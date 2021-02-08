---
title: "ファイルの圧縮や展開関連のコマンド"
date: 2021-02-08T23:56:05+09:00
draft: false
toc: true
images:
tags: 
  - zip
  - gzip
  - bzip2
  - xz
  - tar
---

- オプションとか関連コマンドが多く、とても覚えきれないのでメモ

<!--more-->


## zip

```bash
# ファイル圧縮
zip myzip.zip sample.txt

# ディレクトリ圧縮
zip -r myzip.zip /path/to/dir

# 展開
unzip myzip.zip
```

Manpage of ZIP  
http://linux.math.tifr.res.in/manuals/man/zip.html

zip(1)  
https://www.freebsd.org/cgi/man.cgi?query=zip&apropos=0&sektion=1


## UNIX Compress

```bash
# 圧縮
compress sample.txt

# 展開
uncompress sample.txt.Z
```

compress(1p) - Linux manual page  
https://man7.org/linux/man-pages/man1/compress.1p.html

compress(1)  
https://www.freebsd.org/cgi/man.cgi?query=compress&sektion=1


## gzip

```bash
# ファイル圧縮 (sample.txt が sample.txt.gz になる)
gzip sample.txt

# ディレクトリに含まれるファイルを圧縮 
# ./midir/ 配下のファイルが全て .gz となる
gzip -r ./mydir

# ファイル展開
gunzip sample.txt.gz

# ディレクトリに含まれるファイルを展開
# ./midir/ 配下の .gz ファイルが全て展開される
gunzip -r ./mydir

# gzip 圧縮されたファイルを展開せずに見る
gzcat sample.txt.gz # Linux では gzcat ではなく zcat を使う
zless sample.txt.gz
zmore sample.txt.gz
```

GNU Gzip  
https://www.gnu.org/software/gzip/manual/gzip.html

gzip  
https://www.freebsd.org/cgi/man.cgi?query=gzip&sektion=&n=1

gzcat(1)  
https://www.freebsd.org/cgi/man.cgi?query=gzcat&sektion=1&n=1


## bzip2

```bash
# ファイル圧縮 (sample.txt が sample.txt.bz2 になる)
bzip2 sample.txt

# ディレクトリに含まれるファイルを圧縮 
# -r オプションが無いので find コマンド等と組み合わせる必要がある
find ./mydir -type f -exec bzip2 {} +

# ファイル展開
bunzip2 sample.txt.bz2

# ディレクトリに含まれるファイルを展開
# -r オプションが無いので find コマンド等と組み合わせる必要がある
find ./mydir -type f -exec bunzip2 {} +

# bzip2 圧縮されたファイルを展開せずに見る
bzcat sample.txt.bz2
```

bzip2(1): block-sorting file compressor - Linux man page  
https://linux.die.net/man/1/bzip2

bzip2(1)  
https://www.freebsd.org/cgi/man.cgi?query=bzip2&sektion=1


## xz

```bash
# 圧縮
xz sample.txt

# 圧縮されたデータを展開せずに確認
xzcat sample.txt.xz

# 展開
unxz sample.txt.xz
```

xz(1): Compress/decompress .xz/.lzma files - Linux man page  
https://linux.die.net/man/1/xz

xz(1)  
https://www.freebsd.org/cgi/man.cgi?xz(1)


## tar

tar.Z

```bash
# ディレクトリを指定して tar にしてから UNIX Compress
tar cZvf my.tar.Z /path/to/dir

# 展開
tar xvf my.tar.Z
```

---

tar.gz

```bash
# ディレクトリを指定して tar にしてから gzip
tar czvf my.tar.gz /path/to/dir

# 展開
tar xvf my.tar.gz
```

---

tar.bz2

```bash
# ディレクトリを指定して tar にしてから bzip2
tar jcvf my.tar.bz2 /path/to/dir

# 展開
tar xvf my.tar.bz2
```

---

tar.xz

```bash
# ディレクトリ指定して tar にしてから xz
tar Jcvf my.tar.xz /path/to/dir

# 展開
tar xvf my.tar.xz
```

tar(1) - Linux manual page  
https://man7.org/linux/man-pages/man1/tar.1.html

tar(1)  
https://www.freebsd.org/cgi/man.cgi?tar(1)


## ファイル形式あれこれ

- zip
  - https://support.pkware.com/home/pkzip/developer-tools/appnote
  - https://www.iso.org/standard/60101.html
  - 一般的。 Windows でも使われる
- UNIX Compress
  - https://en.wikipedia.org/wiki/Compress
  - 昔よく使われていたもの
- gzip: GNU zip
  - https://www.gzip.org/
  - https://www.gnu.org/software/gzip/
  - 現代の主流
- bzip2
  - https://www.sourceware.org/bzip2/
  - gzip よりも圧縮効率が高いと言われている。一方で圧縮や展開に必要となるリソースが大きめ、かつ時間がかかる
- xz
  - https://tukaani.org/xz/format.html
  - bzip2 より更に圧縮効率が高いと言われているが、より CPU やメモリや時間を使用する
- tar
  - tar 自体は圧縮ではなく tape archives, tarball 等と呼ばれるファイルアーカイブフォーマット
  - 複数のファイルに対して圧縮を行うよりも、複数のファイルを tar として1つのファイルにアーカイブして圧縮した方がシーケンシャルなデータになるので、一般的に圧縮効率が良い
    - .tar.Z: tar したあとに UNIX Compress したもの。 .taz とも表記される
    - .tar.gz: tar したあとに gzip したもの。 .tgz とも表記される
    - .tar.bz2: tar したあとに bzip2 したもの。
    - .tar.xz: tar したあとに xz したもの。
  - ファイルが一部破損したら破損箇所以降のファイルを取り出すことができないというデメリットもある
- 圧縮効率が高いといっても、必ずしも他よりファイルサイズを小さくできる訳ではない
  - 圧縮対象のファイル内容にも依存する。テキストファイルは得意だけど、動画ファイルは苦手とか。
  - ちゃんと検討するならば具体的なサンプルファイルをいくつか用意して、それぞれの圧縮方式について検証した方が良い


## この記事を試した環境

```zsh
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
```