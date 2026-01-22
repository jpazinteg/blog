---
title: Standard Logic Apps でのランタイム エラー (ネットワーク問題)
date: 2026-01-26 11:00:00
tags:
  - Troubleshooting
  - Azure Integration
  - Azure Logic Apps 
  - Standard Logic Apps
---

こんにちは。 Azure Integration サポート チームの継松です。  
本記事では、閉域ネットワーク環境 (VNet 統合 + Private Endpoint など) で動作する Standard Logic Apps において、ランタイム エラー (runtime error) が発生した際のトラブルシュート方法をまとめています。

Standard Logic Apps は実行に必要な情報を、紐づくストレージ アカウントのファイル共有 (File Share, files) に保存しています。
そのため、何らかの理由でファイル共有へアクセスできなくなると、エラーが発生する原因になります。
このファイル共有へのアクセス問題は、閉域ネットワーク環境を設定する (VNET 統合の実施、関連設定の構成) 際に、設定の漏れやミスによって生じることが多くございます。

*新規構築時*と*既存環境のエラー発生時*の 2 パターンに分けて確認ポイントをご紹介します。

<!-- more -->

## こんな方におすすめです
- 初めて Standard Logic Apps での開発をされる方
- 初めて 閉域環境 での開発をされる方
- Azure リソースの管理者


## 目次
1. [Standard Logic Apps での Runtime Error について](#header1)
2. [新規環境でのトラブルシュート](#header2)
3. [既存環境でのトラブルシュート](#header3)
4. [関連ドキュメント](#header4)
5. [まとめ](#header5)


<h2 id="header1"> Standard Logic Apps での Runtime Error について</h2>

Standard Logic Apps のランタイムは Azure Storage のファイル共有（Azure Files）上にコンテンツを展開する仕組みを持っています。
そのため、以下のようなネットワークやストレージ／DNS 設定の問題によって、ファイルにアクセスできずランタイム エラーが発生します。

よく見られるエラー例：

- “Access to the path 'C:\home\site\wwwroot\host.json' is denied.”
- “The network path was not found: 'C:\home\data\Functions\secrets\Sentinels'.”
- “Could not find a part of the path 'C:\home\site\wwwroot'.”

これらは多くの場合、*ストレージ アカウントへの接続不備*または *VNet/DNS 設定の問題*が原因となります。


<h2 id="header2">新規環境でのトラブルシュート</h2>

まずは Microsoft Learn などの公式ガイドに沿って、必要な設定が正しく実施されているかを確認することが重要です。

**参考:**
- [Standard ワークフローと仮想ネットワーク間のトラフィックをセキュリティで保護する - Azure Logic Apps | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/logic-apps/secure-single-tenant-workflow-virtual-network-private-endpoint)

- [Standard ロジック アプリをプライベート ストレージ アカウントにデプロイする - Azure Logic Apps | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/logic-apps/deploy-single-tenant-logic-apps-private-storage-account)

- (英文) [Deploying Standard Logic App to Storage Account behind Firewall using Service or Private Endpoints | Microsoft Community Hub](https://techcommunity.microsoft.com/blog/integrationsonazureblog/deploying-standard-logic-app-to-storage-account-behind-firewall-using-service-or/2626286)

- [Azure Portal からファイアウォール設定配下のストレージ アカウントに Standard Logic Apps を作成する方法 | Japan Azure Integration Support Blog](https://jpazinteg.github.io/blog/LogicApps/SecBlobStandardLAPortal/)

### 2-1. アプリ設定の必須項目を確認
閉域ネットワーク環境をご利用の場合には、以下の4 つのアプリ設定は必須です。
設定値が空、誤り、余計なスペースがある場合でも動作しなくなります。

| 設定名 | 設定値 | 意味 |
|--------|--------|--------|
| WEBSITE_CONTENTAZUREFILECONNECTIONSTRING   | Azure Files のファイル共有へ接続するための接続文字列   | ストレージ アカウントおよびファイル共有へのアクセスに用いる   |
| WEBSITE_CONTENTSHARE   | Logic Apps が使用する Azure Files 上のファイル共有名   | 紐づくファイル共有の指定   |
| WEBSITE_CONTENTOVERVNET   | 1 (VNet 経由でストレージに接続する場合)   | ストレージへのアクセスを VNet 経由にするかどうかを指定   |
| WEBSITE_DNS_SERVER   | 168.63.129.16 (規定値) またはご利用される DNS サーバー IP アドレス  | Logic Apps が名前解決に利用する DNS サーバーの IP アドレス  |

>[!NOTE]
>これらは Microsoft Learn の「一般的なエラーのトラブルシューティング」でも強調されています。

[Standard ロジック アプリをプライベート ストレージ アカウントにデプロイする - Azure Logic Apps | Microsoft Learn # 一般的なエラーのトラブルシューティング](https://learn.microsoft.com/ja-jp/azure/logic-apps/deploy-single-tenant-logic-apps-private-storage-account#troubleshoot-common-errors)

### 2-2. ストレージ アカウントへの Outbound 接続が許可されているか
Standard Logic Apps のランタイムが Azure Storage にアクセスできる必要があります。

以下のポートが仮想ネットワーク（NSG / UDR 等）で許可されているか確認してください。
| 宛先サービス | 宛先ポート | 内容 |
|--------|--------|--------|
| ストレージ アカウント   | 443   | ストレージ アカウントへのアクセス  |
| ストレージ アカウント   | 445   | ファイル共有 (files) へのアクセス  |

特に ポート 445 が閉域環境で拒否され、エラーとなった状況が多く報告されております。
設定されている仮想ネットワークの構成を十分ご確認ください。

### 2-3. DNS が正しく構成されているか
ストレージの Private Endpoint を利用する場合、DNS が正しく構成されていない場合、アクセスが失敗します。

Standard Logic Apps の高度なツール > Kudu より次が正しく解決するか確認します。

- tcpping <先に取得したストレージ Blob、File、Queue、Table の各エンドポイント>:443
- nameresolver <先に取得したストレージ Blob、File、Queue、Table の各エンドポイント>

詳しい確認方法については下記記事をご参照ください。

[Standard Logic Apps からストレージ アカウントへの疎通を確認する | Japan Azure Integration Support Blog](https://jpazinteg.github.io/blog/LogicApps/StandardLogicApps-Ping/)


<h2 id="header3">既存環境でのトラブルシュート</h2>

「昨日まで動いていたのに突然エラーが発生した」という場合、設定変更または環境側の変化が原因であるケースが大変多くございます。

### 3-1. Logic Apps のアプリ設定変更

ARM テンプレートや CLI のデプロイにより、意図せず以下の値が書き換わることがあります：

- WEBSITE_CONTENTAZUREFILECONNECTIONSTRING
- WEBSITE_CONTENTSHARE
- WEBSITE_CONTENTOVERVNET
- WEBSITE_DNS_SERVER

変更されるとストレージ アカウントへのアクセスに影響を及ぼすため、以前の値と比較して確認します。

### 3-2. ネットワーク側の設定変更

- Private Endpoint を削除／変更した
- NSG で 445 / 443 がブロックされた
- UDR によりストレージへのルーティングが変わった
- DNS のリンクが外れた／ゾーン名が変更された

閉域環境では、ネットワークの些細な変更でもランタイムが影響を受けます。

### 3-3. ストレージアカウント側の変更

- ファイアウォールの許可 VNet が外れた
- アクセスキーを再生成して接続文字列が無効になった
- ファイル共有名を変更した

こうした変更でも「ランタイムが起動しない」という事象が発生いたします。
ご確認の上、構成を必要に応じて更新ください。

<h2 id="header4">3. 関連ドキュメント</h2>

- [Standard ワークフローと仮想ネットワーク間のトラフィックをセキュリティで保護する - Azure Logic Apps | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/logic-apps/secure-single-tenant-workflow-virtual-network-private-endpoint)

- [Standard ロジック アプリをプライベート ストレージ アカウントにデプロイする - Azure Logic Apps | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/logic-apps/deploy-single-tenant-logic-apps-private-storage-account)

- (英文) [Deploying Standard Logic App to Storage Account behind Firewall using Service or Private Endpoints | Microsoft Community Hub](https://techcommunity.microsoft.com/blog/integrationsonazureblog/deploying-standard-logic-app-to-storage-account-behind-firewall-using-service-or/2626286)

- [Azure Portal からファイアウォール設定配下のストレージ アカウントに Standard Logic Apps を作成する方法 | Japan Azure Integration Support Blog](https://jpazinteg.github.io/blog/LogicApps/SecBlobStandardLAPortal/)

<h2 id="header5">4. まとめ</h2>

本記事では、閉域環境上の Standard Logic Apps での Runtime Error およびトラブルシュート方法についてご紹介しました。

最後までお読みいただき、ありがとうございました！



ほか関連記事は以下を参照くださいませ。

[Standard 版 Azure Logic Apps の料金体系を理解するポイント | Japan Azure Integration Support Blog](https://jpazinteg.github.io/blog/LogicApps/LogicApps-StandardPricing/)
[Standard Logic Apps から VNet 経由で別リソースにアクセスする方法 | Japan Azure Integration Support Blog](https://jpazinteg.github.io/blog/LogicApps/vnetIntergration/)
[Standard Logic Apps のワークフローを誤って削除してしまったとき | Japan Azure Integration Support Blog](https://jpazinteg.github.io/blog/LogicApps/LogicApps-Recovery/)
[Azure Portal からファイアウォール設定配下のストレージ アカウントに Standard Logic Apps を作成する方法 | Japan Azure Integration Support Blog](https://jpazinteg.github.io/blog/LogicApps/SecBlobStandardLAPortal/)
[ファイアウォール設定配下の既存ストレージ アカウントに Standard Logic Apps を作成 | Japan Azure Integration Support Blog](https://jpazinteg.github.io/blog/LogicApps/SecBlobStandardLA/)
[Standard Logic Apps の実行履歴にて「Failed to fetch」というエラーが表示される原因と回避策 | Japan Azure Integration Support Blog](https://jpazinteg.github.io/blog/LogicApps/FailedToFetch/)