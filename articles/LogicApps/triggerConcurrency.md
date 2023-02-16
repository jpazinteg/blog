---
title: トリガーのコンカレンシー制御と実行待機数について
date: 2023-02-28 00:00:00
tags:
  - How-To
  - Tips
  - Azure Integration
  - Azure Logic Apps 
  - Trigger
---

こんにちは。Azure Integration サポート チームの髙橋です。

トリガーに設定可能なコンカレンシー制御と最大実行待機数について説明します。

<!-- more -->

## こんな方におすすめです
- トリガーを順次実行とされたい方
- トリガーの並列実行数を制御されたい方

## 関連情報
- トリガーのコンカレンシー制御については、以下の公開情報がございます。
  - 参考ドキュメント : [Azure Logic Apps の制約と構成の参考文献 # コンカレンシーと分割処理](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-limits-and-config?tabs=consumption%2Cazure-portal#concurrency-and-debatching)

- トリガーのコンカレンシーを変更する方法については、以下の公開情報がございます。
  - 参考ドキュメント : [Azure Logic Apps でのトリガーとアクションの種類のスキーマ リファレンス ガイド # トリガーのコンカレンシーを変更する](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-workflow-actions-triggers#change-trigger-concurrency)

- トリガーの最大実行待機数を設定する方法については、以下の公開情報がございます。
  - 参考ドキュメント : [Azure Logic Apps でのトリガーとアクションの種類のスキーマ リファレンス ガイド # 実行待機の制限を変更する](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-workflow-actions-triggers#change-waiting-runs-limit)

## トリガーのデフォルト設定
従量課金タイプおよび Standard タイプの Logic Apps においては、デフォルトではコンカレンシー制御は「オフ」の状態であり、
トリガーが無制限に並列実行できる設定となっております。
![](./triggerConcurrency/triggerConcurrency01.jpg)

## コンカレンシー制御の変更
トリガーの設定にて、コンカレンシー制御を「オン」とし、並列実行する数を制御することが可能です。
**一度コンカレンシー制御を「オン」としますと、元に戻すことができかねますのでご注意ください。**

以下の例では「HTTP 要求の受信時」トリガーを利用しております。
右上の「…」を押下し、「設定」を選択します。
![](./triggerConcurrency/triggerConcurrency02.jpg)

「コンカレンシー制御」にて「上限」を「オン」とします。
「並列処理の次数」にて並列して起動するトリガーの数を制御することが可能です。
以下の例では、順次実行とするために「並列処理の次数」を「1」としております。
![](./triggerConcurrency/triggerConcurrency03.jpg)

以下の公開情報に記載がございますが、「並列処理の次数」には 1 ～ 100 をご設定いただけます。
![](./triggerConcurrency/triggerConcurrency04.jpg)
- 参考ドキュメント : [Azure Logic Apps の制約と構成の参考文献 # コンカレンシーと分割処理](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-limits-and-config?tabs=consumption%2Cazure-portal#concurrency-and-debatching)

## 最大実行待機数の設定
トリガーのコンカレンシー制御にて「並列処理の次数」で設定した値の数だけ並列処理が既に行われていて、
さらに新しいリクエストがトリガーに送られた場合には、先の処理が終わるまで待機させることが可能です。
このときの待機させる数をご設定いただけます。
最小値と最大値につきましては、以下の公開情報に記載がございます。
![](./triggerConcurrency/triggerConcurrency05.jpg)
- 参考ドキュメント : [Azure Logic Apps の制約と構成の参考文献 # コンカレンシーと分割処理](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-limits-and-config?tabs=consumption%2Cazure-portal#concurrency-and-debatching)


[ロジック アプリ] - [<対象の ロジック アプリ>] - [開発ツール - ロジック アプリ コード ビュー] と遷移します。
"triggers" > トリガー名 > "runtimeConfiguration" > "concurrency" に、"maximumWaitingRuns" という項目を追加します。
値には、待機させる最大実行数 (今回の例では 2 ) を指定いたします。
![](./triggerConcurrency/triggerConcurrency06.jpg)

- 参考ドキュメント : [Azure Logic Apps でのトリガーとアクションの種類のスキーマ リファレンス ガイド # 実行待機の制限を変更する](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-workflow-actions-triggers#change-waiting-runs-limit)

上記の設定を行いますと、以下のような動作となります。
※ 以下の動作確認では、「並列処理の次数」を「1」最大実行待機数を「2」としており、従量課金タイプの Logic Apps にて手動でトリガーを起動しております。

まず一つ目の処理の実行を開始いたします。
![](./triggerConcurrency/triggerConcurrency07.jpg)

一つ目の処理が実行されている間に、新しくトリガーを起動しますと以下のように 2 件まで待機いたします。
![](./triggerConcurrency/triggerConcurrency08.jpg)

最大実行待機数を超えてトリガーを起動しようとしますと、以下のようにエラーが発生いたします。
![](./triggerConcurrency/triggerConcurrency09.jpg)

トリガーを外部から呼び出す場合、呼び出し元には 429 (Too Many Requests) が返却され、呼び出しに失敗いたします。
この場合、Logic Apps 側の「トリガーの履歴」にも残りませんので、クライアント側にて時間を空けて再試行する等のご対応が必要となります。
```json
{
    "error": {
        "code": "WorkflowMaximumWaitingRunCountExceeded",
        "message": "The number of waiting workflow runs '2' has exceeded the maximum allowed limit of '2'. Try increasing the value of 'maximumWaitingRuns' in trigger concurrency configuration."
    }
}
```

また、最大実行待機数を上記のように明示的に設定しない場合には、以下の公開情報の通り「10 回の実行 + 同時実行の数」となります。(最大は 100 でございます。)
![](./triggerConcurrency/triggerConcurrency05.jpg)
- 参考ドキュメント : [Azure Logic Apps の制約と構成の参考文献 # コンカレンシーと分割処理](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-limits-and-config?tabs=consumption%2Cazure-portal#concurrency-and-debatching)

たとえば「並列処理の次数」が「1」の場合には、「10 回の実行 + 1」となり、11 件まで待機させることが可能です。
![](./triggerConcurrency/triggerConcurrency10.jpg)
![](./triggerConcurrency/triggerConcurrency11.jpg)

## まとめ
本記事では、トリガーのコンカレンシー制御の設定方法、最大実行待機数についてご案内いたしました。
本記事が少しでもお役に立ちましたら幸いです。