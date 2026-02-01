---
layout: post
title: 비관적 락(Pessimistic Lock), 진짜 뭔지 아세요? - 개발자가 궁금해할 질문 7가지로 정리
date: 2025-08-02 00:00:00
description: 비관적 락(Pessimistic Lock)이란 무엇일까요? 'SELECT FOR UPDATE'의 동작 원리부터 성능 저하 문제, 멀티 서버 환경에서의 동작 방식, 실무 사용 팁까지 개발자가 꼭 알아야 할 7가지 핵심 질문으로 동시성 제어의 모든 것을 완벽하게 정리합니다.
tags: [MySQL, 데이터베이스]
categories: [Database]
giscus_comments: true
author: hwangrolee
image:
  path: /assets/img/understanding-database-pessimistic-lock.png
  alt: "비관적 락(Pessimistic Lock), 진짜 뭔지 아세요? - 개발자가 궁금해할 질문 7가지로 정리"
---

### 비관적 락에 대한 7가지 궁금증

**Q1. 비관적 락은 왜 '비관적'이라고 부르나요?**

- "언젠가 누군가 같은 데이터를 건드릴 거라고 '비관적으로' 생각하기 때문"
- 읽거나 수정하기 전에 바로 락을 걸어버림
- 즉, 동시에 접근하면 안 된다고 미리 가정하는 방식

**Q2. 비관적 락을 걸면 DB는 어떻게 동작하나요?**

- Spring Data JPA → @Lock(LockModeType.PESSIMISTIC_WRITE)
- DB → SELECT ... FOR UPDATE 쿼리 실행
- DB는 조회된 row(행)에 락을 걸어, 트랜잭션이 끝날 때까지 대기 상태로 만듦
- MySQL(InnoDB) / Oracle / PostgreSQL 등 대부분 DB에서 동일 개념 적용

**Q3. 비관적 락은 "row-level lock"인가요, "table-level lock"인가요?**

- 기본은 row-level lock (행 단위 잠금)
- WHERE id = ? 처럼 명확히 특정 행을 조회하면 해당 행만 잠김
- 하지만 조건이 모호하거나 인덱스가 없으면 DB는 더 넓은 범위를 잠글 수 있음
- MySQL → Gap Lock / Next-Key Lock 발생 (범위 락)
- Oracle → Row-level 락이지만 쿼리에 따라 table-level 락으로 확대 가능성 있음
- 실무에서는 PK(기본키) 조회로 비관적 락을 거는 게 안전하다!

**Q4. 비관적 락을 걸면 언제까지 락이 유지되나요?**

- 트랜잭션이 끝날 때까지 유지
- 커밋(COMMIT) 또는 롤백(ROLLBACK) 시 해제됨
- 트랜잭션을 길게 잡으면 락도 오래 유지 → 다른 요청들 대기 → 성능 병목

**Q5. 비관적 락을 쓰면 성능은 얼마나 떨어지나요?**

- DB가 동시에 처리할 수 있는 요청을 줄여버림 → 동시성(Concurrency)↓
- 트랜잭션이 길거나락 걸리는 데이터가 많을수록
- 다른 요청이 줄줄이 대기 (Deadlock 가능성도↑)
- "락을 꼭 필요한 최소 구간에서만" 사용해야 함

**Q6. 그럼 멀티 서버 환경에서는 어떻게 되나요?**

- 비관적 락은 DB 차원에서 락을 거는 것이라 서버가 여러 대여도 안전
- 서버 A에서 SELECT FOR UPDATE로 row 잠그면, 서버 B도 DB에서 대기
- JVM 단위 synchronized 같은 건 멀티 서버에선 소용없지만, DB 락은 글로벌

**Q7. 비관적 락을 꼭 써야 하는 상황은?**

- 꼬이면 절대 안 되는 비즈니스 로직
- 은행 계좌 잔액 차감
- 좌석 예약 / 티켓 예매
- 재고 감소 처리
- 충돌 가능성이 높을 때 (낙관적 락은 충돌 났을 때 재시도가 필요하니까)
- 단, 트래픽이 매우 많고 대기 시간이 길어질 수 있다면 Redis 분산 락이나 메시지 큐로 아키텍처 확장 고려

### 마무리 (Conclusion)

**비관적 락 핵심 요약**

- "읽기 전에 잠금 → 트랜잭션 끝날 때까지 유지"
- "조회한 행(row)만 잠그지만, 조건이 모호하면 더 넓게 잠글 수도 있음"
- "멀티 서버 환경에서도 안전"

**실무 팁**

- PK 기반 조회로 락을 거는 습관
- 트랜잭션 길이는 최소화
- 대규모 트래픽 → Redis 분산 락 or MQ 검토
