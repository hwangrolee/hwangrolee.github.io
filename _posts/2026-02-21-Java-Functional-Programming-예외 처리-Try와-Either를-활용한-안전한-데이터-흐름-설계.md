---
layout: post
title: "Java Functional Programming - 예외 처리: Try와 Either를 활용한 안전한 데이터 흐름 설계"
date: 2026-02-21 00:00:00 +0900
categories: [Java]
tags: [자바, 함수형 프로그래밍]
author: hwangrolee
description: "Vavr의 Try와 Either 타입을 사용하여 예외를 데이터처럼 다루는 Functional Programming 기반 예외 처리 방식을 배우고, try-catch 대신 선언적 메서드 체이닝으로 안전한 데이터 흐름을 설계하는 방법을 익힙니다."
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

Functional Programming 기반 예외 처리는 명령형 프로그래밍의 `try-catch` 블록 대신, 예외(Exception)를 일반적인 **데이터 타입**으로 취급하여 함수 파이프라인 내에서 안전하게 전달하고 처리하는 방식입니다. 이 접근 방식을 사용하면 예외가 발생 가능한 코드를 **순수 함수(Pure Function)**처럼 다룰 수 있게 되며, 코드의 제어 흐름이 `try-catch`로 분리되거나 중단되는 것을 방지합니다. Vavr 라이브러리의 **`Try`** 또는 **`Either`**와 같은 타입을 익히면, 예외 처리 로직을 선언적인(Declarative) 메서드 체이닝으로 통합하여 코드의 가독성과 예측 가능성을 획기적으로 높일 수 있습니다.

---

## 왜 Functional Programming 기반 예외 처리를 학습해야 하는가

전통적인 자바의 예외 처리 메커니즘은 **제어 흐름의 중단**을 야기하며, 이는 Functional Programming의 핵심 원칙인 **데이터 흐름 중심 설계**와 충돌합니다. `try-catch` 블록은 코드의 선형적 흐름을 방해하고, 예외 발생 시점과 처리 시점이 분리되어 코드의 추적성(Traceability)을 저하시킵니다. 특히 Stream API나 람다 표현식 내부에서 예외를 처리해야 할 때, 기존 방식은 함수형 파이프라인의 연속성을 깨뜨리는 문제를 발생시킵니다.

Functional Programming 기반 예외 처리를 학습하면 다음과 같은 실질적 이점을 얻을 수 있습니다:

1. **순수 함수의 보장**: 예외를 던지는 함수는 부수 효과(Side Effect)를 가지므로 순수 함수가 아닙니다. `Try`나 `Either`를 사용하면 예외를 반환 타입으로 명시하여 함수의 순수성을 유지할 수 있습니다.

2. **타입 안정성 강화**: 메서드 시그니처에 실패 가능성이 명시되므로, 컴파일 타임에 예외 처리 누락을 방지할 수 있습니다.

3. **함수 합성(Function Composition) 용이성**: `map`, `flatMap` 등의 고차 함수(Higher-Order Function)를 통해 예외 처리 로직을 자연스럽게 파이프라인에 통합할 수 있습니다.

4. **테스트 용이성**: 예외를 데이터로 다루면 Mock 객체 없이도 성공/실패 케이스를 쉽게 테스트할 수 있습니다.

---

## 1. 예외를 데이터로 다루는 개념

기존의 자바 예외 처리는 **제어 흐름의 중단**을 야기합니다. 메서드가 예외를 던지면(`throw`), 호출 스택을 역추적하며(Stack Unwinding) `catch` 블록을 찾기 때문에, 데이터의 흐름을 중심으로 하는 Functional Programming 파이프라인(예: Stream)에 통합하기 어렵습니다.

### 1.1. 명령형 예외 처리의 문제점

다음은 전통적인 `try-catch` 방식의 예외 처리 코드입니다:

```java
public class ImperativeExceptionHandling {
    public static void main(String[] args) {
        List<String> numbers = Arrays.asList("10", "20", "abc", "30");
        List<Integer> results = new ArrayList<>();
        
        for (String num : numbers) {
            try {
                int parsed = Integer.parseInt(num);
                results.add(parsed * 2);
            } catch (NumberFormatException e) {
                // 예외 발생 시 로깅만 하고 계속 진행
                System.err.println("파싱 실패: " + num);
            }
        }
        
        System.out.println("결과: " + results); // [20, 40, 60]
    }
}
```

이 코드는 다음과 같은 문제점을 가집니다:

- **제어 흐름 분리**: 정상 로직과 예외 처리 로직이 분리되어 있어 코드의 흐름을 파악하기 어렵습니다.
- **Stream API 통합 불가**: `try-catch`는 람다 표현식 내부에서 사용하기 번거롭습니다.
- **부수 효과 발생**: `results` 리스트를 변경하는 부수 효과가 발생하여 불변성(Immutability)을 해칩니다.

### 1.2. Functional Programming 접근 방식

Functional Programming 기반 접근 방식은 예외를 **두 가지 가능한 상태**를 가진 컨테이너로 캡슐화합니다:

- **성공(Success/Right)**: 정상적인 결과 값을 포함합니다.
- **실패(Failure/Left)**: 예외 객체(Exception)를 포함합니다.

이 컨테이너 자체를 반환함으로써, 호출자는 예외 발생 여부를 **`if-else` 검사**가 아닌 **메서드 호출**을 통해 안전하게 처리할 수 있습니다. 이는 **Railway Oriented Programming** 패턴으로도 알려져 있으며, 성공 경로와 실패 경로를 명확히 분리하여 코드의 예측 가능성을 높입니다.

---

## 2. Vavr 라이브러리의 `Try` 타입 활용

Vavr의 `Try<T>` 타입은 **예외가 발생할 가능성이 있는 코드 블록**을 감싸고, 그 결과를 안전하게 포장하여 Functional Programming 파이프라인에 통합하는 데 특화되어 있습니다.

`Try` 객체는 `Success<T>` 또는 `Failure<T>` 중 하나의 인스턴스입니다. 이는 **Monad** 패턴을 구현한 것으로, `map`, `flatMap` 등의 연산을 통해 함수 합성을 가능하게 합니다.

### 2.1. Vavr 의존성 추가

먼저 프로젝트에 Vavr 라이브러리를 추가해야 합니다:

```xml
<!-- Maven -->
<dependency>
    <groupId>io.vavr</groupId>
    <artifactId>vavr</artifactId>
    <version>0.10.4</version>
</dependency>
```

```gradle
// Gradle
implementation 'io.vavr:vavr:0.10.4'
```

### 2.2. `Try` 객체 생성 및 예외 캡슐화

`Try.of()` 메서드는 람다 표현식 내에서 예외를 던질 수 있는 코드를 실행하고, 발생한 예외를 캡처합니다:

```java
import io.vavr.control.Try;

public class TryBasicExample {
    // 예외를 던질 수 있는 함수
    public static int divide(int dividend, int divisor) throws ArithmeticException {
        if (divisor == 0) {
            throw new ArithmeticException("0으로 나눌 수 없습니다.");
        }
        return dividend / divisor;
    }
    
    public static void main(String[] args) {
        // Try를 사용하여 예외를 캡슐화
        Try<Integer> safeDivisionSuccess = Try.of(() -> divide(10, 2)); 
        // Success(5)
        
        Try<Integer> safeDivisionFailure = Try.of(() -> divide(10, 0)); 
        // Failure(ArithmeticException: 0으로 나눌 수 없습니다.)
        
        // 성공 여부 확인
        System.out.println("성공 케이스 - isSuccess: " + safeDivisionSuccess.isSuccess()); 
        // true
        System.out.println("실패 케이스 - isFailure: " + safeDivisionFailure.isFailure()); 
        // true
        
        // 값 추출 (성공 시)
        safeDivisionSuccess.onSuccess(value -> 
            System.out.println("결과: " + value)); // 결과: 5
        
        // 예외 처리 (실패 시)
        safeDivisionFailure.onFailure(throwable -> 
            System.err.println("예외 발생: " + throwable.getMessage()));
        // 예외 발생: 0으로 나눌 수 없습니다.
    }
}
```

이 방식의 핵심은 **예외가 발생해도 프로그램이 중단되지 않고**, `Try` 객체가 실패 상태를 포함한 채로 반환된다는 점입니다. 이는 **지연 평가(Lazy Evaluation)**와 유사한 개념으로, 예외 처리 시점을 호출자가 결정할 수 있게 합니다.

### 2.3. Functional Programming 연산을 통한 안전한 처리

`Try`는 `map`, `flatMap`, `filter`와 같은 모나딕 연산을 제공합니다:

- **`map`**: `Success` 상태일 때만 내부 값을 변형합니다. `Failure` 상태일 때는 변형 로직이 실행되지 않고 `Failure` 상태가 그대로 유지됩니다.
- **`flatMap`**: 중첩된 `Try` 객체를 평탄화(Flatten)하여 연속적인 예외 가능 연산을 체이닝할 수 있습니다.
- **`filter`**: 조건을 만족하지 않으면 `Failure`로 변환합니다.
- **`recover`**: `Failure` 상태일 때 대체 값을 제공하거나 다른 `Try`로 복구합니다.
- **`orElseGet`**: `Failure` 상태일 때만 실행되어 안전한 대체 값을 반환합니다.

```java
import io.vavr.control.Try;

public class TryFunctionalOperations {
    public static void main(String[] args) {
        // 예외 발생 코드의 Functional Programming 처리 파이프라인
        int result = Try.of(() -> divide(100, 10))
            // Success(10) 상태이므로 map 실행
            .map(val -> val * 5)  // Success(50)
            
            // filter를 사용한 조건 검증
            .filter(val -> val > 0)  // Success(50) 유지
            
            // 예외가 발생하면 orElseGet이 실행되고, 
            // 발생하지 않았으므로 내부 값 50을 반환
            .getOrElse(-1);
        
        System.out.println("성공 결과: " + result); // 출력: 50
        
        // --- 실패 예시 ---
        int failureResult = Try.of(() -> divide(100, 0))
            // Failure 상태이므로 map 건너뜀
            .map(val -> val * 5)
            
            // Failure 상태이므로 filter 건너뜀
            .filter(val -> val > 0)
            
            // recover를 사용한 예외 복구
            .recover(ArithmeticException.class, -1)
            .get();
        
        System.out.println("실패 결과: " + failureResult); // 출력: -1
    }
    
    public static int divide(int dividend, int divisor) {
        if (divisor == 0) {
            throw new ArithmeticException("0으로 나눌 수 없습니다.");
        }
        return dividend / divisor;
    }
}
```

### 2.4. `flatMap`을 사용한 연속적 예외 처리

여러 예외 가능 연산을 연결할 때 `flatMap`을 사용하면 중첩된 `Try` 객체를 평탄화할 수 있습니다:

```java
import io.vavr.control.Try;

public class TryFlatMapExample {
    public static Try<Integer> parseInteger(String str) {
        return Try.of(() -> Integer.parseInt(str));
    }
    
    public static Try<Integer> safeDivide(int dividend, int divisor) {
        return Try.of(() -> {
            if (divisor == 0) {
                throw new ArithmeticException("0으로 나눌 수 없습니다.");
            }
            return dividend / divisor;
        });
    }
    
    public static void main(String[] args) {
        // 문자열 파싱 -> 나눗셈 -> 결과 변환의 연속적 파이프라인
        Try<String> result = parseInteger("100")
            .flatMap(dividend -> parseInteger("10")
                .flatMap(divisor -> safeDivide(dividend, divisor)))
            .map(value -> "최종 결과: " + value);
        
        result.onSuccess(System.out::println); // 최종 결과: 10
        result.onFailure(e -> System.err.println("처리 실패: " + e.getMessage()));
        
        // 실패 케이스
        Try<String> failureResult = parseInteger("abc")
            .flatMap(dividend -> parseInteger("10")
                .flatMap(divisor -> safeDivide(dividend, divisor)))
            .map(value -> "최종 결과: " + value);
        
        failureResult.onFailure(e -> 
            System.err.println("처리 실패: " + e.getMessage()));
        // 처리 실패: For input string: "abc"
    }
}
```

이 방식은 **명령형 프로그래밍 대비 다음과 같은 구조적 장점**을 제공합니다:

1. **예외 전파의 자동화**: 중간 단계에서 예외가 발생하면 이후 모든 `map`, `flatMap` 연산이 자동으로 건너뛰어집니다.
2. **불변성 보장**: 원본 데이터를 변경하지 않고 새로운 `Try` 객체를 생성합니다.
3. **병렬 처리 용이성**: 부수 효과가 없으므로 Stream API의 `parallel()` 메서드와 안전하게 결합할 수 있습니다.

### 2.5. Stream API와 `Try`의 통합

`Try`는 Stream API와 결합하여 컬렉션 내 각 요소의 예외를 안전하게 처리할 수 있습니다:

```java
import io.vavr.control.Try;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

public class TryWithStreamExample {
    public static void main(String[] args) {
        List<String> numbers = Arrays.asList("10", "20", "abc", "30", "xyz");
        
        // 각 문자열을 정수로 파싱하고, 실패한 경우 0으로 대체
        List<Integer> parsedNumbers = numbers.stream()
            .map(str -> Try.of(() -> Integer.parseInt(str))
                .getOrElse(0))
            .collect(Collectors.toList());
        
        System.out.println("파싱 결과 (실패 시 0): " + parsedNumbers);
        // [10, 20, 0, 30, 0]
        
        // 성공한 케이스만 필터링
        List<Integer> successfulParsing = numbers.stream()
            .map(str -> Try.of(() -> Integer.parseInt(str)))
            .filter(Try::isSuccess)
            .map(Try::get)
            .collect(Collectors.toList());
        
        System.out.println("성공한 파싱만: " + successfulParsing);
        // [10, 20, 30]
        
        // 실패한 케이스의 예외 메시지 수집
        List<String> errors = numbers.stream()
            .map(str -> Try.of(() -> Integer.parseInt(str)))
            .filter(Try::isFailure)
            .map(tryResult -> tryResult.getCause().getMessage())
            .collect(Collectors.toList());
        
        System.out.println("파싱 실패 메시지: " + errors);
        // [For input string: "abc", For input string: "xyz"]
    }
}
```

이 예제는 **명령형 `try-catch` 방식 대비** 다음과 같은 이점을 제공합니다:

- **선언적 코드**: 예외 처리 로직이 데이터 변환 파이프라인에 자연스럽게 통합됩니다.
- **불변성 유지**: 외부 리스트를 변경하지 않고 새로운 리스트를 생성합니다.
- **테스트 용이성**: 각 단계를 독립적으로 테스트할 수 있습니다.

---

## 3. Vavr 라이브러리의 `Either` 타입 활용

`Either<L, R>` 타입은 **두 가지 가능한 결과** 중 하나를 명시적으로 나타냅니다. 일반적으로 왼쪽(`Left`)에는 **오류(Error)** 타입을, 오른쪽(`Right`)에는 **성공(Success)** 타입을 넣어 사용합니다. `Either`는 예외(Throwable) 객체가 아닌, **도메인 특화 오류 타입**을 반환할 때 유용합니다.

### 3.1. `Try`와 `Either`의 차이점

- **`Try<T>`**: 예외(Throwable)를 캡처하는 데 특화되어 있으며, `Success<T>` 또는 `Failure<Throwable>` 상태를 가집니다.
- **`Either<L, R>`**: 임의의 두 타입을 표현할 수 있으며, 예외가 아닌 **비즈니스 로직 오류**를 타입 안전하게 표현할 수 있습니다.

`Either`를 사용하면 **타입 시스템을 통해 오류 처리를 강제**할 수 있으며, 컴파일 타임에 오류 처리 누락을 방지할 수 있습니다.

### 3.2. 도메인 특화 오류 타입 정의

```java
import io.vavr.control.Either;

// 도메인 특화 오류 타입 정의
class ValidationError {
    enum ErrorType {
        INVALID_FORMAT,
        OUT_OF_RANGE,
        NULL_VALUE
    }
    
    final ErrorType type;
    final String message;
    
    ValidationError(ErrorType type, String message) {
        this.type = type;
        this.message = message;
    }
    
    @Override
    public String toString() {
        return String.format("[%s] %s", type, message);
    }
}

public class EitherBasicExample {
    // Either<ValidationError, Integer>를 반환하는 함수 
    // (Left는 오류, Right는 성공)
    public static Either<ValidationError, Integer> parseAge(String ageStr) {
        if (ageStr == null || ageStr.trim().isEmpty()) {
            return Either.left(new ValidationError(
                ValidationError.ErrorType.NULL_VALUE,
                "나이 값이 비어있습니다."
            ));
        }
        
        try {
            int age = Integer.parseInt(ageStr);
            if (age < 0) {
                return Either.left(new ValidationError(
                    ValidationError.ErrorType.OUT_OF_RANGE,
                    "나이는 음수가 될 수 없습니다."
                ));
            }
            if (age > 150) {
                return Either.left(new ValidationError(
                    ValidationError.ErrorType.OUT_OF_RANGE,
                    "나이가 유효 범위를 초과했습니다."
                ));
            }
            return Either.right(age);
        } catch (NumberFormatException e) {
            return Either.left(new ValidationError(
                ValidationError.ErrorType.INVALID_FORMAT,
                "유효한 숫자가 아닙니다: " + ageStr
            ));
        }
    }
    
    public static void main(String[] args) {
        // Either의 Functional Programming 처리
        Either<ValidationError, Integer> safeAge = parseAge("25");
        
        // map을 사용하면 Right(성공)일 때만 실행
        String message = safeAge
            .map(age -> "사용자 나이는 " + age + "살 입니다.")
            // fold를 사용하여 Left와 Right 모두 처리
            .fold(
                error -> "오류 발생: " + error, // Left 처리 로직
                successMessage -> successMessage  // Right 처리 로직
            );
        
        System.out.println(message); // 출력: 사용자 나이는 25살 입니다.
        
        // 실패 케이스들
        parseAge("abc").peek(
            val -> System.out.println("성공: " + val),
            error -> System.err.println("실패: " + error)
        );
        // 실패: [INVALID_FORMAT] 유효한 숫자가 아닙니다: abc
        
        parseAge("-5").peek(
            val -> System.out.println("성공: " + val),
            error -> System.err.println("실패: " + error)
        );
        // 실패: [OUT_OF_RANGE] 나이는 음수가 될 수 없습니다.
    }
}
```

### 3.3. `Either`의 Functional Programming 연산

`Either`는 `Try`와 유사한 모나딕 연산을 제공하지만, **Right-biased**로 동작합니다. 즉, `map`, `flatMap` 등의 연산은 `Right` 값에만 적용됩니다:

```java
import io.vavr.control.Either;

public class EitherFunctionalOperations {
    public static Either<ValidationError, String> validateName(String name) {
        if (name == null || name.trim().isEmpty()) {
            return Either.left(new ValidationError(
                ValidationError.ErrorType.NULL_VALUE,
                "이름이 비어있습니다."
            ));
        }
        if (name.length() < 2) {
            return Either.left(new ValidationError(
                ValidationError.ErrorType.INVALID_FORMAT,
                "이름은 최소 2자 이상이어야 합니다."
            ));
        }
        return Either.right(name);
    }
    
    public static Either<ValidationError, Integer> parseAge(String ageStr) {
        // 이전 예제의 parseAge 메서드와 동일
        try {
            int age = Integer.parseInt(ageStr);
            if (age < 0 || age > 150) {
                return Either.left(new ValidationError(
                    ValidationError.ErrorType.OUT_OF_RANGE,
                    "나이가 유효 범위를 벗어났습니다."
                ));
            }
            return Either.right(age);
        } catch (NumberFormatException e) {
            return Either.left(new ValidationError(
                ValidationError.ErrorType.INVALID_FORMAT,
                "유효한 숫자가 아닙니다."
            ));
        }
    }
    
    public static void main(String[] args) {
        // 여러 검증을 연결하는 파이프라인
        Either<ValidationError, String> userProfile = validateName("홍길동")
            .flatMap(name -> parseAge("30")
                .map(age -> String.format("이름: %s, 나이: %d세", name, age)));
        
        userProfile.peek(
            profile -> System.out.println("프로필 생성 성공: " + profile),
            error -> System.err.println("프로필 생성 실패: " + error)
        );
        // 프로필 생성 성공: 이름: 홍길동, 나이: 30세
        
        // 실패 케이스 - 이름 검증 실패
        Either<ValidationError, String> failedProfile1 = validateName("김")
            .flatMap(name -> parseAge("30")
                .map(age -> String.format("이름: %s, 나이: %d세", name, age)));
        
        failedProfile1.peekLeft(error -> 
            System.err.println("검증 실패: " + error));
        // 검증 실패: [INVALID_FORMAT] 이름은 최소 2자 이상이어야 합니다.
        
        // 실패 케이스 - 나이 검증 실패
        Either<ValidationError, String> failedProfile2 = validateName("이순신")
            .flatMap(name -> parseAge("200")
                .map(age -> String.format("이름: %s, 나이: %d세", name, age)));
        
        failedProfile2.peekLeft(error -> 
            System.err.println("검증 실패: " + error));
        // 검증 실패: [OUT_OF_RANGE] 나이가 유효 범위를 벗어났습니다.
    }
}
```

### 3.4. `fold` 메서드를 통한 양방향 처리

`Either`의 **`fold`** 메서드는 컨테이너가 `Left`인지 `Right`인지에 따라 두 개의 람다 중 하나를 실행하여, 최종적으로 두 분기(Branch)를 하나의 값으로 합치는 안전한 방법을 제공합니다:

```java
import io.vavr.control.Either;

public class EitherFoldExample {
    public static Either<ValidationError, Double> calculateBMI(
        String heightStr, 
        String weightStr
    ) {
        return parseDouble(heightStr, "키")
            .flatMap(height -> parseDouble(weightStr, "몸무게")
                .flatMap(weight -> {
                    if (height <= 0 || weight <= 0) {
                        return Either.left(new ValidationError(
                            ValidationError.ErrorType.OUT_OF_RANGE,
                            "키와 몸무게는 양수여야 합니다."
                        ));
                    }
                    double bmi = weight / ((height / 100) * (height / 100));
                    return Either.right(bmi);
                }));
    }
    
    private static Either<ValidationError, Double> parseDouble(
        String value, 
        String fieldName
    ) {
        try {
            return Either.right(Double.parseDouble(value));
        } catch (NumberFormatException e) {
            return Either.left(new ValidationError(
                ValidationError.ErrorType.INVALID_FORMAT,
                fieldName + "가 유효한 숫자가 아닙니다: " + value
            ));
        }
    }
    
    public static void main(String[] args) {
        // fold를 사용한 결과 처리
        String result1 = calculateBMI("175", "70")
            .map(bmi -> String.format("%.2f", bmi))
            .fold(
                error -> "BMI 계산 실패: " + error.message,
                bmi -> "BMI: " + bmi
            );
        
        System.out.println(result1); // BMI: 22.86
        
        String result2 = calculateBMI("abc", "70")
            .map(bmi -> String.format("%.2f", bmi))
            .fold(
                error -> "BMI 계산 실패: " + error.message,
                bmi -> "BMI: " + bmi
            );
        
        System.out.println(result2); 
        // BMI 계산 실패: 키가 유효한 숫자가 아닙니다: abc
    }
}
```

---

## 4. 실전 예제: 사용자 등록 시스템

다음은 `Try`와 `Either`를 결합하여 실제 비즈니스 로직에 적용한 종합 예제입니다:

```java
import io.vavr.control.Either;
import io.vavr.control.Try;
import java.util.regex.Pattern;

// 도메인 모델
class User {
    final String email;
    final String name;
    final int age;
    
    User(String email, String name, int age) {
        this.email = email;
        this.name = name;
        this.age = age;
    }
    
    @Override
    public String toString() {
        return String.format("User{email='%s', name='%s', age=%d}", 
            email, name, age);
    }
}

// 검증 오류 타입
class RegistrationError {
    enum ErrorCode {
        INVALID_EMAIL,
        INVALID_NAME,
        INVALID_AGE,
        DATABASE_ERROR
    }
    
    final ErrorCode code;
    final String message;
    
    RegistrationError(ErrorCode code, String message) {
        this.code = code;
        this.message = message;
    }
    
    @Override
    public String toString() {
        return String.format("[%s] %s", code, message);
    }
}

public class UserRegistrationSystem {
    private static final Pattern EMAIL_PATTERN = 
        Pattern.compile("^[A-Za-z0-9+_.-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$");
    
    // 이메일 검증
    public static Either<RegistrationError, String> validateEmail(String email) {
        if (email == null || email.trim().isEmpty()) {
            return Either.left(new RegistrationError(
                RegistrationError.ErrorCode.INVALID_EMAIL,
                "이메일이 비어있습니다."
            ));
        }
        if (!EMAIL_PATTERN.matcher(email).matches()) {
            return Either.left(new RegistrationError(
                RegistrationError.ErrorCode.INVALID_EMAIL,
                "이메일 형식이 올바르지 않습니다: " + email
            ));
        }
        return Either.right(email);
    }
    
    // 이름 검증
    public static Either<RegistrationError, String> validateName(String name) {
        if (name == null || name.trim().isEmpty()) {
            return Either.left(new RegistrationError(
                RegistrationError.ErrorCode.INVALID_NAME,
                "이름이 비어있습니다."
            ));
        }
        if (name.length() < 2 || name.length() > 50) {
            return Either.left(new RegistrationError(
                RegistrationError.ErrorCode.INVALID_NAME,
                "이름은 2자 이상 50자 이하여야 합니다."
            ));
        }
        return Either.right(name);
    }
    
    // 나이 검증
    public static Either<RegistrationError, Integer> validateAge(String ageStr) {
        return Try.of(() -> Integer.parseInt(ageStr))
            .toEither()
            .mapLeft(throwable -> new RegistrationError(
                RegistrationError.ErrorCode.INVALID_AGE,
                "나이가 유효한 숫자가 아닙니다: " + ageStr
            ))
            .flatMap(age -> {
                if (age < 0 || age > 150) {
                    return Either.left(new RegistrationError(
                        RegistrationError.ErrorCode.INVALID_AGE,
                        "나이가 유효 범위를 벗어났습니다: " + age
                    ));
                }
                return Either.right(age);
            });
    }
    
    // 데이터베이스 저장 시뮬레이션 (예외 발생 가능)
    public static Try<User> saveToDatabase(User user) {
        return Try.of(() -> {
            // 실제로는 데이터베이스 연결 및 저장 로직
            System.out.println("데이터베이스에 저장 중: " + user);
            
            // 시뮬레이션: 특정 이메일은 저장 실패
            if (user.email.contains("error")) {
                throw new RuntimeException("데이터베이스 연결 실패");
            }
            
            return user;
        });
    }
    
    // 사용자 등록 파이프라인
    public static Either<RegistrationError, User> registerUser(
        String email, 
        String name, 
        String ageStr
    ) {
        return validateEmail(email)
            .flatMap(validEmail -> validateName(name)
                .flatMap(validName -> validateAge(ageStr)
                    .map(validAge -> new User(validEmail, validName, validAge))))
            .flatMap(user -> saveToDatabase(user)
                .toEither()
                .mapLeft(throwable -> new RegistrationError(
                    RegistrationError.ErrorCode.DATABASE_ERROR,
                    "데이터베이스 저장 실패: " + throwable.getMessage()
                )));
    }
    
    public static void main(String[] args) {
        // 성공 케이스
        Either<RegistrationError, User> result1 = 
            registerUser("user@example.com", "홍길동", "30");
        
        result1.peek(
            user -> System.out.println("등록 성공: " + user),
            error -> System.err.println("등록 실패: " + error)
        );
        // 데이터베이스에 저장 중: User{email='user@example.com', name='홍길동', age=30}
        // 등록 성공: User{email='user@example.com', name='홍길동', age=30}
        
        // 실패 케이스 1: 이메일 형식 오류
        Either<RegistrationError, User> result2 = 
            registerUser("invalid-email", "홍길동", "30");
        
        result2.peekLeft(error -> 
            System.err.println("등록 실패: " + error));
        // 등록 실패: [INVALID_EMAIL] 이메일 형식이 올바르지 않습니다: invalid-email
        
        // 실패 케이스 2: 나이 형식 오류
        Either<RegistrationError, User> result3 = 
            registerUser("user@example.com", "홍길동", "abc");
        
        result3.peekLeft(error -> 
            System.err.println("등록 실패: " + error));
        // 등록 실패: [INVALID_AGE] 나이가 유효한 숫자가 아닙니다: abc
        
        // 실패 케이스 3: 데이터베이스 오류
        Either<RegistrationError, User> result4 = 
            registerUser("error@example.com", "홍길동", "30");
        
        result4.peekLeft(error -> 
            System.err.println("등록 실패: " + error));
        // 데이터베이스에 저장 중: User{email='error@example.com', name='홍길동', age=30}
        // 등록 실패: [DATABASE_ERROR] 데이터베이스 저장 실패: 데이터베이스 연결 실패
    }
}
```

이 예제는 다음과 같은 Functional Programming 원칙을 구현합니다:

1. **순수 함수**: 모든 검증 함수는 부수 효과 없이 입력에 대한 결과만 반환합니다.
2. **불변성**: `User` 객체는 불변(Immutable) 객체로 설계되었습니다.
3. **함수 합성**: `flatMap`을 통해 여러 검증 단계를 하나의 파이프라인으로 연결합니다.
4. **타입 안정성**: 컴파일 타임에 오류 처리를 강제합니다.
5. **테스트 용이성**: 각 검증 함수를 독립적으로 테스트할 수 있습니다.

---

## 5. 명령형 방식과의 비교

동일한 사용자 등록 로직을 명령형 방식으로 구현하면 다음과 같습니다:

```java
public class ImperativeUserRegistration {
    public static User registerUser(String email, String name, String ageStr) 
        throws Exception {
        
        // 이메일 검증
        if (email == null || email.trim().isEmpty()) {
            throw new IllegalArgumentException("이메일이 비어있습니다.");
        }
        if (!EMAIL_PATTERN.matcher(email).matches()) {
            throw new IllegalArgumentException("이메일 형식이 올바르지 않습니다.");
        }
        
        // 이름 검증
        if (name == null || name.trim().isEmpty()) {
            throw new IllegalArgumentException("이름이 비어있습니다.");
        }
        if (name.length() < 2 || name.length() > 50) {
            throw new IllegalArgumentException("이름은 2자 이상 50자 이하여야 합니다.");
        }
        
        // 나이 검증
        int age;
        try {
            age = Integer.parseInt(ageStr);
        } catch (NumberFormatException e) {
            throw new IllegalArgumentException("나이가 유효한 숫자가 아닙니다.");
        }
        
        if (age < 0 || age > 150) {
            throw new IllegalArgumentException("나이가 유효 범위를 벗어났습니다.");
        }
        
        // 사용자 생성 및 저장
        User user = new User(email, name, age);
        
        try {
            return saveToDatabase(user);
        } catch (Exception e) {
            throw new RuntimeException("데이터베이스 저장 실패: " + e.getMessage());
        }
    }
    
    public static void main(String[] args) {
        try {
            User user = registerUser("user@example.com", "홍길동", "30");
            System.out.println("등록 성공: " + user);
        } catch (Exception e) {
            System.err.println("등록 실패: " + e.getMessage());
        }
    }
}
```

### 명령형 방식의 문제점

1. **제어 흐름 중단**: 예외 발생 시 즉시 메서드가 종료되어 이후 로직이 실행되지 않습니다.
2. **타입 안정성 부족**: 메서드 시그니처에 `throws Exception`만 명시되어 어떤 종류의 오류가 발생할 수 있는지 알 수 없습니다.
3. **테스트 어려움**: 예외를 던지는 코드를 테스트하려면 `try-catch` 블록이 필요합니다.
4. **함수 합성 불가**: 예외를 던지는 함수는 다른 함수와 조합하기 어렵습니다.
5. **병렬 처리 어려움**: 예외는 스레드 경계를 넘어 전파되지 않으므로 병렬 처리 시 추가 처리가 필요합니다.

### Functional Programming 방식의 장점

1. **선언적 코드**: 예외 처리 로직이 데이터 변환 파이프라인에 통합되어 있습니다.
2. **타입 안정성**: `Either<RegistrationError, User>` 타입을 통해 가능한 오류 타입이 명확히 드러납니다.
3. **테스트 용이성**: 각 함수가 순수 함수이므로 입력에 대한 출력만 검증하면 됩니다.
4. **함수 합성**: `flatMap`을 통해 여러 검증 단계를 자연스럽게 연결할 수 있습니다.
5. **병렬 처리 용이성**: 부수 효과가 없으므로 Stream API의 `parallel()`과 안전하게 결합할 수 있습니다.

---

## 6. 성능 고려사항

Functional Programming 기반 예외 처리는 추가적인 객체 생성(예: `Try`, `Either` 인스턴스)으로 인해 약간의 오버헤드가 발생할 수 있습니다. 그러나 다음과 같은 이유로 실무에서는 대부분 무시할 수 있는 수준입니다:

1. **JIT 컴파일러 최적화**: 최신 JVM의 JIT 컴파일러는 이러한 패턴을 효율적으로 최적화합니다.
2. **가비지 컬렉션 효율성**: 단명(Short-lived) 객체는 Young Generation에서 빠르게 수집됩니다.
3. **코드 유지보수 비용**: 성능 차이보다 코드의 가독성과 유지보수성 향상이 더 큰 이점을 제공합니다.

성능이 극도로 중요한 핫스팟(Hot Spot) 코드에서는 프로파일링을 통해 실제 병목 지점을 파악한 후 최적화를 고려해야 합니다.

---

## 7. 학습 정리 및 다음 단계

이 문서를 통해 학습한 내용을 정리하면 다음과 같습니다:

1. **예외를 데이터로 다루는 개념**: 예외를 `Success`/`Failure` 또는 `Left`/`Right` 컨테이너로 캡슐화하여 Functional Programming 파이프라인에 통합합니다.

2. **`Try` 타입**: 예외가 발생할 수 있는 코드를 안전하게 감싸고, `map`, `flatMap`, `recover` 등의 연산을 통해 선언적으로 처리합니다.

3. **`Either` 타입**: 도메인 특화 오류 타입을 사용하여 타입 안전한 오류 처리를 구현하고, `fold`를 통해 양방향 분기를 하나의 값으로 통합합니다.

4. **실전 적용**: 사용자 등록 시스템 예제를 통해 여러 검증 단계를 `flatMap`으로 연결하고, `Try`와 `Either`를 결합하여 복잡한 비즈니스 로직을 구현하는 방법을 학습했습니다.

5. **명령형 방식과의 비교**: Functional Programming 방식이 타입 안정성, 테스트 용이성, 함수 합성, 병렬 처리 측면에서 구조적 장점을 제공함을 확인했습니다.

### 다음 학습 단계

이 문서에서 학습한 내용을 바탕으로 다음 주제를 학습하면 Functional Programming 역량을 더욱 강화할 수 있습니다:

- **Monad 패턴 심화**: `Try`, `Either`, `Option` 등이 공통적으로 구현하는 Monad 패턴의 수학적 기반을 이해합니다.
- **Validation 타입**: 여러 검증 오류를 누적(Accumulate)하는 `Validation` 타입을 학습합니다.
- **함수 합성(Function Composition)**: `andThen`, `compose` 등을 사용한 고급 함수 합성 기법을 익힙니다.
- **비동기 처리**: `CompletableFuture`와 `Try`/`Either`를 결합하여 비동기 예외 처리를 구현합니다.

이 문서를 학습한 개발자는 제시된 원리를 응용하여 최소한 **50줄 이상의 복잡한 데이터 변환 로직**을 Stream API와 람다식을 통해 **부수 효과 없이** 작성할 수 있는 지식 기반을 갖추게 되었습니다. 실제 프로젝트에 적용하면서 Functional Programming의 실용성을 체감하시기 바랍니다.