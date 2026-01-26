---
layout: post
title: Java Functional Programming - 람다 표현식 및 메서드 참조
date: 2025-11-29 00:00:00
description: Java 8 Functional Programming의 핵심인 람다 표현식과 메서드 참조 활용법을 마스터하세요. 익명 클래스에서
  람다로, 람다에서 메서드 참조로 코드를 간결화하는 과정을 설명하며, 정적, 인스턴스, 임의 객체, 생성자 참조의 네 가지 유형을 구체적인 Java
  코드로 제시합니다. Stream API에서 선언적인 코드를 작성하고 가독성을 극대화하는 방법을 습득하십시오.
tags: [자바]
keywords: Java 람다, Lambda Expression, 메서드 참조, Method Reference, Java 8, Functional
  Programming, Stream API, 익명 클래스, Anonymous Class, 함수형 인터페이스, Functional Interface,
  고차 함수, Higher-Order Function, 선언적 프로그래밍
categories: [Java]
giscus_comments: true
toc:
  sidebar: left
author: hwangrolee
image:
  path: /assets/img/02-01-java-fp-lambda-expression-and-method-reference.png
  alt: "Java Functional Programming - 람다 표현식 및 메서드 참조"
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

람다 표현식(Lambda Expression)과 메서드 참조(Method Reference)는 Java 8에서 Functional Programming 스타일을 도입하며 코드를 극도로 간결하게 만들고 **동작(Behavior)을 값처럼 다룰 수 있게** 해주는 핵심 요소입니다. 람다는 기존의 장황한 **익명 클래스(Anonymous Class)**를 축약하여 **함수 리터럴**처럼 사용할 수 있게 하며, 메서드 참조는 이 람다 표현식마저도 한 단계 더 줄여 **가독성을 극대화**합니다. 이들을 능숙하게 활용하는 것은 Functional Programming 기반의 **선언적인(Declarative) 코드**를 작성하고 Stream API를 효율적으로 사용하는 데 있어 기본 중의 기본입니다.

---

## 1. 익명 클래스에서 람다 표현식으로의 전환

Java 8 이전에는 함수형 인터페이스(Functional Interface, 추상 메서드가 하나만 있는 인터페이스)를 구현할 때 **익명 클래스**를 사용해야 했습니다. 이는 코드를 장황하게 만드는 주범이었습니다. 람다 표현식은 이 과정을 **인자 리스트, 화살표(`->`), 그리고 바디**로 단순화합니다.

#### 1.1. 익명 클래스 (Java 8 이전)

```java
import java.util.concurrent.Callable;

// Callable<String>은 인자가 없고 String을 반환하는 함수형 인터페이스입니다.
Callable<String> anonymous = new Callable<String>() {
    @Override
    public String call() throws Exception {
        return "익명 클래스 실행 결과";
    }
};
// System.out.println(anonymous.call());
```

#### 1.2. 람다 표현식으로 전환 (Java 8 이후)

익명 클래스에서 불필요했던 인터페이스 이름, 메서드 이름, `return` 키워드 등을 제거하여 핵심 로직만 남깁니다.

```java
import java.util.concurrent.Callable;

// 인자 리스트 () -> 바디 { return "람다 실행 결과"; }
Callable<String> lambda = () -> "람다 실행 결과";
// System.out.println(lambda.call());

// * 일반적인 람다 형태 (Function<Integer, Integer> 예시)
// 입력 타입과 return, 중괄호 생략 가능
java.util.function.Function<Integer, Integer> addOne = x -> x + 1;
// System.out.println(addOne.apply(5)); // 출력: 6
```

람다를 사용하면 코드가 훨씬 간결해지며, 개발자는 **수행할 동작(행동)** 자체에 집중할 수 있습니다.

---

## 2. 람다 표현식에서 메서드 참조로의 전환

메서드 참조(Method Reference)는 람다 표현식이 **이미 존재하는 하나의 메서드를 호출하는 것 외에 다른 작업을 수행하지 않을 때** 람다마저도 더 단순화하는 기법입니다. 메서드 참조는 **`::`** 연산자를 사용하여 코드를 마치 자연어처럼 읽히게 만들어 가독성을 극대화합니다.

메서드 참조는 크게 네 가지 형태로 사용됩니다.

#### 2.1. 정적 메서드 참조 (`ClassName::staticMethodName`)

람다가 정적 메서드를 호출하기만 할 때 사용합니다.

```java
import java.util.function.Function;

// 람다: x -> Integer.parseInt(x)
Function<String, Integer> lambdaParser = s -> Integer.parseInt(s);

// 메서드 참조: Integer::parseInt
Function<String, Integer> methodRefParser = Integer::parseInt;

Integer result = methodRefParser.apply("100");
System.out.println("정적 메서드 참조 결과: " + result); // 출력: 100
```

#### 2.2. 특정 객체의 인스턴스 메서드 참조 (`instance::instanceMethodName`)

특정 인스턴스에 대해 인스턴스 메서드를 호출할 때 사용합니다.

```java
java.util.function.Supplier<Long> getter = System.out::currentTimeMillis;
// Supplier는 인자가 없고 결과를 반환합니다.
long now = getter.get();
System.out.println("인스턴스 메서드 참조 결과: " + now);
```

#### 2.3. 특정 타입의 임의 객체 인스턴스 메서드 참조 (`ClassName::instanceMethodName`)

Stream API에서 가장 흔하게 사용되는 패턴입니다. 람다의 인자가 해당 인스턴스 메서드의 실행 대상이 될 때 사용합니다.

```java
import java.util.List;
import java.util.Arrays;
import java.util.stream.Collectors;

List<String> names = Arrays.asList("a", "b", "c");

// 람다: s -> s.toUpperCase()
List<String> lambdaUpper = names.stream()
    .map(s -> s.toUpperCase())
    .collect(Collectors.toList());

// 메서드 참조: String::toUpperCase
List<String> methodRefUpper = names.stream()
    .map(String::toUpperCase) // String 타입의 각 인스턴스에 toUpperCase()를 적용
    .collect(Collectors.toList());

System.out.println("임의 객체 메서드 참조 결과: " + methodRefUpper); // 출력: [A, B, C]
```

#### 2.4. 생성자 참조 (`ClassName::new`)

람다가 단순히 객체를 생성하는 역할만 할 때 사용합니다.

```java
import java.util.function.Supplier;

// Supplier<MyClass>는 인자가 없고 MyClass를 반환합니다.
// 람다: () -> new String()
Supplier<String> lambdaCreator = () -> new String();

// 메서드 참조: String::new
Supplier<String> methodRefCreator = String::new;
String newString = methodRefCreator.get();
// System.out.println(newString.isEmpty()); // 출력: true
```

## 3. 메서드 참조의 실질적 이점

메서드 참조는 단순히 코드를 짧게 만드는 것 이상의 **표현력**을 제공합니다.

- **가독성 극대화:** 로직을 **함수의 이름**으로 표현하여, 람다의 복잡한 구조 없이 "무엇을 할지"에 대한 의도를 명확하게 드러냅니다. 이는 **선언적인 코드**의 이상에 가깝습니다.
- **유지보수 용이:** 코드의 핵심 동작이 이미 존재하는 메서드에 묶여 있기 때문에, 해당 메서드의 이름을 보면 어떤 동작이 수행될지 즉시 알 수 있습니다.
- **Functional Programming과의 통합:** Stream API와 같은 Functional Programming 구조에서 메서드 참조는 파이프라인의 각 단계를 간결하고 읽기 쉽게 만들어 데이터 처리의 흐름을 명확하게 합니다.
