---
layout: post
title: Java Functional Programming - 참조 투명성 (Referential Transparency)
date: 2025-11-26 00:00:00
description: Functional Programming의 핵심 원칙인 참조 투명성(Referential Transparency)을 완벽히 이해하십시오. 참조 투명성과 순수 함수(Pure Function)의 관계를 명확한 Java 코드 예시로 설명하고, 부수 효과(Side Effect)가 없는 코드가 테스트 용이성, 메모이제이션 최적화, 스레드 안전성(Thread Safety)에 어떻게 기여하는지 상세히 다룹니다.
tags: [자바, 함수형 프로그래밍]
categories: [Java]
author: hwangrolee
image:
  path: /assets/img/01-05-java-fp-referential-transparency.png
  alt: "Java Functional Programming - 참조 투명성 (Referential Transparency)"
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

참조 투명성(Referential Transparency)은 프로그램 내의 어떤 **표현식(Expression)**이 있을 때, 그 표현식을 **표현식이 계산된 최종 결과값**으로 대체해도 프로그램의 전체 동작이나 결과가 변하지 않는 성질을 의미합니다. 이 성질은 **순수 함수(Pure Function)**에 의해 보장되며, Functional Programming의 근간을 이룹니다. 참조 투명성을 이해하고 코드를 작성하는 것은 프로그램의 **예측 가능성**을 극대화하고, **테스트를 용이**하게 하며, 컴파일러나 런타임이 코드를 안전하게 **최적화**할 수 있도록 기반을 마련해 주기 때문에, 견고한 소프트웨어 개발을 위해 매우 중요합니다.

---

## 1. 참조 투명성의 정의와 순수 함수의 관계

어떤 함수 호출이 참조 투명하다는 것은, 그 함수 호출이 **오직 인자값에만 의존**하며 **부수 효과(Side Effect)**를 발생시키지 않는다는 것을 의미합니다.

- **참조 투명한 표현식의 특징:**
  1. **동일한 입력:** 항상 동일한 출력을 반환합니다.
  2. **부수 효과 없음:** 외부 상태(전역 변수, 파일 시스템, 데이터베이스, 콘솔 출력 등)를 변경하지 않습니다.

이러한 특징을 가진 함수가 바로 **순수 함수**입니다. Functional Programming에서는 모든 코드를 순수 함수로 구성하는 것을 이상적으로 삼으며, 참조 투명성을 통해 코드를 수학 공식처럼 다룰 수 있게 됩니다.

#### 1.1. 참조 투명한 예시 (순수 함수)

가장 간단한 참조 투명한 표현식은 상수입니다. `int x = 5;`에서 `x`는 항상 `5`로 대체될 수 있습니다. 순수 함수 역시 마찬가지입니다.

```java
// 순수 함수: 오직 입력 a와 b에만 의존
public static int add(int a, int b) {
    return a + b;
}

// 참조 투명한 사용
int result1 = 5 + add(3, 4); // add(3, 4)는 항상 7
int result2 = 5 + 7;         // add(3, 4)를 결과값 7로 대체해도 동작 동일

// 결과: result1과 result2는 모두 12로 동일합니다.
```

`add(3, 4)`라는 표현식은 프로그램의 어느 위치에서 실행되든 항상 `7`이라는 결과값을 반환하므로, 이 표현식을 `7`로 대체해도 프로그램의 동작은 변하지 않습니다.

#### 1.2. 참조 투명하지 않은 예시 (비순수 함수)

참조 투명하지 않은 표현식은 **외부 상태에 의존**하거나 **외부 상태를 변경**하는 **부수 효과**가 있을 때 발생합니다.

```java
// 외부 상태(전역 변수)
private static int totalCount = 0;

// 비순수 함수: 외부 상태(totalCount)를 변경하는 부수 효과를 가짐
public static int addAndLog(int x) {
    totalCount += 1; // 외부 상태 변경 (부수 효과)
    System.out.println("로그: " + totalCount); // 외부 I/O 발생 (부수 효과)
    return x + totalCount; // 외부 상태에 의존
}

// 참조 투명하지 않은 사용
int resultA = addAndLog(10); // totalCount: 1, resultA: 11
int resultB = addAndLog(10); // totalCount: 2, resultB: 12

// resultA와 resultB는 다른 값을 가집니다.

// 만약 addAndLog(10)를 결과값 11로 대체한다면:
// int resultC = 11;
// int resultD = addAndLog(10); // totalCount: 2, resultD: 12

// resultA != resultC 이므로 참조 투명성이 깨집니다.
// 또한 addAndLog를 호출하는 시점에 콘솔 출력이라는 부수 효과가 발생하므로, 대체할 경우 그 부수 효과도 사라져 프로그램 동작이 변합니다.
```

## 2. 참조 투명성의 실질적 이점

참조 투명성은 단순한 이론이 아니라, 실제 소프트웨어 개발의 품질을 결정하는 중요한 요소입니다.

#### 2.1. 쉬운 테스트와 디버깅

순수 함수는 외부 환경과 독립적이므로, 테스트 케이스를 작성할 때 **특정 입력에 대한 출력만 검증**하면 됩니다. 복잡한 mocking이나 환경 설정을 필요로 하지 않습니다. 이는 **단위 테스트(Unit Test)**의 효율성을 극대화합니다.

#### 2.2. 최적화 가능성 (메모이제이션)

참조 투명한 함수는 동일한 입력에 대해 항상 동일한 출력을 보장합니다. 따라서 컴파일러나 런타임 환경은 이전에 계산된 결과를 저장해 두었다가(캐싱/메모이제이션), 동일한 입력이 들어왔을 때 **함수를 재실행하는 대신 저장된 결과값을 반환**하는 최적화(Optimization)를 안전하게 수행할 수 있습니다.

#### 2.3. 병렬 처리 안전성

순수 함수는 외부 상태를 변경하지 않으므로 **스레드 세이프(Thread Safe)**합니다. 여러 스레드가 동시에 동일한 순수 함수를 호출해도 경쟁 조건(Race Condition)이 발생할 염려가 전혀 없습니다. 이는 Stream API의 `parallelStream()`과 같은 병렬 처리 환경에서 코드를 안전하게 작성하는 기초가 됩니다.

결론적으로, Functional Programming에서 참조 투명성은 **순수 함수**를 통해 얻어지며, 이는 곧 **신뢰할 수 있고, 테스트하기 쉬우며, 높은 성능을 기대할 수 있는** 코드를 작성하기 위한 필수적인 원칙입니다.
