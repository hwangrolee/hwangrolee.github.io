---
layout: post
title: Java Functional Programming - 명령형 vs 선언형 프로그래밍
date: 2025-11-14 00:00:00
description: 명령형(Imperative)과 선언형(Declarative) 프로그래밍 패러다임의 근본적인 차이를 설명합니다. Java Stream
  API를 활용한 함수형 프로그래밍으로 'How'가 아닌 'What'에 집중하여 코드의 가독성, 유지보수성, 병렬 처리 성능을 향상시키는 방법을
  실제 코드 예시와 함께 배웁니다.
tags: [자바]
keywords: Java Functional Programming, 함수형 프로그래밍, 명령형 프로그래밍, 선언형 프로그래밍, Stream API,
  filter, map, reduce
categories: [Java]
giscus_comments: true
toc:
  sidebar: left
author: hwangrolee
image:
  path: /assets/img/01-01-java-fp-imperative-vs-declarative-programming.png
  alt: "Java Functional Programming - 명령형 vs 선언형 프로그래밍"
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

명령형(Imperative) 프로그래밍과 선언형(Declarative) 프로그래밍은 코드를 작성하는 근본적인 패러다임의 차이를 나타냅니다. **명령형**은 **'어떻게(How)'** 목표를 달성할지 컴퓨터에게 **단계별 명령**을 나열하는 방식인 반면, **선언형**은 **'무엇을(What)'** 달성할지 **결과를 정의**하는 데 초점을 맞춥니다.

Functional Programming은 대표적인 선언형 패러다임이며, 코드를 선언적으로 작성하면 개발자는 비즈니스 로직에 집중하고, 코드의 가독성을 높이며, 시스템의 유연성을 확보할 수 있습니다. 이 두 패러다임의 차이를 명확히 이해하는 것은 Java Stream API를 비롯한 Functional Programming 요소를 효과적으로 활용하는 데 필수적입니다.

---

## 1. 명령형 프로그래밍 (Imperative Programming)

명령형 프로그래밍은 개발자가 **상태를 직접 변경**하고, 목표 달성을 위한 **정확한 제어 흐름**을 지정하는 방식입니다. 마치 요리할 때 레시피의 모든 단계를 순서대로 지시하는 것과 같습니다.

### 특징

- **How에 집중:** 목표를 달성하는 구체적인 단계를 기술합니다.
- **가변 상태:** 변수나 객체의 상태를 적극적으로 변경합니다.
- **제어 흐름:** `if`, `for`, `while` 등 명시적인 제어 구조를 사용합니다.

### 명령형 코드 예시: 짝수의 제곱 합계 구하기

다음은 리스트에서 짝수만 필터링하여 제곱한 후 그 합계를 구하는 명령형 방식입니다.

```java
import java.util.List;
import java.util.Arrays;

List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
int sumOfEvenSquares = 0; // 1. 외부 상태(가변 변수) 초기화

// 2. 반복 제어 흐름(for loop)을 명시
for (int number : numbers) {
    // 3. 조건 검사(if)를 명시하여 짝수인지 확인
    if (number % 2 == 0) {
        int square = number * number;
        // 4. 외부 상태를 직접 변경 (Mutating State)
        sumOfEvenSquares += square;
    }
}

System.out.println("명령형 합계: " + sumOfEvenSquares); // 출력: 56 (4 + 16 + 36)
```

이 코드는 **어떻게** (루프를 돌고, if로 검사하고, 변수를 갱신하여) 합계를 계산했는지에 초점을 맞춥니다.

---

## 2. 선언형 프로그래밍 (Declarative Programming)

선언형 프로그래밍은 개발자가 **데이터의 변환 과정**만 정의하고, **구체적인 실행 방법**은 시스템(Stream API 런타임 등)에 위임하는 방식입니다. 마치 여행할 때 최종 목적지만 말하고 구체적인 운전 경로(경로 최적화, 신호 대기 등)는 내비게이션에 맡기는 것과 같습니다. Functional Programming은 이 선언형 패러다임의 가장 강력한 구현체입니다.

### 특징

- **What에 집중:** 원하는 결과와 데이터의 변환 규칙을 기술합니다.
- **불변 상태:** 상태 변경 없이 항상 새로운 값을 반환하는 불변성을 지향합니다.
- **추상화:** `map`, `filter`, `reduce` 등 고차 함수를 통해 반복 및 제어 흐름을 추상화합니다.

### 선언형 코드 예시: 짝수의 제곱 합계 구하기 (Stream API 활용)

Functional Programming의 선언형 방식은 Java Stream API를 통해 구현됩니다.

```java
import java.util.List;
import java.util.Arrays;

List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);

// Functional Programming 스타일의 선언형 파이프라인
int sumOfEvenSquaresDeclarative = numbers.stream()
    // 1. 무엇을 할지 선언: 짝수만 필터링할 것 (filter)
    .filter(n -> n % 2 == 0)
    // 2. 무엇을 할지 선언: 각 요소를 제곱으로 변환할 것 (map)
    .mapToInt(n -> n * n)
    // 3. 무엇을 할지 선언: 결과를 합산할 것 (sum)
    .sum();

System.out.println("선언형 합계: " + sumOfEvenSquaresDeclarative); // 출력: 56
```

---

## 3. 선언형 프로그래밍의 실질적 이점

선언형 코드는 명령형 코드 대비 다음과 같은 이점을 제공합니다.

### 높은 가독성

코드가 비즈니스 로직 자체를 직접적으로 설명합니다. "짝수를 걸러내고, 제곱한 다음, 합산하라"는 **도메인 언어**와 유사하게 읽힙니다.

### 쉬운 유지보수

로직을 변경할 때 전체 루프 구조를 수정할 필요 없이, `filter`나 `map`과 같은 **특정 변환 단계의 함수만 교체**하면 됩니다.

### 최적화 및 병렬성

실행 방법이 런타임에 위임되므로, Java 런타임은 Stream 파이프라인을 **자동으로 병렬 처리**(`parallelStream()`)하거나 최적화(예: 루프 퓨전)할 수 있는 자유를 얻습니다. 개발자는 **어떻게** 병렬 처리할지를 걱정할 필요가 없습니다.

---

## 결론

선언형 프로그래밍은 **추상화 수준**을 높여 코드를 간결하고 유연하며, 플랫폼의 성능 최적화 기능을 최대한 활용할 수 있도록 돕는 현대적인 개발 패러다임입니다.
