---
title: "Kinesis Data Streams のコンシューマーとして動かす AWS Lambda 関数サンプル"
date: 2021-01-19T08:51:43+09:00
toc: true
    - Kinesis Data Streams
    - AWS Lambda
draft: false
---

- Kinesis Data Streams に連携されるデータを確認したい場合等に使う
- ログの出力先等として Kinesis Data Streams に連携するサービスは結構存在するので、サクッと試す分にはマネジメントコンソールから編集可能な Python, Node.js 等を使うのが便利

<!--more-->

## サンプルコード

```py
import json
from logging import getLogger, INFO, DEBUG
import base64

logger = getLogger(__name__)
logger.setLevel(INFO)
logger.info("init")

def lambda_handler(event, context):
    logger.info(json.dumps(event, ensure_ascii=False, indent=2))
    logger.info('base64 decorded records data')
    for record in event['Records']:
        logger.info(base64.b64decode(record['kinesis']['data']))
    return event
```


## AWS CLI を使って Kinesis Data Streams にデータを投入するサンプル

```bash
STREAM_NAME=mystream
PERTITION_KEY=my-partition
REGION=ap-northeast-1
aws --region $REGION kinesis put-record --stream-name $STREAM_NAME --data $(uuidgen|base64) --partition-key $PERTITION_KEY

# ループしたい場合
while sleep 1; do aws --region $REGION kinesis put-record --stream-name $STREAM_NAME --data $(uuidgen|base64) --partition-key $PERTITION_KEY; done
```

- Kinesis Data Streams に連携されているデータは base64 エンコードされる場合が多いため、上記コマンドでも base64 エンコードしているが、必須ではない


## この記事を試した環境

```bash
% sw_vers
ProductName:	Mac OS X
ProductVersion:	10.15.7
BuildVersion:	19H114
% aws --version
aws-cli/2.1.6 Python/3.7.4 Darwin/19.6.0 exe/x86_64 prompt/off
```

- AWS Lambda ランタイム: Python 3.8