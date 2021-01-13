---
title: "Heroku CLI を使って Heroku Postgres をセットアップする方法"
date: 2021-01-14T02:29:30+09:00
toc: true
draft: true
---

- [Heroku が提供するマネージドデータベースである Heroku Postgres](https://jp.heroku.com/postgres) を [Heroku CLI](https://devcenter.heroku.com/ja/articles/heroku-cli) からセットアップして接続するまでを試す手順
- Proc
    - PostgreSQL や SQL を学習したい時にサクッと使えて便利
    - AWS Lambda 等ローカル環境の PostgreSQL が接続できない環境から、ちょっとしたデータベース接続を試したいような場合に便利
        - RDS よりも立ち上がりが速いし、無料
- Cons
    - [無料の Hobby Dev プラン](https://jp.heroku.com/pricing)だと、レコード行数 10 K と同時接続数 20 といった制限があるので注意

<!--more-->

## PostgreSQL をインストール

Heroku CLI 経由で PostgreSQL に接続する際には `psql` が必要になるため、事前にインストールしておく


```bash
brew install postgresql

# PostgreSQL のバージョンを指定したい場合 @ で指定する
brew install postgresql@12

# バージョン確認
psql --version
```

postgresql — Homebrew Formulae  
https://formulae.brew.sh/formula/postgresql

PostgreSQL: Documentation: 12: psql  
https://www.postgresql.org/docs/12/app-psql.html

psql  
https://www.postgresql.jp/document/12/html/app-psql.html


## Heroku CLI のインストールとセットアップ

```bash
brew tap heroku/brew && brew install heroku
heroku login
```

The Heroku CLI | Heroku Dev Center  
https://devcenter.heroku.com/articles/heroku-cli


## Heroku Postgres のセットアップ

```bash
mkdir my-heroku-postgres-env && cd my-heroku-postgres-env

# Heroku App の作成
heroku create

# Heroku Postgres の作成
heroku addons:create heroku-postgresql:hobby-dev --app myapp

# Heroku Postgres のバージョンを指定したい場合
heroku addons:create heroku-postgresql:hobby-dev --version=9.5 --app myapp

# 作成した Heroku Postgres 情報の確認
heroku pg:info --app myapp
```

Heroku Postgres | Heroku Dev Center  
https://devcenter.heroku.com/articles/heroku-postgresql#version-support

---

実行例

```bash
% mkdir my-heroku-postgres-env && cd my-heroku-postgres-env
% heroku create
Creating app... done, ⬢ fierce-depths-xxx
https://fierce-depths-xxx.herokuapp.com/ | https://git.heroku.com/fierce-depths-xxx.git
% heroku addons:create heroku-postgresql:hobby-dev --app fierce-depths-xxx
Creating heroku-postgresql:hobby-dev on ⬢ fierce-depths-xxx... free
Database has been created and is available
 ! This database is empty. If upgrading, you can transfer
 ! data from another database with pg:copy
Created postgresql-concave-yyy as DATABASE_URL
Use heroku addons:docs heroku-postgresql to view documentation
% heroku pg:info --app fierce-depths-xxx
=== DATABASE_URL
Plan:                  Hobby-dev
Status:                Available
Connections:           0/20
PG Version:            12.5
Created:               2021-01-13 17:48 UTC
Data Size:             7.9 MB
Tables:                0
Rows:                  0/10000 (In compliance)
Fork/Follow:           Unsupported
Rollback:              Unsupported
Continuous Protection: Off
Add-on:                postgresql-concave-yyy
```


## Heroku Postgres への接続

```bash
# Heroku CLI での認証情報を使った接続
heroku pg:psql --app myapp

# PostgreSQL のクレデンシャルを取得して普通に psql でも接続できる
heroku pg:credentials --app myapp

# Connection URL での指定(パスワードが含まれるので取り扱い注意)
psql --dbname=postgres://db-username:db-user-password@db-hostname:db-post/db-name

# 冗長な指定(コマンド実行後にパスワードを入力する必要あり)
psql --dbname=db-name --host=ec2-db-hostname --port=db-post --username=db-username --password
```

---

実行例

```bash
% heroku pg:credentials:url --app fierce-depths-xxx
Connection information for default credential.
Connection info string:
   "dbname=xxx host=ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com port=5432 user=yyy password=zzz sslmode=require"
Connection URL:
   postgres://yyy:zzz@ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com:5432/xxx

% psql --dbname=postgres://yyy:zzz@ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com.com:5432/xxx
% psql --dbname=xxx --host=ec2-xxx-xxx-xxx-xxx.compute-1.amazonaws.com --port=5432 --username=yyy --password
```


## クリーンアップ

```bash
# Heroku Postgres Add-on のみ削除したい場合
heroku addons:destroy heroku-postgresql:hobby-dev --app myapp

# Heroku App ごと削除したい場合
heroku apps:destroy --app myapp
```

---

実行例

```bash
% heroku addons:destroy heroku-postgresql:hobby-dev --app fierce-depths-xxx
 ▸    WARNING: Destructive Action
 ▸    This command will affect the app fierce-depths-xxx
 ▸    To proceed, type fierce-depths-xxx or re-run this command with --confirm fierce-depths-xxx

> fierce-depths-xxx
Destroying postgresql-concave-yyy on ⬢ fierce-depths-xxx... done

% heroku apps:destroy --app fierce-depths-xxx
 ▸    WARNING: This will delete ⬢ fierce-depths-xxx including all add-ons.
 ▸    To proceed, type fierce-depths-xxx or re-run this command with --confirm fierce-depths-xxx

> fierce-depths-xxx
Destroying ⬢ fierce-depths-xxx (including all add-ons)... done
```


## この記事を試した環境

```bash
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% psql --version
psql (PostgreSQL) 13.1
% heroku --version
heroku/7.47.7 darwin-x64 node-v12.16.2
```