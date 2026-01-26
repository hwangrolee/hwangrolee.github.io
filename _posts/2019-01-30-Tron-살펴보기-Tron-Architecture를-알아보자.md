---
layout: post
title: Tron 살펴보기:Tron Architecture를 알아보자
date: 2019-01-30
description: Tron 살펴보기 - Tron Architecture를 알아보자
categories: [Blockchain]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Tron-살펴보기-Tron-Architecture를-알아보자/
author: hwangrolee
image:
  path: https://files.readme.io/61639e9-tronarchitecture.jpg
  alt: "Tron 살펴보기:Tron Architecture를 알아보자"
tags: []
---

Tron은 storage layer, core layer, application layer로 구성된 3계층 아키텍처를 채택합니다.



### Storage Layer

Tron의 기술 팀은 state storage와 block storage로 이루어진 고유의 분산된 저장 프로토콜을 설계했습니다.

그래프 데이터베이스의 개념은 현실세계에서 전달된 데이터 저장소의 요구를 보다 잘 충족시키기 위해 storage layer의 설계에 도입되었습니다.

### Core Layer

smart contract module, account management module, consensus module는 core layer를 구성하는 3개의 모듈입니다. TRON의 비전은 쌓여진 가상머신(stacked virtual machine)과 최적화된 명령어 세트에 기반한 기능합니다.

더 나아가 다른 고급 프로그래밍 언어로 보완해야합니다.
또한 특별한 오구를 충족하기 위해 DPOS를 기본으로 TRON의 합의가 이루어졌습니다.

### Application Layer

개발자는 개량된 지갑과 전달된 DApp들의 생성을 위해 인터페이스를 활용할 수 있습니다. TRON 프로토콜은
다국어를 지원하는 Google Protobuf를 사용합니다.

### Node Types

TRON 네트워크상에 있는 3개 유형의 노드는 Super Representative Witness (SR Full Node), Full Node, Solidity Node입니다.
**SR Full Nodes**는 블록생산을 담당합니다.
**Full Nodes**는 APIs를 제공하고 트랜잭션과 블록을 전파합니다.
**Solidity Nodes**는 돌이킬수 없는 블록과 문의 APIs를 제공합니다.
exchange는 **Full Node** 와 **Solidity Node**를 배포하기 위해 필요합니다. **solidity node**는 **Mainnet**에 연결되는 **local Full Node**에 연결합니다.

### Mainnet & Testnet

TRON 네트워크는 Shasta라는 테스트넷뿐만 아니라 Mainnet을 운영중입니다. Mainnet blockchain explorer는 [여기](https://tronscan.org/#/)를 통해 접속할 수있습니다. Shasta blockchain explorer는 [여기](https://shasta.tronex.io/)를 통해 접속할 수 있습니다. 사용자는 Shasta blockchain explorer 사이트에 접속해서 Testnet TRX를 받기위한 Testnet wallet을 열 수 있습니다. [Github](https://github.com/tronprotocol/tron-deployment)을 통해 찾을 수 있는 **Mainnet**과 **Testnet** 설정파일은 node를 세팅할때 필요합니다.
