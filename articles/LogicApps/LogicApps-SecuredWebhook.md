---
title: Azure Monitor のアクション グループから [セキュリティで保護された Webhook] アクションでロジック アプリを呼び出す方法
date: 2025-05-01 00:00:00
tags:
  - How-To
  - Tips
  - Azure Integration
  - 認証  
---

こんにちは。Azure Integration サポート チームの 飯野 です。<br>

Azure Monitor のアクション グループは、Azure Monitor のデータをもとにインフラストラクチャやアプリケーションに問題が発生したことを検知した際に、アラートをトリガーするための機能です。
アラートがトリガーされたときに実行されるアクションのうち、ロジック アプリを呼び出すことができるアクションとして [ロジック アプリ] と [セキュリティ保護された Webhook] の 2 種類がございます。<br>
この記事では Azure Monitor のアクション グループからロジック アプリを呼び出す際に [セキュリティで保護された Webhook] アクションを利用するシナリオと、その際に必要となる手順をご紹介します。<br>
<!-- more -->

## 目次
- アクション グループからの呼び出しの受信トリガー
- セキュリティで保護された Webhook の設定方法

    1. 保護された Web API 用の Microsoft Entra アプリケーションを作成
    1. ロジック アプリ側で [Azure Active Directory 承認ポリシー] を設定
    1. ロジック アプリのエンドポイント URL を取得
    1. アクション グループを設定
    1. アクション グループが Microsoft Entra アプリケーションを使用できるようにするための PowerShell スクリプトを実行

## アクション グループからの呼び出しの受信トリガー
アクション グループから従量課金タイプのロジック アプリを呼び出す際、通常は認証方法を意識することなく、アクション グループの [アクション タイプ] に [ロジック アプリ] を選択いただいてから、呼び出し先のロジック アプリを指定されてアクション グループからロジック アプリを呼び出すことが多いかと思います。
<br>

![](./LogicApps-SecuredWebhook/image-02.png)

この場合、内部的にはロジック アプリの [When a HTTP request is received] トリガーに設定されているエンドポイント URL に対するリクエストが送信されています。
エンドポイント URL 文字列にはアクセス キーを利用した SAS (共有アクセス署名) 認証情報が含まれており、 [ロジック アプリ] を選択した場合のロジック アプリの呼び出しは SAS 認証によって行われることとなります。
従量課金ワークフローの [When a HTTP request is received] トリガーのエンドポイント URL の呼び出しに利用できる認可方式は、以下セクションの記載のとおり、Microsoft Entra ID を使用した OAuth 2.0 または Shared Access Signature (SAS) のいずれか 1 つのみとなります。<br>

[ワークフロー内のアクセスとデータをセキュリティで保護する - Azure Logic Apps | Microsoft Learn ＃ Microsoft Entra ID を使用した OAuth 2.0 を有効にする前の考慮事項](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-securing-a-logic-app?tabs=azure-portal#considerations-before-you-enable-oauth-20-with-microsoft-entra-id)<br>

![](./LogicApps-SecuredWebhook/image-01.png)

このため、アクション グループからロジック アプリを呼び出す際の認証方法として SAS 認証を回避したい場合、OAuth 認証による呼び出しを行う [セキュリティで保護された Webhook] アクション タイプを選択することとなります。<br>
[セキュリティで保護された Webhook] アクションを利用する場合、OAuth 認証の利用のためには事前設定が必要となります。
その設定手順の詳細をご説明いたします。

## セキュリティで保護された Webhook の設定方法
アクション グループで [セキュリティで保護された Webhook] をを選択して呼び出すためには、Webhook 配信をセキュリティで保護するための Microsoft Entra Webhook アプリケーション ロールのメンバーにサービス プリンシパルを作成し、ロジック アプリ側では OAuth 認証を認証方法として設定しておく必要がございます。<br>
設定手順につきましては以下の公開情報もございますので、併せてご確認ください。
[Azure Monitor のアクション グループ - Azure Monitor | Microsoft Learn # セキュリティで保護された Webhook の認証を構成する][(https://learn.microsoft.com/ja-jp/azure/azure-monitor/alerts/action-groups#configure-authentication-for-secure-webhook)

おおまかな手順は以下となります。

1. 保護された Web API 用の Microsoft Entra アプリケーションを作成
1. ロジック アプリ側で [Azure Active Directory 承認ポリシー] を設定
1. ロジック アプリのエンドポイント URL を取得
1. アクション グループを設定
1. アクション グループが Microsoft Entra アプリケーションを使用できるようにするための PowerShell スクリプトを実行

以降から詳細をご説明します。

### 1. 保護された Web API 用の Microsoft Entra アプリケーションを作成
[Microsoft Entra ID] - [管理 - アプリの登録] と遷移し、 [+ 新規登録] を選択します。
　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　
![](./LogicApps-SecuredWebhook/image001.png)

 
アプリケーションを作成しましたら、以下のように [概要] を開き、[アプリケーション ID の URI の追加] を選択します。
 
![](./LogicApps-SecuredWebhook/image002.png)

 
以下の画面の [追加] を選択し、表示されます [アプリケーション ID の URI] をコピーして控えておき、そのまま保存します。

![](./LogicApps-SecuredWebhook/image003.png)

ディレクトリ(テナント) ID とアクセス トークンの形式についても確認しておきます。
ディレクトリ(テナント) ID : 
登録したアプリの [概要] ページにてコピーのうえ、テキスト エディタなどに貼り付けて控えておきます。

![](./LogicApps-SecuredWebhook/image004.png)

 
アクセス トークンの形式 : 
アプリ登録のマニフェストの AccessTokenAcceptedVersion の値の確認し、アクセス トークンの形式を判断します。
1 または null の場合：アクセス トークンの形式は v1
2 の場合：アクセス トークンの形式は v2 となります。<br>
 
![](./LogicApps-SecuredWebhook/image005.png)

以下の技術情報もご参考いただけます。<br>
[Microsoft ID プラットフォームのアクセス トークン - Microsoft identity platform | Microsoft Learn ＃ トークンの形式](https://learn.microsoft.com/ja-jp/entra/identity-platform/access-tokens#token-formats)<br>

このセクションで確認した [アプリケーション ID の URI]、[ディレクトリ(テナント) ID]、[アクセス トークンの形式] は、後述のロジック アプリ側における [Azure Active Directory 承認ポリシー] の設定時に使用します。

### 2. ロジック アプリ側で [Azure Active Directory 承認ポリシー] を設定
[ロジック アプリ] - [<対象の ロジック アプリ>] - [設定 – 認可] と遷移し、 [ポリシーの追加] を選択し、以下の値を設定します。

[ポリシー名] : 任意<br>
[ポリシー タイプ] : AAD<br>
[クレーム] : 以下のクレーム名と設定内容の組み合わせを指定します。[標準要求の追加] で、クレーム名とクレームの値の行を追加することができます。<br>
　[issuer] :<br>
　　アクセス トークンの形式が v1 の場合は https://sts.windows.net/{テナントID}/<br>
　　アクセス トークンの形式が v2 の場合は https://login.microsoftonline.com/{テナントID}/v2.0<br>
　[audience] : api://XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX の形式の文字列<br>

アクセス トークンの形式が v1 の場合の [issuer] に指定する文字列の末尾が / で終了することにご注意ください。
[audience] には 手順 1. で控えたアプリケーション ID の URI を指定します。
上記を設定して [保存] します。

![](./LogicApps-SecuredWebhook/image006.png)

アプリ登録時にマニフェストにて確認したトークンの形式によって issuer (発行者) の設定値が異なることについて、以下の公開技術情報の記載がご参考いただけます。<br>
[ワークフロー内のアクセスとデータをセキュリティで保護する - Azure Logic Apps | Microsoft Learn # Microsoft Entra ID を使用した OAuth 2.0 を有効にする前の考慮事項](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-securing-a-logic-app?tabs=azure-portal#disable-sas)<br>

--- 抜粋 ---<br>
承認ポリシーには少なくとも発行者のクレームが含まれている必要があります。<br>
その Microsoft Entra ID の発行者の値は、https://sts.windows.net/ または https://login.microsoftonline.com/ (OAuth V2) のいずれかで始まります。<br>
--- 抜粋 ---<br>

ロジック アプリにおける AAD ポリシーの有効化手順については、以下の公開技術情報がご参考いただけます。<br>
[Azure AD OAuth を有効にする前の考慮事項 # ロジック アプリに対して OAuth Azure AD を有効にする](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-securing-a-logic-app?tabs=azure-portal#enable-azure-ad-oauth-for-your-logic-app)<br>
 
 ### 3. ロジック アプリのエンドポイント URL を取得
[ロジック アプリ] - [<対象の ロジック アプリ>] - [開発ツール] - [ロジック アプリ デザイナー] と遷移し、 [HTTP 要求の受信時] トリガーを開き、URL を取得します。
 
![](./LogicApps-SecuredWebhook/image007.png)

取得した URL をメモ帳等に貼り付けます。
sp や sv 、sigといったパラメーター値は SAS 認証のために使用され、AAD 認証の場合には不要となるため、削除します。
具体的には以下のようにSAS認証情報を含む URL 文字列から水色箇所を削除し、黄色箇所のみとなるようにします。<br>
 
![](./LogicApps-SecuredWebhook/image008.png)
 
[ワークフロー内のアクセスとデータをセキュリティで保護する - Azure Logic Apps | Microsoft Learn
＃ Shared Access Signature (SAS) 認証を無効にする (従量課金のみ)](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-securing-a-logic-app?tabs=azure-portal#disable-shared-access-signature-sas-authentication-consumption-only)
 
このセクションで解説したロジック アプリのエンドポイント URL の取得手順は、以下技術情報もご参考いただけます。<br>
[ワークフロー内のアクセスとデータをセキュリティで保護する - Azure Logic Apps | Microsoft Learn ＃ 従量課金ロジック アプリ リソースに Microsoft Entra ID を使用した OAuth を有効にする](https://learn.microsoft.com/ja-jp/azure/logic-apps/logic-apps-securing-a-logic-app?tabs=azure-portal#enable-oauth-with-microsoft-entra-id-for-your-consumption-logic-app-resource)

### 4. アクション グループを設定
Azure Monitor 側のアクション グループを開き、以下のように [アクション] - [アクション タイプ] - [セキュリティで保護された Webhook] を選択します。[セキュリティで保護された Webhook] 画面にて前述の手順で確認した内容を以下のように指定します。<br>
[オブジェクト ID] : 手順 1. で登録したアプリケーションを指定します。<br>
　(アプリの登録直後でまだドロップダウン リストの候補に表示されない場合、登録が反映されるまで時間を置いてから再度お試しください。)<br>
[URI] : 手順 3. で SAS 認証の情報を削除した URL を指定します。<br>
 
![](./LogicApps-SecuredWebhook/image009.png)

 
なお、アクション グループで [セキュリティで保護された Webhook] アクションを作成または変更できるようにするには、
Microsoft Entra アプリケーションの所有者ロールをサービス プリンシパルに割り当てる必要があります。
 
![](./LogicApps-SecuredWebhook/image010.png)

[Azure Monitor のアクション グループ - Azure Monitor | Microsoft Learn](https://learn.microsoft.com/ja-jp/azure/azure-monitor/alerts/action-groups#configure-authentication-for-secure-webhook)

以下は上記技術情報からの抜粋となります。
![](./LogicApps-SecuredWebhook/image011.png)<br>

### 5. アクション グループが Microsoft Entra アプリケーションを使用できるようにするための PowerShell スクリプトを実行
以下の技術情報のセクションの手順に従い、スクリプトを実行します。<br>
[Azure Monitor のアクション グループ - Azure Monitor | Microsoft Learn ＃ Secure Webhook PowerShell スクリプト](https://learn.microsoft.com/ja-jp/azure/azure-monitor/alerts/action-groups#secure-webhook-powershell-script)
 
前提条件として以下技術情報の Microsoft Graph PowerShell SDK を作業端末にインストールしておく必要がございます。<br>
[Install the Microsoft Graph PowerShell SDK | Microsoft Learn](https://learn.microsoft.com/ja-jp/powershell/microsoftgraph/installation?view=graph-powershell-1.0&preserve-view=true)
 
スクリプトの実行方法は以下となります。<br>
[Azure Monitor のアクション グループ - Azure Monitor | Microsoft Learn ＃実行手順](https://learn.microsoft.com/ja-jp/azure/azure-monitor/alerts/action-groups#how-to-run)
 
1. 以下のスクリプトをコピーして作業端末上でテキスト エディタに貼り付け、tenantId と、アプリ登録の ObjectID を置き換えます。
tenantIdt と アプリ登録の ObjectID の文字列は 登録したアプリの [概要] ページにて [ディレクトリ(テナント)ID] と [オブジェクト ID] にてご確認いただけます。
1. *.ps1 ファイルとして任意の名称でファイルを保存します。
1. 作業端末上で PowerShell コマンドを開き、*.ps1 スクリプトを実行します。
 
```
Write-Host "================================================================================================="
$scopes = "Application.ReadWrite.All"
$myTenantId = "<<Customer's tenant id>>"
$myMicrosoftEntraAppRegistrationObjectId = "<<Customer's object id from the app registration>>"
$actionGroupRoleName = "ActionGroupsSecureWebhook"
$azureMonitorActionGroupsAppId = "461e8683-5575-4561-ac7f-899cc907d62a" # Required. Do not change.
 
Connect-MgGraph -Scopes $scopes -TenantId $myTenantId
 
Function CreateAppRole([string] $Name, [string] $Description)
{
    $appRole = @{
        AllowedMemberTypes = @("Application")
        DisplayName = $Name
        Id = New-Guid
        IsEnabled = $true
        Description = $Description
        Value = $Name
    }
    return $appRole
}
 
$myApp = Get-MgApplication -ApplicationId $myMicrosoftEntraAppRegistrationObjectId
$myAppRoles = $myApp.AppRoles
$myActionGroupServicePrincipal = Get-MgServicePrincipal -Filter "appId eq '$azureMonitorActionGroupsAppId'"
 
Write-Host "App Roles before addition of new role.."
foreach ($role in $myAppRoles) { Write-Host $role.Value }
 
if ($myAppRoles.Value -contains $actionGroupRoleName)
{
    Write-Host "The Action Group role is already defined. No need to redefine.`n"
    # Retrieve the application again to get the updated roles
    $myApp = Get-MgApplication -ApplicationId $myMicrosoftEntraAppRegistrationObjectId
    $myAppRoles = $myApp.AppRoles
}
else
{
    Write-Host "The Action Group role is not defined. Defining the role and adding it."
    $newRole = CreateAppRole -Name $actionGroupRoleName -Description "This is a role for Action Group to join"
    $myAppRoles += $newRole
    Update-MgApplication -ApplicationId $myApp.Id -AppRole $myAppRoles
 
    # Retrieve the application again to get the updated roles
    $myApp = Get-MgApplication -ApplicationId $myMicrosoftEntraAppRegistrationObjectId
    $myAppRoles = $myApp.AppRoles
}
 
$myServicePrincipal = Get-MgServicePrincipal -Filter "appId eq '$($myApp.AppId)'"
 
if ($myActionGroupServicePrincipal.DisplayName -contains "AzNS AAD Webhook")
{
    Write-Host "The Service principal is already defined.`n"
    Write-Host "The action group Service Principal is: " + $myActionGroupServicePrincipal.DisplayName + " and the id is: " + $myActionGroupServicePrincipal.Id
}
else
{
    Write-Host "The Service principal has NOT been defined/created in the tenant.`n"
    $myActionGroupServicePrincipal = New-MgServicePrincipal -AppId $azureMonitorActionGroupsAppId
    Write-Host "The Service Principal is been created successfully, and the id is: " + $myActionGroupServicePrincipal.Id
}
 
# Check if $myActionGroupServicePrincipal is not $null before trying to access its Id property
# Check if the role assignment already exists
$existingRoleAssignment = Get-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $myActionGroupServicePrincipal.Id | Where-Object { $_.AppRoleId -eq $myApp.AppRoles[0].Id -and $_.PrincipalId -eq $myActionGroupServicePrincipal.Id -and $_.ResourceId -eq $myServicePrincipal.Id }
 
# If the role assignment does not exist, create it
if ($null -eq $existingRoleAssignment) {
    Write-Host "Doing app role assignment to the new action group Service Principal`n"
    New-MgServicePrincipalAppRoleAssignment -ServicePrincipalId $myActionGroupServicePrincipal.Id -AppRoleId $myApp.AppRoles[0].Id -PrincipalId $myActionGroupServicePrincipal.Id -ResourceId $myServicePrincipal.Id
} else {
    Write-Host "Skip assigning because the role already existed."
}
 
Write-Host "myServicePrincipalId: " $myServicePrincipal.Id
Write-Host "My Azure AD Application (ObjectId): " $myApp.Id
Write-Host "My Azure AD Application's Roles"
foreach ($role in $myAppRoles) { Write-Host $role.Value }
 
Write-Host "================================================================================================="
```

このスクリプトの実行は Microsoft Entra アプリケーション管理者ロールが割り当てられているユーザーが行う必要があります。
![](./LogicApps-SecuredWebhook/image012.png)

## まとめ
本記事では、以下についてご案内いたしました。

- アクション グループからの呼び出しの受信トリガー
- セキュリティで保護された Webhook の設定方法
おおまかな手順として以下をご案内いたしました。

1. 保護された Web API 用の Microsoft Entra アプリケーションを作成
1. ロジック アプリ側で [Azure Active Directory 承認ポリシー] を設定
1. ロジック アプリのエンドポイント URL を取得
1. アクション グループを設定
1. アクション グループが Microsoft Entra アプリケーションを使用できるようにするための PowerShell スクリプトを実行

 のご理解の一助として、本記事が少しでもお役に立ちましたら幸いです。最後までお読みいただき、ありがとうございました！
