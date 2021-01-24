---
title: "Apache Bench ノート"
date: 2021-01-25T01:01:38+09:00
draft: false
toc: true
images:
tags: 
  - Apache Bench
---

- `ab`: Apache Bench について調べたノート
- 基本形
  - `ab -c 10 -n 10 http://example.com/`
- オプション
  - `-A`: Basic 認証での user/password 指定
  - `-c`: concurrency. 並行実行するコネクション数
  - `-C`: Cookie 指定
  - `-H`: 任意の HTTP リクエストヘッダー指定
  - `-k`: HTTP KeepAlive を有効化
  - `-m`: HTTP Method を指定
  - `-n`: 生成する HTTP リクエスト数
  - `-p`: POST file
  - `-s`: タイムアウト設定。デフォルト 30 秒
  - `-t`: ベンチマークのタイムリミット。デフォルトでは無制限
  - `-T`: POST, PUT メソッドを使用する場合の Content-Type ヘッダー値。デフォルトでは text/plain だが application/x-www-form-urlencoded 等を指定できる
- 実行結果
  - 1秒 あたりに行ったリクエスト数が知りたい: `Requests per second` を確認
  - 失敗したリクエストを知りたい: `Non-2xx responses` を確認

<!--more-->

## 基本的な使い方

```bash
ab -c 10 -n 10 http://example.com/
```

---

実行例

```bash
$ ab -c 10 -n 10 http://example.com/
This is ApacheBench, Version 2.3 <$Revision: 1879490 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking example.com (be patient).....done


Server Software:        ECS
Server Hostname:        example.com
Server Port:            80

Document Path:          /
Document Length:        1256 bytes

Concurrency Level:      10
Time taken for tests:   0.466 seconds
Complete requests:      10
Failed requests:        0
Total transferred:      16159 bytes
HTML transferred:       12560 bytes
Requests per second:    21.46 [#/sec] (mean)
Time per request:       465.913 [ms] (mean)
Time per request:       46.591 [ms] (mean, across all concurrent requests)
Transfer rate:          33.87 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:      106  112   5.6    110     124
Processing:   106  112   5.6    110     124
Waiting:      106  112   5.6    110     124
Total:        212  224  11.2    220     248

Percentage of the requests served within a certain time (ms)
  50%    220
  66%    224
  75%    232
  80%    234
  90%    248
  95%    248
  98%    248
  99%    248
 100%    248 (longest request)
```


## POST リクエストをベンチマークする

```bash
ab -c 10 -n 10 -p ./sample.json -T application/json https://httpbin.org/post
```

---

実行例

```bash
$ cat ./sample.json 
{
    "key": "value"
}
$ ab -c 10 -n 10 -p ./sample.json -T application/json https://httpbin.org/post
This is ApacheBench, Version 2.3 <$Revision: 1879490 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking httpbin.org (be patient).....done


Server Software:        gunicorn/19.9.0
Server Hostname:        httpbin.org
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES128-GCM-SHA256,2048,128
Server Temp Key:        ECDH P-256 256 bits
TLS Server Name:        httpbin.org

Document Path:          /post
Document Length:        439 bytes

Concurrency Level:      10
Time taken for tests:   1.358 seconds
Complete requests:      10
Failed requests:        0
Total transferred:      6640 bytes
Total body sent:        1580
HTML transferred:       4390 bytes
Requests per second:    7.36 [#/sec] (mean)
Time per request:       1358.083 [ms] (mean)
Time per request:       135.808 [ms] (mean, across all concurrent requests)
Transfer rate:          4.77 [Kbytes/sec] received
                        1.14 kb/s sent
                        5.91 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:      495  502   6.4    500     512
Processing:   167  172   8.1    169     194
Waiting:      167  172   8.2    169     194
Total:        662  674   9.8    672     690

Percentage of the requests served within a certain time (ms)
  50%    672
  66%    681
  75%    682
  80%    684
  90%    690
  95%    690
  98%    690
  99%    690
 100%    690 (longest request)
```


## help

```bash
$ ab
ab: wrong number of arguments
Usage: ab [options] [http[s]://]hostname[:port]/path
Options are:
    -n requests     Number of requests to perform
    -c concurrency  Number of multiple requests to make at a time
    -t timelimit    Seconds to max. to spend on benchmarking This implies -n 50000
    -s timeout      Seconds to max. wait for each response Default is 30 seconds
    -b windowsize   Size of TCP send/receive buffer, in bytes
    -B address      Address to bind to when making outgoing connections
    -p postfile     File containing data to POST. Remember also to set -T
    -u putfile      File containing data to PUT. Remember also to set -T
    -T content-type Content-type header to use for POST/PUT data, eg. 'application/x-www-form-urlencoded' Default is 'text/plain'
    -v verbosity    How much troubleshooting info to print
    -w              Print out results in HTML tables
    -i              Use HEAD instead of GET
    -x attributes   String to insert as table attributes
    -y attributes   String to insert as tr attributes
    -z attributes   String to insert as td or th attributes
    -C attribute    Add cookie, eg. 'Apache=1234'. (repeatable)
    -H attribute    Add Arbitrary header line, eg. 'Accept-Encoding: gzip' Inserted after all normal header lines. (repeatable)
    -A attribute    Add Basic WWW Authentication, the attributes are a colon separated username and password.
    -P attribute    Add Basic Proxy Authentication, the attributes are a colon separated username and password.
    -X proxy:port   Proxyserver and port number to use
    -V              Print version number and exit
    -k              Use HTTP KeepAlive feature
    -d              Do not show percentiles served table.
    -S              Do not show confidence estimators and warnings.
    -q              Do not show progress when doing more than 150 requests
    -l              Accept variable document length (use this for dynamic pages)
    -g filename     Output collected data to gnuplot format file.
    -e filename     Output CSV file with percentages served
    -r              Don\'t exit on socket receive errors.
    -m method       Method name
    -h              Display usage information (this message)
    -I              Disable TLS Server Name Indication (SNI) extension
    -Z ciphersuite  Specify SSL/TLS cipher suite (See openssl ciphers)
    -f protocol     Specify SSL/TLS protocol (SSL3, TLS1, TLS1.1, TLS1.2 or ALL)
    -E certfile     Specify optional client certificate chain and private key
```


## manual

```bash
AB(1)                                                                             ab                                                                            AB(1)

NAME
       ab - Apache HTTP server benchmarking tool

SYNOPSIS
       ab  [  -A  auth-username:password ] [ -b windowsize ] [ -B local-address ] [ -c concurrency ] [ -C cookie-name=value ] [ -d ] [ -e csv-file ] [ -E client-cer‐tificate file ] [ -f protocol ] [ -g gnuplot-file ] [ -h ] [ -H custom-header ] [ -i ] [ -k ] [ -l ] [ -m HTTP-method ] [ -n requests ] [ -p POST-file ] [ -P proxy-auth-username:password  ]  [  -q  ] [ -r ] [ -s timeout ] [ -S ] [ -t timelimit ] [ -T content-type ] [ -u PUT-file ] [ -v verbosity] [ -V ] [ -w ] [ -x <table>-attributes ] [ -X proxy[:port] ] [ -y <tr>-attributes ] [ -z <td>-attributes ] [ -Z ciphersuite ] [http[s]://]hostname[:port]/path

SUMMARY
       ab is a tool for benchmarking your Apache Hypertext Transfer Protocol (HTTP) server. 
       It is designed to give you an impression of how your current Apache installation performs. 
       This especially shows you how many requests per second your Apache installation is capable of serving.

OPTIONS
       -A auth-username:password
              Supply  BASIC  Authentication credentials to the server. 
              The username and password are separated by a single : and sent on the wire base64 encoded. 
              The string is sent regardless of whether the server needs it (i.e., has sent an 401 authentication needed).

       -b windowsize
              Size of TCP send/receive buffer, in bytes.

       -B local-address
              Address to bind to when making outgoing connections.

       -c concurrency
              Number of multiple requests to perform at a time. 
              Default is one request at a time.

       -C cookie-name=value
              Add a Cookie: line to the request. 
              The argument is typically in the form of a name=value pair. 
              This field is repeatable.

       -d     Do not display the "percentage served within XX [ms] table". (legacy support).

       -e csv-file
              Write a Comma separated value (CSV) file which contains for each percentage (from 1% to 100%) the time (in milliseconds) it took to serve that percentage of the requests. 
              This is usually more useful than the 'gnuplot' file; as the results are already 'binned'.

       -E client-certificate-file
              When connecting to an SSL website, use the provided client certificate in PEM format to authenticate with the server. 
              The file is expected to contain the client certificate, followed by intermediate certificates, followed by the private key. Available in 2.4.36 and later.

       -f protocol
              Specify SSL/TLS protocol (SSL2, SSL3, TLS1, TLS1.1, TLS1.2, or ALL). TLS1.1 and TLS1.2 support available in 2.4.4 and later.

       -g gnuplot-file
              Write all measured values out as a 'gnuplot' or TSV (Tab separate values) file. 
              This file can easily be imported into packages like Gnuplot, IDL, Mathematica, Igor or even Excel. 
              The labels are on the first line of the file.

       -h     Display usage information.

       -H custom-header
              Append extra headers to the request. 
              The argument is typically in the form of a valid header line, containing a colon-separated field-value pair (i.e., "Accept-Encoding: zip/zop;8bit").

       -i     Do HEAD requests instead of GET.

       -k     Enable the HTTP KeepAlive feature, i.e., perform multiple requests within one HTTP session. Default is no KeepAlive.

       -l     Do not report errors if the length of the responses is not constant. This can be useful for dynamic pages. Available in 2.4.7 and later.

       -m HTTP-method
              Custom HTTP method for the requests. Available in 2.4.10 and later.

       -n requests
              Number of requests to perform for the benchmarking session. 
              The default is to just perform a single request which usually leads to non-representative benchmarking results.

       -p POST-file
              File containing data to POST. Remember to also set -T.

       -P proxy-auth-username:password
              Supply BASIC Authentication credentials to a proxy en-route. 
              The username and password are separated by a single : and sent on the wire base64 encoded.
              The string is sent regardless of whether the proxy needs it (i.e., has sent an 407 proxy authentication needed).

       -q     When processing more than 150 requests, ab outputs a progress count on stderr every 10% or 100 requests or so. 
              The -q flag will suppress these messages.

       -r     Don\'t exit on socket receive errors.

       -s timeout
              Maximum number of seconds to wait before the socket times out. 
              Default is 30 seconds. 
              Available in 2.4.4 and later.

       -S     Do  not  display  the median and standard deviation values, nor display the warning/error messages when the average and median are more than one or two times the standard deviation apart. 
              And default to the min/avg/max values. (legacy support).

       -t timelimit
              Maximum number of seconds to spend for benchmarking. This implies a -n 50000 internally. 
              Use this to benchmark the server within a fixed total amount of time. 
              Per default there is no timelimit.

       -T content-type
              Content-type header to use for POST/PUT data, eg. application/x-www-form-urlencoded. Default is text/plain.

       -u PUT-file
              File containing data to PUT. Remember to also set -T.

       -v verbosity
              Set verbosity level - 4 and above prints information on headers, 3 and above prints response codes (404, 200, etc.), 2 and above prints warnings and info.

       -V     Display version number and exit.

       -w     Print out results in HTML tables. Default table is two columns wide, with a white background.

       -x <table>-attributes
              String to use as attributes for <table>. Attributes are inserted <table here>.

       -X proxy[:port]
              Use a proxy server for the requests.

       -y <tr>-attributes
              String to use as attributes for <tr>.

       -z <td>-attributes
              String to use as attributes for <td>.

       -Z ciphersuite
              Specify SSL/TLS cipher suite (See openssl ciphers)

OUTPUT
       The following list describes the values returned by ab:

       Server Software
              The value, if any, returned in the server HTTP header of the first successful response. 
              This includes all characters in the header from beginning to the point a character with decimal value of 32 (most notably: a space or CR/LF) is detected.

       Server Hostname
              The DNS or IP address given on the command line

       Server Port
              The port to which ab is connecting. 
              If no port is given on the command line, this will default to 80 for http and 443 for https.

       SSL/TLS Protocol
              The protocol parameters negotiated between the client and server. 
              This will only be printed if SSL is used.

       Document Path
              The request URI parsed from the command line string.

       Document Length
              This is the size in bytes of the first successfully returned document. 
              If the document length changes during testing, the response is considered an error.

       Concurrency Level
              The number of concurrent clients used during the test

       Time taken for tests
              This is the time taken from the moment the first socket connection is created to the moment the last response is received

       Complete requests
              The number of successful responses received

       Failed requests
              The number of requests that were considered a failure. 
              If the number is greater than zero, another line will be printed showing the number of requests that failed due to connecting, reading, incorrect content length, or exceptions.

       Write errors
              The number of errors that failed during write (broken pipe).

       Non-2xx responses
              The number of responses that were not in the 200 series of response codes. If all responses were 200, this field is not printed.

       Keep-Alive requests
              The number of connections that resulted in Keep-Alive requests

       Total body sent
              If  configured  to  send  data  as part of the test, this is the total number of bytes sent during the tests. 
              This field is omitted if the test did not include a body to send.

       Total transferred
              The total number of bytes received from the server. 
              This number is essentially the number of bytes sent over the wire.

       HTML transferred
              The total number of document bytes received from the server. 
              This number excludes bytes received in HTTP headers

       Requests per second
              This is the number of requests per second. 
              This value is the result of dividing the number of requests by the total time taken

       Time per request
              The average time spent per request. 
              The first value is calculated with the formula concurrency * timetaken * 1000 / done while the second value is calculated with the formula timetaken * 1000 / done

       Transfer rate
              The rate of transfer as calculated by the formula totalread / 1024 / timetaken

BUGS
       There are  various  statically  declared buffers of fixed length. 
       Combined with the lazy parsing of the command line arguments, the response headers from the server and other external inputs, this might bite you.

       It does not implement HTTP/1.x fully; only accepts some 'expected' forms of responses. 
       The rather heavy use of strstr(3) shows up top in profile, which might indicate a performance problem; i.e., you would measure the ab performance rather than the server\'s.

Apache HTTP Server                                                            2018-10-10                                                                        AB(1)
```


## Official Web Info

ab - Apache HTTP server benchmarking tool - Apache HTTP Server Version 2.4  
https://httpd.apache.org/docs/2.4/programs/ab.html


## この記事を試した環境

```bash
$ cat /etc/os-release 
NAME="Amazon Linux"
VERSION="2"
ID="amzn"
ID_LIKE="centos rhel fedora"
VERSION_ID="2"
PRETTY_NAME="Amazon Linux 2"
ANSI_COLOR="0;33"
CPE_NAME="cpe:2.3:o\:amazon:amazon_linux:2"
HOME_URL="https://amazonlinux.com/"
$ uname --kernel-name --kernel-release --kernel-version --machine
Linux 4.14.209-160.339.amzn2.x86_64 #1 SMP Wed Dec 16 22:44:04 UTC 2020 x86_64
$ ab -V
This is ApacheBench, Version 2.3 <$Revision: 1879490 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/
```