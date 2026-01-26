---
layout: post
title: Tron 살펴보기 - TronBox 시작하기
date: 2019-01-30
description: Tron 살펴보기 - TronBox 시작하기
categories: [Blockchain]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Tron-살펴보기-TronBox-시작하기/
author: hwangrolee
tags: []
---

터미널에서 TronBox와 TronGrid을 빠르게 세팅하고 실행할 수 있습니다. TronBox와 TronGrid 이 2개의 툴로 MetaCoin App을 실행하고 Smart Contract 컴파일, Contract 배포, 마지막으로 Shasta 테스트넷에서 이벤트 쿼리를 실행할 수있습니다.

> 원래는 튜토리얼 영상이 있는데 게시자가 지운듯 싶습니다.

설치

- Nodejs 5.0+
- Windows, Linux, or Mac OS X

```bash
# tronbox를 글로벌로 설치합니다.
$ npm install -g tronbox
...
```

```bash
# tronbox는 v2.3.11입니다.
# solidity는 v0.4.24입니다.
$ tronbox version
Tronbox v2.3.11 (core: 4.1.13)
Solidity v0.4.24 (tron-solc)
```

### Initialize a Tron-Box Project

> Tronbox project 초기화

```bash
# tronbox 프로젝트를 초기화합니다.
$ tronbox init
Downloading...
Unpacking...
Setting up...
Unbox successful. Sweet!

Commands:

  Compile:        tronbox compile
  Migrate:        tronbox migrate
  Test contracts: tronbox test

$ ls
contracts         migrations        test              tronbox-config.js tronbox.js
```

| contract           | migrations                           | test                                               | tronbox.js                                                                     |
| ------------------ | ------------------------------------ | -------------------------------------------------- | ------------------------------------------------------------------------------ |
| 스마트컨트렉트파일 | 마이그레이션을 위한 자바스크립트파일 | 스마트컨트렉트를 테스트하기 위한 자바스크립트 파일 | 프로젝트의 설정파일로 fullnode address, 서버의 event server 정보가 존재합니다. |

### Basic Commands

> 기본 명령어

| command                         | usage                                                                                                     |
| ------------------------------- | --------------------------------------------------------------------------------------------------------- |
| tronbox compile                 | 모든 스마트 컨트렉트를 컴파일하며 컴파일된 결과는 **./build/contracts**에 저장됩니다.                     |
| tronbox compile --compile-all   | 다시 모든 스마트 컨트렉트를 컴파일 합니다.                                                                |
| tronbox migrate                 | 마지막 마이그레션이후의 변경된 컨트렉트를 배포합니다.                                                     |
| tronbox migrate --reset         | 모든 마이그레이션을 다시 진행합니다.                                                                      |
| tronbox test [test_script_path] | 모든 테스트 스크립트를 실행합니다. 옵션으로 **--reset** flag를 사용하여 테스트파일을 입력하실 수있습니다. |
| tronbox console                 | **tronbox**명령어를 지원하는 console입니다.                                                               |
