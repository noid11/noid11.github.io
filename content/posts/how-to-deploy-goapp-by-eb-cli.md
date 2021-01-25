---
title: "Go で作成した Web アプリを EB CLI を使ってデプロイする方法"
date: 2021-01-26T02:40:20+09:00
draft: true
toc: true
images:
tags: 
  - Go
  - EB
  - EB CLI
---

- ほぼほぼ[公式のチュートリアル](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/go-tutorial.html)をなぞっただけ
- 良い点
  - とりあえず Go で Web アプリ作って動かしたいような場合に便利
    - ミドルウェアの事とか考えなくて良い
  - 5分程度でインターネット経由でアクセス出来る環境にデプロイできる
  - 大体の場合、[12ヶ月間無料枠](https://aws.amazon.com/jp/free/)で済む
    - 新しく始めるプロジェクトの PoC やプロトタイプを動かすようなシナリオに相性が良い
  - やる気になれば[ログを確認](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-logs.html)したり、[アプリケーションが動作する EC2 インスタンスに SSH 接続したり](https://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-ssh.html)できる
- 注意点
  - HTTPS ではなく HTTP でのアクセスとなる
  - デフォルトでは EC2 インスタンスロールを設定しないので AWS SDK を使いたい場合には工夫が必要
    - `eb create` するときに `--instance_profile` で指定しておくとか
  - 簡易的な設定で動作するため、複雑な要件が必要なプロダクション環境には向かないかも
    - ちゃんと設定内容を精査して調整した方が良い
    - 複雑な要件が必要なら EB CLI, EB ではない方法で管理した方が良いかもしれない

<!--more-->

## 手順

プロジェクトディレクトリを作成し `application.go` として Go のコードを書く

```bash
mkdir eb-go && cd eb-go
touch application.go
```

---

application.go

```go
package main

import (
	"fmt"
	"net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
	if r.URL.Path == "/" {
		fmt.Fprintf(w, "Hello World! Append a name to the URL to say hello. For example, use %s/Mary to say hello to Mary.", r.Host)
	} else {
		fmt.Fprintf(w, "Hello, %s!", r.URL.Path[1:])
	}
}

func main() {
	http.HandleFunc("/", handler)
	http.ListenAndServe(":5000", nil)
}
```

---

アプリケーションをローカルで動かす

```zsh
go run application.go

# 別のターミナルやブラウザで http://localhost:5000 にアクセスすれば OK
# EB 環境にデプロイする際にはポート 80 で公開される
curl http://localhost:5000
```

---

Elastic Beanstalk 環境の構築・デプロイ

```zsh
eb init --platform go go-tutorial --region ap-northeast-1

# ここで EB 環境に用いる EC2 インスタンスやらのリソースが CloudFormation 経由でプロビジョニングされるため、 5分 程度の時間がかかる
eb create go-env
```

---

デプロイされた Web アプリをブラウザで開く

```zsh
eb open
```

---

クリーンアップ

```zsh
eb terminate
```


## 実行ログ例

```zsh
% eb init -p go go-tutorial --region ap-northeast-1
Application go-tutorial has been created.
% eb create go-env
Creating application version archive "app-210126_023350".
Uploading go-tutorial/app-210126_023350.zip to S3. This may take a while.
Upload Complete.
Environment details for: go-env
  Application name: go-tutorial
  Region: ap-northeast-1
  Deployed Version: app-210126_023350
  Environment ID: e-mhb6jbhawa
  Platform: arn:aws:elasticbeanstalk:ap-northeast-1::platform/Go 1 running on 64bit Amazon Linux/2.17.2
  Tier: WebServer-Standard-1.0
  CNAME: UNKNOWN
  Updated: 2021-01-25 17:33:52.752000+00:00
Printing Status:
2021-01-25 17:33:51    INFO    createEnvironment is starting.
2021-01-25 17:33:52    INFO    Using elasticbeanstalk-ap-northeast-1-123456789012 as Amazon S3 storage bucket for environment data.
2021-01-25 17:34:14    INFO    Created load balancer named: awseb-e-m-AWSEBLoa-1QR55QWY2AVZX
2021-01-25 17:34:14    INFO    Created security group named: awseb-e-mhb6jbhawa-stack-AWSEBSecurityGroup-1WRY5JQKP69QQ
2021-01-25 17:34:17    INFO    Created Auto Scaling launch configuration named: awseb-e-mhb6jbhawa-stack-AWSEBAutoScalingLaunchConfiguration-GOVWI86F79E3
2021-01-25 17:35:20    INFO    Created Auto Scaling group named: awseb-e-mhb6jbhawa-stack-AWSEBAutoScalingGroup-MW08E9EZ94AT
2021-01-25 17:35:20    INFO    Waiting for EC2 instances to launch. This may take a few minutes.
2021-01-25 17:35:20    INFO    Created Auto Scaling group policy named: arn:aws:autoscaling:ap-northeast-1:123456789012:scalingPolicy:9f066c87-60b4-407f-a339-70b0517dcd2a:autoScalingGroupName/awseb-e-mhb6jbhawa-stack-AWSEBAutoScalingGroup-MW08E9EZ94AT:policyName/awseb-e-mhb6jbhawa-stack-AWSEBAutoScalingScaleUpPolicy-1KC69AR7HGIY7
2021-01-25 17:35:20    INFO    Created Auto Scaling group policy named: arn:aws:autoscaling:ap-northeast-1:123456789012:scalingPolicy:f7442ed6-3f7e-456d-9338-941e85f4057b:autoScalingGroupName/awseb-e-mhb6jbhawa-stack-AWSEBAutoScalingGroup-MW08E9EZ94AT:policyName/awseb-e-mhb6jbhawa-stack-AWSEBAutoScalingScaleDownPolicy-82YZ8T3H24VQ
2021-01-25 17:35:36    INFO    Created CloudWatch alarm named: awseb-e-mhb6jbhawa-stack-AWSEBCloudwatchAlarmHigh-Z3HRCDA02JTS
2021-01-25 17:35:36    INFO    Created CloudWatch alarm named: awseb-e-mhb6jbhawa-stack-AWSEBCloudwatchAlarmLow-8WILR7PPYF5F
2021-01-25 17:36:36    INFO    Application available at go-env.eba-psw5vsuv.ap-northeast-1.elasticbeanstalk.com.
2021-01-25 17:36:37    INFO    Successfully launched environment: go-env
% eb terminate
The environment "go-env" and all associated instances will be terminated.
To confirm, type the environment name: go-env
2021-01-25 17:39:05    INFO    terminateEnvironment is starting.
2021-01-25 17:39:23    INFO    Deleted CloudWatch alarm named: awseb-e-mhb6jbhawa-stack-AWSEBCloudwatchAlarmHigh-Z3HRCDA02JTS 
2021-01-25 17:39:23    INFO    Deleted CloudWatch alarm named: awseb-e-mhb6jbhawa-stack-AWSEBCloudwatchAlarmLow-8WILR7PPYF5F 
2021-01-25 17:39:23    INFO    Deleted Auto Scaling group policy named: arn:aws:autoscaling:ap-northeast-1:123456789012:scalingPolicy:9f066c87-60b4-407f-a339-70b0517dcd2a:autoScalingGroupName/awseb-e-mhb6jbhawa-stack-AWSEBAutoScalingGroup-MW08E9EZ94AT:policyName/awseb-e-mhb6jbhawa-stack-AWSEBAutoScalingScaleUpPolicy-1KC69AR7HGIY7
2021-01-25 17:39:23    INFO    Deleted Auto Scaling group policy named: arn:aws:autoscaling:ap-northeast-1:123456789012:scalingPolicy:f7442ed6-3f7e-456d-9338-941e85f4057b:autoScalingGroupName/awseb-e-mhb6jbhawa-stack-AWSEBAutoScalingGroup-MW08E9EZ94AT:policyName/awseb-e-mhb6jbhawa-stack-AWSEBAutoScalingScaleDownPolicy-82YZ8T3H24VQ
2021-01-25 17:39:23    INFO    Waiting for EC2 instances to terminate. This may take a few minutes.
2021-01-25 17:41:25    INFO    Deleted Auto Scaling group named: awseb-e-mhb6jbhawa-stack-AWSEBAutoScalingGroup-MW08E9EZ94AT
2021-01-25 17:41:25    INFO    Deleted load balancer named: awseb-e-m-AWSEBLoa-1QR55QWY2AVZX
2021-01-25 17:41:25    INFO    Deleted Auto Scaling launch configuration named: awseb-e-mhb6jbhawa-stack-AWSEBAutoScalingLaunchConfiguration-GOVWI86F79E3
2021-01-25 17:41:25    INFO    Deleted security group named: awseb-e-mhb6jbhawa-stack-AWSEBSecurityGroup-1WRY5JQKP69QQ
2021-01-25 17:41:30    INFO    Deleting SNS topic for environment go-env.
2021-01-25 17:41:32    INFO    terminateEnvironment completed successfully.
```


## この記事を試した環境

```zsh
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% go version
go version go1.15.6 darwin/amd64
% eb --version
EB CLI 3.19.3 (Python 3.9.1)
```

