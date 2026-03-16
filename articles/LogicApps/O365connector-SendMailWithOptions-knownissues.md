---
title: [Office 365 Outlook] コネクタなどでの意図しない挙動について
date: 2026-03-17 11:00:00
tags:
  - Troubleshooting
  - Azure Integration
  - Azure Logic Apps 
  - office365
  - Connectors
---

こんにちは。 Azure Integration サポート チームの継松です。  
本記事では、　[Office 365 Outlook] コネクタの [オプションを指定してメールを送信] アクションでの意図しない挙動についてご紹介します。

各コネクタには仕様上、既知の問題や制限事項などがあります。
また、本記事の挙動についても各コネクタ、トリガー、アクションでは既知の問題・制限事項として公開資料にて紹介されている挙動の 1 つでございます。
同アクションをご使用の方や、別のアクションをお使いの方も意図せぬ挙動が発生した場合に同様のトラブルシューティングが有効ですので、参考になれば幸いです。

<!-- more -->

## こんな方におすすめです
- 初めて Logic Apps での開発をされる方


## 目次
1. [事象の概要](#header1)
2. [考えられる要因および回避策](#header2)
3. [その他、既知の問題と制限事項について](#header3)
4. [関連ドキュメント](#header4)
5. [まとめ](#header5)


<h2 id="header1"> 事象の概要</h2>

Azure Logic Apps で Office 365 Outlook コネクタの「オプションを指定してメールを送信」 アクションを利用している際に、受信者がオプションを選択していないにもかかわらず、ワークフロー側で Reject として処理が進んでしまう挙動が、断続的に確認されるケースがあります。

本アクションは、受信者に選択肢付きのメールを送信し、受信者がいずれかのオプションを選択するまでワークフローが待機する想定です。
しかし、問題が発生しているケースでは、
- 受信者側では メールを受信していないように見える
- 受信者が オプションを選択していないにもかかわらず
- Logic Apps 側では Reject 応答を受信したものとして後続処理が進行する
といった、意図しない挙動が確認されることがあります。


<h2 id="header2">考えられる要因および回避策</h2>

### 考えられる要因
このような挙動については、Office 365 Outlook コネクタの公開資料に記載されている既知の問題・制限事項の内容と合致する可能性があります。
まずは公開資料の「既知の問題・制限事項」を確認することをおすすめします。

参考:
[Office 365 Outlook - Connectors | Microsoft Learn # アクションに関する既知の問題と制限事項](https://learn.microsoft.com/ja-jp/connectors/office365/#known-issues-and-limitations-with-actions)

公開資料では、以下のような点が注意事項として挙げられています。

> サードパーティのメール フィルター (G Suite や Mimecast など) は、動作中のユーザー オプションを自動的に選択します。 このため、機能に関連するこの問題を回避するには、 Show HTML confirmation dialog を [はい] に設定します。

[Office 365 Outlook - Connectors | Microsoft Learn # アクションに関する既知の問題と制限事項 - サード パーティのメール フィルターを設定する](https://learn.microsoft.com/ja-jp/connectors/office365/#known-issues-and-limitations-with-actions:~:text=%E3%82%B5%E3%83%BC%E3%83%89%20%E3%83%91%E3%83%BC%E3%83%86%E3%82%A3%E3%81%AE%E3%83%A1%E3%83%BC%E3%83%AB%20%E3%83%95%E3%82%A3%E3%83%AB%E3%82%BF%E3%83%BC%E3%82%92%E8%A8%AD%E5%AE%9A%E3%81%99%E3%82%8B)


### 回避策の一例
公開資料上で案内されている回避策として上述の通り、以下の設定を有効化することで挙動が改善するケースがあります。

- Show HTML confirmation dialog（確認用の HTML ダイアログを表示）

メール クライアントやセキュリティ製品による自動処理の影響を受けにくくするため、同様の事象が発生している場合には、これらの設定変更を一度お試しください。

<h2 id="header3">その他、既知の問題と制限事項について</h2>

[Office 365 Outlook] コネクタではそのほか、以下のような点が紹介されております。

> 電子メール フォルダーが変更されたときに動作をトリガーする
>> 1. トリガーは、電子メールが受信された日時に基づいています。 電子メールを別のフォルダーに移動しても、受信した日付プロパティの値は変更されないため、トリガーは、最新の正常な実行の前に受信したメールをスキップします。
>> 2.トリガーは、トリガーがフロー内に構成されているフォルダー内の電子メールのみをチェックします。 メールをフォルダー/サブ フォルダーに移動するように設定されているルールがある場合、受信トレイ フォルダー内のメールを確認するためにトリガーが構成されている場合、トリガーは発生しません。 適切なフォルダーを選択してください。

> トリガーの起動の遅延
>> ほとんどの場合、対応するイベントの発生に対してトリガーが発生しますが、トリガーの起動の遅延に最大 1 時間かかる場合はまれな状況になる可能性があります。


また、そのほかのコネクタでも同様に「既知の問題と制限事項」が紹介されております。
例えば、 [Azure Blob Storage] コネクタでは以下のような制限事項がございます。

> サブフォルダーにファイルが追加または更新された場合、トリガーは起動しません。 サブフォルダーでトリガーする必要がある場合は、複数のトリガーを作成する必要があります。

> コネクタは、URL デコード可能な文字 ("+" または "%" の後に 2 桁の 16 進数が続く) を含む BLOB 名とパスをサポートしていません。

[Azure Blob Storage - Connectors | Microsoft Learn](https://learn.microsoft.com/ja-jp/connectors/azureblob/#known-issues-and-limitations)

気になる挙動を確認された場合には、まずは公開資料の情報をご確認の上、仕様上の挙動となるものかご確認ください。

<h2 id="header4">3. 関連ドキュメント</h2>

- [ワークフローの状態をチェックし、実行履歴を確認し、アラートを設定する - Azure Logic Apps | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/logic-apps/view-workflow-status-run-history?tabs=consumption)
- [Logic Apps を利用する上でご考慮いただきたい制約 | Japan Azure Integration Support Blog](https://jpazinteg.github.io/blog/LogicApps/logicAppsLimitation/)
- [Logic Apps で使用できるコネクタについて | Japan Azure Integration Support Blog](https://jpazinteg.github.io/blog/LogicApps/LogicAppsConnectorList/)
- [Logic Apps の調査時にサポート エンジニアへ連携するログの取得方法について | Japan Azure Integration Support Blog](https://jpazinteg.github.io/blog/LogicApps/TroubleLogCollection/)

<h2 id="header5">4. まとめ</h2>

- 「オプションを指定してメールを送信」アクションでは、受信者の操作が行われていないにもかかわらず、Reject として処理が進んでしまう挙動が発生する場合がある
- このような場合、公開資料に記載されている既知の問題・制限事項を確認することが有用
- 回避策として、HTML 関連のオプション設定を有効化することで改善が見られるケースがある

最後までお読みいただき、ありがとうございました！
