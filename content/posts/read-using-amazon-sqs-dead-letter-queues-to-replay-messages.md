---
title: "Using Amazon SQS dead-letter queues to replay messages 読んだメモ"
date: 2021-01-24T16:50:17+09:00
draft: true
toc: true
images:
tags: 
  - SQS
---

以下のブログ記事を読んだメモ

Using Amazon SQS dead-letter queues to replay messages | AWS Compute Blog  
https://aws.amazon.com/jp/blogs/compute/using-amazon-sqs-dead-letter-queues-to-replay-messages/

- SQS の DLQ を扱う方法として、 DLQ を 2つ 使う構成
- 1つめの DLQ は Lambda 関数のトリガーとして設定し、アプリケーションで処理されなかったメッセージを Lambda 関数によって再処理する
  - この Lambda 関数で処理できなかったものに関しては SQS メッセージ属性にて再試行回数をカウントして、所定のリトライ回数を超えたら 2つめ の DLQ にメッセージを送信し、あとは手動で処理する
  - Lambda 関数による DLQ のリトライではメッセージ属性を使ってエクスポネンシャルバックオフによる間隔調整を行う

<!--more-->


## Introduction

- SQS はフルマネージドなキューイングサービス
  - マイクロサービス、分散システム、サーバーレスアプリケーションを分離して拡張できる
  - SQS の一般的に使用される機能は、デッドレターキュー
    - DLQ: Dead Letter Queue は、正常に処理（消費）できないメッセージを格納するために使用する
- この記事では、既存の SQS キューに自動復元機能を追加する方法を解説
  - DLQ を監視し、メッセージをメインキューに戻して、再度処理できるかどうかを確認する
    - 特定のアルゴリズムを使用して、これが永久に繰り返されないようにする
    - メッセージの再処理を試みるたびに、メッセージが最終的に無効と見なされるまで、再生時間が長くなる
- SQS DLQ, AWS Lambda, および特定のアルゴリズムを使用して、失敗したメッセージの再試行率を下げる
  - このサーバーレスソリューションはパッケージ化して AWS サーバーレスアプリケーションリポジトリに公開している


## Dead-letter queues and message replay

- DLQ の主なタスクは、メッセージの失敗を処理すること
  - これにより、未処理のメッセージを脇に置いて分離し、処理が失敗した理由を判別できる
  - 多くの場合、これらの失敗したメッセージはアプリケーションエラーが原因で発生する
    - たとえば、コンシューマアプリケーションがメッセージを正しく解析できず、未処理の例外をスローするとする
    - この例外により、メッセージを DLQ に送信するエラーレスポンスがトリガーされる
- 失敗したメッセージを処理するために、エクスポネンシャルバックオフアルゴリズムを実装して再試行メカニズムを構築する
  - エクスポネンシャルバックオフの背後にある考え方は、連続するエラーレスポンスを再試行するまでの待機時間を徐々に長くすること
  - ほとんどのエクスポネンシャルバックオフアルゴリズムは、ジッター（ランダム化された遅延）を使用して、連続する衝突を防ぐ
    - これにより、メッセージの再試行が時間全体でより均等に分散され、より効率的に処理できるようになる


## Solution overview

![sqs-newarch.png](/images/read-using-amazon-sqs-dead-letter-queues-to-replay-messages/sqs-newarch.png)

- プロデューサーから SQS に送信されるメッセージのフロー
  1. プロデューサーはメッセージを SQS キューに送信
  2. コンシューマーが同じ SQS キュー内のメッセージの処理に失敗
  3. メッセージは、コンポーネントの設定に従って、メインの SQS キューからデフォルトのデッドレターキューに移動される
  4. Lambda 関数は SQS メインデッドレターキューをイベントソースとして設定
    - メッセージを受信して元のキューに送り返し、メッセージタイマーを追加
  5. メッセージタイマーは、エクスポネンシャルバックオフおよびジッターアルゴリズムによって定義される
  6. 再試行の回数を制限
    - メッセージがこの制限を超えると、メッセージは 2番目 の DLQ に移動され、オペレーターが手動で処理する


## How the replay function works

- SQS DLQ がメッセージを受信するたびに Lambda がトリガーされて再生機能が実行される
- リプレイコードは SQS メッセージ属性 `sqs-dlq-replay-nb` を、現在試行されている再試行回数の永続的なカウンターとして使用する
  - 再試行の回数は、（アプリケーション構成ファイルで定義されている）最大回数と比較する
    - 最大値を超えると、メッセージは人間が操作するキューに移動する
    - そうでない場合、この関数は AWS Lambda イベントデータを使用して SQS メインキューの新しいメッセージを作成する
    - 最後に、再試行カウンターを更新し、メッセージに新しいメッセージタイマーを追加して、メッセージをメインキューに送り返す

```py
def handler(event, context):
    """Lambda function handler."""
    for record in event['Records']:
        nbReplay = 0
        # number of replay
        if 'sqs-dlq-replay-nb' in record['messageAttributes']:
            nbReplay = int(record['messageAttributes']['sqs-dlq-replay-nb']["stringValue"])

        nbReplay += 1
        if nbReplay > config.MAX_ATTEMPS:
            raise MaxAttempsError(replay=nbReplay, max=config.MAX_ATTEMPS)

        # SQS attributes
        attributes = record['messageAttributes']
        attributes.update({'sqs-dlq-replay-nb': {'StringValue': str(nbReplay), 'DataType': 'Number'}})

        _sqs_attributes_cleaner(attributes)

        # Backoff
        b = backoff.ExpoBackoffFullJitter(base=config.BACKOFF_RATE, cap=config.MESSAGE_RETENTION_PERIOD)
        delaySeconds = b.backoff(n=int(nbReplay))

        # SQS
        SQS.send_message(
            QueueUrl=config.SQS_MAIN_URL,
            MessageBody=record['body'],
            DelaySeconds=int(delaySeconds),
            MessageAttributes=record['messageAttributes']
        )
```


## How to use the application

- このサーバーレスアプリケーションは、次の方法で使用できる
  - Lambda コンソール:「サーバーレスアプリリポジトリの参照」オプションを選択して関数を作成
  - パブリックアプリケーションリポジトリで amazon-sqs-dlq-replay-backoff アプリケーションを選択
    - デフォルトの SQS パラメーターとリプレイ機能パラメーターを使用してアプリケーションを設定
    - このブログ投稿で YanCui によって説明されている、サーバーレスフレームワーク
    - `AWS::ServerlessRepo::Application` リソースを使用した CloudFormation テンプレート
- AWS サーバーレスアプリケーションリポジトリアプリケーションを使用した CloudFormation テンプレートの例

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Transform: AWS::Serverless-2016-10-31

Resources:
  ReplaySqsQueue:
    Type: AWS::Serverless::Application
    Properties:
      Location: 
        ApplicationId: arn:aws:serverlessrepo:eu-west-1:1234123412:applications~sqs-dlq-replay
        SemanticVersion: 1.0.0
      Parameters:
        BackoffRate: "2"
        MaxAttempts: "3"
```


## Conclusion

- エクスポネンシャルバックオフアルゴリズム（ジッターあり）が SQS キューのメッセージ処理機能をどのように強化するかについて説明
  - これで amazon-sqs-dlq-replay-backoff アプリケーションが AWS サーバーレスアプリケーションリポジトリにある
    - GitHub リポジトリでも公開している: https://github.com/aws-samples/amazon-sqs-dlq-replay-backoff
- SQS のデッドレターキューの使用を開始するには、以下
  - http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-dead-letter-queues.html
  - http://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/MonitorSQSwithCloudWatch.html
- 再生メカニズムを実装するには、以下を参照
  - https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/
  - https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/sqs-message-timers.html
- サーバーレス学習リソースの詳細については http:////serverlessland.com
