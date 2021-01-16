---
title: "UUIDv4 を Go で生成する"
date: 2021-01-16T11:31:50+09:00
toc: true
draft: false
---

<!--more-->

## セットアップ

```bash
go mod init generate-uuidv4-by-golang
go get github.com/google/uuid
```


## 実装例

main.go

```go
package main

import (
	"fmt"

	"github.com/google/uuid"
)

func main() {
	uuidv4, err := uuid.NewRandom()
	if err != nil {
		fmt.Println(err)
		return
	}
	fmt.Println(uuidv4.String())
}
```


## 実行例

```bash
% go run main.go
3c85b7b7-0b11-4612-9b1d-1dab37e20c9d
```


## リファレンス

uuid · pkg.go.dev  
https://pkg.go.dev/github.com/google/uuid#NewRandom

google/uuid: Go package for UUIDs based on RFC 4122 and DCE 1.1: Authentication and Security Services.  
https://github.com/google/uuid

RFC 4122 - A Universally Unique IDentifier (UUID) URN Namespace  
https://tools.ietf.org/html/rfc4122

RFC 4122 - A Universally Unique IDentifier (UUID) URN Namespace 日本語訳  
https://tex2e.github.io/rfc-translater/html/rfc4122.html


## この記事を試した環境

```bash
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% go version
go version go1.15.6 darwin/amd64
```

go.mod

```go
module golang-uuid

go 1.15

require github.com/google/uuid v1.1.5 // indirect
```

go.sum

```go
github.com/google/uuid v1.1.5 h1:kxhtnfFVi+rYdOALN0B3k9UT86zVJKfBimRaciULW4I=
github.com/google/uuid v1.1.5/go.mod h1:TIyPZe4MgqvfeYDBFedMoGGpEw/LqOeaOT+nhxU+yHo=
```
