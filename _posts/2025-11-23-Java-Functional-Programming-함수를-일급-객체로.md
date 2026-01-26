---
layout: post
title: Java Functional Programming - 함수를 일급 객체로 (First-Class Citizen)
date: 2025-11-23 00:00:00
description: Java Functional Programming의 핵심인 '함수 일급 객체' 개념을 완벽하게 이해하세요. 함수를 변수에 할당,
  다른 함수의 인자로 전달, 함수의 결과로 반환하는 3가지 조건을 Java 람다 표현식과 함수형 인터페이스를 통해 구체적인 예시 코드와 함께 설명합니다.
  Stream API와 고차 함수(Higher-Order Function) 활용법을 익혀 선언적인 코드 설계 능력을 강화하십시오.
tags: [자바]
keywords: Java Functional Programming, 일급 객체, First-Class Citizen, 람다 표현식, Lambda
  Expression, 함수형 인터페이스, Functional Interface, 고차 함수, Higher-Order Function, Stream
  API, 동작 파라미터화, OCP, 개방-폐쇄 원칙, 클로저, Closure, Java 8
categories: [Java]
giscus_comments: true
toc:
  sidebar: left
author: hwangrolee
image:
  path: /assets/img/01-04-java-fp-first-class-citizen.png
  alt: "Java Functional Programming - 함수를 일급 객체로 (First-Class Citizen)"
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

함수를 **일급 객체(First-Class Citizen)**로 취급한다는 것은, 프로그램에서 함수를 일반적인 **데이터(값)**와 동일한 위치와 권한을 가진 개체로 다룬다는 Functional Programming의 핵심 원칙입니다. 자바에서는 **함수형 인터페이스**와 **람다 표현식**을 통해 함수를 값처럼 사용하여 **변수에 할당**하고, **다른 함수의 인자로 전달**하며, **함수의 결과로 반환**할 수 있게 됩니다. 이 개념을 이해하면 코드를 **유연한 모듈**로 구성하고, 복잡한 로직을 **선언적인 파이프라인** 형태로 설계할 수 있는 Functional Programming 능력의 기반을 다질 수 있습니다.

---

## 1. 일급 객체의 세 가지 조건과 자바에서의 구현

함수가 일급 객체가 되기 위한 세 가지 핵심 조건이 있으며, Java 8 이후 Functional Programming 요소(함수형 인터페이스와 람다)를 통해 이 조건을 충족합니다.

#### 1.1. 함수를 변수에 할당

함수 그 자체를 변수에 저장할 수 있어야 합니다. 자바에서는 **람다 표현식**을 **함수형 인터페이스 타입의 변수**에 할당함으로써 이를 구현합니다. 이 변수는 이제 해당 람다의 동작(Behavior)을 나타내는 **데이터**로 취급됩니다.

```java
import java.util.function.Function;

// Function<T, R>은 T를 입력받아 R을 반환하는 함수형 인터페이스입니다.
// "문자열을 받아 그 길이를 반환하는 함수"를 lengthFunction 변수에 할당
Function<String, Integer> lengthFunction = s -> s.length();

// 변수에 할당된 함수를 실행 (데이터처럼 사용)
Integer result = lengthFunction.apply("FirstClass");
System.out.println("문자열 길이: " + result); // 출력: 12
```

#### 1.2. 함수를 다른 함수의 인자로 전달

함수가 다른 메서드나 함수의 입력 파라미터로 사용될 수 있어야 합니다. 이는 **동작의 파라미터화(Parameterization of Behavior)**를 가능하게 하며, **고차 함수(Higher-Order Function)**를 구현하는 핵심 요소입니다. Stream API의 `map`, `filter`, `forEach` 메서드가 모두 이 방식을 사용합니다.

```java
import java.util.List;
import java.util.Arrays;
import java.util.function.Predicate;

// 리스트와 '판단 로직'이라는 함수(Predicate)를 인자로 받는 메서드
public static List<Integer> filterNumbers(List<Integer> list, Predicate<Integer> condition) {
    // 람다(condition)를 인자로 받아 내부 로직(filter)에 사용
    return list.stream()
               .filter(condition) // 인자로 전달된 함수를 사용
               .collect(java.util.stream.Collectors.toList());
}

// 활용: 짝수 조건 함수(람다)를 filterNumbers 메서드의 인자로 전달
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6);
List<Integer> evenNumbers = filterNumbers(numbers, n -> n % 2 == 0);
System.out.println("짝수 리스트: " + evenNumbers); // 출력: [2, 4, 6]
```

`filterNumbers` 메서드는 **템플릿 로직**을 제공하고, `Predicate`라는 함수를 인자로 받아 **변화하는 로직(전략)**을 주입받아 유연성을 확보합니다.

#### 1.3. 함수를 함수의 결과로 반환

함수가 다른 메서드의 반환값으로 사용될 수 있어야 합니다. 이는 **함수 팩토리(Function Factory)** 역할을 하며, **클로저(Closure)** 특성을 활용하여 특정 인자나 설정을 기억하는 **특화된 함수**를 생성하는 데 사용됩니다. 이는 **고차 함수**의 또 다른 구현 형태입니다.

```java
import java.util.function.Function;

// 설정값(factor)을 받아 새로운 Function 객체를 반환하는 메서드
public static Function<Integer, Integer> createMultiplier(int factor) {
    // 반환되는 람다는 외부 변수 factor를 캡처(클로저)하여 사용
    return (number) -> number * factor;
}

// 활용:
// 3배를 곱하는 특화된 함수를 생성 (함수 팩토리 역할)
Function<Integer, Integer> triple = createMultiplier(3);

// 5배를 곱하는 특화된 함수를 생성
Function<Integer, Integer> quintuple = createMultiplier(5);

System.out.println("3배 결과: " + triple.apply(10)); // 출력: 30
System.out.println("5배 결과: " + quintuple.apply(10)); // 출력: 50
```

## 2. 일급 객체 개념의 실질적 의미

함수를 일급 객체로 다룬다는 것은 곧 **데이터 추상화** 외에도 **행동(Behavior) 추상화**가 가능하다는 의미입니다.

- **코드 재사용성 극대화:** 로직을 작은 단위 함수로 분리하고, 이 함수들을 데이터처럼 필요한 곳에 전달하거나 조합하여 재사용합니다.
- **유연한 설계:** 핵심 알고리즘(템플릿)과 세부 실행 로직(전략)을 분리하여 시스템의 유연성을 높이고, SOLID 원칙 중 **개방-폐쇄 원칙(OCP)**을 Functional Programming 방식으로 준수하게 됩니다.
- **Functional Programming 파이프라인:** Stream API에서 보듯이, 함수를 인자로 전달하고 반환하는 연쇄적인 구조(파이프라인)를 통해 데이터 흐름 중심의 선언적인 프로그래밍이 가능해집니다.
