---
layout: post
title: "Java Functional Programming - 익명 클래스에서 메서드 참조까지: 코드를 극도로 간결하게 만드는 리팩토링 가이드"
date: 2026-01-03 00:00:00
description: "자바 8의 핵심인 람다 표현식과 메서드 참조를 활용해 장황한 익명 클래스를 간결한 함수형 코드로 리팩토링하는 과정을 단계별로 알아봅니다. 가독성을 높이고 부수 효과(Side Effect)를 줄이는 모던 자바 프로그래밍의 기초를 확실하게 다져보세요."
tags: 자바
keywords: JavaFunctionalProgramming, 자바 람다 표현식, 자바 메서드 참조, Java 8, 함수형 프로그래밍, 코드 리팩토링, 익명 클래스, Stream API, 클린 코드, 자바 튜토리얼, 백엔드 개발
categories: 자바
giscus_comments: true
toc:
  sidebar: left
featured: true
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

자바의 람다 표현식(Lambda Expression)과 메서드 참조(Method Reference)는 단순한 문법적 설탕(Syntactic Sugar)을 넘어, 자바 언어가 객체지향의 경계를 넘어 Functional Programming 패러다임으로 확장되었음을 의미합니다. 기존의 명령형 프로그래밍(Imperative Programming) 방식이 '어떻게(How)' 데이터를 처리할지에 집중했다면, 이 새로운 도구들은 '무엇을(What)' 처리할지에 집중하게 함으로써 코드의 표현력을 극대화합니다. 이 글에서는 익명 클래스에서 람다로, 그리고 람다에서 메서드 참조로 이어지는 리팩토링 과정을 통해 Functional Programming의 핵심 철학을 이해하고 실무에 적용하는 방법을 다룹니다.

### Functional Programming 학습의 필요성 및 핵심 개념

현대의 애플리케이션은 복잡한 데이터 변환과 병렬 처리를 요구합니다. 기존의 명령형 코드는 상태를 지속적으로 변경하며 루프와 조건문이 뒤섞여 있어 가독성이 떨어지고, 멀티스레드 환경에서 **부수 효과(Side Effect)**로 인한 버그 발생 위험이 높습니다.

반면, 람다와 메서드 참조를 활용한 Functional Programming 스타일은 다음과 같은 이점을 제공합니다.

- **불변성(Immutability)과 순수 함수(Pure Function):** 데이터 자체를 변경하지 않고 새로운 데이터를 반환하는 방식은 스레드 안전성(Thread-safety)을 보장합니다. 입력이 같으면 결과가 항상 같은 순수 함수를 지향하여 예측 가능한 코드를 작성할 수 있습니다.
- **고차 함수(Higher-Order Function)의 활용:** 함수(동작)를 파라미터로 전달하거나 반환값으로 다룸으로써, 비즈니스 로직을 유연하게 조립할 수 있습니다.
- **지연 평가(Lazy Evaluation):** Stream API와 결합 시, 최종 연산이 호출될 때까지 실제 계산을 미루어 불필요한 연산을 제거하고 성능을 최적화합니다.

### 1. 익명 클래스에서 람다 표현식으로의 전환: 동작의 파라미터화

Java 8 이전에는 특정 동작을 다른 메서드에 전달하기 위해 **익명 클래스(Anonymous Class)**를 사용했습니다. 이는 불필요한 클래스 정의와 메서드 선언을 포함하여 코드를 장황하게 만들었습니다. 람다 표현식은 오직 핵심 로직인 '동작' 그 자체에만 집중합니다.

#### 1.1. 리팩토링 전: 익명 클래스 사용 (명령형 스타일의 잔재)

기존 방식에서는 `Runnable`이나 `Comparator` 같은 함수형 인터페이스(추상 메서드가 하나인 인터페이스)를 구현하기 위해 보일러플레이트 코드가 필수적이었습니다.

```java
// 익명 클래스 사용 예시: 문자열 길이로 정렬
import java.util.Arrays;
import java.util.Comparator;
import java.util.List;

List<String> words = Arrays.asList("apple", "banana", "kiwi");

words.sort(new Comparator<String>() {
    @Override
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});

```

#### 1.2. 리팩토링 후: 람다 표현식 적용

람다 표현식은 컴파일러가 문맥을 통해 타입을 추론(Type Inference)하므로, 불필요한 선언부를 제거할 수 있습니다. 이는 코드를 **선언적(Declarative)**으로 만듭니다.

```java
// 람다 표현식 적용: (인자) -> { 바디 }
// 타입 선언, 메서드 명, return 키워드, 중괄호 생략 가능
words.sort((s1, s2) -> Integer.compare(s1.length(), s2.length()));

```

이 변화는 단순한 코드 단축이 아닙니다. 개발자가 "Comparator를 익명 클래스로 만들어서 오버라이드 한다"는 구현 세부 사항에서 벗어나, "두 문자열의 길이를 비교한다"는 **의도**에 집중하게 합니다.

### 2. 람다 표현식에서 메서드 참조로의 전환: 가독성의 극대화

람다 표현식의 본문이 단순히 다른 메서드 하나를 호출하는 것으로 끝난다면, **메서드 참조(Method Reference)**를 사용하여 이를 더 명확하게 표현할 수 있습니다. `::` 연산자를 사용하며, 이는 해당 메서드를 직접 호출하는 것이 아니라 메서드 자체를 함수 객체로 전달하는 것입니다.

#### 2.1. 정적 메서드 참조 (Static Method Reference)

람다가 `ClassName.staticMethod(args)` 형태일 때 사용합니다.

```java
import java.util.function.Function;

// 람다 표현식
Function<String, Integer> lambdaParser = s -> Integer.parseInt(s);

// 메서드 참조: Integer 클래스의 파싱 기능을 직접 참조
Function<String, Integer> methodRefParser = Integer::parseInt;

```

#### 2.2. 특정 타입의 임의 객체 메서드 참조 (Instance Method Reference of an Arbitrary Object)

Stream API에서 가장 빈번하게 사용되는 패턴입니다. 첫 번째 인자가 메서드의 수신자(Receiver)가 되고, 나머지 인자가 메서드의 인자로 전달되는 형태입니다.

```java
import java.util.List;
import java.util.stream.Collectors;

// 람다: 각 요소 s에 대해 s.toUpperCase() 수행
// words.stream().map(s -> s.toUpperCase()).collect(Collectors.toList());

// 메서드 참조: String 클래스에 정의된 toUpperCase를 각 요소에 적용하라는 의미
List<String> upperCaseWords = words.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

```

#### 2.3. 생성자 참조 (Constructor Reference)

객체 생성을 지연시키거나 팩토리 패턴을 구현할 때 유용합니다.

```java
import java.util.function.Supplier;
import java.util.ArrayList;
import java.util.List;

// 람다: () -> new ArrayList<>()
// 생성자 참조
Supplier<List<String>> listSupplier = ArrayList::new;
List<String> newList = listSupplier.get();

```

### 3. 실전 시나리오: 명령형 코드 vs Functional Programming 코드

실제 비즈니스 로직에서 Functional Programming 스타일이 주는 구조적 이점과 불변성 유지를 비교해 봅니다.

**시나리오:** `Order` 객체 리스트에서 상태가 'DELIVERED'인 주문만 필터링하고, 해당 주문들의 총 가격(Total Price)을 계산하여 반환해야 합니다. `Order` 객체는 불변 객체라고 가정합니다.

#### 3.1. 기존 명령형 접근 (Imperative Approach)

이 방식은 `total`이라는 외부 변수의 상태를 계속 변경(Mutation)합니다. 루프 내부 로직이 복잡해질수록 변수의 상태 추적이 어려워집니다.

```java
public double calculateTotalRevenueImperative(List<Order> orders) {
    double total = 0.0; // 상태 변경이 일어나는 변수
    for (Order order : orders) {
        if ("DELIVERED".equals(order.getStatus())) {
            total += order.getPrice();
        }
    }
    return total;
}

```

#### 3.2. Functional Programming 접근 (Declarative Approach)

Stream API와 메서드 참조를 활용하여 **파이프라인**을 구성합니다.

```java
import java.math.BigDecimal;

public BigDecimal calculateTotalRevenueFunctional(List<Order> orders) {
    return orders.stream()
        .filter(order -> "DELIVERED".equals(order.getStatus())) // 데이터 필터링 (중간 연산)
        .map(Order::getPrice)       // 데이터 매핑: 메서드 참조 활용 (중간 연산)
        .reduce(BigDecimal.ZERO, BigDecimal::add); // 리덕션: 초기값과 누적 연산 (최종 연산)
}

```

**분석 및 이점:**

1. **가독성:** `filter`, `map`, `reduce`라는 동사를 통해 코드의 흐름이 영어 문장처럼 읽힙니다. (`DELIVERED`인 주문을 필터링하고, 가격을 추출하여, 합산한다.)
2. **부수 효과 제거 (No Side Effects):** 외부 변수(`total`)를 수정하지 않고 새로운 결과값을 생성합니다. 이는 병렬 스트림(`parallelStream()`)으로 전환할 때 별도의 동기화 처리 없이도 안전하게 동작함을 보장합니다.
3. **유지보수성:** 가격 추출 로직이나 합산 로직이 변경되더라도 `map`이나 `reduce`의 인자(함수)만 교체하면 되므로 유연합니다.

### 결론

람다 표현식과 메서드 참조는 자바 코드를 간결하게 만드는 것을 넘어, 프로그램을 **값의 변환과 흐름**으로 바라보게 하는 Functional Programming의 핵심 도구입니다. 익명 클래스의 장황함을 람다로, 람다의 반복을 메서드 참조로 정제하는 과정은 곧 불변성을 지키고 부수 효과를 제어하는 견고한 소프트웨어 아키텍처로 나아가는 첫걸음입니다.
