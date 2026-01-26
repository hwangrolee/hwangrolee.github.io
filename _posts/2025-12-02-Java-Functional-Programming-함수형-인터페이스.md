---
layout: post
title: Java Functional Programming - 함수형 인터페이스
date: 2025-12-02 00:00:00
description: Java Functional Programming의 핵심인 표준 함수형 인터페이스(Function, Consumer, Predicate,
  Supplier)를 마스터하십시오. 각 인터페이스의 역할과 Stream API에서의 활용법을 구체적인 Java 8+ 코드로 설명합니다. Bi-접두사
  인터페이스와 Primitive 특화 인터페이스를 통한 박싱/언박싱 오버헤드 제거 및 성능 최적화 전략까지 상세히 다룹니다.
tags: [자바]
keywords: Java 함수형 인터페이스, Functional Interface, java.util.function, 람다 표현식, Lambda
  Expression, Stream API, Function, Consumer, Predicate, Supplier, Primitive 특화, 박싱
  언박싱, Boxing Unboxing, BiFunction, SAM
categories: [Java]
giscus_comments: true
toc:
  sidebar: left
author: hwangrolee
image:
  path: /assets/img/02-02-java-fp-functional-interface.png
  alt: "Java Functional Programming - 함수형 인터페이스"
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

Functional Programming에서 함수형 인터페이스는 자바 코드로 **함수(람다)**를 데이터처럼 취급하고 전달하기 위해 정의된 **단일 추상 메서드(SAM)** 규격입니다. `java.util.function` 패키지의 네 가지 핵심 인터페이스와 그 변형들을 익혀야만, 여러분은 Stream API를 포함한 Functional Programming 코드를 간결하고 효율적으로 작성할 수 있습니다.

---

## 표준 Functional Programming 인터페이스 4가지 기본 형태

이 네 가지 인터페이스는 입력값과 반환값의 존재 여부에 따라 함수가 수행할 기본 역할을 정의합니다.

#### 1. Function<T, R>: 입력과 출력 (변환)

- *`Function`*은 타입 **T**를 입력받아 다른 타입 **R**로 **변환(Mapping)**하는 역할을 수행합니다. 입력과 출력이 모두 존재하여 가장 일반적인 함수의 형태를 띠며, 데이터를 한 형태에서 다른 형태로 바꿀 때 사용됩니다.
- **추상 메서드:** `R apply(T t)`
- **활용 예:** Stream API의 **`map()`** 메서드에 전달되어 데이터 타입을 변환할 때 사용됩니다.

```java
// String을 입력받아 그 길이를 Integer로 반환하는 Function
Function<String, Integer> stringToLength = s -> s.length();

Integer length = stringToLength.apply("Functional Programming"); // 결과: 21
```

#### 2. Consumer<T>: 입력과 소비 (void 반환)

- *`Consumer`*는 타입 **T**를 입력받아 단순히 **소비**하고, 어떠한 값도 반환하지 않습니다 (`void`). 주로 함수 외부의 상태를 변경하는 **부수 효과(Side Effect)**를 일으키는 용도로 사용됩니다.
- **추상 메서드:** `void accept(T t)`
- **활용 예:** Stream API의 **`forEach()`** 메서드에 전달되어 최종 출력을 하거나, 로그를 남기는 등의 작업을 수행할 때 사용됩니다.

```java
// Integer를 입력받아 콘솔에 출력하는 Consumer
Consumer<Integer> printNumber = n -> System.out.println("소비된 데이터: " + n);

printNumber.accept(2025);
```

#### 3. Predicate<T>: 입력과 판단 (boolean 반환)

- *`Predicate`*는 타입 **T**를 입력받아 특정 조건에 맞는지 **논리적으로 판단**하고 `boolean` 값을 반환합니다. 참 또는 거짓을 판별하는 필터링 조건 정의에 사용됩니다.
- **추상 메서드:** `boolean test(T t)`
- **활용 예:** Stream API의 **`filter()`** 메서드에 전달되어 데이터를 걸러내는 조건을 정의할 때 사용됩니다.

```java
// String을 입력받아 길이가 10을 초과하는지 판단하는 Predicate
Predicate<String> isLongerThanTen = s -> s.length() > 10;

boolean result = isLongerThanTen.test("Java Programming"); // 결과: true
```

#### 4. Supplier<T>: 출력만 (입력 없음)

- *`Supplier`*는 아무런 입력 없이, 특정 타입 **T**의 객체를 **제공(Supply)**하는 역할을 합니다.
- **추상 메서드:** `T get()`
- **활용 예:** **지연 로딩(Lazy Loading)**이나 예외 발생 시 기본값을 제공할 때 사용됩니다. 리소스 생성 비용을 필요할 때까지 미루는 데 유용합니다.

```java
// 복잡한 객체 생성을 지연시키는 Supplier
Supplier<String> lazyMessage = () -> {
    // 이 로직은 get()이 호출될 때만 실행됨
    return "초기화된 복잡한 객체";
};

String message = lazyMessage.get();
```

---

## 표준 인터페이스의 변형: 유연성과 성능 확보

표준 4가지 인터페이스는 인자의 개수 확장 및 성능 최적화를 위해 변형을 제공합니다.

#### 1. Bi- 접두사: 인자가 2개인 함수

`Bi-` 접두사는 입력 인자가 두 개인 함수형 인터페이스를 나타내어, 두 데이터를 조합하거나 비교할 때 유연성을 제공합니다.

- **`BiFunction<T, U, R>`:** 두 개의 입력(T, U)을 받아 하나의 결과(R)를 반환합니다.
- **`BiConsumer<T, U>`:** 두 개의 입력(T, U)을 소비하고 리턴이 없습니다.
- **`BiPredicate<T, U>`:** 두 개의 입력(T, U)을 받아 논리적 판단을 수행합니다.

```java
// 두 개의 Integer를 더하는 BiFunction
BiFunction<Integer, Integer, Integer> adder = (a, b) -> a + b;

Integer sum = adder.apply(5, 12); // 결과: 17
```

#### 2. Primitive 특화: 박싱/언박싱 오버헤드 제거

자바의 제네릭은 내부적으로 객체 타입(Wrapper Class)만 사용할 수 있으므로, 기본 타입(`int`, `double`)을 사용할 때마다 **박싱(Boxing)**과 **언박싱(Unboxing)**이라는 객체 변환 과정이 발생하며 성능 오버헤드를 줄 수 있습니다.

이를 방지하기 위해 `int`, `long`, `double` 기본 타입 전용 함수형 인터페이스가 제공됩니다.

- **`ToIntFunction<T>` / `ToDoubleFunction<T>`:** 객체 타입 T를 받아 기본 타입으로 반환하는 함수.
- **`IntConsumer` / `LongConsumer`:** 기본 타입을 입력받아 소비하는 함수.
- **`IntPredicate` / `LongPredicate`:** 기본 타입을 입력받아 판단하는 함수.
- **`IntSupplier` / `LongSupplier`:** 입력 없이 기본 타입을 반환하는 공급자.

```java
// int만 사용하여 성능을 최적화한 IntPredicate
IntPredicate isEven = n -> n % 2 == 0;

boolean result = isEven.test(4); // 결과: true

// Primitive Stream과의 연동
int resultSum = IntStream.rangeClosed(1, 5)
                         .filter(isEven)
                         .sum(); // 결과: 6 (2 + 4)
```

이러한 Primitive 특화 인터페이스는 **`IntStream`**과 같은 기본 타입 스트림에서 자주 사용되며, 성능이 중요한 수치 계산 로직에서 불필요한 객체 변환 비용을 절감하는 데 큰 도움이 됩니다. 이들을 익히는 것이 자바 Functional Programming을 효율적으로 사용하는 지름길입니다.
