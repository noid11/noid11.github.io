---
title: "AWS アカウントのセットアップ TIPS"
date: 2021-02-09T05:54:16+09:00
draft: false
toc: true
images:
tags: 
  - AWS
  - AWS Control Tower
  - AWS Organizations
  - AWS Single Sign-On
  - AWS Service Catalog
---

- 個人的に使っている AWS アカウントの管理とかセットアップに関する TIPS

<!--more-->

## AWS アカウントの管理には Control Tower を使う

- [AWS Control Tower](https://aws.amazon.com/jp/controltower/) はマルチアカウント環境をセキュアに管理・構築してくれるサービス
- 複数の AWS アカウントの管理と言えば [AWS Organizations](https://aws.amazon.com/jp/organizations/) や [AWS Single Sign-On](https://aws.amazon.com/jp/single-sign-on/) といったサービスがメジャーな気がするが、 Control Tower はこれらをまとめて管理してくれる
- 特に便利なのが [AWS Service Catalog](https://aws.amazon.com/jp/servicecatalog/) を使用した AWS アカウント作成で、一々 AWS アカウントのサインアップをする必要がなくなるので大変便利
  - 何か検証が必要になったら検証用の AWS アカウントを作成する・・・みたいな事が手軽にできるようになる

Provision and manage accounts with Account Factory - AWS Control Tower  
https://docs.aws.amazon.com/controltower/latest/userguide/account-factory.html


## AWS CLI v2 の SSO 設定をしておく

- AWS CLI v2 からは AWS SSO に対応したプロファイル設定が可能
- 特定の AWS アカウントに対して AWS CLI で操作したい場面は割とあるので、 AWS アカウントの作成時にセットアップしておくと良い
  - サクッと実行する分には AWS SSO でマネジメントコンソールにアクセスして CloudShell から AWS CLI を実行する・・・という方法でも良い

Configuring the AWS CLI to use AWS Single Sign-On - AWS Command Line Interface  
https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-sso.html


## AWS から送信されるマーケティングメールの登録を解除しておく

AWS Email Preference Center  
https://pages.awscloud.com/jp/communication-preferences.html?languages=japanese

- 作成した AWS アカウントのメールアドレスに同じお知らせメールが届くので、基本的には速攻解除しておくのが吉
  - 油断してメンテナンスのお知らせとか気付けないとヤバいことになる可能性もある

