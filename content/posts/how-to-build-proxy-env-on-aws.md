---
title: "AWS 上に Proxy 環境を構築するメモ"
date: 2021-02-02T22:07:44+09:00
draft: false
toc: true
images:
tags: 
  - EC2
  - Squid
  - AWS CLI
  - curl
  - wget
  - yum
---

## これは何？

- AWS 上に HTTP Proxy を構築して検証する機会があったので、構築手順をメモしておく


## 大まかな流れ

- [事前準備] VPC と public subnet, private subnet を用意しておく
  - public subnet は Internet Gateway を使ってインターネット接続できるようにしておく
  - private subnet は public subnet に構築する proxy を経由してインターネット接続するので NAT Gateway, NAT インスタンスは不要
- public subnet に EC2 インスタンスを作成して [Squid](http://www.squid-cache.org/) をインストールして動かす
- private subnet に EC2 インスタンスを作成して Proxy 設定を行い、動作を確認する
- 各 EC2 インスタンスでは OS として Amazon Linux 2 を使用する


## 各 subnet に EC2 インスタンスを構築して SSH 接続

- public subnet に関してはセキュリティグループで ssh のポートを開放しておけば良い
- private subnet については [public subnet に構築した EC2 インスタンスを踏み台として SSH 接続](https://dev.classmethod.jp/articles/direct-ssh-by-proxycommand/)する
  - [SSH を直接使用せず AWS Systems Manager の Session Manager を使って private subnet の EC2 インスタンスに接続する方法](https://dev.classmethod.jp/articles/private-subnet-instance-ssm-ssh/)もあるものの、 VPC エンドポイントの設定等が必要となり、検証の邪魔になる可能性を考慮して使用を控えた

ssh コマンドを使った接続例

```zsh
# シェル変数のセットアップ
PUBLIC_HOST=ec2-x-x-x-x.ap-northeast-1.compute.amazonaws.com
PUBLIC_HOST_USER=ec2-user
PROVATE_HOST=ip-172-31-200-153.ap-northeast-1.compute.internal
PROVATE_HOST_USER=ec2-user

# public subnet host への接続
ssh $PUBLIC_HOST_USER@$PUBLIC_HOST

# private subnet host への接続
ssh -o ProxyCommand="ssh -W %h:%p $PUBLIC_HOST_USER@$PUBLIC_HOST" $PROVATE_HOST_USER@$PROVATE_HOST
```

- `-o`: ssh config で使用される形式でオプションを指定する
  - https://man.openbsd.org/ssh#o
- `ProxyCommand`: サーバーに接続するためのコマンドを指定する
  - https://man.openbsd.org/ssh_config.5#ProxyCommand
  - 別途定義されているトークンを使用することが可能
    - https://man.openbsd.org/ssh_config.5#TOKENS
    - `%h`: リモートホストネーム
    - `%p`: リモートポート
- `-W`: クライアントの標準入力と標準出力を、セキュリティで保護されたチャネルを介してポート上のホストに転送するようリクエストする
  - https://man.openbsd.org/ssh#W


## public subnet に Squid を構築

```bash
sudo yum update
sudo yum install squid
squid -v
sudo systemctl enable squid
sudo systemctl restart squid
sudo lsof -i:3128
```

- これで Squid をインストールして 3128 ポートで動作することが確認できる
- ログを確認したい場合、以下のようにして確認できる
  - プロキシを経由したアクセスを行っていない状態ではログが出力されないためログファイル自体が存在しない場合があるので注意

```bash
sudo -s
less /var/log/squid/access.log
less /var/log/squid/cache.log
```


## private subnet で Proxy 設定を行い接続を試す

### AWS CLI

- HTTP_PROXY, HTTPS_PROXY 環境変数を設定すれば良い
  - https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-proxy.html
  - EC2 インスタンスで IAM Role を使用する場合、インスタンスメタデータにアクセスが行われるため `NO_PROXY=169.254.169.254` が必要と記載があるが、 NOPROXY を設定しなくても動作が確認できたのでスキップしている

```bash
# Squid を設定した EC2 インスタンスのプライベート IP アドレスが 172.31.28.104 だとする
export HTTP_PROXY=http://172.31.28.104:3128
export HTTPS_PROXY=http://172.31.28.104:3128
```

動作検証

```bash
aws sts get-caller-identity
```

HTTP Proxy 設定を消す場合

```bash
export -n HTTP_PROXY
export -n HTTPS_PROXY
```


### curl

- `~/.curlrc` にプロキシ設定を記述する or `--proxy` オプションでプロキシを指定する
  - `--proxy` オプションで指定するのが楽
  - https://curl.se/docs/manual.html
  - https://curl.se/docs/manpage.html#-x

curl コマンドの動作確認

```bash
curl --proxy http://172.31.28.104:3128 -I http://example.com
curl --proxy http://172.31.28.104:3128 -I https://example.com
```


### wget

- `/etc/wgetrc` にプロキシ設定を記述する or http_proxy, https_proxy 環境変数を設定する
  - 環境変数は case sensitive なので AWS CLI で設定した環境変数とは異なるので注意
  - https://www.gnu.org/software/wget/manual/wget.html#Proxies
- 環境変数使う方法のほうがファイル操作が不要なので個人的にセットアップが楽

```bash
export http_proxy=http://172.31.28.104:3128
export https_proxy=http://172.31.28.104:3128
```

動作確認

```bash
wget http://example.com
wget https://example.com
```

HTTP Proxy 設定を消す場合

```bash
export -n http_proxy
export -n https_proxy
```


### yum

- `/etc/yum.conf` にプロキシ設定を記述する必要がある
  - https://man7.org/linux/man-pages/man5/yum.conf.5.html

`/etc/yum.conf` 設定例

```
proxy=http://172.31.28.104:3128
```

動作確認

```bash
sudo yum update
```


## 参考

Squidで検証用のプロキシを作ってみた | Developers.IO  
https://dev.classmethod.jp/articles/squid-proxy-setup/


## この記事を試した環境

クライアント

```zsh
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% ssh -V
OpenSSH_8.1p1, LibreSSL 2.7.3
```


EC2 インスタンスとして使用した Amazon Linux 2

```bash
[ec2-user@ip-172-31-28-104 ~]$ uname -a
Linux ip-172-31-28-104.ap-northeast-1.compute.internal 4.14.214-160.339.amzn2.x86_64 #1 SMP Sun Jan 10 05:53:05 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
[ec2-user@ip-172-31-28-104 ~]$ cat /etc/os-release 
NAME="Amazon Linux"
VERSION="2"
ID="amzn"
ID_LIKE="centos rhel fedora"
VERSION_ID="2"
PRETTY_NAME="Amazon Linux 2"
ANSI_COLOR="0;33"
CPE_NAME="cpe:2.3:o\:amazon:amazon_linux:2"
HOME_URL="https://amazonlinux.com/"
[ec2-user@ip-172-31-28-104 ~]$ ssh -V
OpenSSH_7.4p1, OpenSSL 1.0.2k-fips  26 Jan 2017
[ec2-user@ip-172-31-28-104 ~]$ squid -v | head -n 2
Squid Cache: Version 3.5.20
Service Name: squid
[ec2-user@ip-172-31-28-104 ~]$ aws --version
aws-cli/1.18.147 Python/2.7.18 Linux/4.14.214-160.339.amzn2.x86_64 botocore/1.18.6
[ec2-user@ip-172-31-28-104 ~]$ curl --version
curl 7.61.1 (x86_64-koji-linux-gnu) libcurl/7.61.1 OpenSSL/1.0.2k zlib/1.2.7 libidn2/2.3.0 libssh2/1.4.3 nghttp2/1.41.0
Release-Date: 2018-09-05
Protocols: dict file ftp ftps gopher http https imap imaps ldap ldaps pop3 pop3s rtsp scp sftp smb smbs smtp smtps telnet tftp 
Features: AsynchDNS IDN IPv6 Largefile GSS-API Kerberos SPNEGO NTLM NTLM_WB SSL libz HTTP2 UnixSockets HTTPS-proxy Metalink
[ec2-user@ip-172-31-28-104 ~]$ wget --version | head -n 1
GNU Wget 1.14 built on linux-gnu.
[ec2-user@ip-172-31-28-104 ~]$ yum --version | head -n 1
3.4.3
```