---
title: 実行中の Logic Apps の状況を確認して制御する
date: 2023-03-16 00:00:00
tags:
  - How-To
  - Tips
  - Azure Integration
  - Azure Logic Apps 
---

こんにちは！Azure Integration サポート チームの山田です。

複数の Logic Apps を実行する際に、既に実行中の Logic Apps の状況を確認して制御する方法についてご紹介いたします。

<!-- more -->

## こんな方におすすめです
- トリガーのコンカレンシー制御のほかに、他の実行中の Logic Apps の状況を任意にフィルターして制御されたい方。

## 目次
- API
- Logic Apps から HTTP コネクタによる API の実行
- 状況判定
- 関連ドキュメント
- まとめ

## API

[Workflow Runs - List](https://learn.microsoft.com/en-us/rest/api/logic/workflow-runs/list?tabs=HTTP)

例として Filter で実行中のフローを絞り込むことができます。

`https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/workflows/{workflowName}/runs?api-version=2016-06-01&$filter={$filter}`

実行中のものを絞り込むフィルタは「status eq 'running'」などですので、エンコードを掛けますと以下のような URL になるかと存じます。
`https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Logic/workflows/{workflowName}/runs?api-version=2016-06-01&$filter=status%20eq%20%27running%27`

実行結果:

![](WorkflowRunsList/WorkflowRunsList-1.png)

## Logic Apps から HTTP コネクタによる API の実行

上の API を、Logic Apps の、HTTP コネクタから実行いただけます。ここでは認証にマネージド ID を利用しております。

![](WorkflowRunsList/WorkflowRunsList-2.png)

実行結果:

![](WorkflowRunsList/WorkflowRunsList-3.png)

当該の API を使用する際は Logic Apps のマネージド ID に対して、「Logic App Operator」ロールを割り当てする必要がある点が、留意点となります。

[Azure 組み込みロール # Logic App Operator](https://learn.microsoft.com/ja-jp/azure/role-based-access-control/built-in-roles#logic-app-operator)


## 状況判定

この HTTP の結果に対して、関数を用いまして以下のような判定が可能となります。

![](WorkflowRunsList/WorkflowRunsList-4.png) 

`Length(body('HTTP')?['value'])`

Length

https://learn.microsoft.com/ja-jp/azure/logic-apps/workflow-definition-language-functions-reference#length

実行結果:

![](WorkflowRunsList/WorkflowRunsList-5.png) 

配列の要素数をカウントする length 関数を用いて、実行中のジョブ一覧「body('HTTP')?['value']」の要素数をカウントできます。このカウントを条件ステートメントで判定し、上のように実行中のジョブが 1 以上である場合は終了アクションでジョブを終了、実行中のジョブがない場合はご想定の処理に移行する、というような構築が可能となります。


## 関連ドキュメント
- トリガーのコンカレンシー制御については、以下の記事がございます。
  - 参考ドキュメント : [トリガーのコンカレンシー制御と最大実行待機数について](https://jpazinteg.github.io/blog/LogicApps/triggerConcurrency/)  

## まとめ

本記事では、複数の Logic Apps を実行する際に、既に実行中の Logic Apps の状況を確認して制御する方法についてご紹介しました。本記事が少しでもお役に立ちましたら幸いです。