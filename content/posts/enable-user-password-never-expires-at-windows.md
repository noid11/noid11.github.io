---
title: "PowerShell を使って Windows ユーザーでパスワード失効を無効化する方法"
date: 2021-01-19T16:11:20+09:00
toc: true
draft: false
---

- Windows Server で特定ユーザーのパスワード失効を無効化したい場合に便利
- デフォルトでは一定期間でローテートするような設定なので、セキュリティに注意した上で使おう

<!--more-->


## コマンド

- Administrator ユーザーのパスワード失効を無効化する例
    - コマンド的には、パスワード有効期限を無期限とする設定を有効化する感じ

```powershell
Set-LocalUser -Name "Administrator" -PasswordNeverExpires $true
```


## Set-LocalUser コマンド

Set-LocalUser (Microsoft.PowerShell.LocalAccounts) - PowerShell | Microsoft Docs  
https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.localaccounts/set-localuser

> The Set-LocalUser cmdlet modifies a local user account. This cmdlet can reset the password of a local user account.

- ローカルユーザーアカウントの変更を行うコマンド
- `-Name`
    - 対象となるローカルユーザーアカウントの名前
- `-PasswordNeverExpires`
    - パスワード有効期限を無期限とする設定を有効化・無効化する


## Net user コマンド

Net user | Microsoft Docs  
https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771865(v=ws.11)

> Adds or modifies user accounts, or displays user account information.

- ユーザーアカウントの追加・変更や、ユーザーアカウント情報を表示するコマンド
- `Set-LocalUser` コマンドを実行する前後で使用することで、変更内容を確認できる


## 実行例

- `Password expires` が `Never` になれば OK

```powershell
PS C:\Users\Administrator> net user Administrator
User name                    Administrator
Full Name
Comment                      Built-in account for administering the computer/domain
User s comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            1/19/2021 4:27:36 AM
Password expires             3/2/2021 4:27:36 AM
Password changeable          1/19/2021 4:27:36 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   1/19/2021 4:41:50 AM

Logon hours allowed          All

Local Group Memberships      *Administrators
Global Group memberships     *None
The command completed successfully.
PS C:\Users\Administrator> Set-LocalUser -Name "Administrator" -PasswordNeverExpires $true
PS C:\Users\Administrator> net user Administrator
User name                    Administrator
Full Name
Comment                      Built-in account for administering the computer/domain
User s comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            1/19/2021 4:27:36 AM
Password expires             Never
Password changeable          1/19/2021 4:27:36 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   1/19/2021 4:41:50 AM

Logon hours allowed          All

Local Group Memberships      *Administrators
Global Group memberships     *None
The command completed successfully.
```


## この記事を試した環境

```powershell
PS C:\Users\Administrator> Get-WmiObject Win32_OperatingSystem


SystemDirectory : C:\Windows\system32
Organization    : Amazon.com
BuildNumber     : 17763
RegisteredUser  : EC2
SerialNumber    : 00430-00000-00000-AA379
Version         : 10.0.17763

PS C:\Users\Administrator> $PSVersionTable

Name                           Value
----                           -----
PSVersion                      5.1.17763.1490
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.17763.1490
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
```