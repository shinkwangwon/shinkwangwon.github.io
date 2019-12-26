---
layout: post
title: "MyBatis # , $ 차이"
tags: [mybatis]
comments: true
date: 2018-02-02
---

## MyBatis # , $ 차이

### PreparedStatement (#)
- SELECT * FROM user WHERE col = #{userId}
- SELECT * FROM user WHERE col = ?
- #부분이 ? 로 치환되면서 파라미터로 들어온 userId 값을 ? 에 대입하여 쿼리를 수행한다.
- #일 경우 해당 부분에 작은 따옴표로 감싸진 상태로 값이 들어간다. ( SELECT * FROM user WHERE col = 'shin';)
- #일 경우는 작은따옴표로 감싸주기 때문에 SQL Injection 대비가 가능하다. 작은따옴표를 감싸주는 파싱작업을 하면서 SQLInjection의 코드를 만나면 오류가 발생한다.
- ? 로 치환된 쿼리는 캐싱하고 있다가 다른 userId가 들어올 때에도 재사용 된다. 즉, SELECT * FROM user WHERE col = ? 이 쿼리를 캐싱하고 있다.
- 단, 옵티마이저의 수행계획이 항상 동일하다는 것이 단점이다. 위 쿼리가 처음 캐싱될 때 col 이란 컬럼으로 인덱스가 스캔되면 그 다음부터 항상 인덱스가 타도록 결정된 상태로 캐싱된다는 것이다.
- 즉, 원치않는 옵티마이저 수행계획이 생성될 수도 있다.


### Statement ($)
- SELECT * FROM user WHERE col = ${userId}
- SELECT * FROM user WHERE col = 값
- $를 사용하면 값이 넣어진 쿼리 자체로 수행이 된다. (SELECT * FROM user WHERE col=shin;)
- 쿼리에 작은따옴표가 붙지 않기 때문에 $를 사용하면 스트링의 경우 작은따옴표로 감싸지지 않기 때문에 에러가 발생한다.
- 또한 스트링 자체를 넣기 때문에 SQL Injection 보안 위험이 있다.
- $일 경우는 주로 조회 조건 컬럼의 값이 아닌 칼럼명이 동적으로 바뀔때 사용한다. 
- SELECT * FROM user WHERE ${col} = 'shin'; 이렇게 칼럼명에 주로 사용
- Statement의 장점은 PreparedStatement와 반대로 쿼리를 캐싱하고 있지 않고 매번 구문분석-의미분석-파싱작업을 한 후에 쿼리를 수행한다. 따라서 매번 옵티마이저 수행계획이 달라진다. 


### SQL Injection
- 보안상의 허점을 이용해 악의적인 SQL문을 실행하도록 하는 공격
- 예를 들어 SELECT * FROM user WHERE col = ${userId} => userId: "shin OR 1==1"


#### 참고
- <https://12bme.tistory.com/164>