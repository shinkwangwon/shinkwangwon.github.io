---
layout: post
title: "Spring Transaction 속성"
tags: [spring]
comments: true
date: 2019-12-13
---


## Transaction Propagation (트랜잭션 전파수준)
- 트랜잭션을 시작하거나 기존 트랜잭션에 참여하는 방법을 결정하는 속성이다.

1. REQUIRED
- 디폴트 속성이며, 미리 시작된 트랜잭션이 있으면 그 트랜잭션에 참여하고 없으면 새로 시작한다. 하나의 REQUIRED 트랜잭션이 먼저 시작하고 REQUIRED로 되어있는 트랜잭션이 또 실행되면 자동으로 같은 트랜잭션으로 묶인다.

2. REQUIRES_NEW
- 항상 새로운 트랜잭션을 시작한다. 이미 진행 중인 트랜잭션이 있으면 그 트랜잭션을 잠시 보류시키고 새롭게 만들어진 트랜잭션을 처리한 후에 다시 기존 트랜잭션을 수행한다. 독립적인 트랜잭션을 만드는 것이기 때문에 새로 만들어진 트랜잭션의 커밋/롤백은 기존 트랜잭션에 영향을 주지 않는다. 반대로 기존 트랜잭션의 커밋/롤백 또한 새로 만들어진 트랜잭션에 영향을 주지 않는다.

3. SUPPORTS
- 이미 시작된 트랜잭션이 있으면 참여하고 그렇지 않으면 트랜잭션 없이 진행한다. 트랜잭션이 있어도 되고 없어도 문제 없는 경우에 사용한다.

4. MANDATORY
- 이미 시작된 트랜잭션이 있으면 참여하고 그렇지 않으면 예외를 발생시킨다. 독립적으로 트랜잭션을 진행하면 안되는 경우에 사용한다.

5. NOT_SUPPORTED
- 트랜잭션 없이 진행하도록 한다. 만약 이미 진행 중인 트랜잭션이 있으면 그 트랜잭션을 보류 시키고 작업을 처리한 후에 다시 기존 트랜잭션을 수행한다.

6. NEVER
- 트랜잭션을 사용하지 않도록 강제한다. 이미 진행 중인 트랜잭션이 있으면 예외를 발생시킨다.

7. NESTED
- 이미 진행중인 트랜잭션이 있으면 중첩 트랜잭션을 생성한다. 즉, 트랜잭션 안에 트랜잭션을 만드는 것이다. 중첩트랜잭션은 먼저 시작된 부모 트랜잭션의 커밋/롤백에는 영향을 받지만, 자신의 커밋/롤백은 부모 트랜잭션에게 영향을 주지 않는다. 메인(부모) 트랜잭션이 롤백되면 같이 롤백되야 하고, 하위(=중첩) 트랜잭션이 롤백되도 메인 트랜잭션은 롤백시키면 안될 경우에 사용한다.


## Transaction Isolation (트랜잭션 격리수준)
- 트랜잭션 격리수준은 동시에 여러 트랜잭션이 진행될 때에 트랜잭션의 작업 결과를 여타 트랜잭션에게 어떻게 노출할 것인지를 결정하는 기준이다.

1. DEFAULT
- 사용하는 데이터 액세스 기술 또는 DB드라이버의 디폴트 설정을 따른다. 보통 드라이버의 격리수준은 DB의 격리 수준을 따르는게 일반적이다. DB의 문서를 참고해서 DB의 디폴트 격리 수준이 무엇인지 확인 후 사용하도록 한다.

2. READ_UNCOMMITTED
- 가장 낮은 격리 수준이다. 하나의 트랜잭션이 커밋되기 전에 그 변화가 다른 트랜잭션에 그대로 노출되는 문제가 있다. 하지만 가장 빠르기 때문에 데이터의 일관성이 조금 떨어지더라도 성능을 극대화할 때 의도적으로 사용하기도 한다.

3. READ_COMMITTED
- READ_UNCOMMITTED와 달리 다른 트랜잭션이 커밋하지 않은 정보는 읽을 수 없다. 대신 하나의 트랜잭션이 읽은 로우를 다른 트랜잭션이 수정할 수 있다. 이 때문에 처음 트랜잭션이 같은 로우를 다시 읽을 경우 다른 내용이 발견될 수 있다. (Non-Repeatable Read 문제)
> MVCC(Multi Version Concurrency Control)기술을 이용해 UNDO영역에 변경 전 데이터를 넣어놓고 같은 트랜잭션 안에서 같은 쿼리 수행시 백업해놓은 데이터를 보여줌으로써 위 문제를 해결한다고 알고 있다.

4. REPEATABLE_READ
- 하나의 트랜잭션이 읽은 로우를 다른 트랜잭션이 수정하는 것을 막아준다. 하지만 새로운 로우를 추가하는 것은 제한하지 않는다. 따라서 SELECT로 조건에 맞는 로우를 전부 가져오는 쿼리를 여러번 수행 했을 경우 트랜잭션이 끝나기 전에 추가된 로우가 발견될 수 있다. (Phantom READ 문제)
> MySQL(InnoDB Engine)의 기본 격리 수준이 REPEATABLE_READ로 알고 있는데 MySQL은 위 문제를 해결하기 위해 기본 Lock수준을 넥스트키락(Record Lock + Gap Lock) 수준으로 해서 검색하는 로우 중간에 새로운 row를 insert하지 못하게 한다(갱신처리 방지). 
> 그리고 MySQL(InnoDB)에서도 MVCC(Multi Version Concurrency Control)기술을 이용하여 위 문제를 해결한다.


5. SERIALIZABLE
- 가장 강력한 트랜잭션 격리 수준이다. 이름 그대로 트랜잭션을 순차적으로 진행시켜 주기 때문에 여러 트랜잭션이 동시에 같은 테이블의 정보를 엑세스하지 못한다. 가장 안전한 격리 수준이지만 가장 성능이 떨어지기 때문에 극단적으로 안전한 작업이 필요한 경우가 아니라면 자주 사용되지 않는다.


#### 참고
- 서적: 토비의 스프링 3.1 Vol.2
- <https://mysqldba.tistory.com/335>