---
layout: post
title: Java Functional Programming - Stream API 기초 및 활용(filter, map, reduce)
date: 2025-12-08 00:00:00
description: Java Stream API의 중간 연산(filter, map, distinct, sorted)과 최종 연산(forEach,
  reduce, collect)을 실전 예제와 함께 완벽 정리. 함수형 프로그래밍으로 데이터 처리 마스터하기. Java 8 이상 필수 기술
tags: [자바]
keywords: Java Stream API, 자바 스트림, filter 사용법, map 메서드, collect 컬렉터, 함수형 프로그래밍, 중간
  연산, 최종 연산, reduce 예제, Lambda 표현식, Java 8, 지연 연산, Lazy Evaluation, parallelStream,
  병렬 처리, 컬렉션 처리, forEach, distinct, sorted, flatMap, groupingBy, 자바 코드 최적화
categories: [Java]
giscus_comments: true
toc:
  sidebar: left
author: hwangrolee
image:
  path: /assets/img/Java-Functional-Programming-Stream-API-Basics-and-Usage-filter-map-reduce.jpg
  alt: "Java Functional Programming - Stream API 기초 및 활용(filter, map, reduce)"
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

## Stream API란 무엇인가?

Java 8에서 도입된 **Stream API**는 컬렉션 데이터를 함수형 프로그래밍 방식으로 처리할 수 있게 해주는 강력한 도구입니다. 기존의 반복문을 사용하던 명령형 프로그래밍에서 벗어나, 선언적이고 간결한 코드 작성이 가능합니다.

### Stream의 핵심 특징

- **데이터 저장소가 아님**: Stream은 데이터를 저장하지 않고 데이터가 흐르는 파이프라인 역할만 수행합니다
- **함수형 프로그래밍**: 불변성과 순수 함수를 기반으로 동작합니다
- **지연 연산**: 최종 연산이 호출되기 전까지 실제 처리를 미룹니다
- **병렬 처리 지원**: `parallelStream()`으로 멀티코어 활용이 간단합니다
- **일회용**: 한 번 소비된 Stream은 재사용할 수 없습니다

---

## Stream API를 사용해야 하는 이유

### 1. 코드 가독성 향상

**기존 방식 (명령형)**

```java
List<String> result = new ArrayList<>();
for (String name : names) {
    if (name.startsWith("A")) {
        result.add(name.toUpperCase());
    }
}
```

**Stream 방식 (선언형)**

```java
List<String> result = names.stream()
    .filter(name -> name.startsWith("A"))
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 2. 병렬 처리 간편화

단 한 줄의 변경으로 병렬 처리가 가능합니다:

```java
list.parallelStream()
    .filter(...)
    .map(...)
    .collect(Collectors.toList());
```

### 3. 성능 최적화

지연 연산 덕분에 불필요한 계산을 자동으로 스킵하여 효율적입니다.

---

## Stream 생성 방법

### 컬렉션에서 생성

```java
List<String> list = Arrays.asList("Java", "Python", "JavaScript");
Stream<String> stream = list.stream();
```

### 배열에서 생성

```java
String[] array = {"Spring", "React", "Node.js"};
Stream<String> stream = Arrays.stream(array);
```

### 직접 생성

```java
Stream<Integer> stream = Stream.of(1, 2, 3, 4, 5);
```

### 무한 Stream 생성

```java
// 0부터 시작하는 무한 숫자 Stream
Stream<Integer> infinite = Stream.iterate(0, n -> n + 1);

// 랜덤 값 생성
Stream<Double> random = Stream.generate(Math::random);
```

---

## 중간 연산 완벽 정리

중간 연산은 Stream을 반환하므로 체이닝이 가능하며, 지연 실행됩니다.

### filter(): 조건에 맞는 요소만 선택

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// 짝수만 필터링
List<Integer> evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
// 결과: [2, 4, 6, 8, 10]
```

**실전 활용 예제**

```java
List<User> activeAdults = users.stream()
    .filter(user -> user.getAge() >= 18)
    .filter(User::isActive)
    .collect(Collectors.toList());
```

### map(): 요소를 변환

```java
List<String> names = Arrays.asList("alice", "bob", "charlie");

// 모든 이름을 대문자로 변환
List<String> upperNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
// 결과: [ALICE, BOB, CHARLIE]
```

**객체 변환 예제**

```java
List<String> userNames = users.stream()
    .map(User::getName)
    .collect(Collectors.toList());
```

### distinct(): 중복 제거

```java
List<Integer> numbers = Arrays.asList(1, 2, 2, 3, 3, 3, 4, 5);

List<Integer> uniqueNumbers = numbers.stream()
    .distinct()
    .collect(Collectors.toList());
// 결과: [1, 2, 3, 4, 5]
```

### sorted(): 정렬

```java
List<String> fruits = Arrays.asList("Banana", "Apple", "Cherry", "Date");

// 기본 정렬 (오름차순)
List<String> sorted = fruits.stream()
    .sorted()
    .collect(Collectors.toList());
// 결과: [Apple, Banana, Cherry, Date]

// 역순 정렬
List<String> reverseSorted = fruits.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
// 결과: [Date, Cherry, Banana, Apple]
```

**객체 정렬 예제**

```java
List<User> sortedUsers = users.stream()
    .sorted(Comparator.comparing(User::getAge))
    .collect(Collectors.toList());
```

### limit()과 skip(): 개수 제한

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// 처음 5개만
List<Integer> first5 = numbers.stream()
    .limit(5)
    .collect(Collectors.toList());
// 결과: [1, 2, 3, 4, 5]

// 처음 3개 건너뛰고 나머지
List<Integer> skip3 = numbers.stream()
    .skip(3)
    .collect(Collectors.toList());
// 결과: [4, 5, 6, 7, 8, 9, 10]
```

### flatMap(): 중첩 구조 평탄화

```java
List<List<String>> nested = Arrays.asList(
    Arrays.asList("a", "b"),
    Arrays.asList("c", "d", "e")
);

List<String> flattened = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
// 결과: [a, b, c, d, e]
```

---

## 최종 연산 완벽 정리

최종 연산은 Stream 파이프라인을 종료하고 결과를 반환합니다.

### forEach(): 각 요소에 작업 수행

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

names.stream()
    .forEach(name -> System.out.println("Hello, " + name));
// 출력:
// Hello, Alice
// Hello, Bob
// Hello, Charlie
```

### collect(): 컬렉션으로 수집

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

// List로 수집
List<String> list = names.stream()
    .collect(Collectors.toList());

// Set으로 수집
Set<String> set = names.stream()
    .collect(Collectors.toSet());

// Map으로 수집
Map<String, Integer> map = names.stream()
    .collect(Collectors.toMap(
        name -> name,
        String::length
    ));
// 결과: {Alice=5, Bob=3, Charlie=7}
```

**그룹핑 예제**

```java
Map<Integer, List<String>> groupedByLength = names.stream()
    .collect(Collectors.groupingBy(String::length));
// 결과: {3=[Bob], 5=[Alice], 7=[Charlie]}
```

### reduce(): 요소를 하나로 결합

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

// 합계 계산
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b);
// 결과: 15

// 곱셈
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b);
// 결과: 120

// 최댓값 찾기
Optional<Integer> max = numbers.stream()
    .reduce((a, b) -> a > b ? a : b);
// 결과: Optional[5]
```

### count(): 요소 개수 세기

```java
long count = numbers.stream()
    .filter(n -> n > 5)
    .count();
```

### anyMatch(), allMatch(), noneMatch(): 조건 검사

```java
List<Integer> numbers = Arrays.asList(1, 2, 3, 4, 5);

boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0);
// 결과: true

boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0);
// 결과: true

boolean noneNegative = numbers.stream()
    .noneMatch(n -> n < 0);
// 결과: true
```

### findFirst(), findAny(): 요소 찾기

```java
Optional<String> first = names.stream()
    .filter(name -> name.startsWith("A"))
    .findFirst();
// 결과: Optional[Alice]

Optional<String> any = names.parallelStream()
    .filter(name -> name.length() > 3)
    .findAny();
// 병렬 처리에서 가장 빨리 찾은 요소 반환
```

---

## 지연 연산으로 성능 최적화하기

Stream의 중간 연산은 최종 연산이 호출될 때까지 실행되지 않습니다. 이를 **지연 연산(Lazy Evaluation)**이라고 합니다.

### 지연 연산의 장점

```java
List<String> fruits = Arrays.asList("Apple", "Banana", "Cherry", "Date");

fruits.stream()
    .filter(f -> {
        System.out.println("필터링: " + f);
        return f.length() > 5;
    })
    .map(f -> {
        System.out.println("변환: " + f);
        return f.toUpperCase();
    })
    .limit(1)
    .forEach(f -> System.out.println("결과: " + f));

// 출력:
// 필터링: Apple
// 필터링: Banana
// 변환: Banana
// 결과: BANANA
```

`limit(1)`로 인해 첫 번째 조건을 만족하는 요소를 찾으면 즉시 파이프라인이 종료됩니다. "Cherry"와 "Date"는 처리조차 되지 않아 성능이 최적화됩니다.

### Short-Circuit 연산

일부 연산은 모든 요소를 처리하지 않고도 결과를 반환할 수 있습니다:

- `limit(n)`
- `findFirst()`
- `findAny()`
- `anyMatch()`
- `allMatch()`
- `noneMatch()`

---

## 실전 예제 모음

### 예제 1: 사용자 데이터 필터링 및 변환

```java
class User {
    private String name;
    private int age;
    private boolean active;

    // 생성자, getter, setter
}

List<User> users = Arrays.asList(
    new User("Alice", 25, true),
    new User("Bob", 17, true),
    new User("Charlie", 30, false),
    new User("David", 22, true)
);

// 활성 성인 사용자의 이름만 추출
List<String> activeAdultNames = users.stream()
    .filter(user -> user.getAge() >= 18)
    .filter(User::isActive)
    .map(User::getName)
    .collect(Collectors.toList());
// 결과: [Alice, David]
```

### 예제 2: 통계 계산

```java
List<Integer> scores = Arrays.asList(85, 92, 78, 90, 88, 95, 82);

// 평균 계산
double average = scores.stream()
    .mapToInt(Integer::intValue)
    .average()
    .orElse(0.0);

// 총합
int sum = scores.stream()
    .mapToInt(Integer::intValue)
    .sum();

// 최고점
int max = scores.stream()
    .mapToInt(Integer::intValue)
    .max()
    .orElse(0);
```

### 예제 3: 문자열 처리

```java
List<String> sentences = Arrays.asList(
    "Java Stream API",
    "Functional Programming",
    "Lambda Expression"
);

// 모든 단어를 소문자로 변환하고 중복 제거
List<String> uniqueWords = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .map(String::toLowerCase)
    .distinct()
    .sorted()
    .collect(Collectors.toList());
// 결과: [api, expression, functional, java, lambda, programming, stream]
```

### 예제 4: 그룹핑과 집계

```java
class Product {
    private String category;
    private double price;

    // 생성자, getter, setter
}

List<Product> products = Arrays.asList(
    new Product("Electronics", 1000),
    new Product("Electronics", 1500),
    new Product("Books", 20),
    new Product("Books", 35),
    new Product("Clothing", 50)
);

// 카테고리별 평균 가격
Map<String, Double> avgPriceByCategory = products.stream()
    .collect(Collectors.groupingBy(
        Product::getCategory,
        Collectors.averagingDouble(Product::getPrice)
    ));
// 결과: {Electronics=1250.0, Books=27.5, Clothing=50.0}
```

---

## 마치며

Java Stream API는 함수형 프로그래밍의 핵심 도구로, 코드의 가독성과 유지보수성을 크게 향상시킵니다. 중간 연산과 최종 연산의 차이를 이해하고, 지연 연산의 장점을 활용하면 효율적이고 간결한 코드를 작성할 수 있습니다.

### 핵심 요약

- **중간 연산**: filter, map, distinct, sorted 등 - Stream 반환, 체이닝 가능
- **최종 연산**: forEach, collect, reduce 등 - 결과 반환, 파이프라인 종료
- **지연 연산**: 최종 연산 호출 전까지 실제 처리 미룸
- **병렬 처리**: parallelStream()으로 간단하게 멀티코어 활용

Stream API를 마스터하면 더 선언적이고 표현력 있는 Java 코드를 작성할 수 있습니다.
