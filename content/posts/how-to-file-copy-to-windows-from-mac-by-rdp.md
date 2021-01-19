---
title: "RDP を使って Mac から Windows にファイルをコピーする方法"
date: 2021-01-19T22:16:16+09:00
draft: true
toc: false
images: 
tags: 
  - RDP
  - Mac
  - Windows
---

- クリップボードにコピーして貼り付るのが手軽で良い
  1. Mac 側でコピーしたいファイルを選択し、クリップボードにコピー
  2. RDP で Windows に接続し、ファイルを送信したいディレクトリをよしなに選択してペースト
- RDP 設定の `Devices & Audio` で `Clipboard` にチェックが入っていることを確認すること

![rdp-setting](/images/how-to-file-copy-to-windows-from-mac-by-rdp/rdp-setting.png)


## この記事を試した環境

Mac

```zsh
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
```

---

Microsoft Remote Desktop

- Version 10.4.0 (1811)
- Download: 
  - https://apps.apple.com/jp/app/microsoft-remote-desktop/id1295203466
  - https://install.appcenter.ms/orgs/rdmacios-k2vy/apps/microsoft-remote-desktop-for-mac/distribution_groups/all-users-of-microsoft-remote-desktop-for-mac
- Release Notes: 
  - https://docs.microsoft.com/ja-jp/windows-server/remote/remote-desktop-services/clients/mac-whatsnew

---

Windows

```powershell
PS C:\Users\Administrator> Get-WmiObject Win32_OperatingSystem


SystemDirectory : C:\Windows\system32
Organization    : Amazon.com
BuildNumber     : 17763
RegisteredUser  : EC2
SerialNumber    : 00430-00000-00000-AA379
Version         : 10.0.17763
```
