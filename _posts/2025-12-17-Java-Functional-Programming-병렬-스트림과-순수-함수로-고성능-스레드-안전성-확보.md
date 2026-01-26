---
layout: post
title: Java Functional Programming - 병렬 스트림과 순수 함수로 고성능 스레드 안전성 확보
date: 2025-12-17 00:00:00
description: Java 병렬 스트림(Parallel Stream)과 순수 함수(Pure Function)의 관계를 심층 분석하여 멀티코어 환경에서 안전하게 성능을 최적화하는 방법을 학습합니다. 부수 효과(Side Effect) 없는 불변성 로직으로 경합 조건을 회피하고, 명령형 코드 대비 선언적인 고성능 데이터 처리 파이프라인을 구축하는 핵심 원칙을 제시합니다.
tags: [자바, 함수형 프로그래밍]
categories: [Java]
giscus_comments: true
author: hwangrolee
pin: true
image:
  path: /assets/img/Java-Functional-Programming-Achieving-High-Performance-Thread-Safety-with-Parallel-Streams-and-Pure-Functions.png
  alt: "Java Functional Programming - 병렬 스트림과 순수 함수로 고성능 스레드 안전성 확보"
---

> 이 글은 Functional Programming 개념 및 활용법을 자바 기반으로 공부하기 위해 Gemini, Claude의 도움을 받아 작성하였습니다.

## Java 병렬 스트림과 순수 함수: 고성능 스레드 안전성 확보

Java 8 Stream API의 **병렬 스트림(Parallel Stream)** 은 선언적인 **Functional Programming** 스타일을 유지하면서 대용량 데이터를 멀티코어 프로세서에 분산 처리하여 성능을 극대화하는 기능입니다. 이 기능을 안전하고 효율적으로 사용하기 위해서는, 데이터 처리 로직을 **순수 함수(Pure Function)**로 구성하는 것이 핵심 원칙입니다.

---

### 왜 병렬 스트림과 순수 함수를 학습해야 하는가

현대의 서버 환경은 대부분 멀티코어 프로세서를 사용하며, 데이터를 **순차적(Sequential)**으로 단일 스레드에서 처리하는 것은 컴퓨팅 자원의 비효율적인 사용으로 이어집니다. 병렬 스트림은 `parallelStream()` 메서드 호출 한 번으로 컬렉션 처리를 여러 CPU 코어에 자동으로 할당하여 데이터 처리 속도를 향상시키는 구조적 이점을 제공합니다.

그러나 병렬 처리는 여러 스레드가 **공유되는 가변 상태(Shared Mutable State)**에 동시에 접근할 때 **경합 조건(Race Condition)**을 필연적으로 발생시킵니다. 이로 인해 예측 불가능하고 비결정적인 결과(**Non-Deterministic Result**)가 초래될 수 있습니다.

**Functional Programming**에서 강조하는 **순수 함수**의 사용은 이러한 병렬 환경에서 **스레드 안전성(Thread Safety)**을 확보하는 가장 강력하고 기본적인 해결책입니다. 순수 함수를 사용함으로써 **명령형(Imperative) 코드 스타일**에서 흔히 발생하는 동시성 문제를 근본적으로 회피하고, 고성능 애플리케이션의 **테스트 용이성**을 높일 수 있습니다. 따라서 고성능 및 고신뢰성 코드를 개발하기 위해서는 병렬 스트림의 성능 이점과 순수 함수의 안전성 원칙을 명확히 이해해야 합니다.

---

### 병렬 스트림의 성능 이점과 순수 함수의 역할

병렬 스트림은 내부적으로 **Spliterator**를 사용하여 데이터를 분할하고, **Fork/Join Framework**를 통해 이 분할된 데이터를 여러 스레드에 할당하여 처리합니다. 이 과정에서 `map`, `filter`, `forEach`와 같은 중간 및 최종 연산에 사용되는 모든 람다 표현식이나 메서드 참조는 반드시 **순수 함수**의 조건을 만족해야 합니다.

#### 순수 함수의 정의와 병렬성

**순수 함수**는 다음 두 가지 핵심 조건을 만족해야 합니다.

1.  **동일한 입력에 대해 항상 동일한 출력을 반환합니다.** (비결정성 회피)
2.  **함수 실행이 함수 외부의 상태를 변경하지 않습니다.** (**No Side Effect**).

병렬 스트림 환경에서 여러 스레드가 동시에 `map` 연산을 수행한다고 가정합니다. 만약 함수가 외부에 정의된 변수(**공유되는 가변 상태**)를 수정하는 **부수 효과**를 유발한다면, 여러 스레드 간의 접근 순서에 따라 데이터 불일치나 **경합 조건**이 발생합니다.

반면, 순수 함수는 오직 입력된 데이터만을 기반으로 결과를 계산하고 외부 상태에 영향을 주지 않으므로, 여러 스레드가 동시에 실행되어도 서로 간섭할 염려가 없습니다. 이것이 Functional Programming의 **불변성(Immutability)** 원칙과 순수성이 병렬 프로그래밍에서 **스레드 안전성**을 자연스럽게 보장하는 근본적인 이유입니다.

#### 코드 예시: 성능 비교 및 순수 함수의 구현

다음 예시는 시간이 소요되는 작업을 **순차 스트림**과 **병렬 스트림**으로 처리할 때의 성능 차이를 보여주며, 이때 `slowSquare` 함수가 **순수 함수**로 정의되어 병렬 실행의 안전성을 보장하고 있음을 시사합니다.

```java
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

public class PureFunctionParallelism {

    // 순수 함수 (Pure Function): 외부 상태 변경 없음, 입력 외에 의존하는 요소 없음.
    private static int slowSquare(int number) {
        try {
            // 실제 환경에서 복잡한 계산이나 I/O 등으로 시간이 걸린다고 가정
            Thread.sleep(1);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        return number * number; // 오직 입력(number)만을 기반으로 출력 결정
    }

    public static void main(String[] args) {
        // 1000개의 데이터 생성
        List<Integer> numbers = IntStream.range(0, 1_000).boxed().collect(Collectors.toList());

        // 1. 순차 스트림 (Sequential Stream) 처리
        long startSeq = System.currentTimeMillis();
        numbers.stream()
               .map(PureFunctionParallelism::slowSquare)
               .collect(Collectors.toList());
        long endSeq = System.currentTimeMillis();
        System.out.println("순차 처리 시간: " + (endSeq - startSeq) + "ms");

        // 2. 병렬 스트림 (Parallel Stream) 처리
        long startPar = System.currentTimeMillis();
        numbers.parallelStream()
               .map(PureFunctionParallelism::slowSquare) // 순수 함수를 사용하므로 안전
               .collect(Collectors.toList());
        long endPar = System.currentTimeMillis();
        System.out.println("병렬 처리 시간: " + (endPar - startPar) + "ms");
    }
}
```

실행 결과, `slowSquare` 함수가 **순수 함수**의 조건을 만족하므로 병렬 스트림 사용 시 성능 향상과 더불어 **스레드 안전성**을 확보할 수 있습니다.

---

### 경고: Side Effect가 있는 함수의 위험성

만약 위의 `slowSquare` 함수 내에서 외부에 정의된 `static int count` 변수를 증가시키는 코드를 추가한다면, 이는 **부수 효과**를 유발하는 함수가 되며 병렬 스트림에서 사용될 경우 반드시 데이터 불일치 문제를 초래합니다.

**명령형 코드 스타일**에서처럼 **공유되는 가변 상태**를 수정해야 하는 경우에는, 개발자가 명시적으로 `synchronized` 블록이나 `Atomic` 계열의 클래스를 사용하여 동시성 제어 문제를 직접 해결해야 합니다. 그러나 이러한 방식은 **Functional Programming**의 철학에 위배될 뿐만 아니라, 코드의 복잡성을 증가시키고 병렬 처리의 성능 이점을 상쇄할 수 있습니다.

따라서 병렬 스트림을 사용할 때는 **순수 함수**를 사용하여 **부수 효과** 자체를 없애고 **불변성**을 유지하는 것이 **선언적**인 코드 설계 방향이자, 안정적인 고성능 애플리케이션을 구축하는 핵심 전략입니다.
