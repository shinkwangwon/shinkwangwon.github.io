---
layout: post 
title: "Java HashMap"
tags: [java]
comments: true
date: 2019-11-28
---

## Hash란 ?
- Hash 알고리즘에 대한 내용은 아래 내용 확인
- [Hash 란?](https://shinkwangwon.github.io/hash/) 


## HashMap vs HashTable
- HashMap과 Hashtable을 정의하면 "키에 대한 해시 값을 사용해 값을 저장하고 조회하며, 키-값 쌍의 개수에 따라 동적으로 크기가 증가하는 연관 배열"이라고 할 수 있다.
> 연관배열(associative array)을 지칭하는 다른 용어는 Map, Dictionary, Symbol Table 등이 있다
- 자바의 Hashtable은 보조해시함수를 사용하지 않는다.
- 자바의 HashMap은 보조 해시함수를 사용 하기 때문에 Hastable에 비해 해시충돌이 덜 발생하기에 상대적으로 성능상 이점이 있다.

## HashMap 해시충돌
- HashMap은 기본적으로 각 객체의 hashCode() 메서드가 반환하는 값을 사용하는데 이 메소드는 리턴타입이 int형이다. 따라서 32비트 정수 자료형으로는 해시충돌이 발생하지 않는 완전한 해시함수를 만드는 것은 사실상 불가능하다. 논리적으로 생성가능한 객체의 수가 2^32승 보다 많을 수 있고 모든 HashMap객체에서 O(1)을 보장하기 위해 랜덤 접근이 가능하게 하려면 원소가 2^32승인 배열을 모든 HashMap이 가지고 있어야 하기 때문이다.

## Open Addressing vs Seperate Chaining
- Open Addressing , Seperate Chaining 둘다 최대 연산횟수는 O(M)이다. (M : 버킷의 수)
- Open Addressing은 데이터를 삭제할 때 효율적으로 처리하기 어려운데, HashMap에서는 remove()메서드가 빈번하기 호출될 수 있다.
- Open Addressing은 연속된 공간에 데이터를 저장하기 때문에 데이터 개수가 충분히 적다면 Open Addressing이 Seperate Chaining보다 더 성능이 좋다. 하지만 배열의 크기(=버킷의 크기)가 커질 수록 캐시 효율이 높다는 Open Addressing의 장점은 사라진다.
- Open Addressing의 경우 해시 버킷을 채운 밀도가 높아질수록 worst case발생 빈도가 더 높다.
> 밀도가 높다는 것은 서로다른 객체가 같은 해시값을 가져서 비슷한 위치에 몰려 있다는 것을 뜻한다.


## Java8 HashMap Algorithm
* Java HashMap 에서 사용하는 방식은 Seperate Chaining 방식 
* HashMap에 저장된 키-값 쌍 개수가 일정 개수 이상으로 많아지면, 일반적으로 Open Addressing은 Separate Chaining보다 느리다.
* Java 8에서는 Seperate Chaining에서 링크드 리스트 와 트리를 혼용해서 사용함
* 하나의 해시버킷에 8개 이상의 key&value 쌍이 모이면 트리로 변경, 하나의 해시버킷이 6개 이하로 줄어들면 다시 링크드 리스트로 변경
* 차이를 "2" 만큼으로 둔것은, 차이가 1이라면 1개의 key&value쌍이 삽입/삭제될때 불필요하게 트리와 링크드 리스트를 변경하는 일이 반복되기 때문이다.
* Hash Map은 데이터 개수가 일정 개수 이상이되면 해시버킷의 크기를 2배로 늘림 => 해시충돌문제를 어느정도 해결 => 단, 모든 데이터에 대한 새로운 Separate Chaining을 구성해야함

## LinkedList vs Tree
- 자바8에서는 해시충돌로 연결리스트를 사용하면 get()메소드 호출시 기댓값은 E(N/M) 이지만 트리로 변경된 경우 기댓값은 E(log N/M)이다.
- 자바8에서는 HashMap을 Entry클래스 대신 Node클래스를 사용한다. Node클래스 자체는 자바 7의 Entry클래스와 내용이 거의 같지만 연결리스트 대신 트리를 사용할 수 있도록 하위 클래스인 TreeNode가 있다는 것이 자바 7의 HashMap과 다르다.
- 이때 사용하는 트리는 Red-Black Tree인데 자바 컬렉션 프레임워크의 TreeMap과 구현이 거의 같다. 
- 트리 순회시 사용하는 대소 판단 기준은 해시 함수 값이다. 해시 값을 대소 판단 기준으로 사용하면 total ordering에 문제가 생기는데 자바 8의 HashMap에서는 tieBreakOrder()메소드로 해결한다,
- 버킷의 최대크기는 2^30승 으로 되어 있다.

```java
// HashMap class 
...
static final int MAXIMUM_CAPACITY = 1 << 30;    // 버킷의 최대 크기

...
static int tieBreakOrder(Object a, Object b) {
    int d;
    if (a == null || b == null ||
        (d = a.getClass().getName().
            compareTo(b.getClass().getName())) == 0)
        d = (System.identityHashCode(a) <= System.identityHashCode(b) ?
                -1 : 1);
    return d;
}
```

## Rehashing (리해싱) 
- 용어 정리 : capacity는 버킷의 총 개수를 말하며, 기본적으로 아무설정 없이 HashMap 생성시 capacity=16, load_factor=0.75로 생성된다.
- if(load_factor == 저장된 데이터 수 / capacity) 가 되는 시점에 자동으로 버킷의 크기를 늘린다. (대략 버킷의 크기를 2배로 늘림)
- 평균적으로 하나의 버킷에 저장된 데이터의 수가 load_factor 랑 같아지면 버킷의 크기를 늘린다는것인데 0.75라는 것은 하나의 버킷에 평균적으로 1개의 엔트리도 생성이 안됐다는 것이다.
- 평균적인 수치일 뿐이지, 자바에서 사용하는 seperate chaining방식은 특정 버킷에 엔트리가 집중적으로 몰릴 수도 있다. (참고로 균등하게 퍼뜨리기 위한 보조해시함수라는 것을 사용하긴 함)
> 보조 해시 함수 : HashMap에서 아래의 hash()라는 메소드를 통해 키값을 해싱하는데, 이때 hashCode에 상위 16비트 값을 XOR연산하는 보조 해시함수를 사용해서 조금이라도 더 균등하게 퍼뜨리기 위한 작업을 한다.

 ```java
 static final int hash(Object key) { int h; return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16); } 
 ```
- load_factor 숫자를 낮게 셋팅하면 capacity가 2배씩 빠르게 늘어나기 때문에 메모리는 많이 차지 하지만 검색 속도가 빨라진다, load_factor 숫자를 높게 세팅(=capacity를 낮게)하면 메모리를 적게 차지하지만 검색 속도가 느려진다. => capacity가 낮다는것은 하나의 버킷에 많은 엔트리가 연결리스트(혹은 트리)로 저장되어 있고 그만큼 비교해야하는 일이 많아지기 때문이다.
- capacity를 증가시킨다는 것은 버킷의 크기를 늘리는 것(=메모리를 늘리는 것) 뿐만이 아니라 기존에 저장되어 있던 모든 데이터의 해시값을 다시 계산하여 버킷에 새롭게 배치하는 작업까지 해야한다. 이러한 과정을 리해싱 이라고 한다. 
- 즉, 리해싱이란 capacity와 load_factor의 설정값에 따라 특정 시점에 버킷의 크기를 늘렸을 때(=메모리를 늘렸을 때 =capacity를 늘렸을 때. 다 같은말) 저장되어 있던 데이터의 해시값을 다시 계산하여 버킷에 재배치하는 것을 말한다.
- HashMap 객체에 저장될 데이터의 개수가 어느 정도인지 예측 가능한 경우에는, capacity값을 HashMap의 생성자의 인자로 지정하여 불필요하게 Separate Chaining을 재구성하지 않게 할 수 있다.


## Java7 -> Java8 변경되면서 HashMap의 변화 
- Seperate Chaining를 위해 연결리스트와 트리를 함께 사용
- 트리를 구현하기 위해 Entry클래스를 Node클래스로 대체 (Node클래스는 하위에 TreeNode를 가지고 있음) 
- 트리를 사용함으로써 생기는 해시값의 대소 판단 문제를 해결하기 위해 tieBreakOrder() 메소드를 사용하는 점
- 보조 해시 함수의 단순화
  * 자바8에서는 해시 충돌이 많이 발생하면 연결리스트 대신 트리를 사용하므로 해시 충돌시 발생할 수 있는 성능 문제가 완화됨
  * 최근의 해시 함수는 균등 분포가 잘되게 만들어지는 경향이 있어 자바 7까지 사용했던 보조 해시 함수의 효과가 크지 않음
  * 위의 2가지 이유로 자바 8에서는 보조해시함수를 단순화 시킴 
- String 객체의 해시함수 변화 (성능향상을 위해 31을 이용)

#### 참고
- <https://onsil-thegreenhouse.github.io/programming/java/2018/02/22/java_tutorial_HashMap_bucket/>
- 서적: 네이버를 만든 기술, 읽으면서 배운다 - 자바편