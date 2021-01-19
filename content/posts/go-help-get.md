---
title: "go get コマンドのヘルプを読む"
date: 2021-01-16T12:10:06+09:00
toc: true
tags:
    - Go
draft: false
---

`go get` コマンドのヘルプ `go help get` を読んだメモ

<!--more-->

## Note

```
usage: go get [-d] [-t] [-u] [-v] [-insecure] [build flags] [packages]
```

---

依存関係の解決挙動あれこれ

- go get は、現在の開発モジュールを、依存関係の解決して追加し、ビルドとインストールを行う
- まず、追加するモジュールの依存関係を解決する
- それぞれのネームドパッケージやパッケージパターンに対して get はバージョンを指定する
    - デフォルトでは最新のタグ付きリリースバージョンとなる
        - `v0.4.5` や `v1.2.2.3` など
    - タグ付けされたリリースのように、最新のタグ付きプレリリース版も検索する
        - `v0.0.1-pre1` など
    - タグ付けされたバージョンが全く無い場合、 get は最新のコミットを見る
    - もし、モジュールが後のバージョンで、まだ required とされていない場合、 get は調べたバージョンを使用する
        - 最新のリリースよりも新しいプレリリースがあるとか
    - そうでなければ get は現在の必須バージョンを指定する
- デフォルトのバージョン選択は `go get golang.org/x/text@v0.3.0` のように `@version` サフィックスをパッケージ引数に追加することで上書きすることが出来る
    - バージョンにはプレフィックスを指定することが出来る。 `@v1` は `v1` で始まる最新の利用可能なバージョンを表す
    - クエリの完全な構文については `go help modules` を参照
- ソース管理リポジトリに保存されているモジュールの場合、バージョンサフィックスはコミットハッシュ、ブランチ識別子、ソース管理システムが知っている他構文 (例えば `go get golang.org/x/text@master`) のような形でも OK
    - 他モジュールのクエリ構文と重複する名前のブランチは、明示的に選択できないことに注意
        - 例えば、 `@v2` というサフィックスは `v2` で始まる最新のバージョンを意味し、 `v2` という名前のブランチを意味しない
- 検討中のモジュールが既に現在の開発モジュールの依存関係にある場合、 get は必要なバージョンを更新する
    - 現在の必須バージョンよりも前のバージョンを指定することは有効であり、依存関係をダウングレードする
    - バージョンサフィックス `@none` は、必要に応じて依存関係をダウングレードしたり、依存関係に依存するモジュールを削除したりして、依存関係を完全に削除すべきであることを示している
- バージョンサフィックス `@latest` は、指定されたパスで指定されたモジュールの最新マイナーリリースを明示的に要求する
    - `@upgrade` というサフィックスは `@latest` と似ているが、モジュールが最新のリリースバージョンよりも新しいリビジョンやプレリリースバージョンで既に required とされている場合は、モジュールをダウングレードしない
    - `@patch` というサフィックスは、最新のパッチリリースを要求する
        - 現在必要とされているバージョンと同じメジャーとマイナーのバージョン番号を持つ、最新のリリースされたバージョン
    - `@upgrade` と同様に `@patch` は既に required とされているモジュールを新しいバージョンにダウングレードすることは無い
    - パスが required されていない場合、 `@upgrade`, `@patch`, `@latest` は同等
- get のデフォルトでは、ネームドパッケージを含むモジュールの最新バージョンを使用するが、そのモジュールの依存関係の最新バージョンは使用しない
    - かわりに、そのモジュールが要求した特定の依存関係バージョンを使用する
        - 例えば、最新の A がモジュール B v1.2.3 を必要とし、一方で B v1.2.4 と v1.3.1 も利用可能である場合、 `go get A` は最新の A を使用するものの A が要求した通り B v1.2.3 を使用する
    - 特定のモジュールに対して競合する要求がある場合 `go get` は要求された最大のバージョンを使用することで、それらの要求を解決する

---

コマンド実行時のフラグあれこれ

- `-d` フラグ
    - 指定されたパッケージのビルドに必要なソースコードをダウンロードするが、ビルドやインストールは行わない
- `-t` フラグ
    - コマンドラインで指定されたパッケージのテストを構築するために必要なモジュールを考慮する
- `-u` フラグ
    - コマンドラインで指定されたパッケージの依存関係を提供するモジュールを更新する
    - 前の例の続きで、 `go get -u A` とした場合には B v1.3.1 の最新の A を使用する
    - B がモジュール C を必要としているが、 C が A のパッケージをビルドするのに必要なパッケージを提供していない場合、 C は更新されない
- `-u=patch` フラグ
    - `-u patch` では無い
    - 依存関係を更新する
    - デフォルトでパッチリリースを選択するように変更
    - `go get -u=patch A@latest` は B v1.2.4 の最新の A を使用するが `go get -u=patch A` は A のパッチリリースを代わりに使用する
- `-t` と `-u` を同時に使うと get はテストの依存関係も更新する
    - 一般的に、新しい依存関係を追加すると、ビルドを動作させるために既存の依存関係をアップグレードする必要があるかもしれないが、 `go get` はこれを自動的に行う
    - 同様に、ある依存関係をダウングレードすると、他の依存関係をダウングレードする必要があるかもしれない
- `-insecure` フラグ
    - HTTP のような安全ではないスキームを使って、リポジトリからの取得やカスタムドメインの解決を許可する
    - 注意して使用すること
    - 環境変数 `GOINSECURE` は、安全ではないスキームを使って、どのモジュールを取得するかを制御することが出来るので、一般的によりよい代替手段となる
    - 詳細は `go help environment` を参照

---

ネームドパッケージのダウンロード、ビルド、インストール

- 引数にモジュールの名前があってもパッケージでは無い場合、ビルドに失敗する代わりに、その引数のインストールステップはスキップされる
    - パッケージではない場合にはモジュールのルートディレクトリに Go コードが無いためビルドに失敗する
    - 例えば、 `go get golang.org/x/perf` は、インポートパスに対応するコードが無くても成功する
- パッケージパターンは許可されており、モジュールのバージョンを解決した後に展開されることに注意
    - 例えば、 `go get golang.org/x/perf/cmd...` は最新の golang.org/x/perf を追加し、その最新バージョンのコマンドをインストールする
- パッケージの引数が無い場合、 `go get` はカレントディレクトリに Go パッケージがある場合には、そのパッケージに適用される
    - 特に `go get -u` と `go get -u=patch` は、そのパッケージの全ての依存関係を更新する
    - パッケージ引数が無く、 `-u` も無い場合、 `go get` は `go install` 以上のものではなく、 `go get -d` は `go list` 以上のものでは無い
- モジュールの詳細については `go help modules`
- パッケージしていについては `go help packages`
- このテキストでは、ソースコードと依存関係を管理するためにモジュールを使用した get の動作について説明している
    - 代わりに go コマンドが `GOPATH` モードで実行されている場合、 get のフラグと効果の詳細は `go help get` と同様に変更される
    - 詳細については `go help modules` と `go help gopath-get` を参照
- `go build`, `go install`, `go clean`, `go mod` も参照しよう


## go help get

```bash
% go help get
usage: go get [-d] [-t] [-u] [-v] [-insecure] [build flags] [packages]

Get resolves and adds dependencies to the current development module and then builds and installs them.

The first step is to resolve which dependencies to add.

For each named package or package pattern, get must decide which version of the corresponding module to use.
By default, get looks up the latest tagged release version, such as v0.4.5 or v1.2.3. If there are no tagged release versions, get looks up the latest tagged pre-release version, such as v0.0.1-pre1. 
If there are no tagged versions at all, get looks up the latest known commit. 
If the module is not already required at a later version (for example, a pre-release newer than the latest release), get will use the version it looked up. 
Otherwise, get will use the currently required version.

This default version selection can be overridden by adding an @version suffix to the package argument, as in 'go get golang.org/x/text@v0.3.0'.
The version may be a prefix: @v1 denotes the latest available version starting with v1. 
See 'go help modules' under the heading 'Module queries' for the full query syntax.

For modules stored in source control repositories, the version suffix can also be a commit hash, branch identifier, or other syntax known to the source control system, as in 'go get golang.org/x/text@master'. 
Note that branches with names that overlap with other module query syntax cannot be selected explicitly. 
For example, the suffix @v2 means the latest version starting with v2, not the branch named v2.

If a module under consideration is already a dependency of the current development module, then get will update the required version.
Specifying a version earlier than the current required version is valid and downgrades the dependency. 
The version suffix @none indicates that the dependency should be removed entirely, downgrading or removing modules depending on it as needed.

The version suffix @latest explicitly requests the latest minor release of the module named by the given path. 
The suffix @upgrade is like @latest but will not downgrade a module if it is already required at a revision or pre-release version newer than the latest released version. 
The suffix @patch requests the latest patch release: the latest released version with the same major and minor version numbers as the currently required version. 
Like @upgrade, @patch will not downgrade a module already required at a newer version. 
If the path is not already required, @upgrade and @patch are equivalent to @latest.

Although get defaults to using the latest version of the module containing a named package, it does not use the latest version of that module's dependencies.
Instead it prefers to use the specific dependency versions requested by that module. 
For example, if the latest A requires module B v1.2.3, while B v1.2.4 and v1.3.1 are also available, then 'go get A' will use the latest A but then use B v1.2.3, as requested by A. 
(If there are competing requirements for a particular module, then 'go get' resolves those requirements by taking the maximum requested version.)

The -t flag instructs get to consider modules needed to build tests of packages specified on the command line.

The -u flag instructs get to update modules providing dependencies of packages named on the command line to use newer minor or patch releases when available. 
Continuing the previous example, 'go get -u A' will use the latest A with B v1.3.1 (not B v1.2.3). 
If B requires module C, but C does not provide any packages needed to build packages in A (not including tests), then C will not be updated.

The -u=patch flag (not -u patch) also instructs get to update dependencies, but changes the default to select patch releases.
Continuing the previous example, 'go get -u=patch A@latest' will use the latest A with B v1.2.4 (not B v1.2.3), while 'go get -u=patch A' will use a patch release of A instead.

When the -t and -u flags are used together, get will update test dependencies as well.

In general, adding a new dependency may require upgrading existing dependencies to keep a working build, and 'go get' does this automatically. 
Similarly, downgrading one dependency may require downgrading other dependencies, and 'go get' does this automatically as well.

The -insecure flag permits fetching from repositories and resolving custom domains using insecure schemes such as HTTP. 
Use with caution. 
The GOINSECURE environment variable is usually a better alternative, since it provides control over which modules may be retrieved using an insecure scheme. 
See 'go help environment' for details.

The second step is to download (if needed), build, and install the named packages.

If an argument names a module but not a package (because there is no Go source code in the module's root directory), then the install step is skipped for that argument, instead of causing a build failure. 
For example 'go get golang.org/x/perf' succeeds even though there is no code corresponding to that import path.

Note that package patterns are allowed and are expanded after resolving the module versions. 
For example, 'go get golang.org/x/perf/cmd/...' adds the latest golang.org/x/perf and then installs the commands in that latest version.

The -d flag instructs get to download the source code needed to build the named packages, including downloading necessary dependencies, but not to build and install them.

With no package arguments, 'go get' applies to Go package in the current directory, if any. In particular, 'go get -u' and 'go get -u=patch' update all the dependencies of that package.
With no package arguments and also without -u, 'go get' is not much more than 'go install', and 'go get -d' not much more than 'go list'.

For more about modules, see 'go help modules'.

For more about specifying packages, see 'go help packages'.

This text describes the behavior of get using modules to manage source code and dependencies. If instead the go command is running in GOPATH mode, the details of get's flags and effects change, as does 'go help get'.
See 'go help modules' and 'go help gopath-get'.

See also: go build, go install, go clean, go mod.
```


## この記事を試した環境

```bash
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% go version
go version go1.15.6 darwin/amd64
```
