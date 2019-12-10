---
layout: post
title: "Java JVM 메모리 구조"
tags: [java]
comments: true
date: 2019-12-10
---

## 자바 메모리 구조(Runtime Data Area)
### 클래스 영역
- Class Area, Method Area, Static Area 라고 불린다.
- 클래스 영역에는 아래와 같은 정보들이 저장되는데, 클래스 사용 이전에 메모리에 할당된다.
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
- New Area(Eden/Survivor) / Old Area / Permanent Generation 영역으로 구분되어 진다. => 자바 8부터는 Permanent Generation영역이 없어지고 Metaspace 영역이 생겨났다.
- String Constant Pool 도 힙영역에 있어서 GC의 대상이 됨

### Permanent Generation 영역
- 자바8 부터는 없어지고 Metaspace영역으로 대체되었다.
- 클래스 로더에 의해 로드되는 클래스와 메소드 등에 대한 메타 정보가 저장되는 영역으로 JVM에 의해 사용된다. 리플렉션을 사용하여 동적으로 클래스가 로딩되는 경우에 사용된다. 
- Class의 메타정보들이 저장되는 영역이며, 이는 JVM에서 class 정보를 토대로 메모리 상에 object로 생성할 때 효과적으로 빠르게 생성하기 위해 저장하는 것이다.
- Permanent Generation 영역은 힙영역이다, 클래스 영역이다, 의견이 분분하다 (네이버를 만든 기술-자바편 책에서는 PermGen 영역을 클래스 영역이라고 함)
- 자바8 전에 PermGen에 저장되는 정보들
  * Class의 메타정보 - 패키지경로+클래스명 , 메소드의 메타 정보 => Class, Method Meta data의 증가로 인한 OOM:permGen 발생 (hotDeploy가 원인)
  * Static Object ==> Collection Object를 static object으로 선언하고 계속 값을 추가하면 OOM:permGen 발생했었음 
  * 상수화된 String Object (interned String)
- 자바8 이후에 옮겨진 위치
  * Class의 메타정보, 메소드의 메타정보 ==> Native 영역으로 이동(Native영역은 Metaspace영역을 뜻함)
  * Static Object ==> Heap 영역으로 이동 (static object를 Heap영역으로 옮김으로써 GC의 대상이 될 수 있도록 함 )
  * 상수화된 String Object (interned String) ==> Heap 영역으로 옮김으로써 GC의 대상이 될 수 있도록 함
    
### Metaspace 영역
- Metaspace는 자바8 이후로 생겨난 메모리 영역이며 Native Memory 영역에 속한다. 즉, OS의 메모리를 직접 할당받아 사용한다.
- Native Memory는 JVM이 아닌 OS가 관리하는 영역이라서 OS에서 사용가능한 메모리 영역 모두 사용 가능한데, 주로 MaxMetaSpaceSize를 지정해서 Native Memory의 특정 공간만 사용한다. (당연히 하나의 자바 어플리케이션이 OS의 모든 메모리를 쓰는건 말이 안되지 않나 싶다.)
- permanant generation 이 metaspace로 변경됨에 따라  java.lang.OutOfMemoryError : PermGen space 이 OOM 에러는 더이상 나오지 않게 됐다.
- 클래스의 메타정보, 메소드의 메타정보 등이 PermGen 영역에서 Metaspace로 옮겨졌는데 PermGen영역은 런타임시에 사이즈를 조절할 수 없었지만 Metaspace는 사이즈가 부족할 경우 자동으로 사이즈를 늘려준다. 그렇기에 PermGen이 있을때보다 OOM 발생확률이 매~우 낮아졌다.

### Native 메소드 영역
- 자바 외의 다른 언어에서 제공되는 메서드들이 저장되는 공간이다.

### PC 레지스터
- Thread가 생성될 때마다 생성되는 공간으로 Thread가 어떤 부분을 어떤 명령으로 실행할지에 대한 기록을 담고 있다. 현재 실행되는 부분의 명령과 주소를 저장한다. 


------------
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
- <https://12bme.tistory.com/142>
- <https://yckwon2nd.blogspot.com/2015/03/java8-permanent.html>