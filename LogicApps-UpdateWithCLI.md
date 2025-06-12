---
title: CLI から従量課金タイプの Logic Apps の設定を更新する
date: 2025-06-16 11:00:00
tags:
  - Tips
  - Azure Integration
  - Azure Logic Apps 
  - How-To
  - Azure CLI
---

こんにちは。  Azure Integration サポート チームの継松です。  
今回は、 従量課金タイプの Logic Apps の設定 (例: コンテンツの IP 範囲) を Azure CLI から更新する方法についてご紹介いたします！

本記事では、具体例として既存のリソースの "コンテンツの IP 範囲" に新しく IP アドレスを追加します。
同様の方法にてそのほかの設定なども更新可能ですのでご参考いただけますと幸いでございます。

<!-- more -->

## 目次
1. [従量課金タイプの Logic Apps に関しての Azure CLI コマンドについて](#header1)
2. [コマンド使用例: 既存のリソースの "コンテンツの IP 範囲" に新しく IP アドレスを追加する](#header2)
3. [関連ドキュメント](#header3)
4. [まとめ](#header4)

<h2 id="header1">1. 従量課金タイプの Logic Apps に関しての Azure CLI コマンドについて</h2>

Azure CLI では Azure CLI Logic Apps 拡張機能 (az logic) を使用し、 Azure Logic Apps で実行されるワークフローを作成、管理することが可能でございます。
例えば、コマンド "az logic workflow create" を使用し、Logic Apps のワークフロー定義の JSON ファイルを使用し、新規リソースを作成することが可能です。
その後、 "az logic workflow update" を使用し、ワークフロー定義の更新や状態の変更をすることが可能です。

また、同コマンド "az logic workflow create" を使用し、上書きのような形で更新することも可能です。
"az logic workflow update" では変更不可能な設定内容 (access-control など) があるため、設定されたい内容によってお使い分けください。

詳しくは以下資料をご参照ください。
[クイックスタート - Azure CLI を使用してワークフローを作成して管理する - Azure Logic Apps | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/logic-apps/quickstart-logic-apps-azure-cli)
[az logic | Microsoft Learn](https://learn.microsoft.com/ja-jp/cli/azure/logic?view=azure-cli-latest)

>[!NOTE]
> Standard タイプの Logic Apps に対して有効なコマンドは従量課金のものと異なります。
>
> 　　従量課金: az logic
> 　　Standard: az logicapp
>
> az logicapp コマンドについては下記資料をご参照ください。
> [az logicapp | Microsoft Learn](https://learn.microsoft.com/ja-jp/cli/azure/logicapp?view=azure-cli-latest)

<h2 id="header2">2. コマンド使用例: 既存のリソースの "コンテンツの IP 範囲" に新しく IP アドレスを追加する</h2>

今回、更新したい項目は "コンテンツの IP 範囲" であり、こちらは access-control に含まれます。
よって、 "az logic workflow create" を使用して、上書きする形で更新します。

**1. 設定を変更したいリソースの情報を取得する**

"az logic workflow show" のコマンドや Azure Portal からワークフローのコードやその他設定を確認し、以下のファイルを用意します。
・ ワークフロー定義の JSON ファイル
・ アクセス制御の構成の JSON (または yaml) ファイル
[az logic workflow | Microsoft Learn # az logic workflow show](https://learn.microsoft.com/ja-jp/cli/azure/logic/workflow?view=azure-cli-latest#az-logic-workflow-show)

その他、 エンドポイントの構成なども設定されている場合には、それらのファイルも別途用意します。
詳しくは、以下資料をご参照ください。
"az logic workflow create" 
[az logic workflow | Microsoft Learn # az logic workflow create](https://learn.microsoft.com/ja-jp/cli/azure/logic/workflow?view=azure-cli-latest#az-logic-workflow-create)

"access-control" ファイルの設定例:
```JSON
{"contents": {
               "allowedCallerIpAddresses": [
                   {
                       "addressRange": "xxx.xxx.xxx.xxx/x"
                   }
               ]
           }
       }
```

**2. コマンドを実行します。**

```
az logic workflow create --resource-group "<リソース グループ名>" -n "ワークフロー名" --definition "ワークフロー定義(コード) json ファイル" --location "<リージョン (例: japaneast)>" --access-control "json/yaml ファイル"
```

<h2 id="header3">3. 関連ドキュメント</h2>

[クイックスタート - Azure CLI を使用してワークフローを作成して管理する - Azure Logic Apps | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/logic-apps/quickstart-logic-apps-azure-cli)
[az logic | Microsoft Learn](https://learn.microsoft.com/ja-jp/cli/azure/logic?view=azure-cli-latest)
[az logicapp | Microsoft Learn](https://learn.microsoft.com/ja-jp/cli/azure/logicapp?view=azure-cli-latest)


<h2 id="header4">4. まとめ</h2>
本記事では従量課金タイプの Logic Apps の設定 (例: コンテンツの IP 範囲) を Azure CLI から更新する方法についてご紹介いたしました。
設定内容によっては方法異なりますので、資料をご参考の上、お試しいただけますと幸いです。

最後までお読みいただき、ありがとうございました！