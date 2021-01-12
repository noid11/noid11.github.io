---
title: "Playwright を golang から使う"
date: 2021-01-12T14:51:07+09:00
toc: ture
draft: true
---

- [playwright](https://playwright.dev/) を Go で使える [playwright-go](https://github.com/mxschmitt/playwright-go) を動かしたメモ

<!--more-->

## プロジェクトのセットアップ

```bash
# directory setup
mkdir my-playwright-go && cd my-playwright-go

# go modules setup
go mod init my-playwright-go
go get github.com/mxschmitt/playwright-go

# コード書くファイル作成
touch main.go
```


## コードを書く

- https://news.ycombinator.com にアクセスして各記事のタイトルを出力するコード
- [公式サンプルがいくつかある](https://github.com/mxschmitt/playwright-go/tree/master/examples)ので、参考にすると良いかも

```go
package main

import (
	"fmt"
	"log"

	"github.com/mxschmitt/playwright-go"
)

func main() {
	pw, err := playwright.Run()
	if err != nil {
		log.Fatalf("could not start playwright: %v", err)
	}
	browser, err := pw.Chromium.Launch()
	if err != nil {
		log.Fatalf("could not launch browser: %v", err)
	}
	page, err := browser.NewPage()
	if err != nil {
		log.Fatalf("could not create page: %v", err)
	}
	if _, err = page.Goto("https://news.ycombinator.com"); err != nil {
		log.Fatalf("could not goto: %v", err)
	}
	entries, err := page.QuerySelectorAll(".athing")
	if err != nil {
		log.Fatalf("could not get entries: %v", err)
	}
	for i, entry := range entries {
		titleElement, err := entry.QuerySelector("td.title > a")
		if err != nil {
			log.Fatalf("could not get title element: %v", err)
		}
		title, err := titleElement.TextContent()
		if err != nil {
			log.Fatalf("could not get text content: %v", err)
		}
		fmt.Printf("%d: %s\n", i+1, title)
	}
	if err = browser.Close(); err != nil {
		log.Fatalf("could not close browser: %v", err)
	}
	if err = pw.Stop(); err != nil {
		log.Fatalf("could not stop Playwright: %v", err)
	}
}
```


## 動かす

```bash
go run main.go
```


## 実行例

```bash
% go run main.go
2021/01/12 13:12:19 Downloading driver to /Users/myusername/Library/Caches/ms-playwright-go/0.171.0
2021/01/12 13:12:34 Downloaded driver successfully
2021/01/12 13:12:34 Downloading browsers...
2021/01/12 13:12:36 Downloaded browsers successfully
1: Theseus: A modern experimental OS written from scratch in Rust
2: Ubiquiti Networks Breach
3: Moderna is announcing three new vaccine programs. Seasonal flu, HIV, and Nipah
4: My ISP Is Killing My Idle SSH Sessions. Yours Might Be Too
5: Two Worlds: So Much Prosperity, So Much Skepticism
6: Simple Anomaly Detection Using Plain SQL
7: Steam's login method is kinda interesting
8: Europe Is Guaranteeing Citizens the Right to Repair
9: What Is a Gamma Squeeze in the Context of Stock Trading?
10: My personal wishlist for a decentralized social network
11: Show HN: Symmetrical Drawing Tool for Doodling
12: Sirum (YC W15 Nonprofit) hiring engineers to improve healthcare for families
13: A bit on scaling chess.com's database
14: Intruder at the top of the 20 meter amateur band?
15: Superintelligence cannot be contained: Lessons from Computability Theory
16: Stealing Your Private YouTube Videos, One Frame at a Time
17: Sinovac's vaccine efficacy less than 60% in Brazil trial -report
18: Getting Ready for Launch
19: Are Experts Real?
20: Google Sued by YouTube Rival Rumble over Search Rankings
21: OpenSocial Specification
22: Is the Schrödinger Equation True?
23: A 432-year-old manual on social distancing
24: iPhone 7 with dead NAND netbooting unmodified Ubuntu 20.04 via USB ethernet
25: We need a new media system
26: XTerm: It's Better Than You Thought
27: Invisible Internet Project (I2P)
28: A Brief History of Consumer Culture
29: How to Hear the ISS
30: Show HN: Thumbnail.ai – Just paste a blog post link and get a thumbnail
```


## この記事を試した環境

```zsh
% sw_vers
ProductName:    Mac OS X
ProductVersion: 10.15.7
BuildVersion:   19H114
% go version
go version go1.15.6 darwin/amd64
```
