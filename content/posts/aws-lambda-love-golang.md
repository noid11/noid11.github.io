---
title: "AWS Lambda で Go ランタイムを使うのが良いと思っている理由"
date: 2021-01-01T15:05:18+09:00
toc: true
tags: 
    - AWS Lambda
    - Go
draft: false
---

個人的に AWS Lambda では zip 形式で golang ランタイム使うのがコスパ最強な気がしているので、根拠をまとめておくメモ

<!--more-->

## パフォーマンス

- コールドスタートの影響が少ない
    - JVM 相当のコンポーネントが存在しないためコールドスタートの影響が少ない
- マルチコアを活かしやすい
    - AWS Lambda では Lambda 関数に設定したメモリサイズに応じて割り当てられる CPU パワーが変化する[^configuration-memory]
        - 今後変化する可能性もあるものの、現状メモリサイズを最低値である 128 MB としても利用コア数は 2 となる[^lambda-go-tips]
        - 1,769 MB のメモリ割り当てで 1vCPU 相当になるらしい
        - 2020/12/01 にメモリ 10GB vCPU 6 コアまでサポートされるようになった[^spec]
    - goroutine との相性が良い

## 開発体験

- 公式の型定義が使える
    - AWS 公式から Lambda 関数が受け取る event の型が定義されている[^aws-lambda-go-events]
    - TypeScript でもあったりするものの、良くも悪くも DefinitelyTyped による型定義なので公式ではなくコミュニティによるもの[^DefinitelyTyped-aws-lambda]


## 料金

- goroutine を使って concurrent な処理を実装しやすいため Lambda 関数の実行時間短縮が期待できる
    - AWS Lambda の料金はリクエスト数と実行時間を元に算出される[^price]
        - さらに 2020/12/01 に AWS Lambda の課金単位が 1ms になった[^bill]
    - そのため基本的に短い時間で処理を完了できるほうがお得
- 基本的にコンテナイメージより zip 形式の方が安く使える
    - 従来の zip 形式ではなく、コンテナイメージ形式でコードをデプロイすることが可能になった[^container]
        - コンテナイメージとカスタムランタイムは Lambda 関数の初期化時にも課金が発生するため、 zip 形式では実現できない要件がある場合にコンテナイメージを検討するのが良さそう[^bootstrap-bill]
        - 数 GB に渡るような巨大なライブラリやコードを動かしたい場合とか

[^configuration-memory]: Configuring Lambda function memory - AWS Lambda  
https://docs.aws.amazon.com/lambda/latest/dg/configuration-memory.html

[^lambda-go-tips]: Serverless連載2: AWS Lambda×Goの開発Tips | フューチャー技術ブログ  
https://future-architect.github.io/articles/20200326/

[^spec]: AWS Lambda now supports up to 10 GB of memory and 6 vCPU cores for Lambda Functions  
https://aws.amazon.com/jp/about-aws/whats-new/2020/12/aws-lambda-supports-10gb-memory-6-vcpu-cores-lambda-functions/

[^aws-lambda-go-events]: aws-lambda-go/events at master · aws/aws-lambda-go  
https://github.com/aws/aws-lambda-go/tree/master/events

[^DefinitelyTyped-aws-lambda]: DefinitelyTyped/types/aws-lambda at master · DefinitelyTyped/DefinitelyTyped  
https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/aws-lambda

[^price]: 料金 - AWS Lambda ｜AWS  
https://aws.amazon.com/jp/lambda/pricing/

[^bill]: AWS Lambda changes duration billing granularity from 100ms down to 1ms  
https://aws.amazon.com/jp/about-aws/whats-new/2020/12/aws-lambda-changes-duration-billing-granularity-from-100ms-to-1ms/

[^container]: AWS Lambda の新機能 – コンテナイメージのサポート | Amazon Web Services ブログ  
https://aws.amazon.com/jp/blogs/news/new-for-aws-lambda-container-image-support/

[^bootstrap-bill]: AWS Lambda のカスタムランタイム - AWS Lambda  
https://docs.aws.amazon.com/ja_jp/lambda/latest/dg/runtimes-custom.html#runtimes-custom-build