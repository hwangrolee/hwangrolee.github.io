---
layout: post
title: "Java Functional Programming - CompletableFuture를 이용한 비동기 파이프라인"
date: 2026-01-31 00:00:00 +0900
categories: [Java]
tags: [자바, 함수형 프로그래밍]
author: hwangrolee
description: "CompletableFuture를 활용하여 Functional Programming의 함수 합성 원리를 비동기 환경에 적용하는 체이닝 기법을 심층적으로 다루어 복잡한 비동기 워크플로우를 구축합니다."
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

Java 8에서 도입된 **CompletableFuture**는 기존의 Future 인터페이스가 가졌던 블로킹(Blocking) 문제와 조합의 어려움을 해결하고, **Functional Programming의 함수 합성(Function Composition) 원리**를 활용하여 복잡한 **비동기 워크플로우**를 유연하게 구축할 수 있도록 설계된 도구입니다. CompletableFuture는 비동기 작업의 결과를 나타내는 **Promise** 객체 역할을 수행하며, 이 객체에 후속 작업을 **체이닝(Chaining)** 방식으로 연결하여 비동기 파이프라인을 구성합니다.

## 왜 CompletableFuture와 체이닝 기법을 학습해야 하는가

현대적인 엔터프라이즈 환경에서는 응답 시간을 최소화하기 위해 **I/O 작업(네트워크 통신, 파일 접근 등)**을 비동기적으로 처리하는 것이 필수적입니다. 기존의 멀티스레딩 방식은 스레드 관리와 동기화 문제로 인해 복잡도가 높았습니다. CompletableFuture는 이러한 비동기 작업을 **선언적인 Functional Programming 스타일**로 처리할 수 있게 해줍니다.

CompletableFuture의 체이닝 기법을 사용하면, 비동기 작업의 논리를 순차적이고 읽기 쉬운 파이프라인 형태로 정의할 수 있습니다. 각 단계의 함수는 **순수 함수(Pure Function)**의 형태를 띠며, 이는 복잡한 비즈니스 로직을 작고 독립적인 단위로 분리하고 연결함으로써 **코드의 가독성, 유지보수성, 그리고 예측 가능성**을 획기적으로 향상시킵니다. 따라서 고성능, 비동기 시스템을 설계하고 구축하기 위해서는 이 체이닝 기법을 능숙하게 다루어야 합니다.

## 비동기 파이프라인 구축을 위한 체이닝 메서드

CompletableFuture는 Functional Programming의 표준 함수형 인터페이스(Function, Consumer, Runnable)를 인수로 받아 후속 작업을 연결하는 다양한 메서드를 제공합니다.

### 1. 결과 변환: thenApply()

thenApply() 메서드는 이전 비동기 작업의 결과를 입력으로 받아 **새로운 값으로 동기적 변환**을 수행합니다. 인수로 **Function** 인터페이스를 사용합니다.

```java
CompletableFuture<String> initialTask = CompletableFuture.supplyAsync(() -> {
    System.out.println("Step 1: Raw data fetched (Thread: " + Thread.currentThread().getName() + ")");
    return "raw_data_123";
});

// Step 2: raw_data_123을 대문자로 변환
CompletableFuture<Integer> transformation = initialTask
    .thenApply(raw -> {
        System.out.println("Step 2: Transforming data (Thread: " + Thread.currentThread().getName() + ")");
        return raw.toUpperCase().length();
    });
```

### 2. 비동기 합성 (FlatMap): thenCompose()

thenCompose() 메서드는 이전 작업의 결과를 받아 **새로운 CompletableFuture를 반환**합니다. 이는 **비동기 작업의 파이프라인을 평탄화(Flattening)**하는 역할을 하며, Functional Programming의 **flatMap**과 동일한 개념으로 이해할 수 있습니다. 이를 통해 중첩된 Future<Future<T>> 구조를 방지하고 깔끔한 체인을 유지할 수 있습니다.

```java
// 비동기적으로 사용자 ID를 조회하는 가상의 함수
public CompletableFuture<String> fetchUserIdAsync() {
    return CompletableFuture.supplyAsync(() -> "user_456");
}

// 비동기적으로 해당 ID의 상세 정보를 조회하는 가상의 함수
public CompletableFuture<String> fetchUserDetailsAsync(String userId) {
    return CompletableFuture.supplyAsync(() -> "Details for " + userId);
}

CompletableFuture<String> chainedResult = fetchUserIdAsync()
    // Step 2: 첫 번째 결과(userId)를 사용하여 두 번째 비동기 작업(fetchUserDetailsAsync)을 실행하고 연결합니다.
    .thenCompose(this::fetchUserDetailsAsync); 
    // 결과 타입은 CompletableFuture<String>으로 평탄화됩니다.
```

### 3. 두 결과 결합: thenCombine()

thenCombine() 메서드는 **서로 독립적인 두 개의 비동기 작업**의 결과가 모두 완료되었을 때, 두 결과를 합쳐 하나의 새로운 결과로 만드는 데 사용됩니다. 인수로 **BiFunction** 인터페이스를 사용합니다.

```java
CompletableFuture<Double> fetchPrice = CompletableFuture.supplyAsync(() -> 100.0);
CompletableFuture<Double> fetchTaxRate = CompletableFuture.supplyAsync(() -> 0.15);

// 두 작업이 병렬로 실행되며, 모두 완료되면 최종 가격을 계산합니다.
CompletableFuture<Double> finalPrice = fetchPrice
    .thenCombine(fetchTaxRate, (price, tax) -> {
        System.out.println("Step 3: Combining results.");
        return price * (1 + tax);
    });
```

fetchPrice와 fetchTaxRate는 서로 병렬적으로 실행되므로 전체 처리 시간이 단축되며, 최종적으로 두 결과가 **순수 함수**인 BiFunction에 입력되어 결합됩니다.

## 명령형 코드 대비 구조적 장점

명령형(Imperative) 방식으로 비동기 작업을 처리할 경우, 각 작업의 완료를 기다리기 위해 블로킹 호출이나 복잡한 콜백 중첩이 발생합니다. 이는 코드의 가독성을 저해하고, 에러 처리 로직이 분산되어 유지보수가 어려워집니다.

반면 CompletableFuture의 체이닝 방식은 다음과 같은 장점을 제공합니다:

1. **선언적 파이프라인**: 각 단계가 명확하게 분리되어 있어 비즈니스 로직의 흐름을 직관적으로 파악할 수 있습니다.
2. **병렬 처리 용이성**: thenCombine()과 같은 메서드를 통해 독립적인 작업을 자동으로 병렬화하여 성능을 최적화할 수 있습니다.
3. **테스트 용이성**: 각 단계의 함수가 순수 함수로 작성되면, 단위 테스트를 독립적으로 수행할 수 있습니다.

## 스레드 관리와 순수 함수의 중요성

CompletableFuture에서 supplyAsync()나 runAsync()와 같이 비동기 작업을 시작하는 메서드는 기본적으로 **ForkJoinPool**의 공통 풀(Common Pool)을 사용합니다. thenApply()나 thenCombine()과 같은 후속 작업들은 이전 작업과 같은 스레드에서 실행되거나, 별도의 스레드 풀에서 실행될 수 있습니다.

이러한 **동시성 환경**에서는 **부수 효과(Side Effect)가 없는 순수 함수**를 사용하는 것이 절대적으로 중요합니다. 체이닝에 사용되는 모든 람다는 외부에 공유된 가변 상태를 변경해서는 안 됩니다. 만약 체인 내에서 부수 효과가 발생하면, 여러 스레드가 동시에 실행되는 과정에서 **데이터 불일치**나 **경합 조건(Race Condition)**이 발생하여 예측 불가능한 결과를 초래할 수 있습니다.

```java
// 잘못된 예시: 외부 상태를 변경하는 부수 효과
List<String> sharedList = new ArrayList<>(); // 가변 상태

CompletableFuture.supplyAsync(() -> "data1")
    .thenApply(data -> {
        sharedList.add(data); // 부수 효과 발생 - 스레드 안전하지 않음
        return data.toUpperCase();
    });
```

위 코드는 여러 스레드가 동시에 sharedList를 수정할 수 있어 ConcurrentModificationException이나 데이터 손실이 발생할 수 있습니다.

```java
// 올바른 예시: 순수 함수 사용
CompletableFuture<String> result = CompletableFuture.supplyAsync(() -> "data1")
    .thenApply(data -> data.toUpperCase()) // 순수 함수 - 외부 상태 변경 없음
    .thenApply(upper -> upper + "_processed");
```

CompletableFuture는 Functional Programming의 원칙을 지킬 때만 **스레드 안전성**과 **예측 가능성**을 보장하며 고성능을 발휘할 수 있습니다.

## 복잡한 비동기 워크플로우 구축 예시

실제 프로젝트에서는 여러 외부 서비스 호출을 조합하여 최종 결과를 생성하는 경우가 많습니다. 다음은 사용자 정보 조회, 주문 내역 조회, 추천 상품 조회를 병렬로 수행한 후 결과를 통합하는 예시입니다.

```java
public class OrderService {
    
    public CompletableFuture<UserProfile> fetchUserProfile(String userId) {
        return CompletableFuture.supplyAsync(() -> {
            // 외부 API 호출 시뮬레이션
            return new UserProfile(userId, "John Doe", "Premium");
        });
    }
    
    public CompletableFuture<List<Order>> fetchOrderHistory(String userId) {
        return CompletableFuture.supplyAsync(() -> {
            // 데이터베이스 조회 시뮬레이션
            return Arrays.asList(
                new Order("ORD001", 150.0),
                new Order("ORD002", 200.0)
            );
        });
    }
    
    public CompletableFuture<List<Product>> fetchRecommendations(String userTier) {
        return CompletableFuture.supplyAsync(() -> {
            // 추천 엔진 호출 시뮬레이션
            if ("Premium".equals(userTier)) {
                return Arrays.asList(new Product("Premium Item 1"), new Product("Premium Item 2"));
            }
            return Arrays.asList(new Product("Standard Item 1"));
        });
    }
    
    public CompletableFuture<DashboardData> buildUserDashboard(String userId) {
        CompletableFuture<UserProfile> profileFuture = fetchUserProfile(userId);
        CompletableFuture<List<Order>> ordersFuture = fetchOrderHistory(userId);
        
        // 사용자 프로필을 먼저 가져온 후, 해당 등급에 맞는 추천 상품을 조회
        CompletableFuture<List<Product>> recommendationsFuture = profileFuture
            .thenCompose(profile -> fetchRecommendations(profile.getTier()));
        
        // 세 가지 비동기 작업의 결과를 모두 결합
        return profileFuture
            .thenCombine(ordersFuture, (profile, orders) -> new PartialDashboard(profile, orders))
            .thenCombine(recommendationsFuture, (partial, recommendations) -> 
                new DashboardData(partial.getProfile(), partial.getOrders(), recommendations)
            );
    }
}
```

이 예시에서 fetchUserProfile, fetchOrderHistory, fetchRecommendations는 모두 독립적으로 실행되며, thenCompose와 thenCombine을 통해 **불변성(Immutability)**을 유지하면서 결과를 조합합니다. 각 단계의 람다 표현식은 순수 함수로 작성되어 있어, 외부 상태를 변경하지 않고 입력값만을 기반으로 출력을 생성합니다.

## 에러 처리와 복구 전략

CompletableFuture는 exceptionally()와 handle() 메서드를 통해 에러 처리를 체이닝 방식으로 통합할 수 있습니다.

```java
CompletableFuture<String> resilientPipeline = CompletableFuture
    .supplyAsync(() -> {
        if (Math.random() > 0.5) {
            throw new RuntimeException("External service failed");
        }
        return "Success Data";
    })
    .exceptionally(ex -> {
        System.err.println("Error occurred: " + ex.getMessage());
        return "Fallback Data"; // 기본값 반환
    })
    .thenApply(data -> data.toUpperCase());
```

exceptionally()는 이전 단계에서 예외가 발생했을 때만 실행되며, 순수 함수 형태로 대체 값을 반환합니다. 이를 통해 전체 파이프라인의 흐름을 중단하지 않고 복구할 수 있습니다.

## 지연 평가와 성능 최적화

CompletableFuture는 **지연 평가(Lazy Evaluation)** 특성을 활용하여 불필요한 연산을 방지할 수 있습니다. 예를 들어, thenCompose()를 사용하면 이전 작업이 완료될 때까지 다음 작업이 시작되지 않으므로, 조건부 로직을 효율적으로 구현할 수 있습니다.

```java
CompletableFuture<String> conditionalPipeline = CompletableFuture
    .supplyAsync(() -> fetchUserTier())
    .thenCompose(tier -> {
        if ("Premium".equals(tier)) {
            return fetchPremiumContent(); // Premium 사용자만 실행
        } else {
            return CompletableFuture.completedFuture("Standard Content");
        }
    });
```

이 방식은 명령형 코드에서 if-else 블록을 사용하는 것보다 **고차 함수(Higher-Order Function)**의 조합으로 로직을 표현하여 가독성과 재사용성을 높입니다.

## 최종 학습 성과

이 문서를 통해 학습한 원리를 응용하면, 최소한 50줄 이상의 복잡한 비동기 데이터 변환 로직을 CompletableFuture와 람다식을 통해 **부수 효과 없이** 작성할 수 있는 지식 기반을 확보할 수 있습니다. 순수 함수, 불변성, 함수 합성의 원칙을 준수하면서 고성능 비동기 시스템을 설계하고 구축하는 능력을 갖추게 됩니다.