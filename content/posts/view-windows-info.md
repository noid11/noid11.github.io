---
title: "Windows の情報を PowerShell から確認する方法"
date: 2021-01-19T14:24:03+09:00
tags:
    - PowerShell
    - Windows
toc: true
draft: false
---

Windows Server のバージョン情報を PowerShell から確認する方法を調べたメモ

<!--more-->

## Get-WmiObject Win32_OperatingSystem

```powershell
Get-WmiObject Win32_OperatingSystem
```


## 実行例

EC2 で Windows Server を構築した例

```powershell
PS C:\Users\Administrator> Get-WmiObject Win32_OperatingSystem


SystemDirectory : C:\Windows\system32
Organization    : Amazon.com
BuildNumber     : 17763
RegisteredUser  : EC2
SerialNumber    : 00430-00000-00000-AA379
Version         : 10.0.17763
```

- SystemDirectory: 
    - OS のシステムディレクトリ
    - System directory of the operating system.
- Organization: 
    - OS 登録ユーザーの会社名
    - Company name for the registered user of the operating system.
- BuildNumber: 
    - OS のビルド番号。製品のリリースバージョン番号よりも正確なバージョン情報として使用できる
    - Build number of an operating system. It can be used for more precise version information than product release version numbers.
- RegisteredUser: 
    - OS の登録ユーザー名
    - Name of the registered user of the operating system.
- SerialNumber: 
    - OS のシリアル ID 番号
    - Operating system product serial identification number.
- Version: 
    - OS のバージョン
    - Version number of the operating system.

Win32_OperatingSystem class - Win32 apps | Microsoft Docs  
https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-operatingsystem


## Get-WmiObject とは

Get-WmiObject (Microsoft.PowerShell.Management) - PowerShell | Microsoft Docs  
https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-wmiobject

> Gets instances of Windows Management Instrumentation (WMI) classes or information about the available classes.

WMI: Windows Managemtn Instrumentation クラスまたは利用可能なクラス情報のインスタンス情報を取得するコマンド


## WMI: Windows Managemtn Instrumentation とは

Windows Management Instrumentation - Win32 apps | Microsoft Docs  
https://docs.microsoft.com/ja-jp/windows/win32/wmisdk/wmi-start-page

> Windows Management Instrumentation (WMI) is the infrastructure for management data and operations on Windows-based operating systems. You can write WMI scripts or applications to automate administrative tasks on remote computers but WMI also supplies management data to other parts of the operating system and products, for example System Center Operations Manager, formerly Microsoft Operations Manager (MOM), or Windows Remote Management (WinRM).

Windows Management Instrumentation (WMI) は Windows ベース OS におけるデータ管理と操作のためのインフラ


## Win32_OperatingSystem とは

Win32_OperatingSystem class - Win32 apps | Microsoft Docs  
https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-operatingsystem

> The Win32_OperatingSystem WMI class represents a Windows-based operating system installed on a computer.

Win32_OperatingSystem は WMI クラスの1つで、コンピューターにインストールされている Windows ベースの OS を扱うクラス



## BuildNumber から Windows Server リリースを確認する

- 公式情報があるので BuildNumber で検索すれば OK
- BuildNumber が `17763` なら、 OS Build カラムから `Windows Server 2019 (Long-Term Servicing Channel) (Datacenter, Essentials, Standard)` か `Windows Server, version 1809 (Semi-Annual Channel) (Datacenter Core, Standard Core)` だと分かる

Windows Server のリリース情報 | Microsoft Docs  
https://docs.microsoft.com/ja-jp/windows-server/get-started/windows-server-release-info


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