---
title: "EB CLI v3 のヘルプを読んだメモ"
date: 2021-01-26T01:44:20+09:00
draft: false
toc: true
images:
tags: 
  - EB
  - EB CLI
---

- [Elastic Beanstalk CLI v3](https://github.com/aws/aws-elastic-beanstalk-cli) の詳細なヘルプがネット上に無さそうだったので読んだメモ  
  - 基本的な使い方なら[ドキュメント](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-getting-started.html)がある
- と思ったら [reference](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-cmd-commands.html) あった・・・

<!--more-->

## eb help

- `abort`: environment の更新または deployment をキャンセルする
- `appversion`: アプリケーションバージョンの一覧表示と管理
- `clone`: environment をクローンする
- `codesource`: EB CLI デフォルトで使用するコードソースを設定
- `config`: environment 設定の変更。サブコマンドを使用して保存された設定を管理
- `console`: EB environment のマネジメントコンソールを開く
- `create`: 新しい environment を作成する
- `deploy`: ソースコードを environment にデプロイする
- `events`: 最近の events を取得する
- `health`: environment health の詳細を表示
- `init`: ディレクトリを初期化してアプリケーションを作成する
- `labs`: 追加の実験用コマンド
- `list`: 全ての environment を表示
- `local`: ローカルマシンでコマンドを実行する
- `logs`: 最新のログを取得
- `open`: アプリケーション URL をブラウザで開く
- `platform`: プラットフォームを管理するコマンド
- `printenv`: 環境変数を表示
- `restore`: terminate した environment を復元
- `scale`: 実行中の EC2 インスタンス数を変更
- `setenv`: 環境変数を設定
- `ssh`: EC2 インスタンスに接続する SSH クライアントを開く
- `status`: environment 情報とステータスを取得
- `swap`: 2つの environment CNAME を入れ替える
- `tags`: environment のタグを追加・削除・更新・一覧表示する
- `terminate`: environment を terminate する
- `upgrade`: environment の platform version を最新にアップデートする
- `use`: デフォルト environment を設定

```zsh
% eb --help     
usage: eb (sub-commands ...) [options ...] {arguments ...}

Welcome to the Elastic Beanstalk Command Line Interface (EB CLI). 
For more information on a specific command, type "eb {cmd} --help".

commands:
  abort        Cancels an environment update or deployment.
  appversion   Listing and managing application versions
  clone        Clones an environment.
  codesource   Configures the code source for the EB CLI to use by default.
  config       Modify an environment\'s configuration. Use subcommands to manage saved configurations.
  console      Opens the environment in the AWS Elastic Beanstalk Management Console.
  create       Creates a new environment.
  deploy       Deploys your source code to the environment.
  events       Gets recent events.
  health       Shows detailed environment health.
  init         Initializes your directory with the EB CLI. Creates the application.
  labs         Extra experimental commands.
  list         Lists all environments.
  local        Runs commands on your local machine.
  logs         Gets recent logs.
  open         Opens the application URL in a browser.
  platform     Commands for managing platforms.
  printenv     Shows the environment variables.
  restore      Restores a terminated environment.
  scale        Changes the number of running instances.
  setenv       Sets environment variables.
  ssh          Opens the SSH client to connect to an instance.
  status       Gets environment information and status.
  swap         Swaps two environment CNAMEs with each other.
  tags         Allows adding, deleting, updating, and listing of environment tags.
  terminate    Terminates the environment.
  upgrade      Updates the environment to the most recent platform version.
  use          Sets default environment.

optional arguments:
  -h, --help            show this help message and exit
  --debug               toggle debug output
  --quiet               suppress all output
  -v, --verbose         toggle verbose output
  --profile PROFILE     use a specific profile from your credential file
  -r REGION, --region REGION
                        use a specific region
  --no-verify-ssl       don\'t verify AWS SSL certificates
  --version             show application/version info

To get started type "eb init". Then type "eb create" and "eb open"
```


## この記事を試した環境

```zsh
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% eb --version
EB CLI 3.19.3 (Python 3.9.1)
```

