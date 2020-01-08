---
layout: post
title: "JPA 와 MyBatis의 차이"
tags: [jpa]
comments: true
date: 2020-01-07
---

## SQL을 직접 다룰 때 발생하는 문제점
### 생산성
- SQL을 직접 다룰때 간단한 CRUD 기능을 위해 반복적인 작업을 해야한다(일일이 컬럼명 맞춰줘야 하고... 귀찮). JPA는 메소드 형태로 CRUD기능을 제공하기 때문에 테이블을 나타내는 엔티티 객체만 만들어 놓으면 손쉽게 CRUD 기능을 구현할 수 있다. 

### DB테이블의 변경
- DB에서 row를 조회해서 객체에 매핑하거나, 객체를 삽입하거나 수정할 때 DB 테이블이 수정된 경우 객체뿐만 아니라 조회,삽입,수정에 관련된 SQL을 전부 수정해야한다. 만약 실수로 수정해야할 쿼리들 중에 하나라도 빼먹으면 나중에 데이터 정합성에 문제가 발생하거나 SqlException이 발생할 수도 있다. 
- JPA의 경우에는 객체(=엔티티)가 곧 테이블이고 SQL을 직접적으로 다루는게 아니라 JPA 문법을 이용해서 작성한 경우에 자동으로 엔티티에 맞는 쿼리를 만들어주기 때문에, DB테이블이 수정되었을 경우 객체(=엔티티)만 수정해주면 끝난다. 즉, 조회,삽입,수정에 대한 문제를 객체의 필드 수정만으로 끝낼 수 있다.

### 연관된 객체의 접근
- DB에서는 외래키를 통해 테이블 간의 연관관계를 맺고 조인을 통해 연관된 테이블을 조회해온다. 이러한 테이블간의 연관관계를 자바에서는 객체간의 참조로 표현한다.

```java
class Member {
    private String memberId;

    private Team team;  // 객체 참조로 테이블간의 관계를 표현 
}
```
- 즉, 위와 같이 Member라는 테이블과 Team이라는 테이블의 관계는 객체참조를 통해 연관관계를 표현한다. (DB는 Member테이블에 Team테이블의 PK를 외래키로 갖는 것으로 표현한다.)
- 이런 경우에 SQL을 직접적으로 사용하여 조회한 경우, member.getTeam() 과 같은 메소드로 team객체를 사용할 수 있는지는 쿼리를 직접 까봐야 알 수 있다.
> member.getTeam() 처럼 member 객체에서 참조하고 있는 team 을 찾는 것을 객체 그래프 탐색이라고 한다.
- SELECT 쿼리에서 team 객체에 대한 데이터를 조인해서 가지고 오고 있는지, 가지고 온다면 team 객체중에 어떤 속성값들을 조회하는지 일일이 봐야 team 객체를 사용할 수 있을지 없을지 알 수 있다.
- 즉, SQL을 직접 다루면 처음 실행하는 SQL에 따라 객체 그래프를 어디까지 탐색할 수 있는지 정해지고, 결국엔 그 SQL을 살펴봐야 어디까지 탐색할 수 있는지 알 수 있다.
- JPA는 연관된 객체를 사용하는 시점에 적절한 SELECT SQL을 실행한다. 위와 동일하게 Member, Team이 있을 때 team = member.getTeam()을 호출하고 team 객체를 실제로 사용하는 시점에 DB에 SELECT 쿼리를 날려서 Team 객체를 조회해 온다. 이러한 기능은 실제 객체를 사용하는 시점까지 DB조회를 미룬다고 해서 **지연로딩**이라 한다. SQL을 직접사용하는 것과는 다르게 JPA가 자동으로 적절한 시점에 Team 객체를 조회해주기 때문에 개발자는 SQL에 덜 의존하게 된다.

### 동일성 비교(equality)
- 동일선 비교는 == 비교로 객체 인스턴스의 주소 값을 비교하는 것을 말한다.
- 논리적으로 DB에서 똑같은 Row를 두번 조회한 경우에 두 객체는 같은 객체라고 볼 수 있다.

```java
// Member를 조회하는 메소드를 아래와 같이 작성했다고 하자.
public Member getMember(String memberId) {
    String sql = "SELECT * FROM MEMBER WHERE MEMBER_ID = ?";
    // JDBC API , SQL 실행
    return new Member(...);
}

Member mem1 = memberDao.getMember(memberId);
Member mem2 = memberDao.getMember(memberId);
mem1 == mem2; // false
```

- 위와 같이 같은 memberId로 2번 조회해서 mem1, mem2 객체에 저장했다. 하지만 getMember 메소드가 새로운 인스턴스를 반환하기 때문에 mem1과 mem2는 서로 다른 인스턴스가 된다.
- 이런 문제를 해결하기 위해 데이터베이스의 같은 Row를 조회할 때마다 같은 인스턴스를 반환하도록 구현하는 것은 쉽지 않다. 여기에 여러 트랜잭션이 동시에 실행되는 상황까지 고려하면 더더욱 구현하기가 어려워진다.
- JPA는 같은 트랜잭션일 때 같은 객체가 조회되는 것을 보장한다. JPA에서는 엔티티를 Persistence Context(영속성 컨텍스트)가 관리하기 때문에 같은 쿼리에 대해 같은 인스턴스가 참조되는 것을 보장하는데 이를 동일성 보장 이라 한다. ( [JPA (Java Persistence API)](https://shinkwangwon.github.io/JPA/) 를 참고하자)


### 객체 상속
- 자바에서 객체는 상속이라는 기능을 가지고 있다. 하지만 DB 테이블에는 상속이라는 기능이 없다.
> 일부 데이터베이스는 상속 기능을 지원한다고 하지만 객체의 상속과는 약간 다르다고 한다.
- 객체지향적인 개발을 위해 상속구조로 객체모델링을 했다고 하자. 이런 경우 SQL을 직접 사용할 때 코드량이 많아지고 개발자가 실수할 확률이 더 높아진다. 아래 예를 보자.

```java
class Item {
    Long id;
    int price;
}

// Item 클래스를 상속받은 Book 클래스
class Book extends Item {
    String author;
}
```

- Item클래스와 이를 상속받는 Book클래스가 있을 때, Book 클래스를 저장하기 위해서는 Insert 쿼리가 2개 만들어 져야 한다.
> INSERT INTO ITEM... 
> INSERT INTO BOOK...
- 이렇게 Item과 Book을 저장하기 위해 2개의 INSERT쿼리를 작성해야하고, 이를 수정할 일이 꼭 둘다 확인을 해주어야 한다. Item 클래스를 상속받는 클래스가 많을 경우에 개발자가 실수하기 딱 좋다.
- JPA에서는 이러한 상속 관련된 문제를 개발자 대신에 해결해준다. jpa.persist(book); 처럼 JPA가 데이터 저장을 위해 제공하는 persist메소드를 통해 book 객체를 저장할 경우, JPA는 자동으로 2개의 쿼리를 만들어서 수행시켜준다.
> INSERT INTO ITEM... 
> INSERT INTO BOOK...
- 조회쿼리에서도 JPA는 빛을 발하는데 SQL을 직접사용할 경우에 ITEM, BOOK 테이블을 조인하는 쿼리를 만들어서 Book 객체를 조회해와야한다. JPA에서는 조회를 위한 find메소드를 사용할 경우 jpa.find(Book.class, id); 자동으로 두 테이블을 조인해서 데이터를 조회해 온다.
> SELECT I.*, B.* FROM ITEM I JOIN BOOK B ON I.ITEM_ID = B.ITEM_ID; 와 같은 쿼리를 생성해 낸다.



## MyBatis는 SqlMapper라고 부르고 JPA는 ORM이라고 부르는데, 그 이유가 무엇일까요 ?
- MyBatis나 스프링 JdbcTemplate을 보통 SqlMapper라고 부른다. 이것은 이름 그대로 객체와 SQL을 매핑한다. 따라서 SQL과 매핑할 객체만 지정하면 지루하게 반복되는 JDBC API사용과 응답 결과를 객체로 매핑하는 일은 SQL 매퍼가 대신 처리해준다. 이런 SqlMapper가 편리하긴 하지만 결국 개발자가 SQL을 직접 작성해야 하므로 SQL에 의존하는 개발을 피할 수 없다. 반면에 ORM은 객체와 테이블을 매핑만 하면 ORM프레임워크가 SQL을 만들어서 데이터베이스와 관련된 처리를 해주므로 SQL에 의존하는 개발을 피할 수 있다.

## 통계 쿼리처럼 매우 복잡한 SQL은 어떻게 처리하나요 ?
- JPA는 통계 쿼리 같이 복잡한 쿼리보다는 실시간 처리용 쿼리에 더 최적화 되어 있다. 상황에 따라 다르지만 정말 복잡한 통계 쿼리는 SQL을 직접 작성하는 것이 더 쉬운 경우가 많다. 이런 경우 MyBatis를 사용하는 것이 나을 수 있다. JPA에서도 직접 쿼리를 작성할 수 있는 Native SQL을 지원하기 때문에 Native SQL을 사용해도 된다.

