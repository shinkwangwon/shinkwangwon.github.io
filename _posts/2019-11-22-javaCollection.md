---
layout: post 
title: "Java Collection Basic"
categories: java
date: 2019-11-28
---

## Collection Interface
- 모든 Collection의 상위 인터페이스로써 핵심 메소드를 선언 (add, contain, isEmpty, remove, size, iterator ...)

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
- HashTable : HashMap보다는 느리지만 동기화를 지원(주요 Syncronized 키워드가 붙어있음), 키와 값으로 null 허용하지 않음
- LinkedHashMap : 입력한 순서대로 저장, HashMap을 상속받기 때문에 HashMap과 거의 흡사
- TreeMap : 정렬된 순서로 저장, 저장시 정렬하기 때문에 저장시간이 다소 오래 걸림

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



## Java HashMap Algorithm
* Java HashMap 에서 사용하는 방식은 seperate chaining 방식 
* HashMap에 저장된 키-값 쌍 개수가 일정 개수 이상으로 많아지면, 일반적으로 Open Addressing은 Separate Chaining보다 느리다.
* Java 8에서는 seperate chaining에서 링크드 리스트 와 트리를 혼용해서 사용함
* 하나의 해시버킷에 8개 이상의 key&value 쌍이 모이면 트리로 변경
* 하나의 해시버킷이 6개 이하로 줄어들면 다시 링크드 리스트로 변경
* 차이를 "2" 만큼으로 둔것은, 차이가 1이라면 1개의 key&value쌍이 삽입/삭제될때 불필요하게 트리와 링크드 리스트를 변경하는 일이 반복되기 때문이다.
* Hash Map은 데이터 개수가 일정 개수 이상이되면 해시버킷의 크기를 2배로 늘림 => 해시충돌문제를 어느정도 해결 => 단, 모든 데이터에 대한 새로운 Separate Chaining을 구성해야함


## Rehashing (리해싱) 
- 용어 정리 : capacity는 버킷의 총 개수를 말하며, 기본적으로 아무설정 없이 HashMap 생성시 capacity=16, load_factor=0.75로 생성된다.
- if(load_factor == 저장된 데이터 수 / capacity) 가 되는 시점에 자동으로 버킷의 크기를 늘린다. (대략 버킷의 크기를 2배로 늘림)
- 평균적으로 하나의 버킷에 저장된 데이터의 수가 load_factor 랑 같아지면 버킷의 크기를 늘린다는것인데 0.75라는 것은 하나의 버킷에 평균적으로 1개의 엔트리도 생성이 안됐다는 것이다.
- 평균적인 수치일 뿐이지, 자바에서 사용하는 seperate chaining방식은 특정 버킷에 엔트리가 집중적으로 몰릴 수도 있다. (참고로 균등하게 퍼뜨리기 위한 보조해시함수라는 것을 사용하긴 함)
> 보조 해시 함수 : HashMap에서 아래의 hash()라는 메소드를 통해 키값을 해싱하는데, 이때 hashCode에 상위 16비트 값을 XOR연산하는 보조 해시함수를 사용해서 조금이라도 더 균등하게 퍼뜨리기 위한 작업을 한다.
> static final int hash(Object key) { int h; return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16); } 
- load_factor 숫자를 낮게 셋팅(=capacity를 높게)하면 메모리는 많이 차지 하지만 검색 속도가 빨라진다, load_factor 숫자를 높게 세팅(=capacity를 낮게)하면 메모리를 적게 차지하지만 검색 속도가 느려진다. => capacity가 낮다는것은 하나의 버킷에 많은 엔트리가 연결리스트(혹은 트리)로 저장되어 있고 그만큼 비교해야하는 일이 많아지기 때문이다.
- capacity를 증가시킨다는 것은 버킷의 크기를 늘리는 것(=메모리를 늘리는 것) 뿐만이 아니라 기존에 저장되어 있던 모든 데이터의 해시값을 다시 계산하여 버킷에 새롭게 배치하는 작업까지 해야한다. 이러한 과정을 리해싱 이라고 한다. 
- 즉, 리해싱이란 capacity와 load_factor의 설정값에 따라 특정 시점에 버킷의 크기를 늘렸을 때(=메모리를 늘렸을 때 =capacity를 늘렸을 때. 다 같은말) 저장되어 있던 데이터의 해시값을 다시 계산하여 버킷에 재배치하는 것을 말한다.
- HashMap 객체에 저장될 데이터의 개수가 어느 정도인지 예측 가능한 경우에는, capacity값을 HashMap의 생성자의 인자로 지정하여 불필요하게 Separate Chaining을 재구성하지 않게 할 수 있다.
  



#### 참고 
- <http://blog.breakingthat.com/2018/05/07/java-collection-%EA%B0%9C%EC%9A%94-%EC%9E%90%EB%A3%8C%EA%B5%AC%EC%A1%B0/>
- <https://onsil-thegreenhouse.github.io/programming/java/2018/02/22/java_tutorial_HashMap_bucket/>