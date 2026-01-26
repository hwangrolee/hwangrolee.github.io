---
layout: post
title: MySQL 트랜잭션 격리수준 Isolation level 알아보기
date: 2024-12-15 00:00:00
description: MySQL 트랜잭션 격리수준 Isolation level 알아보기
tags: [MySQL, 데이터베이스]
keywords: mysql, 트랜잭션, Isolation Level, 트랜잭션 격리수준, Transaction
categories: [Database]
giscus_comments: true
toc:
  sidebar: left
author: hwangrolee
image:
  path: /assets/img/understanding-mysql-transaction-isolation-levels.png
  alt: "MySQL 트랜잭션 격리수준 Isolation level 알아보기"
---

트랜잭션의 격리 수준이란 여러 트랜잭션이 동시에 처리될 때 다른 트랜잭션에서 변경이 일어났을 때 해당 데이터를 또 다른 트랜잭션에서 조회가 가능하게 할지를 허용할지 말지를 결정하는 것이다. 격리 수준에는 총 4개가 있다.

- READ UNCOMMITTED
- READ COMMITTED
- REPEATABLE READ
- SERIALIZABLE
  위 4개 중 첫번째인 READ UNCOMMITTED는 격리수준이 가장 낮고 SERIALIZABLE 은 격리 수준이 가장 높다.

## 1. READ UNCOMMITTED

트랜잭션 격리수준 중 가장 낮은 단계로 성능이 가장 좋지만 다른 트랜잭션에서 커밋되지 않은 데이터를 읽어 올 수 있기 때문에 잘 못 사용할 경우 데이터 정합성에 큰 이슈가 발생 할 수 있다.

Transaction_A 에서 INSERT, UPDATE, DELETE 쿼리를 실행 후 아직 커밋하지 않았지만 Transaction_B 에서 SELECT 쿼리를 실행시키게 되면 INSERT, UPDATE, DELETE 쿼리 결과가 보이게 된다. 이 때 Transaction_A 는 COMMIT 하지 않고 ROLLBACK 을 하게 되면 Transaction_B 는 실제 데이터베이스에 없는 데이터를 읽게 된 것이며 이를 더티리드(Dirty read) 라고 한다.

트랜잭션에서 INSERT, UPDATE, DELETE 쿼리를 실행할 경우 커밋하기 전에는 Undo 영역에 변경 전 데이터를 저장하고 레코드의 값을 변경한다. 이 때 Rollback 을 하게 되면 Undo 영역에 있던 데이터를 레코드로 복원하게 된다.

READ_UNCOMMITTED 는 레코드에 저장된 값을 가저오기 때문에 롤백이 된 데이터를 읽어 올 수 있다.

READ UNCOMMITTED 는 정합성 이슈가 발생하지 않는 SELECT 쿼리에서만 사용할 것을 권장하며 READ UNCOMMITTED로 읽은 데이터로 INSERT, UPDATE, DELETE 에 사용하지 않도록 주의해야 한다.

```sql
-- [SESSION_1]
BEGIN;
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
SELECT * FROM user;
-- name = 'one' 인 데이터가 출력됨.

-- [SESSION_2]
BEGIN;
UPDATE user SET name = 'two' WHERE name = 'one';
-- name = 'one' 을 'two' 로 업데이트함. 아직 커밋하지 않았음.
-- [SESSION_1]
SELECT * FROM user;
-- name = 'two' 인 데이터 출력됨.
-- [SESSION_2]
ROLLBACK;
-- 트랜잭션을 롤백
-- [SESSION_1]
SELECT * FROM user;
-- 롤백전에는 name = 'two' 인 데이터가 출력되었지만 롤백 후에는 이전 데이터인 'one'인 데이터가 출력됨
```

위 내용을 직접 테스트해 보았을 때 name = ‘two’ 는 트랜잭션 내부에서 업데이트문만 실행했을 뿐 COMMIT 하지 않았지만 다른 세션에서 실행한 트랜잭션에서 읽을 수 있다. name = ‘two’가 제대로 커밋 된다면 다행이지만 롤백이 된다면 잘못된 데이터를 불러온 것이 된다.

{: .block-tip }

> ##### TIP
>
> READ UNCOMMITTED 는 자주 호출되지만 INSERT, UPDATE, DELETE 쿼리에는 영향이 가지 않는 곳에서 사용해야만 한다.

## 2. READ COMMITTED

MSSQL, Oracle 의 기본 트랜잭션 격리수준으로 커밋 된 데이터를 읽을 수 있는 격리수준이다. 커밋 된 데이터만 읽기 때문에 READ UNCOMMITTED 처럼 더티리드(Dirty read)가 발생하지 않기 때문에 대부분의 데이터를 신뢰할 수 있다.

READ COMMITTED 격리수준이라고 해서 모든 상황에서 데이터를 신뢰 할 수 있는 것은 아니다. 바로 Unrepeatable read 로 인해 다른 트랜잭션 간에 커밋 타이밍에 따라 데이터 정합성이 어긋 날 수 있기 때문이다. 그래서 READ COMMITTED 는 대부분의 데이터를 신뢰할 수 있다 라고 표현했다.

```sql
-- [SESSION_1]
BEGIN;
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
SELECT * FROM user;
-- name = 'one' 인 데이터가 출력됨.
-- [SESSION_2]
BEGIN;
UPDATE user SET name = 'two' WHERE name = 'one';
-- name = 'one' 을 'two' 로 업데이트함. 아직 커밋하지 않았음.
-- [SESSION_1]
SELECT * FROM user;
-- SESSION_2가 커밋되지 않았기 때문에 name = 'one' 인 데이터 출력됨.
-- [SESSION_2]
COMMIT;
-- 커밋했기 때문에 디비에 데이터 반영됨.
-- [SESSION_1]
SELECT * FROM user;
-- SESSION_2가 커밋되면서 name = 'two' 인 데이터가 출력된.
name='one'
```

위 상황과 같이 SESSION_1 은 트랜잭션이 시작 했을 때 데이터를 확인해보면 name=’one’ 인 상태인데 SESSION_2 에서 트랜잭션 시작 후 업데이트 후 커밋까지 한 후 SESSION_1 에서 다시 조회를 하면 name = ‘two’ 로 바뀌어 있습니다. 이 처럼 같은 트랜잭션 내에게 같은 데이터를 조회했을 때 다른 데이터가 출력된다면 이 또한 데이터 정합성에 문제가 발생할 수 있다.

일반적인 상황에서는 크게 문제되지 않지만 같은 트랜잭션 내에서 같은 SELECT 쿼리 여러번 호출하는 경우에는 이슈가 발생할 수 있다. 되도록이면 같은 트랜잭션 내에서 같은 SELECT 쿼리를 호출하는 케이스를 줄이도록 한다면 위와 같이 Unrepeatable read 현상을 줄일 수 있다.

## 3. REPEATABLE READ

READ COMMITTED 와 같이 커밋 된 데이터를 읽지만 Unrepeatable read 현상이 발생하지 않는 격리 수준이다. MySQL 의 InnoDB 엔진에서 사용하는 기본격리수준이다.

REPEATABLE READ는 Unrepeatable read 가 발생하지 않도록 트랜잭션이 시작하는 시점의 스냅샷을 기준으로 데이터를 조회한다. 다른 트랜잭션에 데이터가 업데이트되었더라도 현재 트랜잭션에서는 스냅샷 기준으로 데이터를 가저오기 때문에 Unrepeatable read 현상이 발생하지 않는다.

트랜잭션을 시작하게 되면 트랜잭션에는 번호를 가지게 되고 언두 영역의 백업된 모든 레코드에는 백업한 트랜잭션 번호가 포함되어 있다. 그리고 언두 영역의 백업된 데이터는 InnoDB 엔진이 불필요하다고 판단하는 시점에 주기적으로 삭제한다.

REPEATABLE READ 는 PHANTOM READ(=PHANTOM ROW) 라는 이슈가 발생한다. PHANTOM READ는 첫번째 SELECT 문에서 보이지 않았던 데이터가 두번째 SELECT 쿼리에서는 보이는 현상이다.

InnoDB 에서의 독특한 매커니즘으로 인해 PHANTOM READ 현상은 발생하지 않는다. 직접 MySQL 로 테스트를 해보았을 때 발생하지 않았다.

INSERT 쿼리 실행 후 커밋하지 않은 상태에서 다른 트랜잭션에서 SELECT ~ FOR UPDATE 를 실행 시 INSERT 를 실행 한 트랜잭션에 의해 SELECT ~ FOR UPDATE 쿼리는 락이 걸리게 되어 PHANTOM READ 가 발생하지 않는다.

## 4. SERIALIZABLE

가장 엄격한 격리수준으로 동시 작업 성능이 가장 떨어지는 격리 수준이다. SELECT 쿼리는 기본적으로 SHARED LOCk 을 기본적으로 획득하게되어 SELECT 중 INSERT, UPDATE, DELETE 가 불가능하다.

## 5. 결론

기본적으로 *READ COMMITTED 혹은 REPEATABLE READ 를 사용할 것을 권장*한다.

되도록이면 1개의 트랜잭션에서 같은 SELECT 쿼리를 실행하지 않도록 주의한다. READ COMMITTED의 경우 1개의 트랜잭션에서 같은 SELECT 문을 실행시 Unrepeatable read 현상이 발생할 여지가 있다. REPEATABLE READ 의 경우 PHANTOM READ가 발생할 수 있다. (InnoDB 외 다른 엔진에서 PHANTOM READ 발생)

SERIALIZABLE 은 사용하지 않는 것이 정신 건강에 좋다.
실시간성을 보장받지 않고 데이터가 정확하지 않아도 되는 기능의 경우 READ UNCOMMITTED 를 사용하면 성능을 높일 수 있다.

## 6. 참고

- https://zzang9ha.tistory.com/381
- Real Mysql 8.0
