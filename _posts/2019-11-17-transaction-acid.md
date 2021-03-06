---
layout: post
title: "Transaction ACID"
tags: [mysql]
comments: true
date: 2019-11-17
---

## Transaction
* DB 작업수행의 논리적인 단위를 뜻한다.
* 하나의 트랜잭션안에 여러개의 쿼리수행이 있을 경우, 모든 쿼리는 전부 commit 되거나 전부 rollback 되어야 한다.
* 멀티 쓰레드 환경에서 여러 쓰레드가 동시에 쿼리를 수행하게 되면 데이터의 무결성이 깨지기 때문에 필요한다

## Transaction의 특성 (ACID)
- Atomicity (원자성)
  * 하나의 트랜잭션을 최소 단위로 본다는 뜻이다.
  * 하나의 트랜잭션안에 여러 쿼리가 있을 경우, 일부분만 실행하지 않는다.
  * All or Nothing의 개념으로, 트랜잭션안의 쿼리 전체를 수행(commit)하거나 전체를 수행하지 않음(rollback)을 뜻한다.
  * 원자성을 보장하기 위해, 이전에 commit된 상태를 임시영역(rollback segment)에 저장해두었다가 트랜잭션 오류 발생시 현재 내역을 지우고 임시 영역에 저장했던 상태로 롤백한다.
- Consistency (일관성)
  * 트랜잭션이 성공적으로 수행되면 DB의 상태는 항상 일관성을 띤다는 뜻이다.
  * 일관적이라는 것은 테이블이 설계된 규칙을 그대로 지닌다는 것이다. (ex: int타입의 컬럼이 varchar로 변경되거나 하지 않음)
  * 일관성은 트랜잭션 전후의 모든 제약조건을 만족하는 것을 통해 보장하는데, 어떤 이벤트와 조건이 발생했을 때 트리거를 통해 보장한다.
- Isolation (격리성=고립성=독립성)
  * 트랜잭션끼리는 서로 간섭할 수 없음을 뜻한다.
  * 하나의 트랜잭션은 다른 트랜잭션에 영향을 주면 안되고 또한 다른 트랜잭션으로 부터 영향을 받으면 안된다.
  * 즉, 어떤 트랜잭션이 수행중일 때 다른 트랜잭션은 앞서 실행중인 트랜잭션의 작업범위를 참조할 수 없게 된다.
  * 고립성을 보장하기 위해 lock을 통해 다른 트랜잭션이 접근하지 못하게 한다.  
- Durability (지속성=영속성)
  * 트랜잭션이 성공적으로 수행되면 수행결과는 영원히 DB에 반영된다는 것을 뜻한다. (다른 작업이 DB를 변경하기 전까지)


## Transaction의 상태
- 활동(Active) : 트랜잭션을 수행중인 상태
- 실패(Failed) : 트랜잭션 수행 중에 오류가 발생하여 중단된 상태
- 철회(Aborted) : 트랜잭션을 비정상적으로 종료하여 Rollback을 수행한 상태
- 부분완료(Partially Committed) : 트랜잭션의 마지막 연산까지 수행했고 Commit 연산을 수행하기 직전의 상태
- 완료(Committed) : 트래잭션을 성공적으로 종료한 상태, Commit 연산을 실행한 이후의 상태


## Transaction Recovery(회복)
- 트랜잭션 도중에 손상된 데이터베이스를 이전 상태로 복구하는 작업
- 트랜잭션의 연산을 수행할 때, 변경 전 데이터베이스 상태를 로그데이터로 생성한다.
- 취소(Undo) 연산으로 이미 데이터베이스에 쓰여진 것도 수정할 수 있다.

## 회복 기법
1. 즉각 갱신 기법 (Immediate Update)
- 트랜잭션의 연산을 즉시 DB에 반영하고 변경된 모든 내용을 로그에 기록한 후, 장애 발생시 로그의 내용을 토대로 회복

2. 지연 갱신 기법 (Deferred Update)
- 트랜잭션을 완료할 때까지 DB에 반영하지 않고 변경할 내용을 로그에 보관
- 트랜잭션 부분완료 시점에 로그 기록을 실제 DB에 반영
- 트랜잭션 수행 중에 장애가 발생해서 롤백해도, DB에 반영한게 없기 때문에 취소(Undo)할 필요가 없다.
- DB가 정상적으로 회복한 후에, 재시도(Redo)작업을 통해 트랜잭션 재실행할 수 있다.

3. 검사점 기법 (Check Point)
- 트랜잭션 중간에 검사점을 로그에 보관하여 트랜잭션 전체를 취소하지 않고 검사점까지 취소할 수 있는 기법
- 회복시 많은 양의 로그를 검색하고 갱신하는 시간을 줄이기 위해 사용한다.

4. 그림자 페이징 기법 (Shadow Paging)
- 트랜잭션 연산으로 DB를 변경할 필요가 있을때, DB의 원본에 대한 복사본을 떠놓고 장애 발생시 롤백할때 이 복사본을 이용하여 회복하는 기법
- 로그를 기록할 필요가 없고, 취소(Undo), 재시도(Redo)할 필요가 없다.

## Redo (재시도)
- 장애가 발생한 후 시스템을 재가동했을 때, 로그 파일에 트랜잭션의 시작(START)이 있고 종료(COMMIT)가 **있는** 경우 로그를 보면서 트랜잭션이 변경한 내용을 다시 기록하는 과정이다. 
- 종료(COMMIT)는 제대로 트랜잭션이 수행되었다는 뜻인데, 장애로 인해 해당 트랜잭션의 변경내용이 DB에 제대로 반영되지 않은 경우 Redo를 수행한다

## Undo (취소)
- 장애가 발생한 후 시스템을 재가동했을 때, 로그 파일에 트랜잭션의 시작(START)이 있고 종료(COMMIT)가 **없는** 경우 로그를 보면서 트랜잭션이 변경한 내용을 기존상태로 돌려 놓는 과정이다.
- 트랜잭션이 제대로 종료(COMMIT)되지 않았는데 트랜잭션의 변경내용이 DB에 반영되어버린 경우 Undo를 수행한다. 

## Trigger
- 트리거란 특정테이블에 DML(Insert, Update, Delete)이 수행될 때, 미리 설정해둔 작업(트리거)이 자동으로 수행되는 것을 말한다.
- 트리거를 설정해두면 직접 호출하지 않고, 데이터베이스 시스템에서 DML 수행 이벤트를 전달받아 자동으로 수행해준다.

## Procedure
- 프로시저란 SQL문을 함수형태로 저장하고 있다가 직접 CALL 명령어로 호출하여 사용하는 것을 말한다.
- 리턴값이 없어도 되고 여러개 일 수도 있다.

## Function
- 데이터베이스에서 함수란 SQL문을 함수형태로 저장하고 있다가 함수명을 통해 호출하여 사용하는 것을 말한다.
- 프로시저와는 다르게 반드시 리턴값이 있어야하고 1개의 리턴값만 있어야 한다.


#### 참고
- <https://victorydntmd.tistory.com/129>
- <http://ehpub.co.kr/tag/%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98%EC%9D%98-%EC%83%81%ED%83%9C/>