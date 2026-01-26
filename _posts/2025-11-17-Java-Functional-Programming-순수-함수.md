---
layout: post
title: Java Functional Programming - 순수함수(Pure Function)
date: 2025-11-17 00:00:00
description: Functional Programming의 핵심인 순수 함수(Pure Function)를 깊이 있게 다룹니다. 같은 입력에 항상 같은 출력을 보장하는 결정론적 특성과 부수 효과 제거를 통해 병렬 처리 안전성, 메모이제이션, 참조 투명성을 확보하는 방법을 실제 Java 코드 예시와 함께 설명합니다.
tags: [자바, 함수형 프로그래밍]
categories: [Java]
author: hwangrolee
image:
  path: /assets/img/01-02-java-fp-pure-function.png
  alt: "Java Functional Programming - 순수함수(Pure Function)"
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

순수 함수(Pure Function)는 Functional Programming의 가장 근본적인 개념입니다. 순수 함수는 두 가지 핵심 속성을 만족해야 합니다. 첫째, **같은 입력에 대해 항상 같은 출력을 보장**해야 하며(결정론적), 둘째, 함수 실행 시 **외부 상태를 변경하거나 외부에 의존하는 부수 효과(No Side Effect)가 없어야** 합니다. 순수 함수를 중심으로 코드를 작성하는 것은 프로그램의 **예측 가능성**과 **신뢰성**을 극대화하며, 병렬 처리와 캐싱을 용이하게 하여 **견고한 시스템 아키텍처**를 구축하는 데 필수적인 원칙입니다.

---

## 1. 순수 함수의 두 가지 핵심 정의

순수 함수는 수학에서의 함수 개념을 프로그래밍에 도입한 것입니다. 즉, 입력(인자)만으로 출력이 결정되고, 그 외의 어떤 것도 함수의 결과에 영향을 미치거나 함수가 외부에 영향을 주지 않습니다.

### 1.1. 결정론적 (Deterministic)

함수에 동일한 인자를 여러 번 전달하더라도, 항상 정확히 동일한 결과 값을 반환해야 합니다. 이는 함수가 호출 시점의 전역 변수나 시스템 시간 등 **외부의 가변적인 상태**에 의존하지 않음을 의미합니다.

- **비결정론적 예시:** 현재 시간을 반환하는 함수, 난수를 생성하는 함수.

### 1.2. 부수 효과 없음 (No Side Effect)

함수가 실행되는 동안 다음 중 어떤 것도 수행하지 않아야 합니다.

- **외부 상태 변경:** 전역 변수나 클래스 인스턴스 필드(상태)를 수정하는 행위.
- **외부 I/O 작업:** 파일 읽기/쓰기, 데이터베이스 접근, 네트워크 통신, 콘솔 출력(`System.out.println`) 등.
- **인자 객체 변경:** 인자로 받은 가변 객체(Mutable Object)의 내부 상태를 수정하는 행위.

## 2. 순수 함수와 비순수 함수의 비교

### 2.1. 순수 함수의 예시

```java
// 순수 함수: 오직 입력 값 a와 b에만 의존하며, 외부 상태를 변경하지 않습니다.
public static int sum(int a, int b) {
    return a + b;
}

// 순수 함수: 인자로 받은 List를 수정하지 않고, 항상 새로운 List를 반환합니다.
public static List<Integer> addElement(List<Integer> originalList, int newElement) {
    List<Integer> newList = new ArrayList<>(originalList);
    newList.add(newElement);
    return newList; // 새로운 객체를 반환
}

// 사용 예시
List<Integer> initial = Arrays.asList(1, 2);
List<Integer> added = addElement(initial, 3);
// initial의 상태는 변경되지 않고, added라는 새로운 List가 생성됩니다.
System.out.println("원본: " + initial); // 출력: [1, 2]
System.out.println("새로운 리스트: " + added); // 출력: [1, 2, 3]
```

`addElement` 함수는 불변성을 지키며 새로운 데이터를 반환하므로 순수 함수입니다.

---

### 2.2. 비순수 함수의 예시

```java
// 외부 상태 (전역 변수)
private static int totalProcessedCount = 0;

// 비순수 함수: 외부 상태를 변경하고, 인자 외의 값에 의존합니다.
public static int processAndIncrement(int data) {
    // 1. 외부 상태 변경 (Side Effect)
    totalProcessedCount++;

    // 2. I/O 발생 (Side Effect)
    System.out.println("처리 횟수: " + totalProcessedCount);

    // 3. 외부 상태에 의존하여 결과 계산 (비결정론적)
    return data + totalProcessedCount;
}

// 사용 예시
int result1 = processAndIncrement(10); // totalProcessedCount: 1, 결과: 11
int result2 = processAndIncrement(10); // totalProcessedCount: 2, 결과: 12
// 같은 입력 10을 사용했지만, totalProcessedCount의 변화로 인해 결과가 다릅니다.
```

## 3. 순수 함수의 실질적인 이점

순수 함수 원칙을 지키는 것은 Functional Programming의 다른 모든 이점을 누릴 수 있는 기반을 제공합니다.

### 3.1. 동시성 및 병렬 처리 안전성

순수 함수는 외부 상태를 변경하지 않으므로 본질적으로 **스레드 안전성(Thread Safety)**이 보장됩니다. 여러 스레드가 동시에 함수를 실행하더라도, 공유 데이터에 대한 경쟁 조건(Race Condition)이 발생하지 않아 락(Lock)이나 동기화 메커니즘 없이도 안전하게 병렬 처리를 수행할 수 있습니다. 이는 Stream API의 `parallelStream()`을 안전하게 사용하는 핵심 이유입니다.

### 3.2. 메모이제이션(Caching)을 통한 성능 최적화

순수 함수는 결정론적이기 때문에, 한 번 계산된 결과는 입력 인자와 함께 캐시에 저장될 수 있습니다. 이후 동일한 인자로 함수가 다시 호출되면, 실제 계산 대신 캐시된 값을 즉시 반환하는 **메모이제이션(Memoization)** 기법을 안전하게 적용할 수 있습니다.

### 3.3. 참조 투명성 확보

순수 함수는 항상 참조 투명성(Referential Transparency)을 만족합니다. 이는 함수 호출 표현식을 계산된 결과값으로 대체해도 프로그램의 의미가 변하지 않음을 보장하여, 코드를 마치 수학 방정식처럼 다룰 수 있게 하며 컴파일러가 코드를 최적화할 수 있는 여지를 제공합니다.
