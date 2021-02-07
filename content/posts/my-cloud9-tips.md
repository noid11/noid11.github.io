---
title: "個人的な AWS Cloud9 の使い方 TIPS"
date: 2021-02-07T19:37:56+09:00
draft: true
toc: true
images:
tags: 
  - Cloud9
---

- [AWS Cloud9](https://aws.amazon.com/jp/cloud9/) を稀に使うのでセットアップ方法メモ


## セットアップ

### SSH key 作成

```bash
ssh-keygen -t rsa -b 4096 -C "Cloud9 環境の ARN"
```

- あとは GitHub に登録したり
  - https://github.com/settings/keys
- [ssh-keygen manual](https://man.openbsd.org/ssh-keygen)
  - `-t`: アルゴリズム
  - `-b`: bit 数
  - `-C`: コメント
    - ユニークな値のほうが後から思い出しやすいので Cloud9 環境の ARN にしている


### Authorized keys 登録

```bash
curl -s https://github.com/noid11.keys >> ~/.ssh/authorized_keys
```

- Cloud9 として使用している EC2 インスタンスに SSH 接続したくなる場合が稀にある
  - [AWS Systems Manager のセッションマネージャーで SSH 接続も出来る](https://docs.aws.amazon.com/ja_jp/systems-manager/latest/userguide/session-manager.html)ので、基本的にはこっちを使ったほうが良い
- `~/.ssh/authorized_keys` には既に Cloud9 環境が使う SSH Key が登録されているっぽいので、ファイルの上書きではなく `>>` で追記する


## 使い方

### しばらく使わないと思ったら環境を削除

- [Cloud9 の利用そのものには料金が発生しない](https://aws.amazon.com/jp/cloud9/pricing/)ものの、 EC2 インスタンスと EBS の料金は発生する
  - [停止中の EC2 インスタンスは課金されない](https://aws.amazon.com/jp/ec2/pricing/on-demand/)が、 [EBS ボリュームは EC2 インスタンスが停止中でも料金が発する](https://aws.amazon.com/jp/premiumsupport/knowledge-center/ebs-charge-stopped-instance/)
  - [EBS の料金は 1GB / 月という単位で計算される](https://aws.amazon.com/jp/ebs/pricing/)ので、 Cloud9 環境を1ヶ月以上放置すると無駄に EBS の料金が発生する
- Cloud9 で使用するデータとは多くの場合、自分が読み書きするソースコード
  - 大体の場合 GitHub, GitLab, CodeCommit などのリモートリポジトリがあるハズ
  - もうしばらく使わ無さそうだな・・・と思ったら `git push` するなりして EBS スナップショット等はとらずに Cloud9 環境毎削除してしまって、また読み書きする際に Cloud9 環境を作成した方が良い
- 定期的にミドルウェア等のパッケージをアップデートするのも面倒


## 注意点

### Linux 環境

- 当たり前だけど普段 Mac 使ってると、コマンドが GNU と BSD でオプション違うみたいなパターンあるので注意


### 現状 AWS CLI が v1

```bash
$ aws --version
aws-cli/1.18.217 Python/2.7.18 Linux/4.14.209-160.339.amzn2.x86_64 botocore/1.19.57
```

- AWS CLI v2 が使いたい場合には自前でインストールする
  - https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2-linux.html


## Cloud9 の好きな所

- 使っている自分の PC スペックが低い場合や、ネットワーク環境が悪い場合、そこそこスペックの高い EC2 インスタンスを使うことでソコソコ快適な開発環境が手に入る
  - [AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html) を使いたい場合等、結構 Docker イメージの容量が大きかったりするのでツラくなったりする
- [様々なプログラミング言語を動かせる環境](https://docs.aws.amazon.com/cloud9/latest/user-guide/language-support.html)が揃っている
  - 一々自分でセットアップするのも面倒なのでデフォルトでインストールされているのは楽
    - 特定のバージョンで試したい！みたいな場合には頑張る必要あり
- EC2 インスタンス上で動作する
  - CloudShell のように透過的に動いて欲しい気もするが、 EC2 インスタンスが見れるので [CloudWatch メトリクスで負荷状況を確認できる](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/viewing_metrics_with_cloudwatch.html)
    - アプリのビルドや実行で、このくらいリソース使うんだな〜みたいなのが分かって良い
    - [EBS Volume のメトリクス](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using_cloudwatch_ebs.html)も見れる
- Cloud9 環境を他の人と共有できる
  - [現状同じ AWS アカウント内の IAM ユーザーのみ共有可能](https://aws.amazon.com/jp/cloud9/faqs/#Environment_Sharing)なので注意
  - まぁ共有したくなったら CloudFormation なり CDK なりで IAM ユーザー作成して連携すれば良い


## この記事を試した環境

```bash
$ cat /etc/system-release
Amazon Linux release 2 (Karoo)
$ sudo yum list installed
Loaded plugins: extras_suggestions, langpacks, priorities, update-motd
224 packages excluded due to repository priority protections
Installed Packages
GeoIP.x86_64                                 1.5.0-11.amzn2.0.2                     installed                          
PyYAML.x86_64                                3.10-11.amzn2.0.2                      installed                          
acl.x86_64                                   2.2.51-14.amzn2                        installed                          
acpid.x86_64                                 2.0.19-9.amzn2.0.1                     installed                          
amazon-linux-extras.noarch                   1.6.13-1.amzn2                         @amzn2-core                        
amazon-linux-extras-yum-plugin.noarch        1.6.13-1.amzn2                         @amzn2-core                        
amazon-ssm-agent.x86_64                      3.0.161.0-1.amzn2                      @amzn2-core                        
apr.x86_64                                   1.6.3-5.amzn2.0.2                      @amzn2-core                        
apr-util.x86_64                              1.6.1-5.amzn2.0.2                      @amzn2-core                        
apr-util-bdb.x86_64                          1.6.1-5.amzn2.0.2                      @amzn2-core                        
at.x86_64                                    3.1.13-24.amzn2                        installed                          
attr.x86_64                                  2.4.46-12.amzn2.0.2                    installed                          
audit.x86_64                                 2.8.1-3.amzn2.1                        installed                          
audit-libs.x86_64                            2.8.1-3.amzn2.1                        installed                          
authconfig.x86_64                            6.2.8-30.amzn2.0.2                     installed                          
autoconf.noarch                              2.69-11.amzn2                          @amzn2-core                        
automake.noarch                              1.13.4-3.1.amzn2                       @amzn2-core                        
avahi-libs.x86_64                            0.6.31-20.amzn2                        @amzn2-core                        
aws-cfn-bootstrap.noarch                     1.4-34.amzn2                           @amzn2-core                        
awscli.noarch                                1.18.147-1.amzn2.0.1                   @amzn2-core                        
basesystem.noarch                            10.0-7.amzn2.0.1                       installed                          
bash.x86_64                                  4.2.46-34.amzn2                        @amzn2-core                        
bash-completion.noarch                       1:2.1-6.amzn2                          installed                          
bc.x86_64                                    1.06.95-13.amzn2.0.2                   installed                          
bind-export-libs.x86_64                      32:9.11.4-26.P2.amzn2.2                @amzn2-core                        
bind-libs.x86_64                             32:9.11.4-26.P2.amzn2.2                @amzn2-core                        
bind-libs-lite.x86_64                        32:9.11.4-26.P2.amzn2.2                @amzn2-core                        
bind-license.noarch                          32:9.11.4-26.P2.amzn2.2                @amzn2-core                        
bind-utils.x86_64                            32:9.11.4-26.P2.amzn2.2                @amzn2-core                        
binutils.x86_64                              2.29.1-30.amzn2                        @amzn2-core                        
bison.x86_64                                 3.0.4-6.amzn2.0.2                      @amzn2-core                        
blktrace.x86_64                              1.0.5-9.amzn2                          installed                          
boost-date-time.x86_64                       1.53.0-27.amzn2.0.3                    installed                          
boost-system.x86_64                          1.53.0-27.amzn2.0.3                    installed                          
boost-thread.x86_64                          1.53.0-27.amzn2.0.3                    installed                          
bridge-utils.x86_64                          1.5-9.amzn2.0.2                        installed                          
byacc.x86_64                                 1.9.20130304-3.amzn2.0.2               @amzn2-core                        
bzip2.x86_64                                 1.0.6-13.amzn2.0.2                     installed                          
bzip2-libs.x86_64                            1.0.6-13.amzn2.0.2                     installed                          
ca-certificates.noarch                       2019.2.32-76.amzn2.0.3                 @amzn2-core                        
chkconfig.x86_64                             1.7.4-1.amzn2.0.2                      installed                          
chrony.x86_64                                3.5.1-1.amzn2.0.1                      @amzn2-core                        
cloud-init.noarch                            19.3-4.amzn2                           @amzn2-core                        
cloud-utils-growpart.noarch                  0.31-2.amzn2                           installed                          
containerd.x86_64                            1.4.1-2.amzn2                          @amzn2extra-docker                 
coreutils.x86_64                             8.22-24.amzn2                          installed                          
cpio.x86_64                                  2.11-28.amzn2                          @amzn2-core                        
cpp.x86_64                                   7.3.1-12.amzn2                         @amzn2-core                        
cracklib.x86_64                              2.9.0-11.amzn2.0.2                     installed                          
cracklib-dicts.x86_64                        2.9.0-11.amzn2.0.2                     installed                          
cronie.x86_64                                1.4.11-23.amzn2                        installed                          
cronie-anacron.x86_64                        1.4.11-23.amzn2                        installed                          
crontabs.noarch                              1.11-6.20121102git.amzn2               installed                          
cryptsetup.x86_64                            1.7.4-4.amzn2                          installed                          
cryptsetup-libs.x86_64                       1.7.4-4.amzn2                          installed                          
cscope.x86_64                                15.8-10.amzn2.0.2                      @amzn2-core                        
ctags.x86_64                                 5.8-13.amzn2.0.2                       @amzn2-core                        
curl.x86_64                                  7.61.1-12.amzn2.0.2                    @amzn2-core                        
cyrus-sasl-lib.x86_64                        2.1.26-23.amzn2                        installed                          
cyrus-sasl-plain.x86_64                      2.1.26-23.amzn2                        installed                          
dbus.x86_64                                  1:1.10.24-7.amzn2                      installed                          
dbus-libs.x86_64                             1:1.10.24-7.amzn2                      installed                          
dejavu-fonts-common.noarch                   2.33-6.amzn2                           @amzn2-core                        
dejavu-sans-fonts.noarch                     2.33-6.amzn2                           @amzn2-core                        
device-mapper.x86_64                         7:1.02.146-4.amzn2.0.2                 installed                          
device-mapper-event.x86_64                   7:1.02.146-4.amzn2.0.2                 installed                          
device-mapper-event-libs.x86_64              7:1.02.146-4.amzn2.0.2                 installed                          
device-mapper-libs.x86_64                    7:1.02.146-4.amzn2.0.2                 installed                          
device-mapper-persistent-data.x86_64         0.7.3-3.amzn2                          installed                          
dhclient.x86_64                              12:4.2.5-77.amzn2.1.1                  installed                          
dhcp-common.x86_64                           12:4.2.5-77.amzn2.1.1                  installed                          
dhcp-libs.x86_64                             12:4.2.5-77.amzn2.1.1                  installed                          
diffstat.x86_64                              1.57-4.amzn2.0.2                       @amzn2-core                        
diffutils.x86_64                             3.3-5.amzn2                            installed                          
dmidecode.x86_64                             1:3.0-5.amzn2.0.2                      installed                          
dmraid.x86_64                                1.0.0.rc16-28.amzn2.0.2                installed                          
dmraid-events.x86_64                         1.0.0.rc16-28.amzn2.0.2                installed                          
docker.x86_64                                19.03.13ce-1.amzn2                     @amzn2extra-docker                 
dosfstools.x86_64                            3.0.20-10.amzn2                        installed                          
doxygen.x86_64                               1:1.8.5-4.amzn2                        @amzn2-core                        
dracut.x86_64                                033-535.amzn2.1.3                      installed                          
dracut-config-generic.x86_64                 033-535.amzn2.1.3                      installed                          
dwz.x86_64                                   0.11-3.amzn2.0.2                       @amzn2-core                        
dyninst.x86_64                               9.3.1-3.amzn2                          @amzn2-core                        
e2fsprogs.x86_64                             1.42.9-19.amzn2                        @amzn2-core                        
e2fsprogs-libs.x86_64                        1.42.9-19.amzn2                        @amzn2-core                        
ec2-hibinit-agent.noarch                     1.0.2-3.amzn2                          @amzn2-core                        
ec2-instance-connect.noarch                  1.1-13.amzn2                           @amzn2-core                        
ec2-net-utils.noarch                         1.4-3.amzn2                            @amzn2-core                        
ec2-utils.noarch                             1.2-3.amzn2                            @amzn2-core                        
ed.x86_64                                    1.9-4.amzn2.0.2                        installed                          
efivar-libs.x86_64                           31-4.amzn2.0.4                         @amzn2-core                        
elfutils.x86_64                              0.176-2.amzn2                          @amzn2-core                        
elfutils-default-yama-scope.noarch           0.176-2.amzn2                          installed                          
elfutils-libelf.x86_64                       0.176-2.amzn2                          installed                          
elfutils-libelf-devel.x86_64                 0.176-2.amzn2                          @amzn2-core                        
elfutils-libs.x86_64                         0.176-2.amzn2                          installed                          
emacs-filesystem.noarch                      1:25.3-3.amzn2.0.2                     @amzn2-core                        
epel-release.noarch                          7-11                                   @amzn2extra-epel                   
ethtool.x86_64                               2:4.8-10.amzn2                         @amzn2-core                        
expat.x86_64                                 2.1.0-12.amzn2                         @amzn2-core                        
file.x86_64                                  5.11-36.amzn2.0.1                      @amzn2-core                        
file-libs.x86_64                             5.11-36.amzn2.0.1                      @amzn2-core                        
filesystem.x86_64                            3.2-25.amzn2.0.4                       installed                          
findutils.x86_64                             1:4.5.11-6.amzn2                       @amzn2-core                        
fipscheck.x86_64                             1.4.1-6.amzn2.0.2                      installed                          
fipscheck-lib.x86_64                         1.4.1-6.amzn2.0.2                      installed                          
flex.x86_64                                  2.5.37-3.amzn2.0.3                     @amzn2-core                        
fontconfig.x86_64                            2.13.0-4.3.amzn2                       @amzn2-core                        
fontpackages-filesystem.noarch               1.44-8.amzn2                           @amzn2-core                        
freetype.x86_64                              2.8-14.amzn2.1                         @amzn2-core                        
gawk.x86_64                                  4.0.2-4.amzn2.1.2                      installed                          
gcc.x86_64                                   7.3.1-12.amzn2                         @amzn2-core                        
gcc-c++.x86_64                               7.3.1-12.amzn2                         @amzn2-core                        
gcc-gfortran.x86_64                          7.3.1-12.amzn2                         @amzn2-core                        
gdb.x86_64                                   8.0.1-30.amzn2.0.3                     @amzn2-core                        
gdb-gdbserver.x86_64                         8.0.1-30.amzn2.0.3                     @amzn2-core                        
gdbm.x86_64                                  1:1.13-6.amzn2.0.2                     installed                          
gdisk.x86_64                                 0.8.10-3.amzn2                         @amzn2-core                        
generic-logos.noarch                         18.0.0-4.amzn2                         installed                          
generic-logos-httpd.noarch                   18.0.0-4.amzn2                         @amzn2-core                        
gettext.x86_64                               0.19.8.1-3.amzn2                       @amzn2-core                        
gettext-common-devel.noarch                  0.19.8.1-3.amzn2                       @amzn2-core                        
gettext-devel.x86_64                         0.19.8.1-3.amzn2                       @amzn2-core                        
gettext-libs.x86_64                          0.19.8.1-3.amzn2                       @amzn2-core                        
git.x86_64                                   2.23.3-1.amzn2.0.1                     @amzn2-core                        
git-core.x86_64                              2.23.3-1.amzn2.0.1                     @amzn2-core                        
git-core-doc.noarch                          2.23.3-1.amzn2.0.1                     @amzn2-core                        
glib2.x86_64                                 2.56.1-7.amzn2.0.1                     @amzn2-core                        
glibc.x86_64                                 2.26-39.amzn2                          @amzn2-core                        
glibc-all-langpacks.x86_64                   2.26-39.amzn2                          @amzn2-core                        
glibc-common.x86_64                          2.26-39.amzn2                          @amzn2-core                        
glibc-devel.x86_64                           2.26-39.amzn2                          @amzn2-core                        
glibc-headers.x86_64                         2.26-39.amzn2                          @amzn2-core                        
glibc-locale-source.x86_64                   2.26-39.amzn2                          @amzn2-core                        
glibc-minimal-langpack.x86_64                2.26-39.amzn2                          @amzn2-core                        
gmp.x86_64                                   1:6.0.0-15.amzn2.0.2                   installed                          
gnupg2.x86_64                                2.0.22-5.amzn2.0.4                     @amzn2-core                        
gnutls.x86_64                                3.3.29-9.amzn2                         @amzn2-core                        
golang.x86_64                                1.15.5-1.amzn2.0.2                     @amzn2-core                        
golang-bin.x86_64                            1.15.5-1.amzn2.0.2                     @amzn2-core                        
golang-src.noarch                            1.15.5-1.amzn2.0.2                     @amzn2-core                        
gpgme.x86_64                                 1.3.2-5.amzn2.0.2                      installed                          
gpm-libs.x86_64                              1.20.7-15.amzn2.0.2                    installed                          
grep.x86_64                                  2.20-3.amzn2.0.2                       installed                          
groff-base.x86_64                            1.22.2-8.amzn2.0.2                     installed                          
grub2.x86_64                                 1:2.02-35.amzn2.0.4                    installed                          
grub2-common.noarch                          1:2.02-35.amzn2.0.4                    installed                          
grub2-pc.x86_64                              1:2.02-35.amzn2.0.4                    installed                          
grub2-pc-modules.noarch                      1:2.02-35.amzn2.0.4                    installed                          
grub2-tools.x86_64                           1:2.02-35.amzn2.0.4                    installed                          
grub2-tools-extra.x86_64                     1:2.02-35.amzn2.0.4                    installed                          
grub2-tools-minimal.x86_64                   1:2.02-35.amzn2.0.4                    installed                          
grubby.x86_64                                8.28-23.amzn2.0.1                      installed                          
gssproxy.x86_64                              0.7.0-17.amzn2                         installed                          
gzip.x86_64                                  1.5-10.amzn2                           installed                          
hardlink.x86_64                              1:1.3-3.amzn2                          installed                          
hibagent.noarch                              1.1.0-5.amzn2                          installed                          
hostname.x86_64                              3.13-3.amzn2.0.2                       installed                          
httpd.x86_64                                 2.4.46-1.amzn2                         @amzn2-core                        
httpd-filesystem.noarch                      2.4.46-1.amzn2                         @amzn2-core                        
httpd-tools.x86_64                           2.4.46-1.amzn2                         @amzn2-core                        
hunspell.x86_64                              1.3.2-16.amzn2                         @amzn2-core                        
hunspell-en.noarch                           0.20121024-6.amzn2.0.1                 installed                          
hunspell-en-GB.noarch                        0.20121024-6.amzn2.0.1                 installed                          
hunspell-en-US.noarch                        0.20121024-6.amzn2.0.1                 installed                          
hwdata.x86_64                                0.252-9.3.amzn2                        @amzn2-core                        
indent.x86_64                                2.2.11-13.amzn2.0.2                    @amzn2-core                        
info.x86_64                                  5.1-5.amzn2                            installed                          
initscripts.x86_64                           9.49.47-1.amzn2.0.1                    installed                          
intltool.noarch                              0.50.2-7.amzn2                         @amzn2-core                        
iproute.x86_64                               4.15.0-1.amzn2.0.4                     @amzn2-core                        
iptables.x86_64                              1.8.4-10.amzn2.1.2                     @amzn2-core                        
iptables-libs.x86_64                         1.8.4-10.amzn2.1.2                     @amzn2-core                        
iputils.x86_64                               20160308-10.amzn2.0.2                  installed                          
irqbalance.x86_64                            2:1.5.0-4.amzn2.0.1                    installed                          
jansson.x86_64                               2.10-1.amzn2.0.2                       installed                          
java-11-amazon-corretto-headless.x86_64      1:11.0.9+12-1.amzn2                    @amzn2-core                        
javapackages-tools.noarch                    3.4.1-11.amzn2                         @amzn2-core                        
jbigkit-libs.x86_64                          2.0-11.amzn2.0.2                       installed                          
json-c.x86_64                                0.11-4.amzn2.0.4                       @amzn2-core                        
kbd.x86_64                                   1.15.5-15.amzn2                        @amzn2-core                        
kbd-legacy.noarch                            1.15.5-15.amzn2                        @amzn2-core                        
kbd-misc.noarch                              1.15.5-15.amzn2                        @amzn2-core                        
kernel.x86_64                                4.14.165-131.185.amzn2                 installed                          
kernel.x86_64                                4.14.209-160.339.amzn2                 @amzn2-core                        
kernel.x86_64                                4.14.214-160.339.amzn2                 @amzn2-core                        
kernel-devel.x86_64                          4.14.209-160.339.amzn2                 @amzn2-core                        
kernel-devel.x86_64                          4.14.214-160.339.amzn2                 @amzn2-core                        
kernel-headers.x86_64                        4.14.214-160.339.amzn2                 @amzn2-core                        
kernel-tools.x86_64                          4.14.214-160.339.amzn2                 @amzn2-core                        
keyutils.x86_64                              1.5.8-3.amzn2.0.2                      installed                          
keyutils-libs.x86_64                         1.5.8-3.amzn2.0.2                      installed                          
keyutils-libs-devel.x86_64                   1.5.8-3.amzn2.0.2                      @amzn2-core                        
kmod.x86_64                                  25-3.amzn2.0.2                         installed                          
kmod-libs.x86_64                             25-3.amzn2.0.2                         installed                          
kpartx.x86_64                                0.4.9-127.amzn2                        @amzn2-core                        
kpatch-runtime.noarch                        0.8.0-4.amzn2                          @amzn2-core                        
krb5-devel.x86_64                            1.15.1-37.amzn2.2.2                    @amzn2-core                        
krb5-libs.x86_64                             1.15.1-37.amzn2.2.2                    installed                          
langtable.noarch                             0.0.31-4.amzn2                         @amzn2-core                        
langtable-data.noarch                        0.0.31-4.amzn2                         @amzn2-core                        
langtable-python.noarch                      0.0.31-4.amzn2                         @amzn2-core                        
less.x86_64                                  458-9.amzn2.0.2                        installed                          
libacl.x86_64                                2.2.51-14.amzn2                        installed                          
libaio.x86_64                                0.3.109-13.amzn2.0.2                   installed                          
libassuan.x86_64                             2.1.0-3.amzn2.0.2                      installed                          
libatomic.x86_64                             7.3.1-12.amzn2                         @amzn2-core                        
libattr.x86_64                               2.4.46-12.amzn2.0.2                    installed                          
libbasicobjects.x86_64                       0.1.1-29.amzn2                         installed                          
libblkid.x86_64                              2.30.2-2.amzn2.0.4                     installed                          
libcap.x86_64                                2.22-9.amzn2.0.2                       installed                          
libcap-ng.x86_64                             0.7.5-4.amzn2.0.4                      installed                          
libcgroup.x86_64                             0.41-21.amzn2                          @amzn2-core                        
libcilkrts.x86_64                            7.3.1-12.amzn2                         @amzn2-core                        
libcollection.x86_64                         0.7.0-29.amzn2                         installed                          
libcom_err.x86_64                            1.42.9-19.amzn2                        @amzn2-core                        
libcom_err-devel.x86_64                      1.42.9-19.amzn2                        @amzn2-core                        
libconfig.x86_64                             1.4.9-5.amzn2.0.2                      installed                          
libcroco.x86_64                              0.6.12-6.amzn2                         @amzn2-core                        
libcrypt.x86_64                              2.26-39.amzn2                          @amzn2-core                        
libcurl.x86_64                               7.61.1-12.amzn2.0.2                    @amzn2-core                        
libdaemon.x86_64                             0.14-7.amzn2.0.2                       installed                          
libdb.x86_64                                 5.3.21-24.amzn2.0.3                    installed                          
libdb-utils.x86_64                           5.3.21-24.amzn2.0.3                    installed                          
libdrm.x86_64                                2.4.97-2.amzn2                         @amzn2-core                        
libdwarf.x86_64                              20130207-4.amzn2.0.2                   installed                          
libedit.x86_64                               3.0-12.20121213cvs.amzn2.0.2           installed                          
libestr.x86_64                               0.1.9-2.amzn2.0.2                      installed                          
libevent.x86_64                              2.0.21-4.amzn2.0.3                     installed                          
libfastjson.x86_64                           0.99.4-3.amzn2                         @amzn2-core                        
libfdisk.x86_64                              2.30.2-2.amzn2.0.4                     installed                          
libffi.x86_64                                3.0.13-18.amzn2.0.2                    installed                          
libffi-devel.x86_64                          3.0.13-18.amzn2.0.2                    @amzn2-core                        
libgcc.x86_64                                7.3.1-12.amzn2                         @amzn2-core                        
libgcrypt.x86_64                             1.5.3-14.amzn2.0.2                     installed                          
libgfortran.x86_64                           7.3.1-12.amzn2                         @amzn2-core                        
libgomp.x86_64                               7.3.1-12.amzn2                         @amzn2-core                        
libgpg-error.x86_64                          1.12-3.amzn2.0.3                       installed                          
libicu.x86_64                                50.2-4.amzn2                           @amzn2-core                        
libidn.x86_64                                1.28-4.amzn2.0.2                       installed                          
libidn2.x86_64                               2.3.0-1.amzn2                          installed                          
libini_config.x86_64                         1.3.1-29.amzn2                         installed                          
libitm.x86_64                                7.3.1-12.amzn2                         @amzn2-core                        
libjpeg-turbo.x86_64                         1.2.90-6.amzn2.0.3                     installed                          
libkadm5.x86_64                              1.15.1-37.amzn2.2.2                    @amzn2-core                        
libmetalink.x86_64                           0.1.3-13.amzn2                         @amzn2-core                        
libmnl.x86_64                                1.0.3-7.amzn2.0.2                      installed                          
libmodman.x86_64                             2.0.1-8.amzn2.0.2                      @amzn2-core                        
libmount.x86_64                              2.30.2-2.amzn2.0.4                     installed                          
libmpc.x86_64                                1.0.1-3.amzn2.0.2                      @amzn2-core                        
libmpx.x86_64                                7.3.1-12.amzn2                         @amzn2-core                        
libnetfilter_conntrack.x86_64                1.0.6-1.amzn2.0.2                      installed                          
libnfnetlink.x86_64                          1.0.1-4.amzn2.0.2                      installed                          
libnfsidmap.x86_64                           0.25-19.amzn2                          installed                          
libnghttp2.x86_64                            1.41.0-1.amzn2                         @amzn2-core                        
libnl3.x86_64                                3.2.28-4.amzn2.0.1                     installed                          
libnl3-cli.x86_64                            3.2.28-4.amzn2.0.1                     installed                          
libpath_utils.x86_64                         0.2.1-29.amzn2                         installed                          
libpcap.x86_64                               14:1.5.3-11.amzn2                      installed                          
libpciaccess.x86_64                          0.14-1.amzn2                           installed                          
libpipeline.x86_64                           1.2.3-3.amzn2.0.2                      installed                          
libpng.x86_64                                2:1.5.13-8.amzn2                       @amzn2-core                        
libproxy.x86_64                              0.4.11-10.amzn2.0.3                    @amzn2-core                        
libpwquality.x86_64                          1.2.3-5.amzn2                          installed                          
libquadmath.x86_64                           7.3.1-12.amzn2                         @amzn2-core                        
libref_array.x86_64                          0.1.5-29.amzn2                         installed                          
libsanitizer.x86_64                          7.3.1-12.amzn2                         @amzn2-core                        
libseccomp.x86_64                            2.4.1-1.amzn2                          installed                          
libsecret.x86_64                             0.18.5-2.amzn2.0.2                     @amzn2-core                        
libselinux.x86_64                            2.5-12.amzn2.0.2                       installed                          
libselinux-devel.x86_64                      2.5-12.amzn2.0.2                       @amzn2-core                        
libselinux-utils.x86_64                      2.5-12.amzn2.0.2                       installed                          
libsemanage.x86_64                           2.5-11.amzn2                           installed                          
libsepol.x86_64                              2.5-8.1.amzn2.0.2                      installed                          
libsepol-devel.x86_64                        2.5-8.1.amzn2.0.2                      @amzn2-core                        
libsmartcols.x86_64                          2.30.2-2.amzn2.0.4                     installed                          
libss.x86_64                                 1.42.9-19.amzn2                        @amzn2-core                        
libssh2.x86_64                               1.4.3-12.amzn2.2.3                     @amzn2-core                        
libsss_idmap.x86_64                          1.16.4-21.amzn2                        installed                          
libsss_nss_idmap.x86_64                      1.16.4-21.amzn2                        installed                          
libstdc++.x86_64                             7.3.1-12.amzn2                         @amzn2-core                        
libstoragemgmt.x86_64                        1.6.1-2.amzn2                          installed                          
libstoragemgmt-python.noarch                 1.6.1-2.amzn2                          installed                          
libstoragemgmt-python-clibs.x86_64           1.6.1-2.amzn2                          installed                          
libsysfs.x86_64                              2.1.0-16.amzn2.0.2                     installed                          
libtasn1.x86_64                              4.10-1.amzn2.0.2                       installed                          
libteam.x86_64                               1.27-9.amzn2                           @amzn2-core                        
libtiff.x86_64                               4.0.3-35.amzn2                         @amzn2-core                        
libtirpc.x86_64                              0.2.4-0.16.amzn2                       @amzn2-core                        
libtool.x86_64                               2.4.2-22.2.amzn2.0.2                   @amzn2-core                        
libunistring.x86_64                          0.9.3-9.amzn2.0.2                      installed                          
libuser.x86_64                               0.60-9.amzn2                           installed                          
libutempter.x86_64                           1.1.6-4.amzn2.0.2                      installed                          
libuuid.x86_64                               2.30.2-2.amzn2.0.4                     installed                          
libuv.x86_64                                 1:1.39.0-1.amzn2                       @amzn2-core                        
libverto.x86_64                              0.2.5-4.amzn2.0.2                      installed                          
libverto-devel.x86_64                        0.2.5-4.amzn2.0.2                      @amzn2-core                        
libverto-libevent.x86_64                     0.2.5-4.amzn2.0.2                      installed                          
libwebp.x86_64                               0.3.0-7.amzn2                          installed                          
libxml2.x86_64                               2.9.1-6.amzn2.5.1                      @amzn2-core                        
libxml2-python.x86_64                        2.9.1-6.amzn2.5.1                      @amzn2-core                        
libxslt.x86_64                               1.1.28-6.amzn2                         @amzn2-core                        
libyaml.x86_64                               0.1.4-11.amzn2.0.2                     installed                          
libyaml-devel.x86_64                         0.1.4-11.amzn2.0.2                     @amzn2-core                        
lm_sensors-libs.x86_64                       3.4.0-8.20160601gitf9185e5.amzn2       @amzn2-core                        
logrotate.x86_64                             3.8.6-15.amzn2                         installed                          
lsof.x86_64                                  4.87-6.amzn2                           @amzn2-core                        
lua.x86_64                                   5.1.4-15.amzn2.0.2                     installed                          
lvm2.x86_64                                  7:2.02.177-4.amzn2.0.2                 installed                          
lvm2-libs.x86_64                             7:2.02.177-4.amzn2.0.2                 installed                          
lz4.x86_64                                   1.7.5-2.amzn2.0.1                      installed                          
m4.x86_64                                    1.4.16-10.amzn2.0.2                    @amzn2-core                        
mailcap.noarch                               2.1.41-2.amzn2                         @amzn2-core                        
make.x86_64                                  1:3.82-24.amzn2                        @amzn2-core                        
man-db.x86_64                                2.6.3-9.amzn2.0.3                      installed                          
man-pages.noarch                             3.53-5.amzn2                           installed                          
man-pages-overrides.x86_64                   7.5.2-1.amzn2                          installed                          
mariadb.x86_64                               3:10.2.10-2.amzn2.0.3                  @amzn2extra-lamp-mariadb10.2-php7.2
mariadb-common.x86_64                        3:10.2.10-2.amzn2.0.3                  @amzn2extra-lamp-mariadb10.2-php7.2
mariadb-config.x86_64                        3:10.2.10-2.amzn2.0.3                  @amzn2extra-lamp-mariadb10.2-php7.2
mariadb-libs.x86_64                          3:10.2.10-2.amzn2.0.3                  @amzn2extra-lamp-mariadb10.2-php7.2
mdadm.x86_64                                 4.0-5.amzn2.0.2                        installed                          
mercurial.x86_64                             2.6.2-10.amzn2                         @amzn2-core                        
microcode_ctl.x86_64                         2:2.1-47.amzn2.0.7                     @amzn2-core                        
mlocate.x86_64                               0.26-8.amzn2                           installed                          
mod_http2.x86_64                             1.15.14-2.amzn2                        @amzn2-core                        
mokutil.x86_64                               1:0.3.0-10.amzn2.0.1                   @amzn2-core                        
mpfr.x86_64                                  3.1.1-4.amzn2.0.2                      @amzn2-core                        
mtr.x86_64                                   2:0.92-2.amzn2                         installed                          
nano.x86_64                                  2.9.8-2.amzn2.0.1                      installed                          
ncurses.x86_64                               6.0-8.20170212.amzn2.1.3               installed                          
ncurses-base.noarch                          6.0-8.20170212.amzn2.1.3               installed                          
ncurses-c++-libs.x86_64                      6.0-8.20170212.amzn2.1.3               @amzn2-core                        
ncurses-devel.x86_64                         6.0-8.20170212.amzn2.1.3               @amzn2-core                        
ncurses-libs.x86_64                          6.0-8.20170212.amzn2.1.3               installed                          
neon.x86_64                                  0.30.0-3.amzn2.0.2                     @amzn2-core                        
net-tools.x86_64                             2.0-0.22.20131004git.amzn2.0.2         installed                          
nettle.x86_64                                2.7.1-8.amzn2.0.2                      @amzn2-core                        
newt.x86_64                                  0.52.15-4.amzn2.0.2                    installed                          
newt-python.x86_64                           0.52.15-4.amzn2.0.2                    installed                          
nfs-utils.x86_64                             1:1.3.0-0.54.amzn2.0.2                 installed                          
nodejs.x86_64                                1:6.17.1-1.el7                         @epel                              
npm.x86_64                                   1:3.10.10-1.6.17.1.1.el7               @epel                              
nspr.x86_64                                  4.25.0-2.amzn2                         @amzn2-core                        
nss.x86_64                                   3.53.1-3.amzn2                         @amzn2-core                        
nss-pem.x86_64                               1.0.3-5.amzn2                          installed                          
nss-softokn.x86_64                           3.53.1-6.amzn2                         @amzn2-core                        
nss-softokn-freebl.x86_64                    3.53.1-6.amzn2                         @amzn2-core                        
nss-sysinit.x86_64                           3.53.1-3.amzn2                         @amzn2-core                        
nss-tools.x86_64                             3.53.1-3.amzn2                         @amzn2-core                        
nss-util.x86_64                              3.53.1-1.amzn2                         @amzn2-core                        
ntsysv.x86_64                                1.7.4-1.amzn2.0.2                      installed                          
numactl-libs.x86_64                          2.0.9-7.amzn2                          installed                          
openldap.x86_64                              2.4.44-22.amzn2                        @amzn2-core                        
openssh.x86_64                               7.4p1-21.amzn2.0.1                     installed                          
openssh-clients.x86_64                       7.4p1-21.amzn2.0.1                     installed                          
openssh-server.x86_64                        7.4p1-21.amzn2.0.1                     installed                          
openssl.x86_64                               1:1.0.2k-19.amzn2.0.4                  @amzn2-core                        
openssl-devel.x86_64                         1:1.0.2k-19.amzn2.0.4                  @amzn2-core                        
openssl-libs.x86_64                          1:1.0.2k-19.amzn2.0.4                  @amzn2-core                        
os-prober.x86_64                             1.58-9.amzn2.0.2                       installed                          
p11-kit.x86_64                               0.23.22-1.amzn2.0.1                    @amzn2-core                        
p11-kit-trust.x86_64                         0.23.22-1.amzn2.0.1                    @amzn2-core                        
pakchois.x86_64                              0.4-10.amzn2.0.2                       @amzn2-core                        
pam.x86_64                                   1.1.8-23.amzn2.0.1                     @amzn2-core                        
parted.x86_64                                3.1-29.amzn2                           installed                          
passwd.x86_64                                0.79-5.amzn2                           @amzn2-core                        
patch.x86_64                                 2.7.1-12.amzn2.0.2                     @amzn2-core                        
patchutils.x86_64                            0.3.3-4.amzn2.0.1                      @amzn2-core                        
pciutils.x86_64                              3.5.1-3.amzn2                          installed                          
pciutils-libs.x86_64                         3.5.1-3.amzn2                          installed                          
pcre.x86_64                                  8.32-17.amzn2.0.2                      installed                          
pcre-devel.x86_64                            8.32-17.amzn2.0.2                      @amzn2-core                        
pcre2.x86_64                                 10.23-2.amzn2.0.2                      installed                          
perl.x86_64                                  4:5.16.3-294.amzn2                     installed                          
perl-Carp.noarch                             1.26-244.amzn2                         installed                          
perl-Data-Dumper.x86_64                      2.145-3.amzn2.0.2                      @amzn2-core                        
perl-Encode.x86_64                           2.51-7.amzn2.0.2                       installed                          
perl-Error.noarch                            1:0.17020-2.amzn2                      @amzn2-core                        
perl-Exporter.noarch                         5.68-3.amzn2                           installed                          
perl-File-Path.noarch                        2.09-2.amzn2                           installed                          
perl-File-Temp.noarch                        0.23.01-3.amzn2                        installed                          
perl-Filter.x86_64                           1.49-3.amzn2.0.2                       installed                          
perl-Getopt-Long.noarch                      2.40-3.amzn2                           installed                          
perl-Git.noarch                              2.23.3-1.amzn2.0.1                     @amzn2-core                        
perl-HTTP-Tiny.noarch                        0.033-3.amzn2                          installed                          
perl-PathTools.x86_64                        3.40-5.amzn2.0.2                       installed                          
perl-Pod-Escapes.noarch                      1:1.04-294.amzn2                       installed                          
perl-Pod-Perldoc.noarch                      3.20-4.amzn2                           installed                          
perl-Pod-Simple.noarch                       1:3.28-4.amzn2                         installed                          
perl-Pod-Usage.noarch                        1.63-3.amzn2                           installed                          
perl-Scalar-List-Utils.x86_64                1.27-248.amzn2.0.2                     installed                          
perl-Socket.x86_64                           2.010-4.amzn2.0.2                      installed                          
perl-Storable.x86_64                         2.45-3.amzn2.0.2                       installed                          
perl-TermReadKey.x86_64                      2.30-20.amzn2.0.2                      @amzn2-core                        
perl-Test-Harness.noarch                     3.28-3.amzn2                           @amzn2-core                        
perl-Text-ParseWords.noarch                  3.29-4.amzn2                           installed                          
perl-Thread-Queue.noarch                     3.02-2.amzn2                           @amzn2-core                        
perl-Time-HiRes.x86_64                       4:1.9725-3.amzn2.0.2                   installed                          
perl-Time-Local.noarch                       1.2300-2.amzn2                         installed                          
perl-XML-Parser.x86_64                       2.41-10.amzn2.0.2                      @amzn2-core                        
perl-constant.noarch                         1.27-2.amzn2.0.1                       installed                          
perl-libs.x86_64                             4:5.16.3-294.amzn2                     installed                          
perl-macros.x86_64                           4:5.16.3-294.amzn2                     installed                          
perl-parent.noarch                           1:0.225-244.amzn2.0.1                  installed                          
perl-podlators.noarch                        2.5.1-3.amzn2.0.1                      installed                          
perl-srpm-macros.noarch                      1-8.amzn2.0.1                          @amzn2-core                        
perl-threads.x86_64                          1.87-4.amzn2.0.2                       installed                          
perl-threads-shared.x86_64                   1.43-6.amzn2.0.2                       installed                          
php-cli.x86_64                               7.2.24-1.amzn2.0.1                     @amzn2extra-lamp-mariadb10.2-php7.2
php-common.x86_64                            7.2.24-1.amzn2.0.1                     @amzn2extra-lamp-mariadb10.2-php7.2
php-devel.x86_64                             7.2.24-1.amzn2.0.1                     @amzn2extra-lamp-mariadb10.2-php7.2
php-fpm.x86_64                               7.2.24-1.amzn2.0.1                     @amzn2extra-lamp-mariadb10.2-php7.2
php-json.x86_64                              7.2.24-1.amzn2.0.1                     @amzn2extra-lamp-mariadb10.2-php7.2
php-mysqlnd.x86_64                           7.2.24-1.amzn2.0.1                     @amzn2extra-lamp-mariadb10.2-php7.2
php-pdo.x86_64                               7.2.24-1.amzn2.0.1                     @amzn2extra-lamp-mariadb10.2-php7.2
php-pear.noarch                              1:1.10.12-4.amzn2.0.1                  @amzn2-core                        
php-process.x86_64                           7.2.24-1.amzn2.0.1                     @amzn2extra-lamp-mariadb10.2-php7.2
php-xml.x86_64                               7.2.24-1.amzn2.0.1                     @amzn2extra-lamp-mariadb10.2-php7.2
pigz.x86_64                                  2.3.4-1.amzn2.0.1                      @amzn2-core                        
pinentry.x86_64                              0.8.1-17.amzn2.0.2                     installed                          
pkgconfig.x86_64                             1:0.27.1-4.amzn2.0.2                   installed                          
plymouth.x86_64                              0.8.9-0.28.20140113.amzn2.0.2          installed                          
plymouth-core-libs.x86_64                    0.8.9-0.28.20140113.amzn2.0.2          installed                          
plymouth-scripts.x86_64                      0.8.9-0.28.20140113.amzn2.0.2          installed                          
pm-utils.x86_64                              1.4.1-27.amzn2.0.1                     installed                          
policycoreutils.x86_64                       2.5-22.amzn2                           installed                          
popt.x86_64                                  1.13-16.amzn2.0.2                      installed                          
postfix.x86_64                               2:2.10.1-6.amzn2.0.3                   installed                          
procps-ng.x86_64                             3.3.10-26.amzn2                        installed                          
psacct.x86_64                                6.6.1-13.amzn2.0.2                     installed                          
psmisc.x86_64                                22.20-15.amzn2.0.2                     installed                          
pth.x86_64                                   2.0.7-23.amzn2.0.2                     installed                          
pygpgme.x86_64                               0.3-9.amzn2.0.2                        installed                          
pyliblzma.x86_64                             0.5.3-11.amzn2.0.2                     installed                          
pystache.noarch                              0.5.3-2.amzn2                          installed                          
python.x86_64                                2.7.18-1.amzn2.0.2                     @amzn2-core                        
python-babel.noarch                          0.9.6-8.amzn2.0.1                      installed                          
python-backports.x86_64                      1.0-8.amzn2.0.2                        installed                          
python-backports-ssl_match_hostname.noarch   3.5.0.1-1.amzn2                        installed                          
python-cffi.x86_64                           1.6.0-5.amzn2.0.2                      installed                          
python-chardet.noarch                        2.2.1-1.amzn2                          installed                          
python-colorama.noarch                       0.3.2-3.amzn2                          installed                          
python-configobj.noarch                      4.7.2-7.amzn2                          installed                          
python-daemon.noarch                         1.6-4.amzn2                            installed                          
python-devel.x86_64                          2.7.18-1.amzn2.0.2                     @amzn2-core                        
python-docutils.noarch                       0.12-0.2.20140510svn7747.amzn2         installed                          
python-enum34.noarch                         1.0.4-1.amzn2                          installed                          
python-idna.noarch                           2.4-1.amzn2                            installed                          
python-iniparse.noarch                       0.4-9.amzn2                            installed                          
python-ipaddress.noarch                      1.0.16-2.amzn2                         installed                          
python-javapackages.noarch                   3.4.1-11.amzn2                         @amzn2-core                        
python-jinja2.noarch                         2.7.2-3.amzn2                          installed                          
python-jsonpatch.noarch                      1.2-4.amzn2                            installed                          
python-jsonpointer.noarch                    1.9-2.amzn2                            installed                          
python-jwcrypto.noarch                       0.4.2-1.amzn2                          installed                          
python-kitchen.noarch                        1.1.1-5.amzn2                          installed                          
python-libs.x86_64                           2.7.18-1.amzn2.0.2                     @amzn2-core                        
python-lxml.x86_64                           3.2.1-4.amzn2.0.2                      @amzn2-core                        
python-markupsafe.x86_64                     0.11-10.amzn2.0.2                      installed                          
python-pillow.x86_64                         2.0.0-21.gitd1c6db8.amzn2.0.1          @amzn2-core                        
python-ply.noarch                            3.4-11.amzn2                           installed                          
python-pycparser.noarch                      2.14-1.amzn2                           installed                          
python-pycurl.x86_64                         7.19.0-19.amzn2.0.2                    installed                          
python-repoze-lru.noarch                     0.4-3.amzn2                            installed                          
python-requests.noarch                       2.6.0-7.amzn2                          installed                          
python-six.noarch                            1.9.0-2.amzn2                          installed                          
python-urlgrabber.noarch                     3.10-9.amzn2.0.1                       installed                          
python-urllib3.noarch                        1.25.7-1.amzn2.0.1                     @amzn2-core                        
python2-botocore.noarch                      1.18.6-1.amzn2.0.1                     @amzn2-core                        
python2-cryptography.x86_64                  1.7.2-2.amzn2                          installed                          
python2-dateutil.noarch                      1:2.6.0-3.amzn2.0.1                    installed                          
python2-futures.noarch                       3.0.5-1.amzn2                          installed                          
python2-jmespath.noarch                      0.9.3-1.amzn2.0.1                      installed                          
python2-jsonschema.noarch                    2.5.1-3.amzn2.0.1                      installed                          
python2-lockfile.noarch                      1:0.11.0-17.el7                        @epel                              
python2-oauthlib.noarch                      2.0.1-8.amzn2.0.1                      installed                          
python2-pip.noarch                           9.0.3-1.amzn2.0.2                      @amzn2-core                        
python2-pyasn1.noarch                        0.1.9-7.amzn2.0.1                      installed                          
python2-rpm.x86_64                           4.11.3-40.amzn2.0.5                    @amzn2-core                        
python2-rsa.noarch                           3.4.1-1.amzn2.0.1                      @amzn2-core                        
python2-s3transfer.noarch                    0.3.3-1.amzn2.0.1                      @amzn2-core                        
python2-setuptools.noarch                    38.4.0-3.amzn2.0.6                     installed                          
python2-simplejson.x86_64                    3.10.0-2.el7                           @epel                              
python3.x86_64                               3.7.9-1.amzn2.0.1                      @amzn2-core                        
python3-devel.x86_64                         3.7.9-1.amzn2.0.1                      @amzn2-core                        
python3-libs.x86_64                          3.7.9-1.amzn2.0.1                      @amzn2-core                        
python3-pip.noarch                           9.0.3-1.amzn2.0.2                      @amzn2-core                        
python3-rpm-macros.noarch                    3-23.amzn2                             @amzn2-core                        
python3-setuptools.noarch                    38.4.0-3.amzn2.0.6                     @amzn2-core                        
pyxattr.x86_64                               0.5.1-5.amzn2.0.2                      installed                          
qrencode-libs.x86_64                         3.4.1-3.amzn2.0.2                      installed                          
quota.x86_64                                 1:4.01-17.amzn2                        installed                          
quota-nls.noarch                             1:4.01-17.amzn2                        installed                          
rcs.x86_64                                   5.9.0-5.amzn2.0.2                      @amzn2-core                        
rdate.x86_64                                 1.4-25.amzn2.0.1                       installed                          
readline.x86_64                              6.2-10.amzn2.0.2                       installed                          
readline-devel.x86_64                        6.2-10.amzn2.0.2                       @amzn2-core                        
rng-tools.x86_64                             6.8-3.amzn2.0.4                        @amzn2-core                        
rootfiles.noarch                             8.1-11.amzn2                           installed                          
rpcbind.x86_64                               0.2.0-44.amzn2                         installed                          
rpm.x86_64                                   4.11.3-40.amzn2.0.5                    @amzn2-core                        
rpm-build.x86_64                             4.11.3-40.amzn2.0.5                    @amzn2-core                        
rpm-build-libs.x86_64                        4.11.3-40.amzn2.0.5                    @amzn2-core                        
rpm-libs.x86_64                              4.11.3-40.amzn2.0.5                    @amzn2-core                        
rpm-plugin-systemd-inhibit.x86_64            4.11.3-40.amzn2.0.5                    @amzn2-core                        
rpm-sign.x86_64                              4.11.3-40.amzn2.0.5                    @amzn2-core                        
rsync.x86_64                                 3.1.2-4.amzn2                          installed                          
rsyslog.x86_64                               8.24.0-52.amzn2                        @amzn2-core                        
ruby.x86_64                                  2.0.0.648-36.amzn2.0.1                 @amzn2-core                        
ruby-irb.noarch                              2.0.0.648-36.amzn2.0.1                 @amzn2-core                        
ruby-libs.x86_64                             2.0.0.648-36.amzn2.0.1                 @amzn2-core                        
rubygem-bigdecimal.x86_64                    1.2.0-36.amzn2.0.1                     @amzn2-core                        
rubygem-io-console.x86_64                    0.4.2-36.amzn2.0.1                     @amzn2-core                        
rubygem-json.x86_64                          1.7.7-36.amzn2.0.1                     @amzn2-core                        
rubygem-psych.x86_64                         2.0.0-36.amzn2.0.1                     @amzn2-core                        
rubygem-rdoc.noarch                          4.0.0-36.amzn2.0.1                     @amzn2-core                        
rubygems.noarch                              2.0.14.1-36.amzn2.0.1                  @amzn2-core                        
runc.x86_64                                  1.0.0-0.1.20200826.gitff819c7.amzn2    @amzn2extra-docker                 
scl-utils.x86_64                             20130529-18.amzn2.0.1                  installed                          
screen.x86_64                                4.1.0-0.25.20120314git3c2946.amzn2     installed                          
sed.x86_64                                   4.2.2-5.amzn2.0.2                      installed                          
selinux-policy.noarch                        3.13.1-192.amzn2.6.5                   @amzn2-core                        
selinux-policy-targeted.noarch               3.13.1-192.amzn2.6.5                   @amzn2-core                        
setserial.x86_64                             2.17-33.amzn2.0.1                      installed                          
setup.noarch                                 2.8.71-10.amzn2.0.1                    @amzn2-core                        
setuptool.x86_64                             1.19.11-8.amzn2.0.1                    installed                          
sgpio.x86_64                                 1.2.0.10-13.amzn2.0.1                  installed                          
shadow-utils.x86_64                          2:4.1.5.1-24.amzn2.0.2                 installed                          
shared-mime-info.x86_64                      1.8-4.amzn2                            installed                          
slang.x86_64                                 2.2.4-11.amzn2.0.2                     installed                          
sqlite.x86_64                                3.7.17-8.amzn2.1.1                     @amzn2-core                        
sqlite-devel.x86_64                          3.7.17-8.amzn2.1.1                     @amzn2-core                        
sssd-client.x86_64                           1.16.4-21.amzn2                        installed                          
strace.x86_64                                4.26-1.amzn2.0.1                       @amzn2-core                        
subversion.x86_64                            1.7.14-16.amzn2.0.1                    @amzn2-core                        
subversion-libs.x86_64                       1.7.14-16.amzn2.0.1                    @amzn2-core                        
sudo.x86_64                                  1.8.23-4.amzn2.2.1                     @amzn2-core                        
swig.x86_64                                  3.0.12-11.amzn2.0.3                    @amzn2-core                        
sysctl-defaults.noarch                       1.0-2.amzn2                            installed                          
sysstat.x86_64                               10.1.5-18.amzn2.0.1                    installed                          
system-release.x86_64                        1:2-13.amzn2                           @amzn2-core                        
system-rpm-config.noarch                     9.1.0-76.amzn2.0.10                    @amzn2-core                        
systemd.x86_64                               219-57.amzn2.0.12                      installed                          
systemd-libs.x86_64                          219-57.amzn2.0.12                      installed                          
systemd-sysv.x86_64                          219-57.amzn2.0.12                      installed                          
systemtap.x86_64                             4.2-1.amzn2.0.1                        @amzn2-core                        
systemtap-client.x86_64                      4.2-1.amzn2.0.1                        @amzn2-core                        
systemtap-devel.x86_64                       4.2-1.amzn2.0.1                        @amzn2-core                        
systemtap-runtime.x86_64                     4.2-1.amzn2.0.1                        installed                          
sysvinit-tools.x86_64                        2.88-14.dsf.amzn2.0.2                  installed                          
tar.x86_64                                   2:1.26-35.amzn2                        @amzn2-core                        
tcp_wrappers.x86_64                          7.6-77.amzn2.0.2                       installed                          
tcp_wrappers-libs.x86_64                     7.6-77.amzn2.0.2                       installed                          
tcpdump.x86_64                               14:4.9.2-4.amzn2.1                     installed                          
tcsh.x86_64                                  6.18.01-15.amzn2.0.2                   installed                          
teamd.x86_64                                 1.27-9.amzn2                           @amzn2-core                        
terraform.x86_64                             0.14.5-1                               @hashicorp                         
time.x86_64                                  1.7-45.amzn2.0.2                       installed                          
traceroute.x86_64                            3:2.0.22-2.amzn2.0.1                   installed                          
trousers.x86_64                              0.3.14-2.amzn2.0.2                     @amzn2-core                        
tzdata.noarch                                2020d-2.amzn2                          @amzn2-core                        
unzip.x86_64                                 6.0-21.amzn2                           @amzn2-core                        
update-motd.noarch                           1.1.2-2.amzn2                          installed                          
usermode.x86_64                              1.111-5.amzn2.0.2                      installed                          
ustr.x86_64                                  1.0.4-16.amzn2.0.3                     installed                          
util-linux.x86_64                            2.30.2-2.amzn2.0.4                     installed                          
vim-common.x86_64                            2:8.1.1602-1.amzn2                     installed                          
vim-enhanced.x86_64                          2:8.1.1602-1.amzn2                     installed                          
vim-filesystem.noarch                        2:8.1.1602-1.amzn2                     installed                          
vim-minimal.x86_64                           2:8.1.1602-1.amzn2                     installed                          
virt-what.x86_64                             1.18-4.amzn2                           installed                          
wget.x86_64                                  1.14-18.amzn2.1                        installed                          
which.x86_64                                 2.20-7.amzn2.0.2                       installed                          
words.noarch                                 3.0-22.amzn2                           installed                          
xfsdump.x86_64                               3.1.8-6.amzn2                          installed                          
xfsprogs.x86_64                              4.5.0-18.amzn2.0.1                     installed                          
xz.x86_64                                    5.2.2-1.amzn2.0.2                      installed                          
xz-libs.x86_64                               5.2.2-1.amzn2.0.2                      installed                          
yajl.x86_64                                  2.0.4-4.amzn2.0.1                      installed                          
yum.noarch                                   3.4.3-158.amzn2.0.4                    @amzn2-core                        
yum-cron.noarch                              3.4.3-158.amzn2.0.4                    @amzn2-core                        
yum-langpacks.noarch                         0.4.2-7.amzn2                          installed                          
yum-metadata-parser.x86_64                   1.1.4-10.amzn2.0.2                     installed                          
yum-plugin-priorities.noarch                 1.1.31-46.amzn2.0.1                    installed                          
yum-utils.noarch                             1.1.31-46.amzn2.0.1                    installed                          
zip.x86_64                                   3.0-11.amzn2.0.2                       installed                          
zlib.x86_64                                  1.2.7-18.amzn2                         @amzn2-core                        
zlib-devel.x86_64                            1.2.7-18.amzn2                         @amzn2-core  
```
