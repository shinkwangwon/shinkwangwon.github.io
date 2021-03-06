---
layout: post
title: "MySQL 동작과정"
tags: [mysql]
comments: true
date: 2019-12-03
---


1. Parser (파서)
- 파서는 사용자 요청으로 들어온 쿼리 문장을 토큰으로 분리해서 트리 형태의 구조로 만들어내는 작업이다. 
- 쿼리 문장의 문법 오류는 이 과정에서 발견되어 오류메세지를 전달한다.
> 토큰 : MySQL이 인식할 수 있는 최소 단위의 어휘나 기호

2. Preprocessor (전처리기)
- 파서 과정에서 만들어진 파서 트리를 기반으로 쿼리 문장에 구조적인 문제점이 있는지 확인한다.
- 각 토큰을 테이블 이름이나 컬럼 이름, 또는 내장 함수와 같은 개체를 매핑해서 해당 개체의 존재 여부와 개체의 접근 권한 등을 확인하는 과정을 수행한다.
- 실제 존재하지 않거나 권한상 사용할 수 없는 개체의 토큰은 이 단계에서 걸러진다.

3. Optimizer (옵티마이저)
- 옵티마이저란 사용자의 요청으로 들어온 쿼리 문장을 어떻게 하면 가장 빠르게 처리할지 결정하는 역할을 담당한다.
- 옵티마이저가 더 나은 선택을 할 수 있게 유도하는 과정이 MySQL 성능 최적화와 관계가 있다.
  * 쿼리분석 : Where 절의 검색 조건인지 Join 조건인지 판단
  * 인덱스 선택 : 각 테이블의 인덱스 정보를 통해 어떤 인덱스를 사용할지 결정
  * 조인 처리 : 여러 테이블의 조인이 있는 경우 어떤 순서로 테이블을 읽을지 결정

4. Execution Engine (실행 엔진)
- 실행엔진은 옵티마이저가 만들어낸 계획대로 스토리지 엔진에게 요청하고 결과를 사용자 혹은 다른 모듈로 넘긴다.

5. Storage Engine (스토리지 엔진)
- 실행엔진의 요청에 따라 데이터를 디스크로 저장하거나 디스크로부터 읽어오는 역할을 담당한다.
- MyISAM, InnoDB 등이 있으며 MySQL은 InnoDB 엔진이 DEFAULT 이다.


## MySQL 쿼리 문장 실행 순서
- FROM
- ON
- JOIN
- WHERE
- GROUP BY
- CUBE or ROLLUP
- HAVING
- SELECT
- DISTINCT
- ORDER BY
- TOP

#### 참고
- <https://12bme.tistory.com/73>
- <https://jdm.kr/blog/91>