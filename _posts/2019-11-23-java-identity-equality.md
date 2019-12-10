---
layout: post
title: "Java 동일성(identity) 와 동등성(equality)"
tags: [java]
comments: true
date: 2019-11-23
---

## Java 동일성(identity) 와 동등성(equality)
- 동일성(identity) : 두 객체가 같은 객체를 바라보고 있는지 비교 (참조하고 있는 주소값이 같은지 비교)
- 동등성(equality) : 두 객체 안의 내용이 같은지 비교


## equals
- Object의 equals
    - == 비교를 통해 참조주소가 같은지를 판단함(동일성:identity)
    - 논리적으로 동등하다(equality)는 안의 내용이 같은지를 비교하는것을 뜻하는데 == 연산은 내용이 아닌 주소를 판별함
    - equals메소드를 오버라이딩하여 문제를 해결함 (ex: 객체의 이름만 비교하여 동등하다고 판단하는 메소드로 오버라이딩)
    ```java
    public boolean equals(Object obj) {
        return (this == obj);
    }

    @Override
    public boolean equlas(Object obj) {
        Person other = (Person) obj;
        return this.name.equals(other.name);
    }   
    ```
- Wrapper Class의 equals
  - 주소값이 아닌 value를 비교 해줌
  ```java
    // Long Wrapper 클래스의 equals
    public boolean equals(Object obj) {
        if (obj instanceof Long) {
            return value == ((Long)obj).longValue();
        }
        return false;
    }
  ```
- String의 equals
    - String의 equals는 문자 하나하나를 비교하며 값이 같은지 체크 
    ```java
    public boolean equals(Object anObject) {
        if (this == anObject) {
            return true;
        }
        if (anObject instanceof String) {
            String anotherString = (String)anObject;
            int n = value.length;
            if (n == anotherString.value.length) {
                char v1[] = value;
                char v2[] = anotherString.value;
                int i = 0;
                while (n-- != 0) {
                    if (v1[i] != v2[i])
                        return false;
                    i++;
                }
                return true;
            }
        }
        return false;
    }
    ```

## HashCode
- Object의 HashCode
    - public native int hashCode();
    - Object의 HashCode는 native call을 하여 해싱된 메모리의 주소값을 리턴함
- String의 HashCode
    - 문자 배열로 이루어진 문자열을 순환하며 hash값을 계산해서 리턴
    - 31을 사용하는 이유는 소수(자신과 1만을 갖는)라서 사용
    > "수식의 31은 소수이기 때문에 선택한 것이다. 만일 이 값이 짝수면서 곱셈의 결과가 오버플로우 되었다면 해시 값이 유실되었을 것이다. 2(의 배수)를 곱한다는 것은 비트의 이동을 의미하기 때문이다. 소수를 사용했을 때 어떤 장점이 있는지는 분명하지 않지만 관례적으로 그렇게 한다. 31의 좋은 점은 비트 이동과 뺄셈으로 곱셈을 대체할 수 있어서 성능을 향상시킬 수 있다는 것이다. 즉 31*i는 (i<<5)-i와 같다. 근래의 자바VM들은 이런 부류의 최적화를 자동적으로 수행한다." - 이펙티브 자바
    ```java
    public int hashCode() {
        int h = hash;
        if (h == 0 && value.length > 0) {
            char val[] = value;

            for (int i = 0; i < value.length; i++) {
                h = 31 * h + val[i];
            }
            hash = h;
        }
        return h;
    }
    ```
- Hash를 사용한 Collection(HashMap, HashTable, HashSet, LinkedHashSet등등)의 key 비교
    - Hash를 사용한 컬렉션들은 put할 때 key의 중복을 허용하지 않는다.
    - key의 hashCode를 이용하여 put을 하는데, 또다시 put할때 이 hashCode를 이용해 key가 중복되는 값인지 체크한다.
    - 이때 HashMap의 Key값을 객체로 하고 싶은 경우, Object의 HashCode는 위에서 말했듯이 메모리의 주소 값이기 때문에 개발자가 예상하는 대로 put이 안될 수도 있다. => 같은 내용을 가진 2개의 객체를 동등하다(equality)고 생각해서 하나만 Map에 들어갈줄 알았지만, 실제는 서로 다른 주소 값이 HashMap의 Key로 사용되기 때문에 두개의 객체가 모두 map에 적재된다.
    - HashCode를 Overring하여 해결할 수 있다.


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

## 클래스변수, 인스턴스변수, 지역변수
- 클래스변수 : 클래스의 static 변수로써 모든 인스턴스가 공유한다. 클래스 로딩시에 메모리에 할당된다(메모리에 한번만 올라감).
- 인스턴스 변수 : 인스턴스마다 다른 값을 갖는다.(공유되지 않는다) 클래스의 인스턴스가 생성될때 메모리에 올라가기 때문에 인스턴스 생성후 접근이 가능하다.
- 지역변수 : 메서느 내에서 선언되는 지역변수. 메서드가 실행될 때 메모리에 할당받고 메소드가 끝나면 소멸된다.
- 멤버변수 : 클래스변수와 인스턴스변수를 멤버변수라고 한다.