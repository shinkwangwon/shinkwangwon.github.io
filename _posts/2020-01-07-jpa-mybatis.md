---
layout: post
title: "JPA 와 MyBatis의 차이"
tags: [jpa]
comments: true
date: 2020-01-07
---

## MyBatis는 SqlMapper라고 부르고 JPA는 ORM이라고 부르는데, 그 이유가 무엇일까요 ?
- MyBatis나 스프링 JdbcTemplate을 보통 SqlMapper라고 부른다. 이것은 이름 그대로 객체와 SQL을 매핑한다. 따라서 SQL과 매핑할 객체만 지정하면 지루하게 반복되는 JDBC API사용과 응답 결과를 객체로 매핑하는 일은 SQL 매퍼가 대신 처리해준다. 이런 SqlMapper가 편리하긴 하지만 결국 개발자가 SQL을 직접 작성해야 하므로 SQL에 의존하는 개발을 피할 수 없다. 반면에 ORM은 객체와 테이블을 매핑만 하면 ORM프레임워크가 SQL을 만들어서 데이터베이스와 관련된 처리를 해주므로 SQL에 의존하는 개발을 피할 수 있다.

## 통계 쿼리처럼 매우 복잡한 SQL은 어떻게 처리하나요 ?
- JPA는 통계 쿼리 같이 복잡한 쿼리보다는 실시간 처리용 쿼리에 더 최적화 되어 있다. 상황에 따라 다르지만 정말 복잡한 통계 쿼리는 SQL을 직접 작성하는 것이 더 쉬운 경우가 많다. 이런 경우 MyBatis를 사용하는 것이 나을 수 있다. JPA에서도 직접 쿼리를 작성할 수 있는 Native SQL을 지원하기 때문에 Native SQL을 사용해도 된다.

