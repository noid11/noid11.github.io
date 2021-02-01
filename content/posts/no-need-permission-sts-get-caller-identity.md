---
title: "AWS STS における GetCallerIdentity API に許可は必要ないという話"
date: 2021-02-01T21:47:26+09:00
draft: true
toc: false
images:
tags: 
  - AWS IAM
  - AWS STS
---

- AWS STS には GetCallerIdentity API という、使用しているクレデンシャルから AWS アカウントや IAM User, IAM Role 情報を取得する API がある
- この API は自分が想定した IAM User, IAM Role のクレデンシャルを取得できているのか確認するのに便利だが、使用する IAM Policy に sts:GetCallerIdentity アクションを Allow しなくても使用することが可能
- API Reference に書いてある

<!--more-->


## API Reference を確認

GetCallerIdentity - AWS Security Token Service  
https://docs.aws.amazon.com/STS/latest/APIReference/API_GetCallerIdentity.html

> No permissions are required to perform this operation.  
> If an administrator adds a policy to your IAM user or role that explicitly denies access to the sts:GetCallerIdentity action, you can still perform this operation.  
> Permissions are not required because the same information is returned when an IAM user or role is denied access.  

- この操作を実行するために権限は必要ない
- 管理者が明示的な拒否ポリシーを設定した場合においても sts:GetCallerIdentity の呼び出しは可能
- IAM User, IAM Role のアクセスが拒否された場合でも同じ情報が返されるため、アクセス許可が不要


## Note

- 明示的な許可が不要なのは便利である反面、明示的もできない点に注意
- 例えば Cognito ID Pool 等を用いて一時的なクレデンシャルを発行するようなアプリケーションでは、そのクレデンシャルを使って GetCallerIdentity を呼び出すことによって、ゲストユーザーであっても AWS アカウント ID や IAM Role の ARN を取得することができてしまう
  - 具体的な実害があるかは思いつかないが、ちょっとしたリスクではある
- GetCallerIdentity による情報取得を防ぎたい場合、エンドユーザーに対して一時的なクレデンシャルを発行するようなアーキテクチャは避けたほうが良いかもしれない
