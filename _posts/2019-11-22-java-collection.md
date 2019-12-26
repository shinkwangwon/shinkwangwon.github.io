---
layout: post 
title: "Java Collection Basic"
tags: [java]
comments: true
date: 2019-11-28
---


## Collection 이란 ?
- 컬렉션은 여러 요소들을 담을 수 있는 자료구조다.
- 배열과 비슷하지만 배열은 크기가 고정되지만, 컬렉션은 크기가 리사이징이 가능한 특성을 지닌다.
- 즉, 컬렉션 프레임워크는 배열의 문제점을 해결하고, 널리 알려져 있는 자료구조를 바탕으로 객체들을 효율적으로 추가,삭제,검색할 수 있도록 java.util 패키지에 컬렉션과 관련된 인터페이스와 클래스들을 포함시킨 것이다.

## Collections 란 ?
- Collections 클래스는 모든 컬렉션의 알고리즘을 담당한다. -> Collection 인터페이스가 아니라 Collections 클래스다(끝에 "s" 주의)
- 유틸리티 클래스로 static 메소드로 구성되어 있고 컬렉션들을 컨트롤하는데 사용된다.
- Collections 클래스는 정렬, 셔플링, 이진검색, 뒤집기 등의 메서드를 가지고 있다.

## Collection Interface
- 컬렉션 인터페이스들은 제네릭으로 표현되어 컴파일 시점에서 객체의 타입을 체크하기 때문에 런타임 에러를 줄이는데 도움이 된다.
- 컬렉션 프레임워크의 대표적인 인터페이스 : List인터페이스 , Set인터페이스, Map인터페이스
- List와 Set은 Collection인터페이스를 상속(인터페이스끼리는 extends이다)받기 때문에 List와 Set의 공통된 부분은 Collection인터페이스에 정의하고 있다.
- 반면 Map 인터페이스는 구조상의 차이(Key & value)로 인해 Collection 인터페이스를 상속받지 않고 별도로 정의된다.

![collection](/assets/posts/20191122/java_collection.png)

1. List
- 요소들의 순서가 있으며, index를 사용하여 탐색/삽입하고 중복 값을 허용
- ArrayList : 내부적으로 데이터를 배열로 관리하며 데이터의 추가/삭제시 임시 배열을 생성해서 데이터를 복사하는 방법을 사용함. 추가/삭제가 많을 경우 데이터의 복사가 많이 일어나서 성능이 느릴 수 있지만 인덱스를 알고 있기 때문에 접근하는데는 빠르다.
- LinkedList : 각 데이터노드가 이전/다음 노드의 상태만 알고 있다. ArrayList와는 다르게 데이터 복사가 없기 때문에 추가/삭제시에 유리한 반면 검색시에는 처음부터 순회해야 하기 때문에 성능상 느릴 수 있다.
- Vector : 가변 길이의 배열과 비슷하며 동기화(synchronized)를 지원한다. 배열용량이 초과하면 기존 사이즈의 2배로 늘린다. 2배로 늘려놓고 안쓸 수 있기 때문에 성능이 좋지 않을 수 있다.

2. Set
- 집합을 정의하며 요소의 중복을 허용하지 않음
- HashSet : 순서를 전혀 알 수 없음. 임의의 접근 속도는 가장 빠름
- LinkedHashSet : 추가된 순서대로 저장
- TreeSet : 정렬된 순서로 저장하며 정렬방법을 지정할 수 있음

3. Map
- Key & Value 쌍으로 연관지어 저장하고, 중복을 허용하지 않음
- HashMap : 해시테이블을 사용한 클래스, 중복을 허용하지않고 순서를 보장하지 않음, 키와 값으로 null을 허용
- HashTable : HashMap보다는 느리지만 동기화를 지원(주요 Syncronized 키워드가 붙어있음), 키와 값으로 null 허용하지 않음 (NullPointerException)
- LinkedHashMap : 입력한 순서대로 저장, HashMap을 상속받기 때문에 HashMap과 거의 흡사
- TreeMap : 정렬된 순서로 저장, 저장시 정렬하기 때문에 저장시간이 다소 오래 걸림

```java
Map<String, String> hashMap = new HashMap<>();
hashMap.put("a",  "aa");
hashMap.put("a",  "bb");	// 중복허용 안함 : "bb"로 덮어 씌워짐
hashMap.put(null, null);
System.out.println(hashMap);	// {null=null, a=bb}

Map<String, String> hashTable = new Hashtable<>();
hashTable.put("a", "aa");
hashTable.put("a", "bb");	// 중복허용 안함 : "bb"로 덮어 씌워짐
// hashTable.put(null, null);	// ERROR: java.lang.NullPointerException
System.out.println(hashTable);	// {a=bb}

Map<String, String> concurrentHashMap = new ConcurrentHashMap<>();
concurrentHashMap.put("a", "aa");
concurrentHashMap.put("a", "bb");	// 중복허용 안함 : "bb"로 덮어 씌워짐
// concurrentHashMap.put(null, null);	// ERROR: java.lang.NullPointerException
System.out.println(concurrentHashMap);
```

## HashTable, HashMap, ConcurrentHashMap
* HashTable
  - 주요 메소드에 synchronized 키워드가 선언되어 있어서 동기화를 지원함(그렇기에 속도가 조금 느림)
  - key, value에 null을 허용하지 않음 - key, value 둘중 하나라도 null이면 에러

* HashMap
  - synchronized 키워드가 없고, key,value에 null 입력 가능
  - 개발자가 직접 synchronized 블록을 명시적으로 선언해서 사용은 가능

* ConcurrentHashMap
  - HashMap을 thread-safe하게 만든 클래스
  - 다만, hashMap과 다르기 key, value에 null 허용하지 않음
  - putIfAbsent(key, value) 라는 메소드를 제공함 - key 값이 존재하면 기존의 값을 반환하고 없다면 입력한 값을 저장한뒤 반환 


#### 참고 
- <http://blog.breakingthat.com/2018/05/07/java-collection-%EA%B0%9C%EC%9A%94-%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/>
- <https://gbsb.tistory.com/247>