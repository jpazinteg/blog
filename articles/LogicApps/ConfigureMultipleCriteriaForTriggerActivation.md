---
title: 
date: 2024-05-13 00:00:00
tags:
  - How-To
  - Tips
  - Azure Integration
  - Azure Logic Apps
  - Trigger
---

こんにちは。Azure Integration チームの武田です。 

本記事では、トリガー発火条件として、1 つの項目に複数の条件を設定する方法についてご案内させていただきます。<br>
なお、本記事では従量課金プランの Logic Apps を用いてご説明をしておりますが、Standard Logic Apps をご使用の場合でも、同じように設定をすることが可能です。

<!-- more -->

## 目次
- トリガーの発火条件に複数条件を設定する方法
- 基本的な設定例

    1. 同じ項目に AND 条件を指定
    2. 同じ項目に OR 条件を指定
- 応用的な設定方法
- まとめ


## トリガーの発火条件に複数条件を設定する方法
通常、トリガーの [パラメーター] タブにてトリガーの起動方法をご設定いただきますが、パラメーター欄では同じ項目に複数の条件を付けること、例えば 「メールの件名に『日報』か『月報』が含まれる場合」 のような条件設定はできません。<br>
このような設定が必要な場合、[パラメーター] タブではなく、[設定] タブ内にございます [トリガーの条件] を使用して、トリガーの起動条件を設定していきます。<br><br>

この時、[パラメーター] タブでご設定いただくのと違い、[トリガーの条件] 欄にて指定いただきます条件はコードで記載します。<br>
詳細な設定方法は後述いたしますが、比較したい項目の記載方法と条件式に使用できる関数について、弊社公開ドキュメントに概要をおまとめしております。<br>
ご一読いただけますと幸いです。<br><br>

- [トリガーとアクションの種類のスキーマ リファレンス - Azure Logic Apps | Microsoft Learn]

(https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-workflow-actions-triggers)<br><br>


- [式関数のリファレンス ガイド - Azure Logic Apps | Microsoft Learn]

(https://learn.microsoft.com/ja-jp/azure/logic-apps/workflow-definition-language-functions-reference)<br><br>


## 基本的な設定例
では、実際にトリガー条件を設定して Logic Apps を起動させてみます。<br>
今回は例として、Office 365 Outlook コネクタより [新しいメールが届いたとき (V3)] トリガーを使用します。<br><br>

まず、基本形として宛先メールアドレスと件名の条件を設定したトリガーの動作を確認します。<br>
また、トリガーの挙動がわかりやすいよう、トリガーの後続処理として受信した (トリガー発火の起因となった) メールの件名を取得するアクションを設定しておきます。
![](./trigger-multiple-condition/blog_001)


宛先に設定したアドレス宛に、件名が『テスト』のメールを送ってみます。
![](./trigger-multiple-condition/blog_002)


トリガー発火条件を満たすメールを受信したことで、Logic Apps が起動していることが確認できます。
![](./trigger-multiple-condition/blog_003)


### 1. 同じ項目に AND 条件を指定
ここからは、トリガーの発火条件に複数条件を設定していきます。今回は、件名に複数条件を設定して動作確認をします。<br><br>

先述の通り、複数の条件を指定する場合、トリガーの [パラメーター] タブではなく、[設定] タブにございます [トリガーの条件] を使用します。<br>
条件として、『件名に "テスト" と "TEST" の表記が含まれる場合』という内容を設定してみます。
![](./trigger-multiple-condition/blob_004)

トリガー発火の確認をするため、それぞれ件名が『テスト』、『TEST』、『テスト_TEST』となっているメールを、宛先に指定したアドレス宛に送ります。
![](./trigger-multiple-condition/blob_005)

実行結果を確認しますと、件名が『テスト_TEST』のメールを受信した時だけトリガーが発火したことが確認できます。
![](./trigger-multiple-condition/blob_006)


### 2. 同じ項目に OR 条件を指定
同じように、OR 条件も設定してみます。<br>
先ほど設定した条件を AND 条件から OR 条件に変更し、先ほどと同じ件名でメールを出してみます。
![](./trigger-multiple-condition/blob_007)
![](./trigger-multiple-condition/blob_008)


実行結果を確認しますと、OR 条件に変えたことで 3 通全てがトリガーの発火条件を満たすメールとなり、それぞれトリガーが発火していることが確認できます。
![](./trigger-multiple-condition/blob_009)
![](./trigger-multiple-condition/blob_010)
![](./trigger-multiple-condition/blob_011)


## 応用的な設定方法
さて。ここまでで同じ項目に AND 条件と OR 条件を指定する基本的な設定方法をご案内いたしましたが、設定の組み合わせによっては、より高度な条件を設定することができます。<br>
例えば、「日次起動の Logic Apps で、祝祭日は起動させない」、「Logic Apps を毎日 11:00 と 12:30 に起動させる」などが可能となります。<br><br>

高度な条件の設定方法については、別途ブログ記事でご紹介をしております。<br>
そちらも併せてご参照いただけますと幸いです。

- Logic Apps で日付や時刻を判定してワークフローを制御する | Japan Azure Integration Support Blog

(https://jpazinteg.github.io/blog/LogicApps/LogicApps-Functions/)


## まとめ
本記事では、以下についてご案内いたしました。

- トリガーの発火条件に複数条件を設定する方法
- 複数条件の基本的な設定例
- 複数条件の応用的な設定方法


トリガーに複数条件を設定する方法として、本記事が少しでもお役に立ちましたら幸いです。<br>
最後までお読みいただき、ありがとうございました。