---
layout: post
title: "클러스터링 인덱스"
tags: [mysql]
comments: true
date: 2019-11-23
---

## 클러스터링 인덱스
- MySQL에서 클러스터링 인덱스는 InnoDB 와 TokuDB 스토리지 엔진에서만 지원하며 나머지에선 지원되지 않음
- 프라이머리 키값이 비슷한 레코드끼리 묶어서 저장하는 것을 클러스터링 인덱스라고 한다. (=프라이머리 키로 인덱스를 만듬)
- 프라이머리 키값에 의해 레코드의 저장위치가 결정되고 PK값이 바뀌면 레코드의 물리적인 저장 위치가 바뀌어야 한다.(PK가 바뀔일은 거의 없겠지만)
  * 다른 엔진에서는 INSERT될 때 데이터 파일의 끝에(또는 임의의 빈 공간) 저장함.
- 일반적으로 B+Tree 인덱스도 인덱스 키값으로 정렬되어 저장한다. 하지만 B+Tree인덱스는 클러스터링 인덱스라고 부르지 않는다.
> MySQL에서 인덱스는 B+Tree 구조로 저장된다.


## InnoDB 스토리지 엔진이 클러스터링 인덱스를 만드는 우선순위 
1. 프라이머리 키가 있으면 기본적으로 프라이머리 키를 클러스터 키로 선택
2. (1번이 없을 경우) NOT NULL 옵션의 유니크 인덱스(UNIQUE INDEX) 중에서 첫번째 인덱스를 클러스터 키로 선택
3. (1,2번이 없을 경우) 자동으로 유니크한 값을 가지도록 증가되는 칼럼을 내부적으로 추가한 후, 클러스터 키로 선택


## 보조 인덱스
- InnoDB 엔진에서는 모든 레코드의 저장위치가 PK(=클러스터링 인덱스)에 의해 결정지어진다고 했다.
- 그럼 보조 인덱스도 레코드의 저장위치를 가지고 있을까 ?
- 그렇지 않다. InnoDB에서 보조인덱스는 레코드가 저장된 주소가 아니라 PK값을 저장하도록 되어 있다.
- 보조 인덱스를 사용하여 조회쿼리를 수행하면, 보조 인덱스를 통해 PK값을 찾아온다. 찾아온 PK값으로 레코드를 다시 조회하여 레코드를 가져온다.
- 보조인덱스로 PK값을 찾아오는 과정을 한번 더 수행해야 하지만, InnoDB는 PK값으로 레코드를 읽어오는 과정이 매~~우 빠르기 때문에 상관없다.

## 클러스터링 인덱스 장/단점
1. 장점
- PK(=클러스터링 인덱스)로 검색할 때 매우 빠르다.
- 테이블의 모든 보조 인덱스가 PK값을 가지고 있기 때문에 인덱스만으로 처리될 수 있는 경우가 많다. (커버링 인덱스라고 함)
  > 커버링 인덱스 : 원하는 데이터를 인덱스에서만으로 추출할 수 있는 것을 의미한다. B+Tree 스캔만으로 원하는 데이터를 가져올 수 있으며, 칼럼을 읽기 위해 굳이 데이터 블록을 보지 않아도 된다.
  > ex) userName 과 age라는 컬럼이 있고 각각 보조인덱스로 설정되어 있을 때
  > SELECT userName, age FROM tableName; // 조회할 데이터가 모두 인덱스로만 이루어져 있음

2. 단점
- 테이블의 모든 보조 인덱스가 PK값을 갖기 때문에 클러스터 키값의 크기가 클 경우 전체적으로 인덱스의 크기가 커짐
- 보조 인덱스를 통해 검색할 때 프라이머리 키로 다시 한번 검색해야 하므로 처리 성능이 조금 느림
- INSERT할 때 프라이머리 키에 의해 레코드의 저장 위치가 결정되기 때문에 처리 성능이 느림
- 프라이머리 키를 변경할 때 레코드를 DELETE하고 INSERT하는 작업이 필요하기 때문에 처리 성능이 느림



#### 참고
- <https://12bme.tistory.com/149>
