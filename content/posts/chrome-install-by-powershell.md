---
title: "PowerShell を使って Google Chrome をインストールする方法"
date: 2021-01-19T15:12:28+09:00
toc: true
tags:
    - PowerShell
    - Windows
    - Google Chrome
draft: false
---

Windows Server 構築で毎回 Internet Explorer の設定を変える・・・みたいな作業が減るので便利

<!--more-->

## コマンド

```powershell
$Path = $env:TEMP; $Installer = "chrome_installer.exe"; Invoke-WebRequest "https://dl.google.com/tag/s/appguid%3D%7B8A69D345-D564-463C-AFF1-A69D9E530F96%7D%26browser%3D0%26usagestats%3D1%26appname%3DGoogle%2520Chrome%26needsadmin%3Dprefers%26brand%3DGTPM/update2/installers/ChromeSetup.exe" -OutFile $Path\$Installer; Start-Process -FilePath $Path\$Installer -Args "/silent /install" -Verb RunAs -Wait; Remove-Item $Path\$Installer
```


## 何をしているのか

`;` はコマンドの区切りなので、実質以下のようなコマンドを連続して実行している

```powershell
$Path = $env:TEMP
$Installer = "chrome_installer.exe"
Invoke-WebRequest "https://dl.google.com/tag/s/appguid%3D%7B8A69D345-D564-463C-AFF1-A69D9E530F96%7D%26browser%3D0%26usagestats%3D1%26appname%3DGoogle%2520Chrome%26needsadmin%3Dprefers%26brand%3DGTPM/update2/installers/ChromeSetup.exe" -OutFile $Path\$Installer
Start-Process -FilePath $Path\$Installer -Args "/silent /install" -Verb RunAs -Wait
Remove-Item $Path\$Installer
```

1. `$Path = $env:TEMP`
    - Google Chrome のインストーラーを保存する一時ディレクトリの定義
2. `$Installer = "chrome_installer.exe"`
    - Google Chrome のインストーラーを保存する際のファイル名
3. `Invoke-WebRequest "https://dl.google.com/tag/s/appguid%3D%7B8A69D345-D564-463C-AFF1-A69D9E530F96%7D%26browser%3D0%26usagestats%3D1%26appname%3DGoogle%2520Chrome%26needsadmin%3Dprefers%26brand%3DGTPM/update2/installers/ChromeSetup.exe" -OutFile $Path\$Installer`
    - `Invoke-WebRequest`
        - HTTP, HTTPS リクエストを行うコマンド
        - https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest
    - `https://dl.google.com/...`
        - Google Chrome インストーラーがホストされている URL
        - 直接ダウンロードできるリンクについては Google 公式の情報は発見できなかった
    - `-OutFile $Path\$Installer`
        - HTTP, HTTPS レスポンス内容をファイルに出力するオプション
        - `$Path\$Installer` は前に定義した一時ディレクトリとファイル名
4. `Start-Process -FilePath $Path\$Installer -Args "/silent /install" -Verb RunAs -Wait`
    - `Start-Process`
        - 新しいプロセスを開始するコマンド
        - https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/start-process
    - `-FilePath $Path\$Installer`
        - プロセスとして実行するファイルのパス
    - `-Args "/silent /install"`
        - `-ArgumentList` のエイリアス
        - プロセスを開始するときの引数
        - そのため `"/silent /install"` は Google Chrome インストーラーに対して指定するオプション
            - サイレントインストールのためのオプションだと思われるが、 Google Chrome のインストーラーで使用できるオプションについては情報を発見できなかった
    - `-Verb RunAs`
        - ファイルを動作するときの verb を指定するオプション
            - verb は実行するアクション的な意味合い
        - 指定可能な verb は、実行するファイルの拡張子によって異なる
        - `RunAs` は `.exe` 形式のファイルに指定できる verb
            - 要するに管理者権限でプロセスを実行するということ。 `sudo` に似ている
    - `-Wait`
        - 実行するプロセスと、その子孫となるプロセスが完了するのを待つオプション
5. `Remove-Item $Path\$Installer`
    - ファイルを削除するコマンド
        - https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/remove-item
    - `Start-Process ... -Wait` と合わせて、イントールプロセス完了後にインストーラーを削除する


## 参考

コマンドでWindowsにChromeをインストールする - Qiita  
https://qiita.com/Arahabica/items/04f8469842f29550b831

Install Google Chrome with Powershell command-line - YouTube  
https://www.youtube.com/watch?v=M0qX46M3mnE


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