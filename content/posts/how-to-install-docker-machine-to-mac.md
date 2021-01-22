---
title: "Mac に Docker Machine をインストールする方法"
date: 2021-01-22T09:30:07+09:00
draft: false
toc: false
images:
tags: 
  - Docker
  - Mac
---

ググっても公式情報がヒットしづらかったのでメモ

<!--more-->

- [Docker Machine](https://docs.docker.com/machine/) は [Docker Desktop on Mac](https://docs.docker.com/docker-for-mac/install/) に含まれなくなったので別途個別にインストールが必要
- インストール方法は[公式ドキュメント](https://docs.docker.com/machine/install-machine/)に全て書いてある

```bash
# インストール
base=https://github.com/docker/machine/releases/download/v0.16.0 && curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/usr/local/bin/docker-machine && chmod +x /usr/local/bin/docker-machine

# インストールできたか確認
docker-machine version
```

---

この記事を試した環境

```zsh
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% docker-machine version
docker-machine version 0.16.0, build 702c267f
```