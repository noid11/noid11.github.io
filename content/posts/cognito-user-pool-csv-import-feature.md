---
title: "Cognito User Pool の CSV インポート機能"
date: 2021-01-05T20:18:51+09:00
toc: true
tags:
    - Cognito
draft: false
---

Cognito User Pool における CSV インポート機能についてのメモ

<!--more-->

## ユーザーインポート機能の概要

- Cognito User Pool におけるユーザーインポート機能は2種類ある[^cognito-user-pools-import-users]
    1. ユーザー移行 Lambda トリガーを用いたユーザーインポート
        - ユーザーが既存のパスワードを使用して移行可能
        - ユーザーのサインインアクションが行われたタイミングで移行が行われるため、移行のためにユーザーアクションが必要
            - つまり非アクティブなユーザーについては移行されない
    2. CSV ファイルを用いたユーザーインポート
        - CSV ファイルを用意することでバルクでユーザー移行が可能
            - 非アクティブなユーザーについても CSV ファイルに含めれば移行されるので、ある程度まとまった単位でユーザー移行したい場合に適している
        - ユーザーのパスワードは CSV で指定できないため、インポート後各ユーザーのパスワードリセットが必要


## CSV インポート機能とは？

- 先述の Cognito User Pool にユーザーをインポートする機能の1つ
- ユーザー情報を記述した CSV ファイルを元に、 Cognito User Pool にユーザーをインポートする機能[^cognito-user-pools-using-import-tool]
    - CSV ファイルでユーザーパスワードの設定はサポートされていないため、この方法でインポートされたユーザーは  RESET_REQUIRED ステータスとなり、パスワードリセットが必要となる[^API_UserType]

Cognito User Pool におけるユーザーステータスの遷移図

> ![user-status](/images/cognito-user-pool-import-feature/amazon-cognito-sign-in-confirm-user.png)

https://docs.aws.amazon.com/cognito/latest/developerguide/signing-up-users-in-your-app.html より引用


## ユーザーインポートの流れ

1. IAM ロールを作成する
    - これは Cognito User Pool が CSV インポートする際に CloudWatch Logs にログ出力を行うため必要となるもの
2. CSV ファイルを作成
    - ユーザー情報を記述した CSV ファイルを作成する
3. ユーザーインポートジョブを作成
    - CSV ファイルをインポートするためのジョブを作成
    - ジョブは非同期に実行されるようになっており、ジョブを作成するだけではインポートは行われず、明示的にジョブの開始を行う必要がある
4. CSV ファイルをアップロード
    - ジョブを作成した際に、 CSV ファイルをアップロードするための URL が発行されるので `curl` コマンド等を用いて CSV ファイルのアップロードを行う
5. ユーザーインポートジョブを開始
6. インポートジョブのログを確認
    - CloudWatch Logs に出力される
7. インポートユーザーがパスワードをリセットするように要求
    - これはアプリケーション実装側のトピックとなるので、この記事では省略する


## AWS CLI を使ってインポートを試す

### IAM ロール

- IAM ロールについてはマネジメントコンソールからインポートを試した際に作成されるサービスロールを使用する[^cognito-user-pools-using-import-tool-cli-cloudwatch-iam-role]

`get-role` 結果

```zsh
% aws iam get-role --role-name Cognito-UserImport-Role   
{
    "Role": {
        "Path": "/service-role/",
        "RoleName": "Cognito-UserImport-Role",
        "RoleId": "AROAIQGN5YPP2UMLFTJVM",
        "Arn": "arn:aws:iam::123456789012:role/service-role/Cognito-UserImport-Role",
        "CreateDate": "2018-06-30T09:54:58+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "cognito-idp.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        },
        "MaxSessionDuration": 3600,
        "RoleLastUsed": {
            "LastUsedDate": "2021-01-05T18:35:15+00:00",
            "Region": "ap-northeast-1"
        }
    }
}
```


自分の環境では、以下のようなマネージドポリシー `Cognito-1530352497476` がアタッチされた

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Stmt1467314338000",
            "Effect": "Allow",
            "Action": [
                "logs:CreateLogGroup",
                "logs:CreateLogStream",
                "logs:DescribeLogStreams",
                "logs:PutLogEvents"
            ],
            "Resource": [
                "arn:aws:logs:ap-northeast-1:123456789012:log-group:/aws/cognito/*"
            ]
        }
    ]
}
```


### CSV ファイルを用意

get-csv-header コマンドで CSV に必要となるヘッダーを取得できる[^get-csv-header]

```zsh
USER_POOL_ID=ap-northeast-1_xxx
aws cognito-idp get-csv-header --user-pool-id $USER_POOL_ID
aws cognito-idp get-csv-header --user-pool-id $USER_POOL_ID --query CSVHeader
```


実行例

```zsh
% aws cognito-idp get-csv-header --user-pool-id $USER_POOL_ID --query CSVHeader --output text
[
    "name",
    "given_name",
    "family_name",
    "middle_name",
    "nickname",
    "preferred_username",
    "profile",
    "picture",
    "website",
    "email",
    "email_verified",
    "gender",
    "birthdate",
    "zoneinfo",
    "locale",
    "phone_number",
    "phone_number_verified",
    "address",
    "updated_at",
    "cognito:mfa_enabled",
    "cognito:username"
]
```


`--output` を `text` とするとコマンドを使って加工しやすいかもしれない

```zsh
% aws cognito-idp get-csv-header --user-pool-id $USER_POOL_ID --query CSVHeader --output text
name    given_name      family_name     middle_name     nickname        preferred_username      profile picture website email   email_verified  gender  birthdate       zoneinfo        locale  phone_number       phone_number_verified   address updated_at      cognito:mfa_enabled     cognito:username
```


これで取得したヘッダーを元に CSV ファイルを作成する

```csv
name,given_name,family_name,middle_name,nickname,preferred_username,profile,picture,website,email,email_verified,gender,birthdate,zoneinfo,locale,phone_number,phone_number_verified,address,updated_at,cognito:mfa_enabled,cognito:username
```


サンプル CSV (User Pool 設定によって必要となる header や属性が異なるので注意)

```csv
name,given_name,family_name,middle_name,nickname,preferred_username,profile,picture,website,email,email_verified,gender,birthdate,zoneinfo,locale,phone_number,phone_number_verified,address,updated_at,cognito:mfa_enabled,cognito:username
johndoe,,,,,,,,,johndoe@example.com,TRUE,,,,,,FALSE,,,FALSE,johndoe
janeroe,,,,,,,,,janeroe@example.com,TRUE,,,,,,FALSE,,,FALSE,janeroe
```


### ジョブを作成

`create-user-import-job` コマンドでインポートジョブを作成[^create-user-import-job]

```zsh
JOB_NAME=myjob
CLOUD_WATCH_LOGS_ROLE_ARN=arn:aws:iam::123456789012:role/service-role/Cognito-UserImport-Role
aws cognito-idp create-user-import-job --job-name $JOB_NAME --user-pool-id $USER_POOL_ID --cloud-watch-logs-role-arn $CLOUD_WATCH_LOGS_ROLE_ARN
```


実行例

```zsh
% aws cognito-idp create-user-import-job --job-name $JOB_NAME --user-pool-id $USER_POOL_ID --cloud-watch-logs-role-arn $CLOUD_WATCH_LOGS_ROLE_ARN
{
    "UserImportJob": {
        "JobName": "myjob",
        "JobId": "import-xxx",
        "UserPoolId": "ap-northeast-1_xxx",
        "PreSignedUrl": "https://aws-cognito-idp-user-import-nrt.s3.ap-northeast-1.amazonaws.com/123456789012/ap-northeast-1_xxx/import-xxx?...",
        "CreationDate": "2021-01-05T21:06:37.381000+09:00",
        "Status": "Created",
        "CloudWatchLogsRoleArn": "arn:aws:iam::123456789012:role/service-role/Cognito-UserImport-Role",
        "ImportedUsers": 0,
        "SkippedUsers": 0,
        "FailedUsers": 0
    }
}
```


CSV ファイルのアップロード

```zsh
PRE_SIGNED_URL="https://aws-cognito-idp-user-import-nrt.s3.ap-northeast-1.amazonaws.com/123456789012/ap-northeast-1_xxx/import-xxx?..."
curl -v -T ./user-import-csv-template.csv -H "x-amz-server-side-encryption:aws:kms" $PRE_SIGNED_URL
```

### 作成したジョブを開始

`start-user-import-job` でジョブを開始[^start-user-import-job]

```zsh
JOB_ID=import-xxx
aws cognito-idp start-user-import-job --user-pool-id $USER_POOL_ID --job-id $JOB_ID
```


実行例

```zsh
% aws cognito-idp start-user-import-job --user-pool-id $USER_POOL_ID --job-id $JOB_ID
{
    "UserImportJob": {
        "JobName": "myjob",
        "JobId": "import-xxx",
        "UserPoolId": "ap-northeast-1_xxx",
        "PreSignedUrl": "https://aws-cognito-idp-user-import-nrt.s3.ap-northeast-1.amazonaws.com/123456789012/ap-northeast-1_xxx/import-xxx?...",
        "CreationDate": "2021-01-06T03:49:04.035000+09:00",
        "StartDate": "2021-01-06T03:50:45.478000+09:00",
        "Status": "Pending",
        "CloudWatchLogsRoleArn": "arn:aws:iam::123456789012:role/service-role/Cognito-UserImport-Role",
        "ImportedUsers": 0,
        "SkippedUsers": 0,
        "FailedUsers": 0
    }
}
```


### インポートジョブのステータス確認

`describe-user-import-job` コマンドでインポートジョブのステータスを確認可能[^describe-user-import-job]

```zsh
aws cognito-idp describe-user-import-job --user-pool-id $USER_POOL_ID --job-id $JOB_ID
```


実行例

```zsh
% aws cognito-idp describe-user-import-job --user-pool-id $USER_POOL_ID --job-id $JOB_ID
{
    "UserImportJob": {
        "JobName": "myjob",
        "JobId": "import-xxx",
        "UserPoolId": "ap-northeast-1_xxx",
        "PreSignedUrl": "https://aws-cognito-idp-user-import-nrt.s3.ap-northeast-1.amazonaws.com/123456789012/ap-northeast-1_xxx/import-xxx?...",
        "CreationDate": "2021-01-06T03:49:04.035000+09:00",
        "StartDate": "2021-01-06T03:50:47.752000+09:00",
        "CompletionDate": "2021-01-06T03:50:48.531000+09:00",
        "Status": "Succeeded",
        "CloudWatchLogsRoleArn": "arn:aws:iam::123456789012:role/service-role/Cognito-UserImport-Role",
        "ImportedUsers": 2,
        "SkippedUsers": 0,
        "FailedUsers": 0,
        "CompletionMessage": "Import Job Completed Successfully."
    }
}
```


### ユーザーが追加されたか確認

`list-users` コマンドで確認[^list-users]

```zsh
aws cognito-idp list-users --user-pool-id $USER_POOL_ID
```


実行例

```zsh
% aws cognito-idp list-users --user-pool-id $USER_POOL_ID
{
    "Users": [
        {
            "Username": "janeroe",
            "Attributes": [
                {
                    "Name": "sub",
                    "Value": "c754c8af-55e4-413f-afdb-895bd17e8078"
                },
                {
                    "Name": "email_verified",
                    "Value": "true"
                },
                {
                    "Name": "name",
                    "Value": "janeroe"
                },
                {
                    "Name": "phone_number_verified",
                    "Value": "false"
                },
                {
                    "Name": "email",
                    "Value": "janeroe@example.com"
                }
            ],
            "UserCreateDate": "2021-01-06T03:50:48.369000+09:00",
            "UserLastModifiedDate": "2021-01-06T03:50:48.369000+09:00",
            "Enabled": true,
            "UserStatus": "RESET_REQUIRED"
        },
        {
            "Username": "johndoe",
            "Attributes": [
                {
                    "Name": "sub",
                    "Value": "556cdeca-59af-49e2-903c-8d2452ff1cae"
                },
                {
                    "Name": "email_verified",
                    "Value": "true"
                },
                {
                    "Name": "name",
                    "Value": "johndoe"
                },
                {
                    "Name": "phone_number_verified",
                    "Value": "false"
                },
                {
                    "Name": "email",
                    "Value": "johndoe@example.com"
                }
            ],
            "UserCreateDate": "2021-01-06T03:50:48.363000+09:00",
            "UserLastModifiedDate": "2021-01-06T03:50:48.363000+09:00",
            "Enabled": true,
            "UserStatus": "RESET_REQUIRED"
        }
    ]
}
```


## ジョブのステータス

- API Reference の DataType に定義がある[^API_UserImportJobType]
    - `Created`: ジョブが作成されたものの未開始
    - `Pending`: ジョブが開始されたもののユーザーはインポートしていない遷移状態
    - `InProgress`: ジョブが開始されユーザーをインポート中
    - `Stopping`: ジョブの停止をリクエストしたが、ジョブがユーザーインポートを停止していない
    - `Stopped`: ジョブがユーザーインポートを停止した
    - `Succeeded`: ジョブが正常に完了した
    - `Faild`: エラーのためジョブが失敗した
    - `Expired`: ジョブを作成したが 24 - 48 時間の間にジョブを開始しなかった。ジョブに関連付けられているデータは全て削除され、ジョブは開始できない


## トラブルシュート Tips

- CSV のインポートが失敗する
    - CloudWatch Logs に出力されるログを見るとヒントがあるかも
    - 以下のようなフォーマットで CloudWatch Logs に出力が行われる
        - Log Group: `/aws/cognito/userpools/ap-northeast-1_xxx/my-user-pool-name`
        - Log Stream: `import-job-id/import-job-name`


## この記事を試した環境

```zsh
% uname -v
Darwin Kernel Version 19.6.0: Tue Nov 10 00:10:30 PST 2020; root:xnu-6153.141.10~1/RELEASE_X86_64
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% aws --version
aws-cli/2.1.6 Python/3.7.4 Darwin/19.6.0 exe/x86_64 prompt/off
```

[^cognito-user-pools-import-users]: Importing Users into a User Pool - Amazon Cognito  
https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-import-users.html

[^cognito-user-pools-using-import-tool]: Importing Users into User Pools From a CSV File - Amazon Cognito  
https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-using-import-tool.html

[^API_UserType]: UserType - Amazon Cognito Identity Provider  
https://docs.aws.amazon.com/cognito-user-identity-pools/latest/APIReference/API_UserType.html

[^cognito-user-pools-using-import-tool-cli-cloudwatch-iam-role]: Creating the CloudWatch Logs IAM Role (AWS CLI, API) - Amazon Cognito  
https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-using-import-tool-cli-cloudwatch-iam-role.html

[^get-csv-header]: https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cognito-idp/get-csv-header.html

[^create-user-import-job]: https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cognito-idp/create-user-import-job.html

[^start-user-import-job]: https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cognito-idp/start-user-import-job.html

[^describe-user-import-job]: https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cognito-idp/describe-user-import-job.html

[^list-users]: https://awscli.amazonaws.com/v2/documentation/api/latest/reference/cognito-idp/list-users.html

[^API_UserImportJobType]: UserImportJobType - Amazon Cognito Identity Provider
https://docs.aws.amazon.com/cognito-user-identity-pools/latest/APIReference/API_UserImportJobType.html
