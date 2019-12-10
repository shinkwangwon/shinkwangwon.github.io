---
layout: post
title: "Java 컴파일 과정"
tags: [java]
comments: true
date: 2019-12-06
---

## Java 컴파일 과정
1. 개발자가 작성한 .java 파일을 토대로 컴파일러(javac)가 .class(=바이트코드)를 생성해낸다.
  - 어휘 분석(Lexical Ananlysis) : 소스코드를 문자 단위로 읽어서 유의미한 토큰을 만듬
  - 구문 분석(Syntax Analysis) : Parser라고도 하며 어휘분석의 결과로 나온 토큰들이 정해진 문법에 맞는지 검사 후 파스트리 생성. 맞지 않으면 컴파일 에러
  - 의미 분석(Symantic Analysis) : 타입 검사, 자동 타입 변환 등 검사
  - 중간 코드 생성(Intermediate Code Generation) : 기계어로 변환하기 좋은 형태로 코드를 생성. 자바의 바이트 코드에 해당한다고 볼 수 있음. 이 단계에서 클래스나 인터페이스별 상수 풀이 만들어진다. 상수 풀에 저장된 정보는 해당 클래스나 인터페이스가 실제 생성될 때 런타임 상수 풀을 구성하는데 사용된다.
  - 중간 코드 최적화(Code Optimization) : 더 효율적인 기계어로 변환될 수 있도록 최적화 과정 수행
2. Class Loader는 바이트코드를 토대로 JVM이 운영체제로부터 할당받은 메모리 영역인 Runtime Data Area로 적재하는 역할을 한다.
  - 런타임 시에 동적으로 클래스를 메모리 영역에 배치 시킨다.
3. 실행 엔진은 Class Loader에 의해 메모리에 적재된 바이트 코드를 기계어로 변경해 명령어 단위로 실행하는 역할을 한다.
  - 자바 실행엔진 종류
  - Interprter (인터프리터) : 자바 바이트 코드를 한줄씩 해석하며 기계어 코드로 변환 후 실행하는 방식
  - JIT Compiler (Just In Time) : 바이트 코드를 한번에 기계어 코드로 변환 후 캐싱하고 있다가 실행만 시킴


#### 참고
- <https://homoefficio.github.io/2019/01/31/Back-to-the-Essence-Java-%EC%BB%B4%ED%8C%8C%EC%9D%BC%EC%97%90%EC%84%9C-%EC%8B%A4%ED%96%89%EA%B9%8C%EC%A7%80-1/>