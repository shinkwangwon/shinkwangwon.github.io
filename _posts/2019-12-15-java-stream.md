---
layout: post
title: "Java Stream"
tags: [java]
comments: true
date: 2019-12-15
---

## Stream 이란
- JAVA의 Stream은 컬렉션(Collection)의 요소를 하나씩 참조하여 람다식으로 처리할 수 있게 해주는 일종의 반복자이다.

## Stream 특징
- 스트림은 외부 반복을 통해 작업하는 컬렉션과는 달리 내부 반복(internal iteration)을 통해 작업을 수행
- 스트림은 재사용이 가능한 컬렉션과는 달리 단 한 번만 사용 가능
- 스트림은 원본 데이터를 변경하지 않음
- 스트림의 연산은 필터-맵(filter-map) 기반의 API를 사용하여 지연(lazy) 연산을 통해 성능 최적화
- 스트림은 parallelStream() 메소드를 통한 손쉬운 병렬 처리 지원 
- 스트림 생성 - 중계(중간) - 최종 연산을 거친다. 최종 연산을 수행하기 전에는 중간 연산을 수행하지 않는다. (지연된 연산 수행)
- 간단한 예제 

```java
Person p1 = new Person("shin", 29, "SILVER");
Person p2 = new Person("park", 30, "SILVER");
Person p3 = new Person("kim", 31, "GOLD");
Person p4 = new Person("lee", 31, "VIP");

List<Person> personList = Arrays.asList(p1,p2,p3,p4);

// 등급이 SILVER인 사람의 이름만 추출해내기
List<String> silverPersonNameList = personList.stream()
        .filter(p -> p.getGrade().equals("SILVER"))
        .map(Person::getName)
        .collect(Collectors.toList());

System.out.println(silverPersonNameList);

// 결과 : [shin, park]
```

## 원시타입 스트림
- 자바 8에서는 int double같은 원시 타입을 지원하는 IntStream, DoubleStream과 같은 인터페이스도 제공한다.
- 자바에서 원시타입 스트림을 사용하면 객체 생성 비용이 없기 때문에 데이터를 훨씬 효율적으로 처리할 수 있다.

## 메소드 참조 (::)
- 메소드 참조(Method Reference)는 말 그대로 메소드를 참조해서 매개 변수의 정보 및 리턴 타입을 알아내어, 람다식에서 불필요한 매개 변수를 제거하는 것이 목적이다.
- 위의 예제에서 .map(Person::getName) 이 메소드 참조를 사용한 방식인데, 메소드 참조를 안쓰면 .map(p -> p.getName()) 과 같이 풀어써야 한다.
- [Java 람다(Lambda)](https://shinkwangwon.github.io/java-lambda/) 

## 원시타입 스트림
- 자바 8에서는 int double같은 원시 타입을 지원하는 IntStream, DoubleStream과 같은 인터페이스도 제공한다.
- 자바에서 원시타입 스트림을 사용하면 객체 생성 비용이 없기 때문에 데이터를 훨씬 효율적으로 처리할 수 있다.

## parallelStream()
- 위 예제에서 stream() 대신 parallelStream()을 사용할 경우 처리를 병렬적으로 수행할 수 있다.
- parallelStream() 메서드로 하는 병렬 처리는 내부적으로 ForkJoinPool.commonPool() 메서드에서 반환하는 공통 스레드 풀을 사용한다. 스레드 개수의 기본 값은 (CPU개수 - 1)개를 기본으로 한다.
- 스레드 개수를 지정하고 싶다면 시스템 속성값을 지정해야한다.
- System.setProperty("java.util.concurrent.ForkJoinPool.common.parallelism", "20");


## 클로저
- filter메소드에 전달한 익명함수는 함수 범위 밖의 자유 변수인 grade를 참조하는 클로저다. 
- 아래 예제를 보듯이 grade 변수는 final 형태가 아니다. 변경 가능한 상태라고 생각할 수 있다.
- 하지만, grade의 값을 클로저 안에서 바꿀 수 있는 것은 아니다(1번 컴파일에러). 심지어 클로저 밖에서 바꾸려고 해도 filter에서 접근하려는 라인에서 컴파일 에러(2번 컴파일에러)가 발생한다.
- 즉, final키워드만 붙이지 않았을 뿐 사실상 final 과 같이 취급한 변수만 클로저안에서 접근할 수 있는 것이다.
> 컴파일 에러 : Local variable grade defined in an enclosing scope must be final or effectively final.

```java
public static void closureStream(List<Person> personList, String grade) {
    
    // grade = "VIP"; // 2.컴파일에러
    List<String> silverPersonNameList = personList.stream()
    .filter(p -> {
        // grade = "VIP"; // 1. 컴파일 에러
        return p.getGrade().equals(grade);	// 2. 컴파일에러
    })
    .map(Person::getName)
    .collect(Collectors.toList());
    
    System.out.println(silverPersonNameList);
}
```