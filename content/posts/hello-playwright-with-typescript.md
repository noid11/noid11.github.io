---
title: "Playwright を TypeScript で動かすまで"
date: 2021-01-12T12:39:32+09:00
toc: true
draft: false
---

- [Playwright](https://playwright.dev/) は Microsoft 製のブラウザ自動化ツール
    - [Chromium](https://www.chromium.org/), [Firefox](https://www.mozilla.org/en-US/firefox/), [WebKit](https://webkit.org/) に対応している
    - [公式 Docker イメージ](https://playwright.dev/docs/docker/README/)も提供されている
- 組み込みで TypeScript をサポートしているとあったので、最低限動かす所まで試したメモ


<!--more-->

## プロジェクトのセットアップ

```zsh
mkdir my-playwright-project && cd my-playwright-project

# npm 初期化 
npm init --yes

# playwright のインストール
#     --save-dev いらないような気がする・・・
#     chromium, firefox, webkit がダウンロードされ、 300 MB 程度のトラフィックが発生するので注意
npm install --save-dev playwright

# typescript セットアップ
npm install --save-dev typescript @types/node@12 ts-node
npx tsc --init

# コードを書いていく
touch main.ts
```


## ミニマムなコード実装

```ts
import { webkit } from 'playwright'

(async () => {
  const browser = await webkit.launch()
  const page = await browser.newPage()
  await page.goto('http://whatsmyuseragent.org/')
  await page.screenshot({ path: `example.png` })
  await browser.close()
})()
```


## コード実行

```zsh
npx ts-node main.ts
```

- 同じディレクトリに http://whatsmyuseragent.org/ にアクセスしたスクリーンショットが example.png として保存されれば OK
- 上記コードの場合 User-Agent が WebKit になる


## Useful Links

CLI Commands | npm Docs  
https://docs.npmjs.com/cli/v6/commands

Getting Started | Playwright  
https://playwright.dev/docs/intro

TypeScript: The starting point for learning TypeScript  
https://www.typescriptlang.org/docs


## この記事を試した環境

```zsh
% node -v
v12.16.1
% npm -v
6.14.8
```

package.json

```json
{
  "name": "my-playwright-project",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "dependencies": {
    "playwright": "^1.7.1"
  },
  "devDependencies": {
    "@types/node": "^12.19.12",
    "ts-node": "^9.1.1",
    "typescript": "^4.1.3"
  }
}
```

package-lock.json

```json
{
  "name": "my-playwright-project",
  "version": "1.0.0",
  "lockfileVersion": 1,
  "requires": true,
  "dependencies": {
    "@types/node": {
      "version": "12.19.12",
      "resolved": "https://registry.npmjs.org/@types/node/-/node-12.19.12.tgz",
      "integrity": "sha512-UwfL2uIU9arX/+/PRcIkT08/iBadGN2z6ExOROA2Dh5mAuWTBj6iJbQX4nekiV5H8cTrEG569LeX+HRco9Cbxw=="
    },
    "@types/yauzl": {
      "version": "2.9.1",
      "resolved": "https://registry.npmjs.org/@types/yauzl/-/yauzl-2.9.1.tgz",
      "integrity": "sha512-A1b8SU4D10uoPjwb0lnHmmu8wZhR9d+9o2PKBQT2jU5YPTKsxac6M2qGAdY7VcL+dHHhARVUDmeg0rOrcd9EjA==",
      "optional": true,
      "requires": {
        "@types/node": "*"
      }
    },
    "agent-base": {
      "version": "6.0.2",
      "resolved": "https://registry.npmjs.org/agent-base/-/agent-base-6.0.2.tgz",
      "integrity": "sha512-RZNwNclF7+MS/8bDg70amg32dyeZGZxiDuQmZxKLAlQjr3jGyLx+4Kkk58UO7D2QdgFIQCovuSuZESne6RG6XQ==",
      "requires": {
        "debug": "4"
      }
    },
    "arg": {
      "version": "4.1.3",
      "resolved": "https://registry.npmjs.org/arg/-/arg-4.1.3.tgz",
      "integrity": "sha512-58S9QDqG0Xx27YwPSt9fJxivjYl432YCwfDMfZ+71RAqUrZef7LrKQZ3LHLOwCS4FLNBplP533Zx895SeOCHvA==",
      "dev": true
    },
    "balanced-match": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/balanced-match/-/balanced-match-1.0.0.tgz",
      "integrity": "sha1-ibTRmasr7kneFk6gK4nORi1xt2c="
    },
    "brace-expansion": {
      "version": "1.1.11",
      "resolved": "https://registry.npmjs.org/brace-expansion/-/brace-expansion-1.1.11.tgz",
      "integrity": "sha512-iCuPHDFgrHX7H2vEI/5xpz07zSHB00TpugqhmYtVmMO6518mCuRMoOYFldEBl0g187ufozdaHgWKcYFb61qGiA==",
      "requires": {
        "balanced-match": "^1.0.0",
        "concat-map": "0.0.1"
      }
    },
    "buffer-crc32": {
      "version": "0.2.13",
      "resolved": "https://registry.npmjs.org/buffer-crc32/-/buffer-crc32-0.2.13.tgz",
      "integrity": "sha1-DTM+PwDqxQqhRUq9MO+MKl2ackI="
    },
    "buffer-from": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/buffer-from/-/buffer-from-1.1.1.tgz",
      "integrity": "sha512-MQcXEUbCKtEo7bhqEs6560Hyd4XaovZlO/k9V3hjVUF/zwW7KBVdSK4gIt/bzwS9MbR5qob+F5jusZsb0YQK2A==",
      "dev": true
    },
    "concat-map": {
      "version": "0.0.1",
      "resolved": "https://registry.npmjs.org/concat-map/-/concat-map-0.0.1.tgz",
      "integrity": "sha1-2Klr13/Wjfd5OnMDajug1UBdR3s="
    },
    "create-require": {
      "version": "1.1.1",
      "resolved": "https://registry.npmjs.org/create-require/-/create-require-1.1.1.tgz",
      "integrity": "sha512-dcKFX3jn0MpIaXjisoRvexIJVEKzaq7z2rZKxf+MSr9TkdmHmsU4m2lcLojrj/FHl8mk5VxMmYA+ftRkP/3oKQ==",
      "dev": true
    },
    "debug": {
      "version": "4.3.1",
      "resolved": "https://registry.npmjs.org/debug/-/debug-4.3.1.tgz",
      "integrity": "sha512-doEwdvm4PCeK4K3RQN2ZC2BYUBaxwLARCqZmMjtF8a51J2Rb0xpVloFRnCODwqjpwnAoao4pelN8l3RJdv3gRQ==",
      "requires": {
        "ms": "2.1.2"
      }
    },
    "diff": {
      "version": "4.0.2",
      "resolved": "https://registry.npmjs.org/diff/-/diff-4.0.2.tgz",
      "integrity": "sha512-58lmxKSA4BNyLz+HHMUzlOEpg09FV+ev6ZMe3vJihgdxzgcwZ8VoEEPmALCZG9LmqfVoNMMKpttIYTVG6uDY7A==",
      "dev": true
    },
    "end-of-stream": {
      "version": "1.4.4",
      "resolved": "https://registry.npmjs.org/end-of-stream/-/end-of-stream-1.4.4.tgz",
      "integrity": "sha512-+uw1inIHVPQoaVuHzRyXd21icM+cnt4CzD5rW+NC1wjOUSTOs+Te7FOv7AhN7vS9x/oIyhLP5PR1H+phQAHu5Q==",
      "requires": {
        "once": "^1.4.0"
      }
    },
    "extract-zip": {
      "version": "2.0.1",
      "resolved": "https://registry.npmjs.org/extract-zip/-/extract-zip-2.0.1.tgz",
      "integrity": "sha512-GDhU9ntwuKyGXdZBUgTIe+vXnWj0fppUEtMDL0+idd5Sta8TGpHssn/eusA9mrPr9qNDym6SxAYZjNvCn/9RBg==",
      "requires": {
        "@types/yauzl": "^2.9.1",
        "debug": "^4.1.1",
        "get-stream": "^5.1.0",
        "yauzl": "^2.10.0"
      }
    },
    "fd-slicer": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/fd-slicer/-/fd-slicer-1.1.0.tgz",
      "integrity": "sha1-JcfInLH5B3+IkbvmHY85Dq4lbx4=",
      "requires": {
        "pend": "~1.2.0"
      }
    },
    "fs.realpath": {
      "version": "1.0.0",
      "resolved": "https://registry.npmjs.org/fs.realpath/-/fs.realpath-1.0.0.tgz",
      "integrity": "sha1-FQStJSMVjKpA20onh8sBQRmU6k8="
    },
    "get-stream": {
      "version": "5.2.0",
      "resolved": "https://registry.npmjs.org/get-stream/-/get-stream-5.2.0.tgz",
      "integrity": "sha512-nBF+F1rAZVCu/p7rjzgA+Yb4lfYXrpl7a6VmJrU8wF9I1CKvP/QwPNZHnOlwbTkY6dvtFIzFMSyQXbLoTQPRpA==",
      "requires": {
        "pump": "^3.0.0"
      }
    },
    "glob": {
      "version": "7.1.6",
      "resolved": "https://registry.npmjs.org/glob/-/glob-7.1.6.tgz",
      "integrity": "sha512-LwaxwyZ72Lk7vZINtNNrywX0ZuLyStrdDtabefZKAY5ZGJhVtgdznluResxNmPitE0SAO+O26sWTHeKSI2wMBA==",
      "requires": {
        "fs.realpath": "^1.0.0",
        "inflight": "^1.0.4",
        "inherits": "2",
        "minimatch": "^3.0.4",
        "once": "^1.3.0",
        "path-is-absolute": "^1.0.0"
      }
    },
    "graceful-fs": {
      "version": "4.2.4",
      "resolved": "https://registry.npmjs.org/graceful-fs/-/graceful-fs-4.2.4.tgz",
      "integrity": "sha512-WjKPNJF79dtJAVniUlGGWHYGz2jWxT6VhN/4m1NdkbZ2nOsEF+cI1Edgql5zCRhs/VsQYRvrXctxktVXZUkixw=="
    },
    "https-proxy-agent": {
      "version": "5.0.0",
      "resolved": "https://registry.npmjs.org/https-proxy-agent/-/https-proxy-agent-5.0.0.tgz",
      "integrity": "sha512-EkYm5BcKUGiduxzSt3Eppko+PiNWNEpa4ySk9vTC6wDsQJW9rHSa+UhGNJoRYp7bz6Ht1eaRIa6QaJqO5rCFbA==",
      "requires": {
        "agent-base": "6",
        "debug": "4"
      }
    },
    "inflight": {
      "version": "1.0.6",
      "resolved": "https://registry.npmjs.org/inflight/-/inflight-1.0.6.tgz",
      "integrity": "sha1-Sb1jMdfQLQwJvJEKEHW6gWW1bfk=",
      "requires": {
        "once": "^1.3.0",
        "wrappy": "1"
      }
    },
    "inherits": {
      "version": "2.0.4",
      "resolved": "https://registry.npmjs.org/inherits/-/inherits-2.0.4.tgz",
      "integrity": "sha512-k/vGaX4/Yla3WzyMCvTQOXYeIHvqOKtnqBduzTHpzpQZzAskKMhZ2K+EnBiSM9zGSoIFeMpXKxa4dYeZIQqewQ=="
    },
    "jpeg-js": {
      "version": "0.4.3",
      "resolved": "https://registry.npmjs.org/jpeg-js/-/jpeg-js-0.4.3.tgz",
      "integrity": "sha512-ru1HWKek8octvUHFHvE5ZzQ1yAsJmIvRdGWvSoKV52XKyuyYA437QWDttXT8eZXDSbuMpHlLzPDZUPd6idIz+Q=="
    },
    "make-error": {
      "version": "1.3.6",
      "resolved": "https://registry.npmjs.org/make-error/-/make-error-1.3.6.tgz",
      "integrity": "sha512-s8UhlNe7vPKomQhC1qFelMokr/Sc3AgNbso3n74mVPA5LTZwkB9NlXf4XPamLxJE8h0gh73rM94xvwRT2CVInw==",
      "dev": true
    },
    "mime": {
      "version": "2.4.7",
      "resolved": "https://registry.npmjs.org/mime/-/mime-2.4.7.tgz",
      "integrity": "sha512-dhNd1uA2u397uQk3Nv5LM4lm93WYDUXFn3Fu291FJerns4jyTudqhIWe4W04YLy7Uk1tm1Ore04NpjRvQp/NPA=="
    },
    "minimatch": {
      "version": "3.0.4",
      "resolved": "https://registry.npmjs.org/minimatch/-/minimatch-3.0.4.tgz",
      "integrity": "sha512-yJHVQEhyqPLUTgt9B83PXu6W3rx4MvvHvSUvToogpwoGDOUQ+yDrR0HRot+yOCdCO7u4hX3pWft6kWBBcqh0UA==",
      "requires": {
        "brace-expansion": "^1.1.7"
      }
    },
    "ms": {
      "version": "2.1.2",
      "resolved": "https://registry.npmjs.org/ms/-/ms-2.1.2.tgz",
      "integrity": "sha512-sGkPx+VjMtmA6MX27oA4FBFELFCZZ4S4XqeGOXCv68tT+jb3vk/RyaKWP0PTKyWtmLSM0b+adUTEvbs1PEaH2w=="
    },
    "once": {
      "version": "1.4.0",
      "resolved": "https://registry.npmjs.org/once/-/once-1.4.0.tgz",
      "integrity": "sha1-WDsap3WWHUsROsF9nFC6753Xa9E=",
      "requires": {
        "wrappy": "1"
      }
    },
    "path-is-absolute": {
      "version": "1.0.1",
      "resolved": "https://registry.npmjs.org/path-is-absolute/-/path-is-absolute-1.0.1.tgz",
      "integrity": "sha1-F0uSaHNVNP+8es5r9TpanhtcX18="
    },
    "pend": {
      "version": "1.2.0",
      "resolved": "https://registry.npmjs.org/pend/-/pend-1.2.0.tgz",
      "integrity": "sha1-elfrVQpng/kRUzH89GY9XI4AelA="
    },
    "playwright": {
      "version": "1.7.1",
      "resolved": "https://registry.npmjs.org/playwright/-/playwright-1.7.1.tgz",
      "integrity": "sha512-dOSWME42wDedJ/PXv8k0zG0Kxd6d6R2OKA51/05++Z2ISdA4N58gHlWqlVKPDkBog1MI6lu/KNt7QDn19AybWQ==",
      "requires": {
        "debug": "^4.1.1",
        "extract-zip": "^2.0.1",
        "https-proxy-agent": "^5.0.0",
        "jpeg-js": "^0.4.2",
        "mime": "^2.4.6",
        "pngjs": "^5.0.0",
        "progress": "^2.0.3",
        "proper-lockfile": "^4.1.1",
        "proxy-from-env": "^1.1.0",
        "rimraf": "^3.0.2",
        "ws": "^7.3.1"
      }
    },
    "pngjs": {
      "version": "5.0.0",
      "resolved": "https://registry.npmjs.org/pngjs/-/pngjs-5.0.0.tgz",
      "integrity": "sha512-40QW5YalBNfQo5yRYmiw7Yz6TKKVr3h6970B2YE+3fQpsWcrbj1PzJgxeJ19DRQjhMbKPIuMY8rFaXc8moolVw=="
    },
    "progress": {
      "version": "2.0.3",
      "resolved": "https://registry.npmjs.org/progress/-/progress-2.0.3.tgz",
      "integrity": "sha512-7PiHtLll5LdnKIMw100I+8xJXR5gW2QwWYkT6iJva0bXitZKa/XMrSbdmg3r2Xnaidz9Qumd0VPaMrZlF9V9sA=="
    },
    "proper-lockfile": {
      "version": "4.1.1",
      "resolved": "https://registry.npmjs.org/proper-lockfile/-/proper-lockfile-4.1.1.tgz",
      "integrity": "sha512-1w6rxXodisVpn7QYvLk706mzprPTAPCYAqxMvctmPN3ekuRk/kuGkGc82pangZiAt4R3lwSuUzheTTn0/Yb7Zg==",
      "requires": {
        "graceful-fs": "^4.1.11",
        "retry": "^0.12.0",
        "signal-exit": "^3.0.2"
      }
    },
    "proxy-from-env": {
      "version": "1.1.0",
      "resolved": "https://registry.npmjs.org/proxy-from-env/-/proxy-from-env-1.1.0.tgz",
      "integrity": "sha512-D+zkORCbA9f1tdWRK0RaCR3GPv50cMxcrz4X8k5LTSUD1Dkw47mKJEZQNunItRTkWwgtaUSo1RVFRIG9ZXiFYg=="
    },
    "pump": {
      "version": "3.0.0",
      "resolved": "https://registry.npmjs.org/pump/-/pump-3.0.0.tgz",
      "integrity": "sha512-LwZy+p3SFs1Pytd/jYct4wpv49HiYCqd9Rlc5ZVdk0V+8Yzv6jR5Blk3TRmPL1ft69TxP0IMZGJ+WPFU2BFhww==",
      "requires": {
        "end-of-stream": "^1.1.0",
        "once": "^1.3.1"
      }
    },
    "retry": {
      "version": "0.12.0",
      "resolved": "https://registry.npmjs.org/retry/-/retry-0.12.0.tgz",
      "integrity": "sha1-G0KmJmoh8HQh0bC1S33BZ7AcATs="
    },
    "rimraf": {
      "version": "3.0.2",
      "resolved": "https://registry.npmjs.org/rimraf/-/rimraf-3.0.2.tgz",
      "integrity": "sha512-JZkJMZkAGFFPP2YqXZXPbMlMBgsxzE8ILs4lMIX/2o0L9UBw9O/Y3o6wFw/i9YLapcUJWwqbi3kdxIPdC62TIA==",
      "requires": {
        "glob": "^7.1.3"
      }
    },
    "signal-exit": {
      "version": "3.0.3",
      "resolved": "https://registry.npmjs.org/signal-exit/-/signal-exit-3.0.3.tgz",
      "integrity": "sha512-VUJ49FC8U1OxwZLxIbTTrDvLnf/6TDgxZcK8wxR8zs13xpx7xbG60ndBlhNrFi2EMuFRoeDoJO7wthSLq42EjA=="
    },
    "source-map": {
      "version": "0.6.1",
      "resolved": "https://registry.npmjs.org/source-map/-/source-map-0.6.1.tgz",
      "integrity": "sha512-UjgapumWlbMhkBgzT7Ykc5YXUT46F0iKu8SGXq0bcwP5dz/h0Plj6enJqjz1Zbq2l5WaqYnrVbwWOWMyF3F47g==",
      "dev": true
    },
    "source-map-support": {
      "version": "0.5.19",
      "resolved": "https://registry.npmjs.org/source-map-support/-/source-map-support-0.5.19.tgz",
      "integrity": "sha512-Wonm7zOCIJzBGQdB+thsPar0kYuCIzYvxZwlBa87yi/Mdjv7Tip2cyVbLj5o0cFPN4EVkuTwb3GDDyUx2DGnGw==",
      "dev": true,
      "requires": {
        "buffer-from": "^1.0.0",
        "source-map": "^0.6.0"
      }
    },
    "ts-node": {
      "version": "9.1.1",
      "resolved": "https://registry.npmjs.org/ts-node/-/ts-node-9.1.1.tgz",
      "integrity": "sha512-hPlt7ZACERQGf03M253ytLY3dHbGNGrAq9qIHWUY9XHYl1z7wYngSr3OQ5xmui8o2AaxsONxIzjafLUiWBo1Fg==",
      "dev": true,
      "requires": {
        "arg": "^4.1.0",
        "create-require": "^1.1.0",
        "diff": "^4.0.1",
        "make-error": "^1.1.1",
        "source-map-support": "^0.5.17",
        "yn": "3.1.1"
      }
    },
    "typescript": {
      "version": "4.1.3",
      "resolved": "https://registry.npmjs.org/typescript/-/typescript-4.1.3.tgz",
      "integrity": "sha512-B3ZIOf1IKeH2ixgHhj6la6xdwR9QrLC5d1VKeCSY4tvkqhF2eqd9O7txNlS0PO3GrBAFIdr3L1ndNwteUbZLYg==",
      "dev": true
    },
    "wrappy": {
      "version": "1.0.2",
      "resolved": "https://registry.npmjs.org/wrappy/-/wrappy-1.0.2.tgz",
      "integrity": "sha1-tSQ9jz7BqjXxNkYFvA0QNuMKtp8="
    },
    "ws": {
      "version": "7.4.2",
      "resolved": "https://registry.npmjs.org/ws/-/ws-7.4.2.tgz",
      "integrity": "sha512-T4tewALS3+qsrpGI/8dqNMLIVdq/g/85U98HPMa6F0m6xTbvhXU6RCQLqPH3+SlomNV/LdY6RXEbBpMH6EOJnA=="
    },
    "yauzl": {
      "version": "2.10.0",
      "resolved": "https://registry.npmjs.org/yauzl/-/yauzl-2.10.0.tgz",
      "integrity": "sha1-x+sXyT4RLLEIb6bY5R+wZnt5pfk=",
      "requires": {
        "buffer-crc32": "~0.2.3",
        "fd-slicer": "~1.1.0"
      }
    },
    "yn": {
      "version": "3.1.1",
      "resolved": "https://registry.npmjs.org/yn/-/yn-3.1.1.tgz",
      "integrity": "sha512-Ux4ygGWsu2c7isFWe8Yu1YluJmqVhxqK2cLXNQA5AcC3QfbGNpM7fu0Y8b/z16pXLnFxZYvWhd3fhBY9DLmC6Q==",
      "dev": true
    }
  }
}
```