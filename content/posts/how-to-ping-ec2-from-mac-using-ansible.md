---
title: "Mac に Ansible をインストールして EC2 に ping するまで"
date: 2021-02-11T17:05:07+09:00
draft: true
toc: true
images:
tags: 
  - Mac
  - Ansible
---

<!--more-->

## Mac に Ansible をインストール

```bash
brew install ansible
```

ansible — Homebrew Formulae  
https://formulae.brew.sh/formula/ansible


## EC2 Linux 環境を構築

- EC2 で Linux 環境構築する手順は省略
- EC2 インスタンスに ssh 接続できるようにしておく


## /etc/ansible/hosts に EC2 インスタンスを登録

```bash
% cat /etc/ansible/hosts 
ec2-xxx-xxx-xxx-xxx.ap-northeast-1.compute.amazonaws.com
```

---

`/etc/ansible/hosts` って何？

- Ansible では、インベントリと呼ばれる単位で、複数のホストに対して操作を行う
  - インベントリを定義したらパターンを使用して Ansible を実行したいホストやグループを指定する
  - このインベントリのデフォルトの場所が `/etc/ansible/hosts` となる。
- コマンドラインで `-i` or `--inventory` or `--inventory-file` オプションを使用して、任意のインベントリファイルを指定することも可能

How to build your inventory — Ansible Documentation  
https://docs.ansible.com/ansible/latest/user_guide/intro_inventory.html


## ping

```bash
% ansible --user ec2-user all --module-name ping
[WARNING]: Platform linux on host ec2-xxx-xxx-xxx-xxx.ap-northeast-1.compute.amazonaws.com is using the discovered Python interpreter at /usr/bin/python, but future
installation of another Python interpreter could change the meaning of that path. See https://docs.ansible.com/ansible/2.10/reference_appendices/interpreter_discovery.html
for more information.
ec2-xxx-xxx-xxx-xxx.ap-northeast-1.compute.amazonaws.com | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

---

この ping って何？

- Ansible の ping モジュール
  - リモートノードに ssh による通信が可能か、リモートノードで Python が動かくか確認するもの
  - ICMP での ping ではない

ansible.builtin.ping – Try to connect to host, verify a usable python and return pong on success — Ansible Documentation  
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/ping_module.html#ansible-collections-ansible-builtin-ping-module

Ansible.Builtin — Ansible Documentation  
https://docs.ansible.com/ansible/latest/collections/ansible/builtin/

---

この警告は何なのか

- POSIX 環境で実行されるほとんどの Ansible モジュールは、ターゲットホスト上で Python インタプリタを必要としまする
- 特に設定しない限り Ansible は、 Python モジュールがそのホストで初めて実行されたときに、各ターゲットホスト上で適切な Python インタプリタを検出しようとする
  - この動作を Interpreter Discovery という
  - Interpreter Discovery を防ぐには2つの方法がある
    1. 個々のホストやグループに対して ansible_python_interpreter インベントリ変数を設定する
    2. グローバルに ansible.cfg の [defaults] セクションで interpreter_python キーを使用する
  - グローバルに設定してしまったほうが楽なので、今回は 2. の方法で対処する

[小ネタ] Pythonのバージョンを指定してAnsible実行時に表示される警告を消す | DevelopersIO  
https://dev.classmethod.jp/articles/ansible-interpreter-warning/

Interpreter Discovery — Ansible Documentation  
https://docs.ansible.com/ansible/2.9/reference_appendices/interpreter_discovery.html


## 警告を消すために ~/.ansible.cfg を作成

```bash
% cat ~/.ansible.cfg
[defaults]
interpreter_python=/usr/bin/python
```

---

`~/.ansible.cfg` って何？

- Ansible の設定ファイル

Controlling how Ansible behaves: precedence rules — Ansible Documentation  
https://docs.ansible.com/ansible/latest/reference_appendices/general_precedence.html


## 再度 ping

```bash
% ansible --user ec2-user all --module-name ping
ec2-xxx-xxx-xxx-xxx.ap-northeast-1.compute.amazonaws.com | SUCCESS => {
    "changed": false,
    "ping": "pong"
}
```

- 警告が消えたのでヨシ！


## この記事を試した環境

```bash
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H512
% ansible --version
ansible 2.10.5
  config file = /Users/myname/.ansible.cfg
  configured module search path = ['/Users/myname/.ansible/plugins/modules', '/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/local/Cellar/ansible/2.10.6/libexec/lib/python3.9/site-packages/ansible
  executable location = /usr/local/bin/ansible
  python version = 3.9.1 (default, Feb  3 2021, 07:04:15) [Clang 12.0.0 (clang-1200.0.32.29)]
```