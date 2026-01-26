---
layout: post
title: Tron 살펴보기:TronBox로 Smart Contract 배포하기
date: 2019-01-30
description: Tron 살펴보기 - TronBox로 Smart Contract 배포하기
categories: [Blockchain]
giscus_comments: true
sitemap:
  changefreq: daily
  priority: 1.0
redirect_from:
- /Tron-살펴보기-TronBox로-Smart-Contract-배포하기/
author: hwangrolee
tags: []
---

### Example Tutorial Project

TronBox MetaCoin 예제 프로젝트는 다음 링크에서 진행 할 수 있습니다.
[https://github.com/sullof/metacoin-box](https://github.com/sullof/metacoin-box)

### Smart Contract Develpment

> 스마트 컨트렉트 개발

**./contracts** 디렉토리에 컨트랙트 파일을 담고 있습니다. 기본적으로 .sol 확자내에 컨트랙트 파일과 라이브러리 파일이 존재합니다. 라이브러리 파일에 단순한 몇가지 고유의 속성이 있지만 이를 contract라고 합니다.

Tronbox는 동일한 contract 이름과 파일 이름으로 정의해야합니다. 예를 들어 _Test.sol_ 파일이 있다면 다음과 같이 contract를 작성해야합니다.

```solidity
pragma solidity ^0.4.23;
contract Test{
    function f() returns (string){
        return "method f()";
    }
    function g() returns (string){
        return "method g()";
    }
}
```

contract type은 대소문자를 구별합니다.

### Increase Deploy Configuration

설정파일 migrations/2_deploy_contracts.js는 다음과 같습니다.

```javascript
// migrations/2_deploy_contracts.js
var Migrations = artifacts.require("./Migrations.sol");
var Test = artifacts.require("./Test.sol");
module.exports = function (deployer) {
  deployer.deploy(Migrations);
  deployer.deploy(Test);
};
```

위 코드를 보면 눈여겨봐야할 코드 2줄이 있습니다. 하나는 **var Test = artifacts.require("./Test.sol")**, 새로운 contract 인스턴스를 Test에 선언합니다. **deployer.deploy(Test)**는 **Test**를 배포합니다.

### Configuration

tronbox.js는 contract를 배포하기 위한 설정파일입니다. **migration** 혹은 **test**시 네트워크를 설정하기 위한 **--network NETWORK_NAME**을 입력할 수있습니다.

```javascript
// tronbox.js
module.exports = {
  networks: {
      development: {
          from: 'some address',
          privateKey: 'some private key',
          consume_user_resource_percent: 30,
          fee_limit: 100000000,
          fullNode: "https://api.trongrid.io",
          solidityNode: "https://api.trongrid.io",
          eventServer:  "it is optional",
          network_id: "*" // Match any network id
      },
      production: {
          from: 'some other address',
          privateKey: 'some other private key',
          consume_user_resource_percent: 30,
          fee_limit: 100000000,
          fullNode: "https://api.trongrid.io",
          solidityNode: "https://api.trongrid.io",
          eventServer: "https://api.trongrid.io",
          network_id: "*" // Match any network id
      },
      shasta: {
          from: 'some other address',
          privateKey: 'some other private key',
          consume_user_resource_percent: 30,
          fee_limit: 100000000,
          fullNode: "https://api.shasta.trongrid.io",
          solidityNode: "https://api.shasta.trongrid.io",
          eventServer: "https://api.shasta.trongrid.io",
          network_id: "*" // Match any network id
      }
    ..... you can define other network configuration as well
  }
};
```

> **tronbox.js**의 privateKey는 tron wallet에서 확인 할 수 있습니다.
> 저는 Chrome browser extension으로 설치 가능한 [TronLink](https://chrome.google.com/webstore/detail/tronlink/ibnejdfjmmkpcnlpebklmnkoeoihofec?hl=ko)를 사용했습니다.

### Contract Compiling

> 컨트랙트 컴파일

```bash
# 컨트랙트를 컴파일합니다.
$ tronbox compile
Compiling ./contracts/Migrations.sol...
Writing artifacts to ./build/contracts
```

기본적으로 tronbox 컴파일러는 불필요한 컴파일을 줄이기 위해 마지막 컴파일 이후의 수정된 contract만 컴파일 합니다. 만약 전체 파일을 컴파일하고 싶다면 **--compile-all**을 사용할 수있습니다.

```bash
$ tronbox compile --compile-all
Compiling ./contracts/Migrations.sol...
Writing artifacts to ./build/contracts
```

컴파일 결과는 **./build/contracts** 디렉토리에 저장됩니다. 디렉토리가 존재하지 않다면 자동으로 생성됩니다.

### Contract Deployment

> 컨트랙트 배포

Script Migration은 Tron 네트워크로 전파하기 위해 구성된 Javascript 파일로 구성되 있습니다. script migration은 boardcast(전파) responsibility(책임?)을 캐시합니다. script migration은 broadcast needs를 기반으로 하며 그에 따라 조정될 것입니다. 당신의 일에 상당한 변화가 생겼을때, 당신이 생성한 migration script는 변화를 블록체인을 통해 전파할 것입니다.이전 migration 기록 내력은 고유한 migration contract을 거처 블록체인에 기록되어집니다. 아래는 자세한 지시사항입니다.

```bash
$ tronbox migrate
```

이 명령어 **migration**디렉토리에 있는 모든 migration script를 초기화합니다. 이전 migration이 성공했다면, **tronbox migrate**는 새로운 migration으로 초기화합니다. 새로운 migration script가 아니라면 이 명령어 동작하지 않을 것입니다. 따라서 새로 배포하기 위해서는 **--reset**를 사용할수 있습니다.

```bash
$ tronbox migrate --reset --network shasta
Running migration: 1_initial_migration.js
  Deploying Migrations...
  Migrations:
    (base58) TLe9SwehECmQPzZEPHHTHJiVoWRmaW9h5j
    (hex) 41750e838b993e722fdade967dae22de8b13c3fc6c
Saving successful migration to network...
Saving artifacts...
Running migration: 2_deploy_contracts.js
  Deploying Test...
  Test:
    (base58) TNpYcmZKvi75fpPrmNuhM21xSQHMtQCXmz
    (hex) 418cf64411285f96ff188c669f421633831e28b34c
Saving successful migration to network...
Saving artifacts...
```

### Trigger Contract

> 컨트랙트 강제 실행

**./tests**디렉토리에 테스트할 수 있는 스크립트가 있습니다. TronBox .js, .es, .es6 및 .jsx를 제외한 다른 확장자는 무시합니다.

```javascript
// test.js
var Test = artifacts.require("./Test.sol");
contract("Test", function (accounts) {
  it("call method g", function () {
    Test.deployed()
      .then(function (instance) {
        return instance.call("g");
      })
      .then(function (result) {
        assert.equal("method g()", result[0], "is not call method g");
      });
  });
  it("call method f", function () {
    Test.deployed()
      .then(function (instance) {
        return instance.call("f");
      })
      .then(function (result) {
        assert.equal("method f()", result[0], "is not call method f");
      });
  });
});
```

### Running Testing Script

```bash
$ tronbox test ./test/test.js  --network shasta
Using network 'shasta'.

Deploying contracts to development network...

Warning: This version does not support tests written in Solidity.

Preparing Javascript tests (if any)...


  0 passing (0ms)
```
