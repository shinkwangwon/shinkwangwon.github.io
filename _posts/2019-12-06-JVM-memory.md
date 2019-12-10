---
layout: post
title: "Java 컴파일 과정 & JVM 메모리 구조"
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


## 자바 메모리 구조(Runtime Data Area)
### 클래스 영역
- Class Area, Method Area, Static Area 라고 불린다.
  * 클래스 영역에는 아래와 같은 정보들이 저장되는데, 클래스 사용 이전에 메모리에 할당된다.
  * 필드 정보 : 멤버변수의 이름, 데이터 타입, 접근 제어자에 대한 정보
  * 메소드 정보 : 메소드의 이름, 리턴 타입, 매개변수, 접근 제어자에 대한 정보
  * 타입 정보 : Type의 속성이 Class 인지 Interface인지 여부 저장 - Type의 전체 이름(패키지 경로+클래스명) 저장
  * Constant Pool(상수 풀) : Type에서 사용된 상수를 저장. 문자상수, 타입, 필드, Method의 Symbolic Reference(객체 이름으로 참조)를 상수풀에 저장
  * 클래스 변수 : Static변수라고도 하며 모든 객체에서 공유하는 변수 저장
  * final 클래스 변수 : final 클래스 변수의 경우 상수로 치환되어 상수 풀에 저장

### 스택 영역
- 메소드 호출 시마다 각각의 스택프레임이 생성되고, 메소드가 끝나면 해당하는 스택프레임이 삭제된다. (Last In First Out)
- 메소드 안에서 사용되어지는 값들을 저장한다 - 호출된 메소드의 매개변수, 지역변수, 리턴값, 연산시 일어나는 임시 값 등

### 힙 영역
- new 연산자로 생성된 객체와 배열을 저장하는 영역이다. 클래스 영역에 로드된 클래스만 생성이 가능하다
- GC(Garbage Collection)의 대상이 되는 영역으로써 GC 알고리즘에 따라 메모리를 반환하게 된다.
- New Area(Eden/Survivor) / Old Area / Permanent Generation 영역으로 구분되어 진다.
- String Constant Pool 도 힙영역에 있어서 GC의 대상이 됨
- ######## ######## ######## 
   - Permanent Generation 
   - 생성된 객체들의 정보의 주소 값이 저장된 공간 
   - 클래스 로더에 의해 로드되는 클래스와 메소드 등에 대한 메타 정보가 저장되는 영역으로 JVM에 의해 사용된다. 리플렉션을 사용하여 동적으로 클래스가 로딩되는 경우에 사용된다. 
   - Class의 메타정보들이 저장되는 영역이며, 이는 JVM에서 class 정보를 토대로 메모리 상에 object로 생성할 때 효과적으로 빠르게 생성하기 위해 저장하는 것이다.
   - Hot-deploy 수행 시 Custom ClassLoader가 해제되지 않고 계속 존재하여 동일한 class들이 ClassLoader로 로드되어 Permanent 영역 Full이 나는 것입니다.
   - 기존에 PermGen저장되는 정보들
     * Class의 메타정보 - 패키지경로+클래스명 , 메소드의 메타 정보 => Class, Method Meta data의 증가로 인한 OOM:permGen 발생 (hotDeploy가 원인)
     * Static Object ==> Collection Object를 static으로 선언하고 계속 값을 추가하면 OOM:permGen 발생했었음 
     * 상수화된 String Object (interned String)
     * Class와 관련된 배열 객체 메타정보
     * JVM 내부적인 객체들과 최적화 컴파일러의 최적화 정보
   - PermGen에서 옮겨진 위치
     * Class의 메타정보, 메소드의 메타정보 ==> Native 영역으로 이동(Native영역은 Metaspace영역을 뜻함)
     * Static Object ==> Heap 영역으로 이동 (static object를 힙영역으로 옮김으로써 GC의 대상이 될 수 있도록 함 )
     * 상수화된 String Object (interned String) ==> Heap 영역으로 이동
     * Class와 관련된 배열 객체 메타정보 ==> 조사 필요
     * JVM 내부적인 객체들과 최적화 컴파일러의 최적화 정보 ==> 조사 필요

- 네이버를 만든 기술-자바편 책에서는 PermGen 영역을 메소드 영역이라고 함 

### Native 메소드 영역
- 자바 외의 다른 언어에서 제공되는 메서드들이 저장되는 공간이다.

### PC 레지스터
- Thread가 생성될 때마다 생성되는 공간으로 Thread가 어떤 부분을 어떤 명령으로 실행할지에 대한 기록을 담고 있다. 현재 실행되는 부분의 명령과 주소를 저장한다. 


## 클래스변수, 인스턴스변수, 지역변수
- 클래스변수 : 클래스의 static 변수로써 모든 인스턴스가 공유한다. 클래스 로딩시에 메모리에 할당된다(메모리에 한번만 올라감).
- 인스턴스 변수 : 인스턴스마다 다른 값을 갖는다.(공유되지 않는다) 클래스의 인스턴스가 생성될때 메모리에 올라가기 때문에 인스턴스 생성후 접근이 가능하다.
- 지역변수 : 메서느 내에서 선언되는 지역변수. 메서드가 실행될 때 메모리에 할당받고 메소드가 끝나면 소멸된다.
- 멤버변수 : 클래스변수와 인스턴스변수를 멤버변수라고 한다.


## Java 초기화 블럭 (Initialization Block)
1. 클래스 초기화 블럭
    - 클래스 변수(static변수) 의 복잡한 초기화에 사용됨. 클레스가 처음 로딩될 때 한번만 수행된다.
2. 인스턴스 초기화 블럭
    - 인스턴스 변수의 복잡한 초기화에 사용된다. 인스턴스가 생성될 때마다 수행된다. (생성자보다 먼저 수행됨)
```java
class TestBlock {
    static { /* 클래스 초기화 블럭 */ }

    { /* 인스턴스 초기화 블럭 */ }
}
```

#### 참고
- <https://homoefficio.github.io/2019/01/31/Back-to-the-Essence-Java-%EC%BB%B4%ED%8C%8C%EC%9D%BC%EC%97%90%EC%84%9C-%EC%8B%A4%ED%96%89%EA%B9%8C%EC%A7%80-1/>
- <https://12bme.tistory.com/142>
- <https://yckwon2nd.blogspot.com/2015/03/java8-permanent.html>