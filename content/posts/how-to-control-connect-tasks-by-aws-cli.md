---
title: "AWS CLI を使って Amazon Connect Tasks を作成・削除する"
date: 2021-01-20T19:54:25+09:00
draft: true
toc: true
images:
tags: 
  - Amazon Connect
  - AWS CLI
---

<!--more-->

```bash
REGION=ap-northeast-1
INSTANCE_ID=xxx
CONTACT_FLOW_ID=yyy
TASK_NAME=mytask
CONTACT_ID=$(aws --region $REGION connect start-task-contact --instance-id $INSTANCE_ID --contact-flow-id $CONTACT_FLOW_ID --name $TASK_NAME --query ContactId --output text)
aws --region $REGION connect stop-contact --contact-id $CONTACT_ID --instance-id $INSTANCE_ID
```


## Amazon Connect Tasks とは

- Amazon Connect を使用している各エージェントに対して「タスク」を与える機能[^1]
- タスクとは、雑に言うと ToDo リストみたいなもの
  - コールと同じようにエージェントにルーティングされて、エージェントが承諾したら特定のタスクに取り組む・・・といった感じで使う
    - Salesforce 等の外部サービスと連携することも出来るらしい[^2]
  - ドキュメントには以下のようなユースケースが書いてある[^3]
    - Salesforce 等の CRM: Customer Relationship Management ソリューションと連携したカスタマーへのフォローアップ
    - コールを使用したカスタマーへのフォローアップ
      - タスクを確認して、エージェントから特定のカスタマーにアウトバウンドコールする的な
    - カスタマーに関連して行うビジネス固有のアクション


## AWS CLI でタスクを作成

[start-task-contact](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/connect/start-task-contact.html) コマンドを使う[^4]

```bash
REGION=ap-northeast-1
INSTANCE_ID=xxx
CONTACT_FLOW_ID=yyy
TASK_NAME=mytask
aws --region $REGION connect start-task-contact --instance-id $INSTANCE_ID --contact-flow-id $CONTACT_FLOW_ID --name $TASK_NAME
```


## AWS CLI でタスクを削除

[stop-contact](https://awscli.amazonaws.com/v2/documentation/api/latest/reference/connect/stop-contact.html) コマンドを使う[^5]

```bash
REGION=ap-northeast-1
INSTANCE_ID=xxx
CONTACT_ID=yyy
aws --region $REGION connect stop-contact --contact-id $CONTACT_ID --instance-id $INSTANCE_ID
```


## この記事を試した環境

```zsh
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% aws --version
aws-cli/2.1.6 Python/3.7.4 Darwin/19.6.0 exe/x86_64 prompt/off
```


[^1]: Amazon Connect Tasks (エージェントのタスクを自動化、追跡、管理) | AWS  
https://aws.amazon.com/jp/connect/tasks/

[^2]: Amazon Connect – よりスマートになり、サードパーティーツールとの統合が向上 | Amazon Web Services ブログ  
https://aws.amazon.com/jp/blogs/news/amazon-connect-smarter-and-more-integrated/

[^3]: Tasks - Amazon Connect  
https://docs.aws.amazon.com/connect/latest/adminguide/tasks.html

[^4]: StartTaskContact - Amazon Connect Service  
https://docs.aws.amazon.com/connect/latest/APIReference/API_StartTaskContact.html

[^5]: StopContact - Amazon Connect Service  
https://docs.aws.amazon.com/connect/latest/APIReference/API_StopContact.html
