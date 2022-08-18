---
title: Standard Logic Apps で AAD 認証する方法について
date: 2022-08-17 00:00:00
tags:
  - How-To
  - Tips
  - Azure Integration
  - Azure Logic Apps 
  - Standard Logic Apps
---

こんにちは！Azure Integration サポート チームの 川合 です。  
Standard Logic Apps （シングルテナント） をご利用のお客様も多いかと思いますが、App Service や Azure Functions と異なり、Standard Logic Apps では Azure Portal から AAD 認証の設定が出来ません。

![](./AadAuthentication/image000.png)

今回は、REST API を用いて Standard Logic Apps でも AAD 認証を行う方法をご紹介いたします。

<!-- more -->

## 目次
- 事前準備
- Azure ActiveDirectory へのアプリ登録
- トークン取得
- authsettingsV2 API の呼び出し
- AAD 認証による Standard Logic Apps の実行
- まとめ

## 事前準備
今回の記事については以下の情報を参考にしております。

- [Trigger workflows in Standard logic apps with Easy Auth](https://techcommunity.microsoft.com/t5/integrations-on-azure-blog/trigger-workflows-in-standard-logic-apps-with-easy-auth/ba-p/3207378)

また、REST API の実行方法については [postman](https://www.postman.com/downloads/) を利用してリクエストを送信します。そのため、Postman のインストール設定を事前に実施いただくことが前提の記事となります。

途中でトークンのデコードを行う手順がございますが、 [jwt.io](https://jwt.io/) というサイトで行います。

以上を基に、本記事では認証手順を記載いたします。

## Azure ActiveDirectory へのアプリ登録
始めに、トークンを取得する為にお客様がご利用いただいている Azure ActiveDirectory に対して "アプリの登録" を行います。
なお、トークンの取得方法については複数ございますため、お客様にとって容易な方法でご対応いただければと存じます。

＜アプリの登録手順＞
１．Azure Portal より Azure Active Directory を選択し、[アプリの登録] を選択。
![](./AadAuthentication/image001.png)

２．以下の内容で登録。
・名前 = 任意の値
・サポートされているアカウントの種類 = "この組織ディレクトリのみに含まれるアカウント (Microsoft のみ - シングル テナント)

以降変更なし。
![](./AadAuthentication/image002.png)


アプリの登録が出来ましたら、[概要] より以下の内容を控えていただきます。

![](./AadAuthentication/image003.png)

＜控える項目＞
・アプリケーション (クライアント) ID
・オブジェクト ID
・ディレクトリ (テナント) ID


次に、[証明書とシークレット] のメニューより、[新しいクライアント シークレット] を作成いただきます。
こちらについては有効期限はお客様の要件によって任意の値を設定下さいますようお願いいたします。
ここで表示されるクライアントシークレットの値は一度しか表示されませんので、確実にこの作成したタイミングで控えていただきますようお願いいたします。

![](./AadAuthentication/image004.png)

＜控える項目＞
・値

Azure ActiveDirectory へのアプリ登録の方法については以上となります。次の手順で、認証に利用するトークンの情報を取得致します。

## トークン取得
続いて、AAD 認証に必要な REST API を設定いたします。まず、先に記載しました Postman を起動し、トークンを取得します。
![](./AadAuthentication/image005.png)

・POSTメソッド
https://login.microsoftonline.com/{ディレクトリ(テナント)ID}/oauth2/token 
![](./AadAuthentication/image00501.png)

Body には [x-www-from-urlencoded] を指定し、以下の内容を設定します。

![](./AadAuthentication/image006.png)
client_id = AADに登録したアプリの [アプリケーション (クライアント) ID] を指定
resource = https://management.azure.com 
client_secret = クライアントシークレットの [値]
grant_type = client_credentials

それ以外のパラメーターは特に設定不要でございます。

以上の設定が設定出来ましたら、画面右上の [Send] ボタンを選択し、リクエストを送信します。
![](./AadAuthentication/image00502.png)

リクエストが成功すると、Body の中にトークンの値が表示されますので、 "access_token" を全て控えていただきます。

＜控える項目＞
・access_token
![](./AadAuthentication/image00503.png)


続いて、取得した "access_token" をデコードし、oid を取得します。

[jwt.io](https://jwt.io/)

![](./AadAuthentication/image00701.png)

ブラウザで上記サイトを開いたら以下の [Encoded] に先ほどコピーした "access_token" を全て貼り付けます。
![](./AadAuthentication/image007.png)

画面右側の [Decoded] - [PAYLOAD] の中段あたりに "oid" が表示されるので、そちらの値を控えていただきます。
![](./AadAuthentication/image008.png)

＜控える項目＞
・oid

トークン取得の作業としては以上となりますが、"access_token" および "oid" については後続の手順で利用しますので、そのまま残していただきますようお願いいたします。


## authsettingsV2 API の呼び出し
続いて、authsettingsV2 API を呼び出します。こちらは Standard Logic Apps に対して AAD 認証を行うための事前処理を行うものでございます。
こちらの手順では再度 Postman を用いてリクエストを送信いたします。

始めに、HTTP リクエストは以下の通り設定いただきます。

・PUT メソッド
```
https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.Web/sites/{logicAppName}/config/authsettingsV2?api-version=2021-02-01
```

・subscriptionId = Standard Logic Apps が作成されている サブスクリプション ID
・resourceGroupName = Standard Logic Apps が作成されている リソースグループ
・logicAppName = Standard Logic Apps （ワークフロー名ではない）

それぞれの値は Standard Logic Apps の [概要] から取得いただくことが可能でございます。
![](./AadAuthentication/image009.png)


PUT メソッドで設定する Body については以下の記事を参考に設定いただきます。

- [Trigger workflows in Standard logic apps with Easy Auth](https://techcommunity.microsoft.com/t5/integrations-on-azure-blog/trigger-workflows-in-standard-logic-apps-with-easy-auth/ba-p/3207378)

＜Body＞
```
{
    "id": "/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.Web/sites/{logicAppName}/config/authsettingsV2",
    "name": "authsettingsV2",
    "type": "Microsoft.Web/sites/config",
    "location": "{locationOfLogicApp}",
    "tags": {},
    "properties": {
        "platform": {
            "enabled": true,
            "runtimeVersion": "~1"
        },
        "globalValidation": {
            "requireAuthentication": true,
            "unauthenticatedClientAction": "AllowAnonymous"
        },
        "identityProviders": {
            "azureActiveDirectory": {
                "enabled": true,
                "registration": {
                    "openIdIssuer": "{issuerId}",
                    "clientId": "{clientId}"
                },
                "login": {
                    "disableWWWAuthenticate": false
                },
                "validation": {
                    "jwtClaimChecks": {},
                    "allowedAudiences": [
                        "{audience1}",
                        "{audience2}"
                    ],
                    "defaultAuthorizationPolicy": {
                        "allowedPrincipals": {
                            "identities": [
                                "{ObjectId of AAD app}"
                            ]
                        }
                    }
                }
            },
            "facebook": {
                "enabled": false,
                "registration": {},
                "login": {}
            },
            "gitHub": {
                "enabled": false,
                "registration": {},
                "login": {}
            },
            "google": {
                "enabled": false,
                "registration": {},
                "login": {},
                "validation": {}
            },
            "twitter": {
                "enabled": false,
                "registration": {}
            },
            "legacyMicrosoftAccount": {
                "enabled": false,
                "registration": {},
                "login": {},
                "validation": {}
            },
            "apple": {
                "enabled": false,
                "registration": {},
                "login": {}
            }
        },
        "login": {
            "routes": {},
            "tokenStore": {
                "enabled": false,
                "tokenRefreshExtensionHours": 72.0,
                "fileSystem": {},
                "azureBlobStorage": {}
            },
            "preserveUrlFragmentsForLogins": false,
            "cookieExpiration": {
                "convention": "FixedTime",
                "timeToExpiration": "08:00:00"
            },
            "nonce": {
                "validateNonce": true,
                "nonceExpirationInterval": "00:05:00"
            }
        },
        "httpSettings": {
            "requireHttps": true,
            "routes": {
                "apiPrefix": "/.auth"
            },
            "forwardProxy": {
                "convention": "NoProxy"
            }
        }
    }
}
```

隅付き括弧で記載されているパラメーターについては以下の通り設定いたします。

・locationOfLogicApp = japaneast や japanwest 等 Standard Logicc Apps が存在するリージョン
・issuerId = "https://sts.windows.net/{ディレクトリ(テナント)ID}/ "
・clientId = トークンから取得した oid
・audience1 = https://management.azure.com 
・audience2 = https://{logicAppName}.azurewebsites.net
・ObjectId of AAD app = トークンから取得した oid

上記の通り Body が設定出来ましたら、Postman の Authorization について、以下の通り設定いたします。

・Type = "Bearer Token"
・Token = "access_token"
![](./AadAuthentication/image014.png)

以上の設定が出来ましたら、画面右上の [Send] ボタンを選択し、リクエストを送信します。

Status 200 が返却されていればこちらの手順については完了です。
![](./AadAuthentication/image011.png)


## AAD 認証による Standard Logic Apps の実行
以上の事前準備が出来ましたら、以下の REST API を実行し、Standard Logic Apps を実行致します。
なお、AAD 認証で呼び出す Standard Logic Apps のトリガーについては "HTTP 要求の受信時" トリガーである必要がございますため、ご留意いただきますようお願い申し上げます。

![](./AadAuthentication/image01101.png)


・POST メソッド
```
https://{logicAppName}.azurewebsites.net:443/api/{workflowName}/triggers/manual/invoke?api-version=2020-05-01-preview
```

・logicAppName = Standard Logic Apps （ワークフロー名ではない）
・workflowName = Standard Logic Apps に構築しているワークフロー名
![](./AadAuthentication/image012.png)
![](./AadAuthentication/image013.png)

続いて、Postman Headers に以下を追加いたします。

・KEY = Authorization
・VALUE = Bearer "access_token"
![](./AadAuthentication/image014.png)

以上の設定が出来ましたら、画面右上の [Send] ボタンを選択し、リクエストを送信します。


Status 202 が表示されましたら無事 Standard Logic Apps のワークフローが AAD 認証によって実行されます。
![](./AadAuthentication/image015.png)

Standard Logic Apps の実行結果に Postman から実行されたログが出力されます。
![](./AadAuthentication/image016.png)

以上の手順にて、AAD 認証を用いてStandard Logic Apps を実行することが可能でございます。


## まとめ
本記事では、以下についてご案内いたしましたが、ご理解いただけましたでしょうか。
- 事前準備
- Azure ActiveDirectory へのアプリ登録
- API の設定方法
- Standard Logic Apps の実行方法
- まとめ

本記事が少しでもお役に立ちましたら幸いです。最後までお読みいただき、ありがとうございました！

<Azure Logic Apps の参考サイト>
-- 概要 - Azure Logic Apps とは
https://docs.microsoft.com/ja-jp/azure/logic-apps/logic-apps-overview
Azure Logic Apps とは、ロジック アプリ デザイナーでフロー チャートを用いて作成したワークフローを自動実行するソリューションです。
Azure Logic Apps では、条件分岐などを実装することができ、ワークフローの実行状況に応じて実行する処理を分岐することが可能です。
