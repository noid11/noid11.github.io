---
title: "AWS CLI v2 を使って CloudWatch Logs を `tail -f` っぽく見る方法"
date: 2021-01-23T14:33:50+09:00
draft: true
toc: false
images:
tags: 
  - AWS CLI
  - CloudWatch Logs
---

- AWS Lambda を動かしている時にマネジメントコンソールを開かなくてもログを確認できて便利

```bash
LOG_GROUP=/aws/lambda/myfunc
aws logs tail $LOG_GROUP --follow
```

<!--more-->

## help

https://awscli.amazonaws.com/v2/documentation/api/latest/reference/logs/tail.html

- `trail -f` のような形式じゃなくても、過去 1時間 のログデータを取得するような使い方も可能
  - `aws logs tail $LOG_GROUP --since 1h`
- `--format` を `short` と設定することで、ログストリーム名やタイムスタンプのミリ秒単位を省略して表示できる
  - `aws logs tail $LOG_GROUP --follow --format short`
- `--filter-pattern` を指定することで、表示内容をフィルタできる
  - 特定 Lambda 関数における、過去1週間の Duration や Max Memory Used を確認したい場合・・・
    - `aws logs tail $LOG_GROUP --since 7d --filter-pattern "?REPORT"`
  - 特定リクエスト ID でフィルタしたい場合・・・
    - `aws logs tail $LOG_GROUP --since 2d --filter-pattern '"my-request-id"'`
      - CloudWatch Logs におけるフィルタパターンで完全一致はダブルクォーテーションで囲む必要があるので `'"hoge"'` のような指定が必要
  - フィルタパターンとして指定できる構文については CloudWatch Logs のドキュメントを参照
    - https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/FilterAndPatternSyntax.html

```zsh
% aws logs tail help
TAIL()                                                                  TAIL()



NAME
       tail -

DESCRIPTION
       Tails the logs for a CloudWatch Logs group. 
       By default, the command returns logs from all associated CloudWatch Logs streams during the past ten minutes. 
       Note that there is no guarantee for exact timestamp ordering of logs.

       See 'aws help' for descriptions of global parameters.

SYNOPSIS
            tail
          group_name <value>
          [--since <value>]
          [--follow]
          [--format <value>]
          [--filter-pattern <value>]

OPTIONS
       group_name (string) The name of the CloudWatch Logs group.

       --since (string) From what time to begin displaying logs.  
       By  default, logs will be displayed starting from ten minutes in the past. 
       The value provided can be an ISO 8601 timestamp or a relative time. 
       For relative times, provide a number and a single unit. 
       Supported units include:

          o s - seconds
          o m - minutes
          o h - hours
          o d - days
          o w - weeks

          For  example, a value of 5m would indicate to display logs starting
          five minutes in the past.
          Note that multiple units are not supported (i.e. 5h30m)

       --follow (boolean) Whether to continuously poll for new logs. 
       By default, the command will exit once there are no more logs to display.
       To exit from this mode, use Control-C.

       --format (string) The format to display the logs. 
       The following formats are supported:

          o detailed - This the default format. 
            It prints  out  the  timestamp with millisecond precision and timezones, the log stream name, and the log message.

          o short - A shortened format. 
            It prints out the a shortened timestamp and the log message.

       --filter-pattern (string) The filter pattern to use. 
       See Filter and Pattern Syntax for details. 
       If not provided, all the events are matched

       See 'aws help' for descriptions of global parameters.



                                                                        TAIL()
```


## この記事を試した環境

```zsh
% sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.7
BuildVersion:	19H114
% aws --version
aws-cli/2.1.6 Python/3.7.4 Darwin/19.6.0 exe/x86_64 prompt/off
```