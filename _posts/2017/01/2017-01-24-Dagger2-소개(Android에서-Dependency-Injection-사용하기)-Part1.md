---
layout: post
category : Android
title : "[Dagger2] Dagger2 소개(Android에서 Dependency Injection 사용하기) Part1"
description : ""
tags : [Android, DI]
---

{% include JB/setup %}

> *이 문서는 [Introduction to Dagger 2, Using Dependency Injection in Android: Part 1](https://blog.mindorks.com/introduction-to-dagger-2-using-dependency-injection-in-android-part-1-223289c2a01b#.8fzywkmju)를 번역한 문서입니다.*

이 문서에는 많은 정보가 포함되어있습니다. 가독성을 위하여 두 부분으로 나누었습니다.

**Part 1**: Dagger2의 개념 및 개요에 대한 소개
**Part 2**: 예제를 통한 Dagger2의 구현

Android에서 Dagger2의 사용법을 이해하려면 먼저 필요성을 이해해야합니다. 중요한 질문은 다음과 같습니다.

## *왜 Dependency Injection이 필요한가?*

**Dependency Injection**은 **[Inversion of Control](https://ko.wikipedia.org/wiki/%EC%A0%9C%EC%96%B4_%EB%B0%98%EC%A0%84)**의 개념을 바탕으로 합니다. 클래스가 외부로부터 의존성을 가져야한다고 말합니다. 간단히 말해 어떤 클래스도 다른 클래스를 인스턴스화 하지 않아도 되지만 구성 클래스에서 인스턴스를 가져와야합니다.

Java 클래스가 new 연산자를 통해서 다른 클래스의 인스턴스를 생성하면 해당 클래스와 독립적으로 사용하고 테스트 할 수 없으므로 **Hard Injection**이라 합니다.

## *그럼, 클래스 외부에서 Dependency를 제공하면 어떤 이점이 있나요?*

가장 중요한 이점은 클래스를 재사용 할 가능성을 높이고 다른 클래스와 독립적으로 클래스를 테스트 할 수 있습니다. 이것은 비지니스 로직의 특정 구현이 아닌 클래스를 생성하는데 굉장한 것으로 들립니다. 이제 조금 이해가 되었으므로 Dependency Injection 탐색을 진행할 수 있습니다.

## *문제는 DI(Dependency Injection)를 어떻게 할 것인가?*

이 질문에 답하기 위해서 우리는 과거를 되돌아 봐야합니다. dependency container라는 프레임워크 클래스는 클래스의 종속성을 분석하는데 사용되었습니다. [Java Reflections](http://gyrfalcon.tistory.com/entry/Java-Reflection)를 통해 클래스의 인스턴스를 생성하고 정의된 종속성에 객체를 삽입하는 것을 분석 할 수 있었습니다. 분석한 결과를 토대로 Hard Injection이 제거되었습니다. 그런식으로 클래스는 모의 객체를 사용하여 테스트 할 수 있었습니다. 이것이 Dagger1입니다.

Dagger1은 두 가지의 주요 단점이 있습니다. 첫째, 리플렉션 자체가 느리고 둘째, 런타임에 종속성 해결을 수행하여 예기치 않은 크래시가 발생했습니다.

**이것이 Dagger2의 탄생으로 이어졌습니다. Dagger2는 Google이 Square의 Dagger1에서 fork했습니다.**

Dagger2에서의 큰 변화는 어노테이션 프로세서를 사용하여 [dependency graph](https://en.wikipedia.org/wiki/Dependency_graph)를 생성하는 것이였습니다. 의존성을 제공하는 클래스는 이제 `javax.injection` 패키지를 사용하여 빌드 시 생성됩니다.이것은 어플리케이션이 실행되기 전에 가능한 오류 검사를 용이하게 합니다. 생성 된 소스는 손으로 쓴 것처럼 쉽게 읽을 수 있습니다.

참고 : [어노테이션 프로세서](http://hannesdorfmann.com/annotation-processing/annotationprocessing101)는 프로젝트에서 사용할 소스 코드 파일을 생성하기 위해 빌드하는 동안 컴파일 된 파일을 읽는 방법입니다.

위 내용들이 당신을 압도했다면 이제 진정한 Android 예제를 시작 할 때까지 기다려야합니다.

DI에 대한 몇 가지 사실을 말씀드리고자 합니다.

클래스의 종속성을 표현하기 위한 표준 Java 어노테이션은 [Java Specification Request 330 (JSR 330)](https://www.jcp.org/en/jsr/detail?id=330)에 정의되어 있습니다.

- 주입방식
  - 생성자 주입 : 메소드 파라미터 주입
  - 필드 주입 : 맴버 변수 주입(private이면 안됨)
  - 메소드 주입 : 메소드 파라미터 주입


- JSR330에 따른 의존성 주입 순서
  - 생성자
  - 필드
  - 메소드

**참고**
1. `@Inject`으로 어노테이션 된 메소드나 필드가 호출되는 순서는 JSR330에는 정의되지 않았습니다. 메소드 또는 필드가 클래스에서 선언된 순서대로 호출된다고 가정 할 수 없습니다.
2. 생성자가 호출 된 후에 필드와 메소드 파라미터가 삽입되므로 생성자에서 삽입된 멤버 변수를 사용할 수 없습니다.

이제 충분한 배경과 정당화를 가지고 Dagger를 이해하기 위해서 계속 진행 할 수 있습니다.

Dagger2를 사용하여 의존성 주입 프로세스를 시각화하는 경향이 있습니다.

**의존성 소비자**는 커넥터를 통해서 **의존성 제공자**의 dependency(Object)를 묻습니다.

- 의존성 제공자(Dependency provider) : `@Module`로 어노테이션된 클래스는 삽입 할 수 있는 객체를 제공합니다. 이러한 클래스는 `@Provides`로 어노테이션된 메소드들을 정의합니다. 이 메소드에서 리턴 된 객체는 dependency injection에 이용할 수 있습니다.
- 의존성 소비자(Dependency consumer) : `@Inject` 어노테이션은 의존성을 정의하는데 사용합니다.
- 소비자와 제공자를 연결(Connect) : `@Component` 어노테이션이 달린 인터페이스는 객체 제공자(`@Module`)와 의존성을 소비하는 클래스 사이의 연결을 정의합니다. 이 연결에 대한 클래스는 Dagger에 의해 생성됩니다.

## Dagger2의 한계

1. Dagger는 필드를 자동으로 주입을 할 수 없습니다.
2. private 필드는 주입 할 수 없습니다.
3. 필드 주입을 사용하려면 `@Component` 어노테이션이 달린 인터페이스에 멤버 변수를 주입 할 클래스의 인스턴스를 취하는 메소드를 정의해야합니다.

이제 Part 2로 이동하여 Dagger2를 실제로 사용해 봅시다. [이 링크](/2017-01-18-Dagger2-소개(Android에서-Dependency-Injection-사용하기)-Part2.md)를 따라주십시오.
