---
title: "curl で外部 json ファイルを元に POST リクエストする方法"
date: 2021-01-07T08:00:34+09:00
toc: true
draft: false
---

`--data`, `-d` オプションで `@/file/to/path` という感じで指定すれば良い。

<!--more-->

## 実行例

```zsh
ENDPOINT=https://www.example.com

# ミニマムなリクエスト
# -d, --data オプションを使用する場合、自動で POST リクエストになるので -X POST は不要
curl --data @./request.json $ENDPOINT

# Content-Type として application/json を指定
# 明示的に Content-Type を指定しない場合、 Content-Type は application/x-www-form-urlencoded になる
curl --data @./request.json -H "Content-Type: application/json" $ENDPOINT

# 標準出力の json を受け取ってリクエスト
echo '{ "hoge": "fuga"}' | curl --data @- $ENDPOINT
```

動作検証には http://httpdump.io/ が便利


## request.json サンプル

JSON 形式であれば何でも良いが、以下の URL にサンプルがあるので試してみると良いかも

JSON Example  
https://json.org/example.html


## curl の manual を読む

curl - How To Use  
https://curl.se/docs/manpage.html

> -d, --data <data>  
> (HTTP MQTT) Sends the specified data in a POST request to the HTTP server, in the same way that a browser does when a user has filled in an HTML form and presses the submit button. This will cause curl to pass the data to the server using the content-type application/x-www-form-urlencoded. Compare to -F, --form.  
> --data-raw is almost the same but does not have a special interpretation of the @ character. To post data purely binary, you should instead use the --data-binary option. To URL-encode the value of a form field you may use --data-urlencode.
> If any of these options is used more than once on the same command line, the data pieces specified will be merged together with a separating &-symbol. Thus, using '-d name=daniel -d skill=lousy' would generate a post chunk that looks like 'name=daniel&skill=lousy'.  
> If you start the data with the letter @, the rest should be a file name to read the data from, or - if you want curl to read the data from stdin. Posting data from a file named 'foobar' would thus be done with -d, --data @foobar. When -d, --data is told to read from a file like that, carriage returns and newlines will be stripped out. If you don't want the @ character to have a special interpretation use --data-raw instead.  
> See also --data-binary, --data-urlencode and --data-raw. This option overrides -F, --form and -I, --head and -T, --upload-file. 

- `-d`, `--data` オプションは POST リクエストで指定されたデータを HTTP サーバーに送信する
    - `content-type` として `application/x-www-form-urlencoded` を使用する
- `--data-raw` は殆ど同じだが、 `@` 文字の特別な解釈が無い
    - データを純粋にバイナリで送信する場合には `--data-binary` オプションを使用する必要がある
    - フォームフィールドの値を URL エンコードするには `--data-urlencode` を使用できる
- これらのオプションのいずれかが同じコマンドラインで複数回使用される場合、指定されたデータ部分は区切り文字と一緒にマージされる
    - `-d name=daniel -d skill=lousy` と指定をすると `name=daniel&skill=lousy` という post chunk が生成される
- データを `@` で始める場合、残りはデータを読み取るためのファイル名である必要がある
    - curl で stdin からデータを読み取る場合は `-`
    - foobar というファイルからのデータ投稿をする場合 `-d, --data @foobar` のように指定する
    - `-d`, `--data` がファイル読み取りを指示されると、キャリッジリターンと改行が削除される
    - 純粋に `@` を文字として使用したい場合、 `--data-raw` を使用する
- `--data-binary`, `--data-urlencode`, `--data-raw` も参照すると良い
    - これらのオプションは `-F`, `--form`, `-I`, `--head`, `-T`, `--upload-file` を上書きする


## この記事を試した環境

```zsh
% uname -v
Darwin Kernel Version 19.6.0: Tue Nov 10 00:10:30 PST 2020; root:xnu-6153.141.10~1/RELEASE_X86_64
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% curl --version
curl 7.64.1 (x86_64-apple-darwin19.0) libcurl/7.64.1 (SecureTransport) LibreSSL/2.8.3 zlib/1.2.11 nghttp2/1.39.2
Release-Date: 2019-03-27
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp smb smbs smtp smtps telnet tftp 
Features: AsynchDNS GSS-API HTTP2 HTTPS-proxy IPv6 Kerberos Largefile libz MultiSSL NTLM NTLM_WB SPNEGO SSL UnixSockets
```