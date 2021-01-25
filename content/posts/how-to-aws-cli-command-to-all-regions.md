---
title: "AWS CLI コマンドを利用可能な全てのリージョンに対して実行する方法"
date: 2021-01-25T14:42:12+09:00
draft: true
toc: true
images:
tags: 
  - AWS CLI
  - jq
  - xargs
---

- `aws ec2 describe-regions` と `jq` によって利用可能な全てのリージョンを取得し、 `xargs` コマンドで全てのリージョンに対して任意の AWS CLI コマンドを実行すれば OK

全リージョンに対して `ec2 describe-instances` を実行する例

```bash
aws ec2 describe-regions | jq --raw-output ".Regions[].RegionName" | xargs -n 1 -I{} -P 5 aws --region {} ec2 describe-instances
```

<!--more-->


## 利用可能な AWS リージョンの取得

- EC2 で提供されている [DescribeRegions API](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeRegions.html) でリージョン情報を取得できる
- AWS CLI の場合 [`ec2 describe-regions` コマンド](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/ec2/describe-regions.html)
- 厳密に全ての AWS リージョンを取得する場合 `--all-regions` オプションを指定する必要があるが、これは使用するためにオプトインが必要なリージョンがあり、どの AWS アカウントでも全てのリージョンが利用可能では無いので注意

```bash
aws ec2 describe-regions
```

---

- `xargs` コマンドに連携するためにリージョン情報を改行された raw text に整形するため `jq` を使う
- [`jq` マニュアル](https://stedolan.github.io/jq/manual/)

```bash
aws ec2 describe-regions | jq --raw-output ".Regions[].RegionName"
```


## xargs あれこれ

- `xargs` は標準入力で受け取った文字列を使って任意のコマンドを実行できる
  - この任意のコマンドはマルチプロセスで動かすことも可能なので、複数の処理が必要な場合に便利
- オプション
  - `-n`: 1コマンドラインが受け取る最大引数の個数
  - `-P`: 最大プロセス数。 0 を指定すると可能な限り多くのプロセスを使う
    - しかし Mac 環境で 0 指定すると `xargs: max. processes must be >0` というエラーが発生する。 BSD というより Mac 独自の制約かもしれない。
  - `-I`: 入力内容の置換
    - `-I{}` と指定することで、任意のコマンドを実行する際に `{}` を使い、入力を挿入することができる
    - 標準入力関係なく、あるコマンドをマルチプロセスで実行したいような場合 `seq 10 | xargs -n 1 -P 5 -I{} uuidgen` という感じで `seq` コマンドで回数指定しつつ `-I{}` の `{}` を使わないことで特定コマンドを任意の回数、マルチプロセスで動かせる
      - マルチプロセスで SQS にメッセージ投入する例: `seq 100 | xargs -n 1 -P 0 -I{} aws sqs send-message --queue-url $QUEUE_URL --message-body mybody`

```bash
xargs -n 1 -I{} -P 3 aws --region {} ec2 describe-instances
```

---

■ `-P` 調整例

`-P`: 1

```zsh
% time aws ec2 describe-regions | jq --raw-output ".Regions[].RegionName" | xargs -n 1 -I{} -P 1 aws --region {} ec2 describe-instances
...
aws ec2 describe-regions  0.65s user 0.24s system 87% cpu 1.009 total
jq --raw-output ".Regions[].RegionName"  0.03s user 0.01s system 3% cpu 1.008 total
xargs -n 1 -I{} -P 1 aws --region {} ec2 describe-instances  11.51s user 4.33s system 44% cpu 35.704 total
```

`-P`: 5

```zsh
% time aws ec2 describe-regions | jq --raw-output ".Regions[].RegionName" | xargs -n 1 -I{} -P 5 aws --region {} ec2 describe-instances
...
aws ec2 describe-regions  0.73s user 0.32s system 65% cpu 1.600 total
jq --raw-output ".Regions[].RegionName"  0.03s user 0.00s system 2% cpu 1.599 total
xargs -n 1 -I{} -P 5 aws --region {} ec2 describe-instances  15.39s user 5.32s system 148% cpu 13.934 total
```


## この記事を試した環境

```zsh
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% aws --version
aws-cli/2.1.6 Python/3.7.4 Darwin/19.6.0 exe/x86_64 prompt/off
% jq --version
jq-1.6
```

- `xargs` はバージョン表示する機能が見当たらなかったが Mac に標準でインストールされているものを使用した
