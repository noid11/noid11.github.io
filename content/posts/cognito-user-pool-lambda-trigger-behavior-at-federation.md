---
title: "Cognito User Pool における Lambda Trigger は Federation 時にどう動くのか"
date: 2021-02-09T14:36:09+09:00
draft: false
toc: true
images:
tags: 
  - Amazon Cognito
---

- [Cognito User Pool では Google, Facebook といった外部 IdP を使用したフェデレーションが可能](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-identity-federation.html)
- このフェデレーションを行った際に、どう [Lambda トリガー](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools-working-with-aws-lambda-triggers.html)が起動するのか気になったので調べたメモ

<!--more-->

## どうやって調べたのか

- 各種 Lambda トリガーを User Pool に設定して、実際にフェデレーションしてみて Lambda 関数の起動状況を確認すれば OK


## 初回フェデレーション時のみ起動される Lambda トリガー  

### [Pre Sign-up Lambda Trigger](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-lambda-pre-sign-up.html)

Lambda 関数に連携される event 例

```json
{
  "version": "1",
  "region": "ap-northeast-1",
  "userPoolId": "ap-northeast-1_xxx",
  "userName": "IdP_userid",
  "callerContext": {
    "awsSdkVersion": "aws-sdk-unknown-unknown",
    "clientId": "xxx"
  },
  "triggerSource": "PreSignUp_ExternalProvider",
  "request": {
    "userAttributes": {
      "email_verified": "false",
      "cognito:email_alias": "",
      "cognito:phone_number_alias": "",
      "email": "sample@example.com"
    },
    "validationData": {}
  },
  "response": {
    "autoConfirmUser": false,
    "autoVerifyEmail": false,
    "autoVerifyPhone": false
  }
}
```


### [Post Confirmation Lambda Trigger](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-lambda-post-confirmation.html)

Lambda 関数に連携される event 例

```json
{
  "version": "1",
  "region": "ap-northeast-1",
  "userPoolId": "ap-northeast-1_xxx",
  "userName": "IdP_userid",
  "callerContext": {
    "awsSdkVersion": "aws-sdk-unknown-unknown",
    "clientId": "xxx"
  },
  "triggerSource": "PostConfirmation_ConfirmSignUp",
  "request": {
    "userAttributes": {
      "sub": "3c6fcb0d-b156-4d9b-8dd3-77a8fd9a8954",
      "identities": "[{\"userId\":\"userid\",\"providerName\":\"IdP\",\"providerType\":\"XXX\",\"issuer\":null,\"primary\":true,\"dateCreated\":1611313260723}]",
      "cognito:user_status": "EXTERNAL_PROVIDER",
      "email_verified": "false",
      "email": "sample@example.com"
    }
  },
  "response": {}
}
```

## 初回以降のフェデレーション時のみ起動される Lambda トリガー  

### [Pre Authentication Lambda Trigger](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-lambda-pre-authentication.html)

Lambda 関数に連携される event 例

```json
{
  "version": "1",
  "region": "ap-northeast-1",
  "userPoolId": "ap-northeast-1_xxx",
  "userName": "IdP_userid",
  "callerContext": {
    "awsSdkVersion": "aws-sdk-unknown-unknown",
    "clientId": "xxx"
  },
  "triggerSource": "PreAuthentication_Authentication",
  "request": {
    "userAttributes": {
      "sub": "3c6fcb0d-b156-4d9b-8dd3-77a8fd9a8954",
      "email_verified": "false",
      "cognito:user_status": "EXTERNAL_PROVIDER",
      "identities": "[{\"userId\":\"userid\",\"providerName\":\"IdP\",\"providerType\":\"XXX\",\"issuer\":null,\"primary\":true,\"dateCreated\":1611313260723}]",
      "email": "sample@example.com"
    },
    "validationData": {}
  },
  "response": {}
}
```


### [Post Authentication Lambda Trigger](https://docs.aws.amazon.com/cognito/latest/developerguide/user-pool-lambda-post-authentication.html)

Lambda 関数に連携される event 例

```json
{
  "version": "1",
  "region": "ap-northeast-1",
  "userPoolId": "ap-northeast-1_xxx",
  "userName": "IdP_userid",
  "callerContext": {
    "awsSdkVersion": "aws-sdk-unknown-unknown",
    "clientId": "xxx"
  },
  "triggerSource": "PostAuthentication_Authentication",
  "request": {
    "userAttributes": {
      "sub": "3c6fcb0d-b156-4d9b-8dd3-77a8fd9a8954",
      "identities": "[{\"userId\":\"userid\",\"providerName\":\"IdP\",\"providerType\":\"XXX\",\"issuer\":null,\"primary\":true,\"dateCreated\":1611313260723}]",
      "cognito:user_status": "EXTERNAL_PROVIDER",
      "email_verified": "false",
      "email": "sample@example.com"
    },
    "newDeviceUsed": false
  },
  "response": {}
}
```


## NOTE

- なぜ初回と初回以降で起動する Lambda トリガーが異なるのか？
  - Cognito User Pool でフェデレーションすると、フェデレーションしたユーザー情報が User Pool に作成される
    - 初回のフェデレーション時のみ、このユーザー作成が行われるため、呼び出される Lambda トリガーが異なるのだと思われる
      - Pre Sign-up と Post Confirmation が暗黙的に呼び出される
    - 初回以降は既にユーザー情報が作成済みであり、 Cognito User Pool としてはユーザー作成ではなく認証が行われるものとして扱われているのだと思われる
      - Pre Authentication, Post Authentication が呼び出される
    - ちなみにユーザーが作成されてから意図的に該当ユーザーを削除し、再度フェデレーションするとユーザーは再作成される
      - つまりフェデレーション時のユーザー作成は PUT 的な処理なのだと思われる
- 何に使えるのか？
  - 例えばフェデレーションする際でも許可したユーザーのみ利用を許可したい場合、 Pre Sign-up Lambda Trigger でメールアドレス等のユーザー情報をチェックして、所定のユーザー以外がフェデレーションしてきたらブロックする・・・みたいな構成ができそう
