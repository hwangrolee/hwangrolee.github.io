---
layout: post
title: Java Functional Programming - 불변성 (Immutability)
date: 2025-11-20 00:00:00
description: 자바(Java) Functional Programming의 핵심 설계 원칙인 불변성(Immutability)에 대한 심층 가이드입니다. 불변 객체가 고품질 애플리케이션 구축의 필수 요소인 이유를 설명하고, 가변 객체(Mutable Object) 사용 시 발생하는 동시성 문제와 예측 불가능성을 명확히 비교합니다. 완벽한 불변 클래스를 설계하기 위한 5가지 필수 원칙(final 클래스, private final 필드, 깊은 복사, 방어적 복사)을 구체적인 Java 8+ 코드 예시와 함께 제시합니다. 이를 통해 멀티스레딩 환경에서 스레드 안전성(Thread Safety)을 확보하고, Functional Programming의 데이터 변환 패턴(`with*` 메서드)을 활용하는 방법을 습득할 수 있습니다.
tags: [자바, 함수형 프로그래밍]
categories: [Java]
author: hwangrolee
image:
  path: /assets/img/01-03-java-fp-immutability.png
  alt: "Java Functional Programming - 불변성 (Immutability)"
---

> 이 글은 Functional Programming 개념 및 활용법을 자바기반으로 공부하기 위해 Gemini, Claude 의 도움을 받아 작성하였습니다.

불변성(Immutability)은 객체가 생성된 후에는 **내부 상태를 절대 변경할 수 없도록** 설계하는 Functional Programming의 핵심 원칙입니다. 데이터를 수정하는 대신, 변경이 필요할 때마다 **기존 데이터를 복사하여 새로운 객체**를 반환하도록 설계합니다. 이 원칙을 준수하면 객체의 상태를 추적하기 쉬워져 프로그램의 **예측 가능성**이 극대화되며, 특히 멀티스레딩 환경에서 **스레드 안전성(Thread Safety)**을 보장하여 동시성 문제를 근본적으로 해결할 수 있습니다. 불변성은 고품질의 견고한 애플리케이션을 구축하기 위한 필수적인 설계 기법입니다.

---

## 1. 불변성의 중요성과 이점

가변 객체(Mutable Object)는 여러 메서드나 스레드에 의해 예측할 수 없는 시점에 상태가 변경될 수 있습니다. 이는 복잡한 시스템의 버그를 유발하는 주된 원인입니다.

- **스레드 안전성(Thread Safety) 확보:** 불변 객체는 상태를 변경할 수 없으므로, 여러 스레드가 동시에 접근해도 데이터 충돌이나 경쟁 조건(Race Condition)이 발생하지 않습니다. 별도의 동기화(Synchronization) 메커니즘이 필요 없습니다.
- **쉬운 캐싱 및 공유:** 상태가 영구적으로 고정되어 있으므로, 한 번 생성된 객체는 복사본 없이 안전하게 공유(Share)할 수 있으며, 해시 코드(Hash Code)를 캐싱하여 성능 최적화에 사용할 수 있습니다.
- **코드의 명확성:** 상태 변화가 없기 때문에 객체의 동작을 예측하기 쉽고, 디버깅이 용이해집니다.

---

## 2. 불변 클래스 설계의 필수 원칙

자바에서 완벽하게 불변하는 클래스를 설계하려면 다음의 다섯 가지 원칙을 엄격하게 준수해야 합니다.

#### 2.1. 클래스 선언: `final` 키워드 사용

클래스를 `final`로 선언하여 다른 클래스가 이 클래스를 **상속(Extend)하는 것을 방지**해야 합니다. 상속을 허용하면 자식 클래스에서 객체의 불변성을 해치는 메서드를 추가할 위험이 있습니다.

#### 2.2. 모든 필드: `private final` 키워드 사용

클래스 내부의 **모든 필드**를 `private`으로 선언하여 외부 접근을 차단하고, `final`로 선언하여 **객체 생성 시 단 한 번만 초기화**되도록 강제해야 합니다.

#### 2.3. Setter 메서드 제공 금지

필드의 값을 변경하는 **Setter 메서드**(`set*`)를 절대로 제공하지 않아야 합니다. 객체의 상태 변경을 허용하는 모든 공용(public) 메서드는 금지되어야 합니다.

#### 2.4. 생성자: 깊은 복사(Deep Copy)를 통한 초기화

생성자에서 인자로 **가변 객체(Mutable Object, 예: `List`, `Date`)**를 받았다면, 반드시 그 객체를 **깊은 복사**하여 내부 필드에 저장해야 합니다. 만약 원본 객체의 참조를 그대로 저장하면, 외부에서 원본 객체를 수정했을 때 내부 필드의 상태가 바뀌어 불변성이 깨집니다.

#### 2.5. Getter: 방어적 복사(Defensive Copy)를 통한 반환

Getter 메서드가 내부의 **가변 객체**를 반환해야 할 경우, 단순히 참조를 반환하는 대신 **반드시 복사본**을 반환해야 합니다. 참조를 그대로 반환하면, 외부에서 반환된 객체를 수정하여 내부 상태가 변할 수 있습니다.

---

## 3. Immutable 클래스 설계 예시

Java의 `String` 클래스나 `Integer` 클래스가 불변 객체의 대표적인 예시입니다. 다음은 간단한 `Point` 클래스를 불변 객체로 설계하는 예시입니다.

```java
import java.util.Collections;
import java.util.List;

// 1. 클래스를 final로 선언
public final class ImmutablePoint {

    // 2. 모든 필드를 private final로 선언
    private final int x;
    private final int y;

    // (예시를 위한 가변 객체 필드)
    private final List<String> tags;

    // 생성자
    public ImmutablePoint(int x, int y, List<String> tags) {
        this.x = x;
        this.y = y;

        // 4. 생성자에서 가변 인자를 깊은 복사(새로운 List 생성)하여 초기화
        this.tags = List.copyOf(tags); // Java 10 이후 List.copyOf 사용
    }

    // 3. Setter 메서드 없음 (상태 변경 메서드 없음)

    // Getter 메서드
    public int getX() { return x; }
    public int getY() { return y; }

    // 5. Getter: 가변 객체를 반환할 때 방어적 복사본을 반환
    // List.copyOf()로 이미 불변 리스트를 만들었지만,
    // 만약 clone()이 불가능한 클래스라면 Collections.unmodifiableList() 등을 사용
    public List<String> getTags() {
        return Collections.unmodifiableList(tags); // 외부 수정을 막기 위한 방어
    }

    // Functional Programming 스타일의 상태 변경(새 객체 생성) 메서드
    public ImmutablePoint move(int dx, int dy) {
        // 기존 객체를 수정하지 않고, 새로운 상태를 가진 새 객체를 생성하여 반환
        return new ImmutablePoint(this.x + dx, this.y + dy, this.tags);
    }
}
```

## 4. `with*` 메서드를 이용한 Functional Programming 패턴

불변 객체에서 상태를 변경하는 것처럼 보이는 메서드는 실제로는 기존 객체를 기반으로 **새로운 객체**를 생성하여 반환합니다. 관례적으로 이러한 메서드에는 **`with`** 접두사를 붙여 `withName()`, `withTags()`처럼 명명합니다. 이는 Functional Programming에서 흔히 사용되는 **데이터 변환(Transformation)** 패턴입니다.

```java
ImmutablePoint p1 = new ImmutablePoint(10, 20, List.of("A"));
System.out.println("P1: " + p1.getX()); // 출력: 10

// move() 메서드는 P1의 상태를 바꾸지 않고, 새로운 객체 P2를 생성
ImmutablePoint p2 = p1.move(5, 5);
System.out.println("P2: " + p2.getX()); // 출력: 15
System.out.println("P1 (unchanged): " + p1.getX()); // 출력: 10 (P1은 그대로 유지)
```

이러한 불변 객체 설계 방식을 통해, 데이터 처리 파이프라인에서 데이터가 안전하게 흐르고 예측 가능한 결과를 제공하도록 보장할 수 있습니다.
