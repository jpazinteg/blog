---
title: Logic Apps の Availability Zone 対応に伴う通知メールについて
date: 2022-04-07 00:00:00
tags:
  - How-To
  - Tips
  - Azure Integration
  - Logic Apps
---

こんにちは！Azure Integration サポート チームの 川合 です。  
つい先日、Logic Apps が Availability Zone に対応するという情報が Tech Community のブログ記事で公開されましたね。

- Azure Logic Apps - Availability Zone support is coming soon
https://staticsint.teams.cdn.office.net/evergreen-assets/safelinks/1/atp-safelinks.html

こちらの要件に伴い、お客様の中では以下の件名でメールが届いた方もいらっしゃるのではないでしょうか。

**Action required: Update firewall configurations that allow or deny IP addresses of Azure Logic Apps**

今回は上記メール本文の概要と、お客様にてご対応いただく内容について整理いたしましたのでご案内させていただきます。


<!-- more -->

## 目次
- メール本文の概要について
- 本メールに対するアクション
- まとめ

## メール本文の概要について
まず、本メールの概要については大きく以下の内容となります。

![](./AzNotificationMail/image01.png)


・2022年3月25日より、アベイラビリティゾーンをサポートするために
　Logic Apps で利用する IP アドレスの追加が予定されている。
・Logic AppsのIPアドレスとの通信を許可するファイアウォール設定がある場合、
　2022年5月15日までにLogic Appsとコネクターの両方について新しいアドレスに更新する必要がある。
・既存および新規で作成するロジック アプリに今回追加される IP アドレスが利用される。

以上が本メールの概要となります。

## 本メールに対するアクション
お客様にご対応いただく内容として、メールにも記載されております通り ファイアウォール の設定変更がございます。

ここで記載されております　**ファイアウォール**　が何を指しているかですが、Azure FireWall だけではなく、ネットワークセキュリティグループやストレージ アカウントのファイアウォール等、Logic Apps からの IP アドレス通信制御を行っている全てのリソースを指します。

例えば、ストレージ アカウントのファイアウォールにて、ロジック アプリの送信 IP アドレスを許可されている場合、
IP アドレスが変更となりますのでファイアウォール側の設定を見直していただく必要がございます。

![](./AzNotificationMail/image02.png)

ファイアウォールに対して設定すべき IP アドレスにつきましては、以下の通りご利用いただいておりますコネクタの種類によって異なります。

マネージド コネクタをご利用されている際には、
以下の公開情報の該当リージョンの IP アドレスをすべて追加いただきますようお願いいたします。

<ご参考情報>
- マネージド コネクタのアウトバウンド IP アドレス
https://docs.microsoft.com/ja-JP/connectors/common/outbound-ip-addresses#azure-logic-apps

組み込みコネクタをご利用されている際には、
以下の公開情報の該当リージョンの IP アドレスをすべて追加いただきますようお願いいたします。

<ご参考情報>
- Azure Logic Apps の制約と構成の参考文献 # マルチテナントとシングルテナント - 送信 IP アドレス
https://docs.microsoft.com/ja-JP/azure/logic-apps/logic-apps-limits-and-config?tabs=azure-portal#multi-tenant--single-tenant---outbound-ip-addresses

ファイアウォール経由で、Logic Apps への 通信を制御している際には、
以下の公開情報の該当リージョンの IP アドレスをすべて追加いただきますようお願いいたします。

<ご参考情報>
- Azure Logic Apps の制約と構成の参考文献 # 受信 IP アドレス
https://docs.microsoft.com/ja-JP/azure/logic-apps/logic-apps-limits-and-config?tabs=azure-portal#inbound-ip-addresses


なお、お客様のコネクタでご利用されているサービスについて、これまで特にファイアウォール等の設定をしておらず、
ロジック アプリの送信 IP アドレスをブロックしていない際には、対応は不要となります。


## まとめ
本記事では、以下についてご案内いたしましたが、ご理解いただけましたでしょうか。
- メール本文の概要について
- 本メールに対するアクション

本記事が少しでもお役に立ちましたら幸いです。最後までお読みいただき、ありがとうございました！

<Logic Apps の参考サイト>
-- 概要 - Azure Logic Apps とは
https://docs.microsoft.com/ja-jp/azure/logic-apps/logic-apps-overview
Logic App とは、ロジック アプリ デザイナーでフロー チャートを用いて作成したワークフローを自動実行するソリューションです。
Logic App では、条件分岐などを実装することができ、ワークフローの実行状況に応じて実行する処理を分岐することが可能です。
