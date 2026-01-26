---
layout: post
title: Java Functional Programming - Java record 활용
date: 2025-12-05 00:00:00
description: Java record를 활용하여 불변(Immutable) 데이터 모델을 효율적으로 정의하는 방법을 학습하십시오. record가
  보일러플레이트 코드를 제거하고, Functional Programming의 핵심인 불변성을 어떻게 보장하는지 설명합니다. Stream API와
  메서드 참조를 이용해 record 객체를 안전하게 활용하고 데이터 중심의 선언적 코드를 작성하는 실용적인 Java 예시를 제공합니다.
tags: [자바]
keywords: Java record, Java 16, 불변성, Immutability, Functional Programming, 보일러플레이트
  제거, Stream API, 메서드 참조, Canonical Constructor, 데이터 모델, 스레드 안전성, Thread Safety
categories: [Java]
giscus_comments: true
toc:
  sidebar: left
author: hwangrolee
image:
  path: /assets/img/02-03-java-fp-java-record.png
  alt: "Java Functional Programming - Java record 활용"
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

Java `record`는 **불변(Immutable) 데이터 모델**을 정의하는 과정을 획기적으로 단순화하여, 데이터 중심의 코드를 작성하는 Functional Programming 스타일을 Java 환경에 깊숙이 통합하는 핵심 요소입니다. 이 문법은 데이터를 표현하는 클래스가 지녀야 할 핵심 속성(상태)만을 명시하고, 나머지 보일러플레이트(boilerplate) 코드를 컴파일러가 자동으로 생성해 줌으로써 개발자의 부담을 줄여줍니다.

### 왜 `record`를 학습하고 활용해야 하는가

Functional Programming 패러다임에서 **불변성**은 프로그램의 예측 가능성과 스레드 안전성을 보장하는 가장 중요한 원칙입니다. 기존 Java에서 불변 클래스를 설계하려면 다음과 같은 요소를 반드시 구현해야 했습니다. 모든 필드를 `private final`로 선언하고, 모든 인수를 받는 생성자(Constructor), 필드 접근자(`getter`), 그리고 객체 비교 및 해시를 위한 `equals()`와 `hashCode()`, 디버깅을 위한 `toString()` 메서드를 수동으로 작성하거나 IDE의 도움을 받아야 했습니다.

이러한 수많은 반복적인 코드(보일러플레이트)는 가독성을 저해하고 유지보수 시 실수를 유발하는 원인이 되었습니다. `record`는 단 한 줄의 선언만으로 이 모든 필수 요소들을 **불변성의 원칙에 맞게 자동 생성**해 줍니다. 따라서 코드가 극도로 간결해지며, 정의된 **상태(State)**와 **행위(Behavior)**를 분리하는 Functional Programming의 목표에 완벽하게 부합하게 됩니다.

### `record`를 사용한 불변 데이터 모델 정의

`record`는 데이터 컨테이너 역할을 수행하는 클래스를 명시적으로 정의하며, 필드(Field)가 아닌 **컴포넌트(Component)** 목록을 선언합니다. 컴파일러는 이 목록을 기반으로 다음과 같은 코드를 자동으로 생성합니다.

1. 모든 컴포넌트를 `private final` 필드로 선언합니다. (불변성 확보)
2. 모든 컴포넌트를 인수로 받는 **정식 생성자(Canonical Constructor)**를 생성합니다.
3. 모든 컴포넌트에 대한 접근자 메서드(Accessors), 즉 `getter` 역할을 하는 메서드를 **컴포넌트 이름과 동일하게** 생성합니다.
4. `equals()`, `hashCode()`, `toString()` 메서드를 컴포넌트 전체를 기반으로 자동으로 오버라이딩합니다.

### **코드 예시: `record`를 사용한 불변 객체 정의**

고객의 주문 정보를 담는 불변 객체를 정의하는 예시입니다.

```java
// 기존 클래스 방식에서는 수십 줄의 코드가 필요했던 부분을 한 줄로 정의합니다.
public record OrderItem(String productName, int quantity, double price) {

    // 정식 생성자 (Canonical Constructor)의 압축형을 사용하여 유효성 검증 로직을 추가할 수 있습니다.
    public OrderItem {
        if (quantity <= 0) {
            throw new IllegalArgumentException("Quantity must be greater than zero.");
        }
        if (price <= 0) {
            throw new IllegalArgumentException("Price must be greater than zero.");
        }
    }

    // record는 불변 객체이므로, 상태를 변경하는 대신 새로운 객체를 반환하는 행위를 추가할 수 있습니다.
    public double getTotalPrice() {
        return quantity * price;
    }
}
```

### **코드 예시: `record` 객체의 활용 및 Functional Programming 요소와의 통합**

`record`로 정의된 불변 객체는 `Stream API`와 같은 Functional Programming 도구와 결합할 때 그 진가를 발휘합니다.

```java
import java.util.List;
import java.util.stream.Collectors;

// OrderItem record는 불변이므로, 스트림 파이프라인에서 안전하게 사용됩니다.
List<OrderItem> items = List.of(
    new OrderItem("Laptop", 1, 1200.00),
    new OrderItem("Mouse", 2, 25.00),
    new OrderItem("Keyboard", 1, 75.00)
);

// 1. Stream API를 사용하여 총 금액을 계산합니다.
double totalOrderPrice = items.stream()
    .mapToDouble(OrderItem::getTotalPrice) // 메서드 참조 (Method Reference) 사용
    .sum();

System.out.println("Total Order Price: " + totalOrderPrice);

// 2. map 연산을 통해 불변 객체를 새로운 불변 객체로 변환합니다. (불변성 유지)
// 모든 상품 가격을 10% 인상한 새로운 리스트를 생성합니다.
List<OrderItem> updatedItems = items.stream()
    .map(item -> new OrderItem(
        item.productName(),
        item.quantity(),
        item.price() * 1.10) // 새로운 record 객체 생성
    )
    .collect(Collectors.toList());

System.out.println("Updated Prices: " + updatedItems);
```

이 예시에서 `record`는 `mapToDouble(OrderItem::getTotalPrice)`와 같은 **메서드 참조**를 통해 간결하게 사용됩니다. 또한, 데이터를 수정하는 대신 `map` 연산을 통해 **새로운 `OrderItem` 객체를 생성**하는 방식은 데이터를 변경하지 않는(Non-Mutating) Functional Programming의 핵심 원칙인 **불변성**을 명확하게 실현합니다. `record`는 이러한 불변 데이터 중심의 프로그래밍 스타일을 Java에서 가장 효율적으로 지원하는 표준 문법입니다.
