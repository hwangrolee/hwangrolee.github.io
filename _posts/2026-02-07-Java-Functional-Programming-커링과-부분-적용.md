---
layout: post
title: "Java Functional Programming - 커링(Currying)과 부분 적용: 다중 인자 함수를 단일 인자 함수들의 연속으로 변환하는 기법"
date: 2026-02-07 00:00:00 +0900
categories: [Java]
tags: [자바, 함수형 프로그래밍]
author: hwangrolee
description: "커링과 부분 적용을 통해 다중 인자 함수를 단일 인자 함수의 연속으로 변환하고, 특정 인자를 고정하여 재사용 가능한 전문화된 함수를 생성하는 방법을 학습합니다."
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

커링(Currying)과 부분 적용(Partial Application)은 다중 인자를 받는 함수를 **단일 인자를 받는 함수들의 연속**으로 변환하는 Functional Programming 기법입니다. 이 두 기법은 함수를 유연하게 재사용하고, 필요한 인자만 미리 고정하여 **특정 목적에 맞는 전문화된 함수**를 생성함으로써 코드의 재사용성을 극대화합니다. 이는 복잡한 로직을 작은 단위로 쪼개고 조합하여 시스템을 구축하는 Functional Programming 설계 원칙을 따르는 데 핵심적인 기술입니다.

## 학습 동기: 왜 커링과 부분 적용을 배워야 하는가

명령형(Imperative) 프로그래밍에서는 동일한 로직을 반복적으로 작성하거나, 매번 모든 인자를 전달해야 하는 불편함이 존재합니다. 커링과 부분 적용을 활용하면 **고차 함수(Higher-Order Function)**를 통해 함수의 일부 인자를 미리 설정하고, 나머지 인자만 받는 새로운 함수를 생성할 수 있습니다. 이는 다음과 같은 실질적인 이점을 제공합니다.

- **코드 재사용성 향상**: 공통 로직을 함수로 추상화하고, 특정 상황에 맞게 인자를 고정하여 재사용 가능한 유틸리티 함수를 생성할 수 있습니다.
- **불변성(Immutability) 증진**: 함수형 접근 방식을 통해 상태 변경 없이 새로운 함수를 생성하므로, 부수 효과(Side Effect)를 최소화합니다.
- **테스트 용이성**: 순수 함수(Pure Function)로 설계된 커링 함수는 입력에 대해 항상 동일한 출력을 보장하므로, 단위 테스트 작성이 용이합니다.
- **함수 조합(Function Composition) 강화**: 단일 인자 함수들의 연속으로 변환된 커링 함수는 다른 함수와 조합하기 쉬워, 복잡한 데이터 파이프라인 구축에 유리합니다.

명령형 코드에서는 동일한 설정을 반복적으로 전달해야 하지만, 커링과 부분 적용을 사용하면 설정을 한 번만 정의하고 이를 재사용할 수 있어 코드의 간결성과 유지보수성이 크게 향상됩니다.

---

## 1. 커링(Currying)의 개념과 구현

**커링**은 여러 인자를 받는 함수를 **인자를 하나만 받는 함수들의 연쇄적인 호출**로 변환하는 과정을 의미합니다. 함수가 하나의 인자를 받으면, 다음 인자를 받을 새로운 함수를 반환하고, 이 과정이 모든 인자가 채워질 때까지 반복됩니다.

### 커링의 기본 구조

세 개의 인자를 받는 함수가 있다고 가정할 때, 커링을 적용하면 다음과 같은 형태로 변환됩니다.

**일반 함수**: `f(a, b, c)`  
**커링 적용**: `f(a)(b)(c)`

자바에서는 `java.util.function.Function`을 중첩하여 이를 구현합니다.

```java
import java.util.function.Function;

public class CurryingExample {
    
    // 커링 적용 전: 세 인자를 모두 받는 일반 함수
    public static Integer add(int a, int b, int c) {
        return a + b + c;
    }

    // 커링 적용 후: Integer를 받고 -> Function<Integer, Function<Integer, Integer>>를 반환
    public static Function<Integer, Function<Integer, Function<Integer, Integer>>> curriedAdd() {
        // a를 받는다.
        return a -> 
            // b를 받는다.
            b -> 
                // c를 받고 최종 결과를 반환한다.
                c -> a + b + c;
    }

    public static void main(String[] args) {
        // 활용: 인자를 하나씩 순차적으로 전달
        Integer result = curriedAdd().apply(1).apply(2).apply(3); // 1 + 2 + 3 = 6
        System.out.println("커링 적용 결과: " + result); // 출력: 6
    }
}
```

### 커링의 구조적 장점

커링은 함수를 **단일 인자 함수의 연속**으로 변환함으로써, 각 단계에서 부분 적용이 가능하도록 만듭니다. 이는 함수형 프로그래밍의 핵심 원칙인 **함수 조합**과 **지연 평가(Lazy Evaluation)**를 가능하게 하며, 필요한 시점에 인자를 전달하여 최종 결과를 계산할 수 있습니다.

---

## 2. 부분 적용(Partial Application)의 개념과 활용

**부분 적용**은 **다중 인자 함수**나 **커링된 함수**에 **일부 인자만** 미리 전달하여, 나머지 인자를 기다리는 **새로운 함수**를 반환하는 기법입니다. 이는 특정 인자들을 고정하여 특화된 함수를 생성하는 데 사용됩니다.

### 커링과 부분 적용의 차이점

- **커링**: 함수를 "인자가 하나인 연속 함수"로 **변환**하는 기법 자체를 의미합니다.
- **부분 적용**: 변환된 함수(또는 일반 함수)에 **일부 인자만 주입**하여 재사용 가능한 함수를 **생성**하는 활용 방식을 의미합니다.

### 부분 적용을 통한 함수 특화

부분 적용의 실질적인 이점은 **특정 인자 값이 자주 사용될 때** 그 인자를 고정하여 새로운 유틸리티 함수를 만들 수 있다는 것입니다.

```java
import java.util.function.Function;

public class PartialApplicationExample {
    
    // curriedAdd 함수 정의
    public static Function<Integer, Function<Integer, Function<Integer, Integer>>> curriedAdd() {
        return a -> b -> c -> a + b + c;
    }

    public static void main(String[] args) {
        Function<Integer, Function<Integer, Function<Integer, Integer>>> addCurried = curriedAdd();

        // 인자 '1'을 첫 번째 인자로 고정 (부분 적용)
        Function<Integer, Function<Integer, Integer>> addOne = addCurried.apply(1);

        // 인자 '2'를 두 번째 인자로 고정 (추가 부분 적용)
        Function<Integer, Integer> addOneAndTwo = addOne.apply(2); 

        // 이제 addOneAndTwo는 세 번째 인자만 받으면 되는 특화된 함수가 됨
        Integer specializedResult = addOneAndTwo.apply(10); // 1 + 2 + 10 = 13
        System.out.println("부분 적용 결과: " + specializedResult); // 출력: 13
        
        // 다양한 값으로 재사용 가능
        System.out.println("부분 적용 결과: " + addOneAndTwo.apply(20)); // 출력: 23
        System.out.println("부분 적용 결과: " + addOneAndTwo.apply(30)); // 출력: 33
    }
}
```

### 명령형 코드 대비 구조적 장점

명령형 프로그래밍에서는 동일한 설정을 반복적으로 전달해야 합니다.

```java
// 명령형 방식: 매번 모든 인자를 전달
int result1 = add(1, 2, 10);
int result2 = add(1, 2, 20);
int result3 = add(1, 2, 30);
```

반면, 부분 적용을 사용하면 공통 인자를 한 번만 설정하고 재사용할 수 있습니다.

```java
// 함수형 방식: 공통 인자를 고정하고 재사용
Function<Integer, Integer> addOneAndTwo = curriedAdd().apply(1).apply(2);
int result1 = addOneAndTwo.apply(10);
int result2 = addOneAndTwo.apply(20);
int result3 = addOneAndTwo.apply(30);
```

이러한 접근 방식은 **불변성**을 유지하면서도 코드의 **재사용성**과 **가독성**을 크게 향상시킵니다.

---

## 3. 실전 적용: 로깅 함수 특화

실무에서 로깅 함수를 부분 적용을 사용하여 특화하는 예시입니다. 자주 변하지 않는 인자(예: 로거 이름, 로그 레벨)를 미리 고정하여, 매번 로깅 시점에 모든 인자를 전달할 필요 없이 메시지만 전달하는 함수를 생성합니다.

### 명령형 방식의 로깅

명령형 프로그래밍에서는 매번 로거 이름과 레벨을 전달해야 합니다.

```java
public class ImperativeLogging {
    
    public static void log(String loggerName, String level, String tag, String message) {
        System.out.println(String.format("[%s] [%s] %s: %s", level, loggerName, tag, message));
    }
    
    public static void main(String[] args) {
        // 매번 모든 인자를 전달해야 함
        log("UserService", "INFO", "DB", "데이터베이스 연결 성공");
        log("UserService", "INFO", "AUTH", "사용자 로그인 시도");
        log("UserService", "INFO", "CACHE", "캐시 초기화 완료");
    }
}
```

### 함수형 방식의 로깅: 커링과 부분 적용

커링과 부분 적용을 사용하면 로거 이름과 레벨을 미리 고정하여 재사용 가능한 로깅 함수를 생성할 수 있습니다.

```java
import java.util.function.BiConsumer;
import java.util.function.Function;

public class FunctionalLogging {
    
    // 커링된 로깅 함수: loggerName -> level -> BiConsumer<tag, message>
    public static Function<String, Function<String, BiConsumer<String, String>>> createCurriedLogger() {
        return loggerName -> 
            level -> 
                (tag, message) -> {
                    System.out.println(String.format("[%s] [%s] %s: %s", level, loggerName, tag, message));
                };
    }

    public static void main(String[] args) {
        // 1. UserService 로거를 생성 (부분 적용)
        Function<String, BiConsumer<String, String>> userServiceLogger = 
            createCurriedLogger().apply("UserService");

        // 2. INFO 레벨의 특화된 로깅 함수를 최종 생성 (추가 부분 적용)
        BiConsumer<String, String> logInfo = userServiceLogger.apply("INFO");

        // 3. 이제 태그와 메시지만 전달하면 됨
        logInfo.accept("DB", "데이터베이스 연결 성공"); 
        logInfo.accept("AUTH", "사용자 로그인 시도");
        logInfo.accept("CACHE", "캐시 초기화 완료");
        
        // 출력:
        // [INFO] [UserService] DB: 데이터베이스 연결 성공
        // [INFO] [UserService] AUTH: 사용자 로그인 시도
        // [INFO] [UserService] CACHE: 캐시 초기화 완료
        
        // 4. 다른 레벨의 로거도 쉽게 생성 가능
        BiConsumer<String, String> logError = userServiceLogger.apply("ERROR");
        logError.accept("DB", "데이터베이스 연결 실패");
        // 출력: [ERROR] [UserService] DB: 데이터베이스 연결 실패
    }
}
```

### 실전 시나리오: 다중 서비스 로거 관리

실제 프로젝트에서는 여러 서비스에 대한 로거를 관리해야 합니다. 부분 적용을 활용하면 각 서비스별로 전문화된 로거를 효율적으로 생성할 수 있습니다.

```java
import java.util.function.BiConsumer;
import java.util.function.Function;
import java.util.HashMap;
import java.util.Map;

public class MultiServiceLogging {
    
    // 커링된 로깅 함수
    public static Function<String, Function<String, BiConsumer<String, String>>> createCurriedLogger() {
        return loggerName -> 
            level -> 
                (tag, message) -> {
                    System.out.println(String.format("[%s] [%s] %s: %s", level, loggerName, tag, message));
                };
    }
    
    public static void main(String[] args) {
        // 로거 팩토리
        Function<String, Function<String, BiConsumer<String, String>>> loggerFactory = createCurriedLogger();
        
        // 각 서비스별 로거 생성
        Map<String, BiConsumer<String, String>> loggers = new HashMap<>();
        loggers.put("UserService-INFO", loggerFactory.apply("UserService").apply("INFO"));
        loggers.put("OrderService-INFO", loggerFactory.apply("OrderService").apply("INFO"));
        loggers.put("PaymentService-ERROR", loggerFactory.apply("PaymentService").apply("ERROR"));
        
        // 사용
        loggers.get("UserService-INFO").accept("AUTH", "사용자 인증 성공");
        loggers.get("OrderService-INFO").accept("ORDER", "주문 생성 완료");
        loggers.get("PaymentService-ERROR").accept("PAYMENT", "결제 처리 실패");
        
        // 출력:
        // [INFO] [UserService] AUTH: 사용자 인증 성공
        // [INFO] [OrderService] ORDER: 주문 생성 완료
        // [ERROR] [PaymentService] PAYMENT: 결제 처리 실패
    }
}
```

---

## 4. 커링과 부분 적용의 성능 고려사항

커링과 부분 적용은 코드의 재사용성과 가독성을 향상시키지만, 함수 객체 생성으로 인한 오버헤드가 발생할 수 있습니다. 그러나 이러한 오버헤드는 대부분의 비즈니스 로직에서 무시할 수 있는 수준이며, 다음과 같은 상황에서는 성능보다 코드 품질이 더 중요합니다.

### 성능 이점이 있는 경우

- **반복적인 설정 제거**: 동일한 설정을 반복적으로 전달하는 대신, 한 번만 설정하고 재사용하므로 코드 실행 경로가 단순해집니다.
- **병렬 처리 용이성**: 순수 함수로 설계된 커링 함수는 부수 효과가 없으므로, 병렬 처리 시 동기화 오버헤드가 없습니다.
- **테스트 용이성**: 각 단계별로 독립적인 함수를 테스트할 수 있어, 디버깅 시간이 단축됩니다.

### 성능 최적화가 필요한 경우

고성능이 요구되는 시스템(예: 실시간 데이터 처리)에서는 다음과 같은 최적화를 고려할 수 있습니다.

```java
import java.util.function.BiConsumer;
import java.util.concurrent.ConcurrentHashMap;

public class OptimizedLogging {
    
    // 로거 캐싱을 통한 성능 최적화
    private static final ConcurrentHashMap<String, BiConsumer<String, String>> loggerCache = 
        new ConcurrentHashMap<>();
    
    public static BiConsumer<String, String> getLogger(String loggerName, String level) {
        String key = loggerName + "-" + level;
        return loggerCache.computeIfAbsent(key, k -> 
            (tag, message) -> {
                System.out.println(String.format("[%s] [%s] %s: %s", level, loggerName, tag, message));
            }
        );
    }
    
    public static void main(String[] args) {
        // 캐시된 로거 사용
        BiConsumer<String, String> logger = getLogger("UserService", "INFO");
        logger.accept("DB", "데이터베이스 연결 성공");
        logger.accept("AUTH", "사용자 로그인 시도");
        
        // 동일한 설정의 로거는 캐시에서 재사용됨
        BiConsumer<String, String> sameLogger = getLogger("UserService", "INFO");
        System.out.println("동일한 로거 인스턴스: " + (logger == sameLogger)); // true
    }
}
```

---

## 5. Stream API와 커링의 조합

커링과 부분 적용은 Stream API와 결합하여 복잡한 데이터 변환 로직을 간결하게 표현할 수 있습니다.

### 실전 예시: 사용자 데이터 필터링 및 변환

```java
import java.util.List;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.stream.Collectors;

public class StreamCurryingExample {
    
    static class User {
        String name;
        int age;
        String department;
        
        User(String name, int age, String department) {
            this.name = name;
            this.age = age;
            this.department = department;
        }
        
        @Override
        public String toString() {
            return String.format("User{name='%s', age=%d, department='%s'}", name, age, department);
        }
    }
    
    // 커링된 필터 생성 함수
    public static Function<String, Predicate<User>> createDepartmentFilter() {
        return department -> user -> user.department.equals(department);
    }
    
    public static Function<Integer, Predicate<User>> createAgeFilter() {
        return minAge -> user -> user.age >= minAge;
    }
    
    public static void main(String[] args) {
        List<User> users = List.of(
            new User("김철수", 25, "개발팀"),
            new User("이영희", 30, "개발팀"),
            new User("박민수", 28, "마케팅팀"),
            new User("정수진", 35, "개발팀"),
            new User("최지훈", 22, "개발팀")
        );
        
        // 부분 적용: 개발팀 필터 생성
        Predicate<User> isDeveloper = createDepartmentFilter().apply("개발팀");
        
        // 부분 적용: 30세 이상 필터 생성
        Predicate<User> isExperienced = createAgeFilter().apply(30);
        
        // Stream API와 조합하여 복잡한 필터링 수행
        List<User> experiencedDevelopers = users.stream()
            .filter(isDeveloper)
            .filter(isExperienced)
            .collect(Collectors.toList());
        
        System.out.println("경력 개발자 목록:");
        experiencedDevelopers.forEach(System.out::println);
        
        // 출력:
        // 경력 개발자 목록:
        // User{name='이영희', age=30, department='개발팀'}
        // User{name='정수진', age=35, department='개발팀'}
    }
}
```

### 지연 평가(Lazy Evaluation)와의 결합

Stream API는 지연 평가를 지원하므로, 커링된 함수와 결합하면 필요한 시점에만 연산이 수행되어 성능이 최적화됩니다.

```java
import java.util.List;
import java.util.function.Function;
import java.util.stream.Stream;

public class LazyEvaluationExample {
    
    // 커링된 변환 함수
    public static Function<String, Function<Integer, String>> createFormatter() {
        return prefix -> 
            number -> {
                System.out.println("변환 수행: " + number); // 실제 변환 시점 확인
                return prefix + number;
            };
    }
    
    public static void main(String[] args) {
        List<Integer> numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);
        
        // 부분 적용: "NUM-" 접두사를 가진 포매터 생성
        Function<Integer, String> formatter = createFormatter().apply("NUM-");
        
        System.out.println("Stream 생성 (아직 변환 수행 안 됨)");
        Stream<String> stream = numbers.stream()
            .map(formatter)
            .filter(s -> s.endsWith("5"));
        
        System.out.println("\n최종 결과 수집 시작 (이제 변환 수행됨)");
        List<String> result = stream.collect(Collectors.toList());
        
        System.out.println("\n최종 결과: " + result);
        
        // 출력:
        // Stream 생성 (아직 변환 수행 안 됨)
        //
        // 최종 결과 수집 시작 (이제 변환 수행됨)
        // 변환 수행: 1
        // 변환 수행: 2
        // 변환 수행: 3
        // 변환 수행: 4
        // 변환 수행: 5
        // 변환 수행: 6
        // 변환 수행: 7
        // 변환 수행: 8
        // 변환 수행: 9
        // 변환 수행: 10
        //
        // 최종 결과: [NUM-5]
    }
}
```

---

## 6. 최종 학습 성과: 복잡한 데이터 변환 파이프라인 구축

이제 학습한 내용을 종합하여 50줄 이상의 복잡한 데이터 변환 로직을 Stream API와 커링을 통해 구현해보겠습니다.

### 실전 시나리오: 주문 데이터 처리 시스템

```java
import java.util.List;
import java.util.Map;
import java.util.function.BiConsumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.stream.Collectors;

public class OrderProcessingSystem {
    
    static class Order {
        String orderId;
        String customerId;
        String product;
        int quantity;
        double price;
        String status;
        
        Order(String orderId, String customerId, String product, int quantity, double price, String status) {
            this.orderId = orderId;
            this.customerId = customerId;
            this.product = product;
            this.quantity = quantity;
            this.price = price;
            this.status = status;
        }
        
        double getTotalPrice() {
            return quantity * price;
        }
        
        @Override
        public String toString() {
            return String.format("Order{id='%s', customer='%s', product='%s', qty=%d, price=%.2f, status='%s'}", 
                orderId, customerId, product, quantity, price, status);
        }
    }
    
    // 커링된 로거 생성
    public static Function<String, Function<String, BiConsumer<String, String>>> createLogger() {
        return serviceName -> 
            level -> 
                (tag, message) -> {
                    System.out.println(String.format("[%s] [%s] %s: %s", level, serviceName, tag, message));
                };
    }
    
    // 커링된 필터 생성 함수들
    public static Function<String, Predicate<Order>> createStatusFilter() {
        return status -> order -> order.status.equals(status);
    }
    
    public static Function<Double, Predicate<Order>> createMinPriceFilter() {
        return minPrice -> order -> order.getTotalPrice() >= minPrice;
    }
    
    public static Function<String, Predicate<Order>> createCustomerFilter() {
        return customerId -> order -> order.customerId.equals(customerId);
    }
    
    // 커링된 변환 함수
    public static Function<Double, Function<Order, Double>> createDiscountCalculator() {
        return discountRate -> 
            order -> order.getTotalPrice() * (1 - discountRate);
    }
    
    public static void main(String[] args) {
        // 로거 설정 (부분 적용)
        BiConsumer<String, String> logInfo = createLogger().apply("OrderService").apply("INFO");
        BiConsumer<String, String> logWarning = createLogger().apply("OrderService").apply("WARNING");
        
        // 샘플 주문 데이터
        List<Order> orders = List.of(
            new Order("ORD001", "CUST001", "노트북", 2, 1500000, "COMPLETED"),
            new Order("ORD002", "CUST002", "마우스", 5, 30000, "PENDING"),
            new Order("ORD003", "CUST001", "키보드", 3, 150000, "COMPLETED"),
            new Order("ORD004", "CUST003", "모니터", 1, 500000, "COMPLETED"),
            new Order("ORD005", "CUST001", "헤드셋", 2, 200000, "CANCELLED"),
            new Order("ORD006", "CUST002", "웹캠", 1, 100000, "COMPLETED"),
            new Order("ORD007", "CUST004", "스피커", 4, 80000, "PENDING")
        );
        
        logInfo.accept("INIT", "주문 처리 시스템 시작");
        logInfo.accept("DATA", "총 주문 건수: " + orders.size());
        
        // 필터 생성 (부분 적용)
        Predicate<Order> isCompleted = createStatusFilter().apply("COMPLETED");
        Predicate<Order> isHighValue = createMinPriceFilter().apply(500000.0);
        Predicate<Order> isCustomer001 = createCustomerFilter().apply("CUST001");
        
        // 할인 계산기 생성 (부분 적용: 10% 할인)
        Function<Order, Double> applyDiscount = createDiscountCalculator().apply(0.1);
        
        // 복잡한 데이터 변환 파이프라인 1: 고가 완료 주문 분석
        logInfo.accept("PROCESS", "고가 완료 주문 분석 시작");
        List<Order> highValueOrders = orders.stream()
            .filter(isCompleted)
            .filter(isHighValue)
            .collect(Collectors.toList());
        
        logInfo.accept("RESULT", "고가 완료 주문 건수: " + highValueOrders.size());
        highValueOrders.forEach(order -> 
            logInfo.accept("ORDER", order.toString())
        );
        
        // 복잡한 데이터 변환 파이프라인 2: 특정 고객의 완료 주문 총액 계산
        logInfo.accept("PROCESS", "CUST001 고객 주문 분석 시작");
        double customer001Total = orders.stream()
            .filter(isCustomer001)
            .filter(isCompleted)
            .mapToDouble(Order::getTotalPrice)
            .sum();
        
        logInfo.accept("RESULT", String.format("CUST001 총 주문 금액: %.2f원", customer001Total));
        
        // 복잡한 데이터 변환 파이프라인 3: 할인 적용 후 금액 계산
        logInfo.accept("PROCESS", "할인 적용 계산 시작");
        Map<String, Double> discountedPrices = orders.stream()
            .filter(isCompleted)
            .collect(Collectors.toMap(
                order -> order.orderId,
                applyDiscount
            ));
        
        logInfo.accept("RESULT", "할인 적용 완료 주문 건수: " + discountedPrices.size());
        discountedPrices.forEach((orderId, discountedPrice) -> 
            logInfo.accept("DISCOUNT", String.format("%s: %.2f원", orderId, discountedPrice))
        );
        
        // 복잡한 데이터 변환 파이프라인 4: 상태별 주문 그룹화 및 통계
        logInfo.accept("PROCESS", "상태별 주문 통계 생성 시작");
        Map<String, Long> ordersByStatus = orders.stream()
            .collect(Collectors.groupingBy(
                order -> order.status,
                Collectors.counting()
            ));
        
        logInfo.accept("RESULT", "상태별 주문 통계:");
        ordersByStatus.forEach((status, count) -> 
            logInfo.accept("STATS", String.format("%s: %d건", status, count))
        );
        
        // 경고: PENDING 주문이 많은 경우
        Long pendingCount = ordersByStatus.getOrDefault("PENDING", 0L);
        if (pendingCount > 2) {
            logWarning.accept("ALERT", String.format("대기 중인 주문이 %d건 있습니다. 처리가 필요합니다.", pendingCount));
        }
        
        logInfo.accept("COMPLETE", "주문 처리 시스템 종료");
    }
}
```

### 실행 결과

```
[INFO] [OrderService] INIT: 주문 처리 시스템 시작
[INFO] [OrderService] DATA: 총 주문 건수: 7
[INFO] [OrderService] PROCESS: 고가 완료 주문 분석 시작
[INFO] [OrderService] RESULT: 고가 완료 주문 건수: 2
[INFO] [OrderService] ORDER: Order{id='ORD001', customer='CUST001', product='노트북', qty=2, price=1500000.00, status='COMPLETED'}
[INFO] [OrderService] ORDER: Order{id='ORD004', customer='CUST003', product='모니터', qty=1, price=500000.00, status='COMPLETED'}
[INFO] [OrderService] PROCESS: CUST001 고객 주문 분석 시작
[INFO] [OrderService] RESULT: CUST001 총 주문 금액: 3450000.00원
[INFO] [OrderService] PROCESS: 할인 적용 계산 시작
[INFO] [OrderService] RESULT: 할인 적용 완료 주문 건수: 4
[INFO] [OrderService] DISCOUNT: ORD001: 2700000.00원
[INFO] [OrderService] DISCOUNT: ORD003: 405000.00원
[INFO] [OrderService] DISCOUNT: ORD004: 450000.00원
[INFO] [OrderService] DISCOUNT: ORD006: 90000.00원
[INFO] [OrderService] PROCESS: 상태별 주문 통계 생성 시작
[INFO] [OrderService] RESULT: 상태별 주문 통계:
[INFO] [OrderService] STATS: COMPLETED: 4건
[INFO] [OrderService] STATS: PENDING: 2건
[INFO] [OrderService] STATS: CANCELLED: 1건
[INFO] [OrderService] COMPLETE: 주문 처리 시스템 종료
```

---

## 결론

커링과 부분 적용은 Functional Programming의 핵심 기법으로, 다중 인자 함수를 단일 인자 함수의 연속으로 변환하고, 특정 인자를 고정하여 재사용 가능한 전문화된 함수를 생성합니다. 이를 통해 다음과 같은 이점을 얻을 수 있습니다.

- **코드 재사용성**: 공통 로직을 함수로 추상화하고, 부분 적용을 통해 특화된 유틸리티 함수를 생성합니다.
- **불변성 증진**: 상태 변경 없이 새로운 함수를 생성하여 부수 효과를 최소화합니다.
- **테스트 용이성**: 순수 함수로 설계되어 단위 테스트 작성이 용이합니다.
- **함수 조합 강화**: Stream API와 결합하여 복잡한 데이터 파이프라인을 간결하게 구축할 수 있습니다.

명령형 프로그래밍 대비 커링과 부분 적용은 코드의 간결성, 유지보수성, 병렬 처리 용이성을 크게 향상시키며, 실무에서 복잡한 비즈니스 로직을 효율적으로 관리하는 데 필수적인 기술입니다.

