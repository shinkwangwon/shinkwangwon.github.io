---
layout: post
title: "MySQL Explain"
tags: [mysql]
comments: true
date: 2021-07-16
---

# 6. 실행계획

## 6.2 실행계획 분석

- UPDATE, DELETE, INSERT 는 실행계획 분석 불가능
- WHERE 절이 동일한 SELECT 문을 만들어서 대략적으로 계획확인만 가능

### 6.2.1  ID 칼럼

- 단위 SELECT 쿼리별로 부여되는 식별자 값
- 하나의 SELECT 문장은 다시 1개 이상의 SUB SELECT 문장 포함 가능

```sql
SELECT ...
FROM (SELECT ... FROM tb_test1) tb1,
	tb_test2 tb2 
WHERE tb1.id=tb2.id;

-- 위의 쿼리 문장의 각 SELECT 는 다음 과 같이 분리해서 생각해볼 수 있다. 
-- 이렇게 SELECT 키워드 단위로 구분한 것을 단위(SLLECT)쿼리 라고 표현 
SELECT ... FROM tb_test1;
SELECT ... FROM tb1, tb_test2 tb2 WHERE tb1.id = tb2.id
```

- 실행 계획에서 가장 왼쪽에 표시되는 id 칼럼은 단위 SELECT 쿼리별로 부여되는 식별자
- SELECT 문장은 하나인데 여러개의 테이블이 조인되는 경우, 실행계획 결과의 row는 테이블 개수만큼 나오지만 id 값은 증가하지 않고 같은 id가 부여됨
- 반대로 쿼리 문장이 3개의 단위 SELECT 쿼리로 구성되어 있으면 실행계획의 각 row는 각기 다른 id를 지님

### 6.2.2 select_type 칼럼

**SIMPLE**

- UNION 이나 서브 쿼리를 사용하지 않는 단순한 SELECT 쿼리인 경우
- 쿼리에 조인이 포함된 경우에도 SIMPLE
- 쿼리가 아무리 복잡해도 실행계획에서 select_type이 SIMPLE인 단위 쿼리는 반드시 하나만 존재 (단위 쿼리 기준임. SELECT는 하나인데 조인이 많아서 실행계획 row가 많이 나오더라도 모두 SIMPLE 타입으로 나타남)
- 일반적으로 가장 바깥 SELECT 쿼리의 select_type이 SIMPLE로 표시됨

**PRIMARY**

- UNION 이나 서브쿼리가 포함된 SELECT 쿼리의 실행 계획에서 가장 바깥쪽(Outer)에 있는 단위 쿼리
- SIMPLE과 마찬가지로 select_type이 PRIMARY인 쿼리는 하나만 존재하며, 쿼리의 가장 바깥쪽 SELECT 단위 쿼리가 PRIMARY로 표시됨

**UNION**

- UNION으로 결합하는 단위 SELECT 쿼리 가운데 첫번째를 제외한 두번째 이후 단위 SELECT 쿼리의 select_type은 UNION 으로 표시
- 첫번째 단위 SELECT는 UNION이 아니라 UNION 쿼리로 결합된 전체 집합의 select_type이 표시됨

```sql
EXPLAIN
SELECT * FROM (
  (SELECT emp_no FROM employees e1 LIMIT 10)
  UNION ALL
  (SELECT emp_no FROM employees e2 LIMIT 10)
  UNION ALL
  (SELECT emp_no FROM employees e3 LIMIT 10)
) tb;
```

![No image](/assets/posts/20210716/Untitled.png)

- 첫번째 SELECT만 UNION이 아니고 나머지 모두 UNION으로 표시됨
- 여기서 3개의 서브쿼리로 조회된 결과를 UNION ALL로 결합해서 임시 테이블로 만들어 사용하므로 첫번쨰 쿼리는 DERIVED 라는 select_type을 가짐

**DEPENDENT UNION**

- UNION 이나 UNION_ALL로 결합된 단위 쿼리가 외부의 영향을 받는 것을 의미
- UNION 으로 결합되는 각 쿼리가 외부에 정의된 값을 참조해서 처리 될 때 DEPENDENT 키워드가 select_type에 표시됨
- 아래 예시에서 UNION 으로 결합되는 각 쿼리가 외부에서 정의된 employees.emp_no 칼럼을 사용하고 있음

```sql
EXPLAIN
SELECT 
  e.first_name,
  ( SELECT CONCAT('Salary change count : ', count(*)) AS message
    FROM salaries s WHERE s.emp_no=e.emp_no
    UNION
    SELECT CONCAT('Department change count : ', count(*)) AS message
    FROM dept_emp de WHERE de.emp_no=e.emp_no
  ) AS message
FROM employees e
WHERE e.emp_no=10001;
```

- 하나의 단위 SELECT 쿼리가 다른 단위 SELECT를 포함하고 있는 것을 서브쿼리라고 하는데, 이처럼 서브 쿼리가 사용된 경우에는 외부 쿼리보다 서브 쿼리가 먼저 실행된다(이 방식이 반대의 경우보다 대부분 빠르게 처리됨). 하지만 select_type에 DEPENDENT 키워드를 포함하는 서브 쿼리는 외부 쿼리에 의존적이므로 절대 외부 쿼리보다 먼저 실행될 수가 없다. 그래서 select_type에 DEPENDENT 키워드가 포함된 쿼리는 비효율적인 경우가 많다.

**DEPENDENT SUBQUERY**

- 서브쿼리가 바깥쪽 SELECT 쿼리에서 정의된 칼럼을 사용하는 경우
- 바깥쪽 쿼리가 먼저 수행된 후 내부 쿼리(서브쿼리)가 실행되야 하므로 일반 서브쿼리보다는 처리 속도 느림

```sql
EXPLAIN
SELECT e.first_name,
	(SELECT COUNT(*)
	 FROM dept_emp de, dept_manager dm
	 WHERE dm.dept_no=de.dept_no AND de.emp_no=e.emp_no) AS cnt
FROM employees e
WHERE e.emp_no=10001;
```

**DERIVED**

- 서브쿼리가 FROM 절에 사용된 경우는 select_type이 항상 DERIVED인 실행계획을 만듬
- DERIVED는 단위 SELECT 쿼리의 실행결과를 메모리나 디스크에 임시테이블을 생성하는 것을 의미 (이때 임시테이블을 파생테이블이라고도 함)
- 파생테이블에는 인덱스가 전혀 없으므로 다른 테이블과 조인할 때 성능상 불리
- 가능하다면 FROM절에 서브쿼리를 제거하고 조인으로 처리하는 것이 좋다.

```sql
EXPLAIN
SELECT * 
FROM 
	(SELECT de.emp_no FROM dept_emp de) tb,
	employees e
WHERE e.emp_no=tb.emp_no;
```

### 6.2.3 Table

- MySQL의 실행계획은 단위 SELECT 쿼리 기준이 아니라 테이블 기준으로 표시됨
- 테이블에 별칭이 부여된 경우 별칭이 표시되고 별도의 테이블을 사용하지 않는 SELECT 쿼리는 null이 표시됨
- Table 칼럼에 <derived>, <union> 같이 "<>"로 둘러싸인 경우가 있는데, 이 테이블은 임시테이블을 의미하고, "<>"안에 표시되는 숫자는 단위 SELECT 쿼리의 id를 지칭함

![No image](/assets/posts/20210716/Untitled1.png)

### 6.2.4 type

타입 종류는 아래와 같고(MySQL버전별로 조금씩 다를 수 있음) 성능이 빠른 순서대로 나열해서 표시한 것이다

- system
- const
- eq_ref
- ref
- fulltext
- ref_or_null
- unique_subquery
- index_subquery
- range
- index_merge
- index
- ALL

**system**

- 레코드가 1건만 존재하는 테이블 또는 한 건도 존재하지 않는 테이블을 참조할 때
- InnoDB 테이블에서는 나타나지 않고 MyISAM 이나 MEMORY 테이블에서만 사용되는 접근 방법

**const**

- 테이블 레코드 건수에 관계없이 쿼리가 프라이머리 키나 유니크 키 칼럼을 이용하는 WHERE 조건절을 가지고 있으며, 반드시 1건을 반환하는 쿼리인 경우
- 프라이머리 키나 유니크 키 중에서 인덱스의 일부 컬럼만 조건으로 사용할 때는 const 타입 사용 불가. 이 경우 실제 레코드가 1건만 저장돼 있더라도 const 사용불가

```sql
EXPLAIN
SELECT * FROM employees WHERE emp_no=10001;
```

![No image](/assets/posts/20210716/Untitled2.png)

**eq_ref**

- eq_ref 접근 방법은 여러 테이블이 조인되는 쿼리의 실행계획에서만 표시됨
- 조인에서 처음 읽은 테이블의 컬럼 값을, 그 다음 읽어야할 테이블의 프라이머리 키나 유니크 키 칼럼의 검색 조건에 사용할 때를 eq_ref라고 표시됨

```sql
EXPLAIN
SELECT * FROM dept_emp de, employees e
WHERE e.emp_no=de.emp_no AND de.dept_no='d005';
```

![No image](/assets/posts/20210716/Untitled3.png)

**ref**

- ref 접근 방법은 eq_ref와는 달리 조인의 순서와 관계없이 사용되며, 또한 프라이머리 키나 유니크 키 등의 제약 조건도 없다
- 인덱스의 종류와 관계없이 동등(equal)조건으로 검색할 때는 ref 접근 방법이 사용된다
- ref 타입은 반환되는 레코드가 반드시 1건이라는 보장이 없으므로 const나 eq_ref 보다는 빠르지 않지만, 동등 조건으로만 비교되므로 매우 빠른 레코드 조회방법 중 하나이다

```sql
-- PK(dept_no + emp_no)
EXPLAIN
SELECT * FROM dept_emp WHERE dept_no='d0005';
```

![No image](/assets/posts/20210716/Untitled4.png)

**unique_subquery**

- WHERE 조건절에서 사용될 수 있는 IN (subquery) 형태의 쿼리를 위한 접근 방식
- unique_subquery의 의미 그대로 서브 쿼리에서 중복되지 않은 유니크한 값만 반환할 때 이 접근방법 사용

```sql
EXPLAIN
SELECT * FROM departments WHERE dept_no IN (
	SELECT dept_no FROM dept_emp WHERE emp_no=10001);
```

**index_subquery**

- IN (subquery) 에서 subquery가 중복된 값을 반환할 수는 있지만 중복된 값을 인덱스를 이용해 제거할 수 있을때 이 접근방법이 사용됨
- 아래에서 서브쿼리는 프라이머리 키의 dept_no 칼럼을 기준으로 'd001' 부터 'd003' 까지 읽으면서 dept_no 값만 가져오면 된다. 또한 이미 프라이머리 키는 dept_no 칼럼 기준으로 정렬되어 있어서 중복된 dept_no 제거를 위해 별도의 정렬작업이 필요하지 않다. 이런 경우 index_subquery 방식이 사용된다

```sql
-- PK(dept_no + emp_no)
EXPLAIN
SELECT * FROM departments WHERE dept_no IN (
	SELECT dept_no FROM dept_emp WHERE dept_no BETWEEN 'd001' AND 'd003');
```

**range**

- 익히 알고 있는 인덱스 레인지 스캔 형태의 접근 방법
- range는 인덱스를 하나의 값이 아니라 범위로 검색하는 경우를 의미하는데, 주로 "< , > , IS NULL , BETWEEN , IN , LIKE" 등의 연산자를 이용해 인덱스를 검색할 때 사용
- 접근 방법 중 우선순위가 낮지만, 이 접근 방법도 상당히 빠르며 모든 쿼리가 이 접근 방법만 사용해도 어느 정도의 성능은 보장된다고 볼 수 있음
- 이 책에서 인덱스 레인지 스캔이라고 하면 const, ref, range 라는 세가지 접근 방법을 모두 묶어서 지칭하는 것이다. "인덱스를 효율적으로 사용한다" 또는 "범위 결정(제한) 조건으로 인덱스를 사용한다"는 표현 모두 이 세가지 접근 방법을 의미한다.

**index_merge**

- index_merge 접근 방식은 이전까지 설명한 것과 달리 2개 이상의 인덱스를 이용해 각각의 검색 결과를 만들어낸 후 그 결과를 병합하는 처리 방식이다
- 아래쿼리는 emp_no는 프라이머리키를 이용해 조회하고, first_name은 ix_firstname인덱스를 이용해 조회한 후 두 결과를 병합하는 형태로 처리

```sql
EXPLAIN
SELECT * FROM employees
WHERE emp_no BETWEEN 10001 AND 11000
	OR first_name='Smith';
```

**index**

- index 접근 방법은 이름이 index라서 효율적으로 인덱스를 사용한다고 생각할 수 있지만, 이는 인덱스 풀스캔을 의미한다
- range 접근 방식과 같이 효율적으로 인덱스의 필요한 부분만 읽는 것을 의미하는 것은 아니라는 점을 유의!
- index 접근 방법은 아래 조건 3개 가운데 (첫번째+두번째) 조건을 충족하거나, (첫번째+세번째) 조건을 충족하는 쿼리에서 사용된다
    - ragne나 const 또는 ref와 같은 접근 방식으로 인덱스를 사용하지 못하는 경우
    - 인덱스에 포함된 칼럼만으로 처리할 수 있는 쿼리인 경우(즉, 데이터 파일을 읽지 않아도 되는 경우)
    - 인덱스를 이용해 정렬이나 그룹핑 작업이 가능한 경우(즉, 별도의 정렬 작업을 피할 수 있는 경우)
- 아래 쿼리는 WHERE 조건이 없으므로 range, const, ref 접근 방식 사용이 불가능하지만, 정렬하려는 칼럼은 인덱스(ux_deptname)가 있으므로 별도의 정렬 처리를 피하려고 index 접근 방식이 사용된다

```sql
EXPLAIN
SELECT * FROM departments ORDER BY dept_name DESC LIMIT 10;
```

**ALL**

- 풀 테이블 스캔을 의미하는 접근 방식

### 6.2.5 possible_keys

- MySQL 옵티마이저가 최적의 실행 계획을 만들기 위해 후보로 선정했던 접근 방식에서 사용되는 인덱스의 목록일뿐, 말 그대로 "사용될 법했던 인덱스의 목록"
- 실행계획 확인할 때 Possible_keys 칼럼은 그냥 무시하자.

### 6.2.6 key

- 최종 선택된 인덱스
- PRIMARY인 경우에는 프라이머리 키를 사용한다는 의미이며, 그 이외의 값은 모두 테이블이나 인덱스를 생성할 때 부여했던 고유 이름
- 실행 계획의 type 칼럼이 index_merge 가 아닌 경우에는 반드시 테이블 하나당 하나의 인덱스만 이용 가능
- ALL 과 같이 인덱스를 전혀 사용하지 못하면 key 칼럼은 NULL로 표시

### 6.2.7 key_len

- 인덱스에서 몇개의 칼럼까지 사용했는지 알 수 있음. 더 정확하게는 인덱스의 각 레코드에서 몇 바이트까지 사용했는지 알려주는 값이다

```sql
EXPLAIN
SELECT * FROM dept_emp WHERE dept_no='d005';
-- key_len 12 표시됨
-- dept_no 칼럼타입이 CHAR(4)이기 때문
-- MySQL에서는 utf8 문자를 위해 1개의 문자당 3바이트로 고정해서 메모리 할당
-- 그래서 key_len 칼럼의 값은 12바이트(4 * 3 바이트)로 표시된 것 
```

```sql
EXPLAIN
SELECT * FROM dept_emp WHERE dept_no='d005' AND emp_no=10001;
-- key_len 16 표시됨
-- emp_no 칼럼 타입은 INTEGER 4바이트 
-- 그래서 key_len 칼럼의 값은 12바이트(4 * 3) + 4바이트 = 16바이트로 표시된 것
```

### 6.2.8 ref

- 접근 방법이 ref 방식이면 참조조건(Equal 비교 조건)으로 어떤 값이 제공됐는지 표시
- 상수 값 지정했다면 const로 표시되고, 다른 테이블의 칼럼값이면 그 테이블명과 칼럼명 표시
- 종종 func 라고 표시될 때가 있는데 이는 참조용으로 사용되는 값을 그대로 사용한 것이 아니라, 콜레이션 변환이나 값 자체의 연산을 거쳐서 참조됐다는 것을 의미함. 최대한 없애는 것이 좋다

### 6.2.9 rows

- rows 컬럼에 표시되는 정보는 반환하는 레코드의 예측치가 아니라, 쿼리를 처리하기 위해 얼마나 많은 레코드를 디스크로부터 읽고 체크해야 하는지를 의미

### 6.2.10 Extra

**Not exists**

- A 테이블에는 존재하지만 B테이블에는 없는 값을 조회할 때 NOT IN (subquery) 또는 NOT EXISTS 연산자를 사용할때가 있음
- 이런 형태의 조인을 안티조인(Anti-JOIN)이라고 하는데, 똑같은 처리를 OUTER JOIN을 이용해도 구현 가능. 일반적으로 레코드 건수가 많을때는 NOT IN (subquery) 또는 NOT EXISTS 보다 OUTER JOIN을 이용하는게 성능상 빠름
- `ANTI-JOIN은 일반 조인(Inner join) 했을 때 나오지 않는 결과만 가져오는 방법`

```sql
EXPLAIN
SELECT *
FROM dept_emp de
	LEFT JOIN departments d ON de.dept_no=d.dept_no
WHERE d.dept_no IS NULL;
```

- 이렇게 아우터 조인을 이용해 안티조인을 수행하는 쿼리에서는 Extra 칼럼에 Not exists 메시지 표시됨
- Not exists 는 이 쿼리를 NOT EXISTS 형태의 쿼리로 변환해서 처리했음을 의미하는 것이 아니라 MySQL이 내부적으로 어떤 최적화를 했는데 그 최적화의 이름이 "Not exists"인 것이다.
- Extra 칼럼의 `Not exists` 와 SQL의 `NOT EXISTS 연산자` 를 혼동하지 않도록 주의

**Using filesort**

- Order by 처리를 위해 인덱스를 사용하지 못하는 경우 표시됨
- 조회된 레코드를 정렬용 메모리 버퍼에 복사해 퀵소트 알고리즘을 수행하는 절차가 필요
- 쿼리에 많은 부하를 일으키므로 쿼리를 튜닝하거나 인덱스를 생성하는 것이 좋음

**Using index(커버링 인덱스)**

- 데이터 파일을 전혀 읽지 않고 인덱스만 읽어서 쿼리를 모두 처리할 수 있는 경우
- 커러인 인덱스가 사용되지 않는 쿼리보다 훨씬 빠르게 처리됨

**Using index for group-by**

- MySQL은 GROUP BY 처리를 위해 기룹핑 기준 칼럼을 이용해 정렬작업을 수행한 후 다시 정렬된 결과를 그룹핑하는 형태의 고부하 작업을 필요로 함
- GROUP BY 처리에 인덱스를 이용하면 정렬작업이 필요 없기 때문에 상당히 효율적
- GROUP BY 처리가 인덱스를 이용할 때 Using index for group-by 가 표시됨

**Using join buffer**

- 실제로 조인에 필요한 인덱스는 조인되는 양쪽 테이블 칼럼 모두가 필요한 것이 아니라 조인에서 뒤에 읽는 테이블의 칼럼에만 필요
- 조인에서 뒤에 읽는 테이블에 인덱스가 없다면 드라이빙 테이블로부터 읽은 레코드의 건수만큼 매번 드리븐 테이블을 풀 테이블 스캔이나 인덱스 풀 스캔을 해야 함
- 이런 드리븐 테이블의 비효율적인 검색을 보완하기 위해 MySQL서버는 드라이빙 테이블에서 읽은 레코드를 임시 공간(조인버퍼)에 보관해두고  재사용할 수 있게 함
- 이렇게 조인 버퍼가 사용되는 실행계획의 Extra 칼럼에 Using joib buffer 라고 표시됨
- 조인 버퍼는 join_buffer_size 라는 시스템 설정 변수에  최대값을 설정할 수 있지만 일반 온라인 웹 서비스용에서는 1MB 정도로 충분함
- 조인 조건이 없는 카테시안 조인의 경우에는 항상 조인 버퍼를 사용

```sql
EXPLAIN
SELECT *
FROM dept_emp de, employees e  -- 카테시안 곱(조인)
WHERE de.from_date > '2005-0101' AND e.emp_no < 10904;
```

**Using sort_union , Using union , Using intersect**

쿼리가 Index_merge 접근 방식으로 실행되는 경우 2개의 인덱스가 동시에 사용될 수 있다. 이때 Extra 칼럼에는 두 인덱스로부터 읽은 결과를 어떻게 병합했는지 조금 더 상세하게 설명하기 위해 아래 중 하나를 표시한다

- Using intersect
    - 각각의 인덱스를 사용하는 조건이 AND로 연결된 경우 각 처리 결과에서 교집합을 추출해내는 작업을 수행했다는 의미
- Using union
    - 각 인덱스를 사용할 수 있는 조건이 OR로 연결된 경우 각 처리 결과에서 합집합을 추출해내는 작업을 수행했다는 의미
- Using sort_union
    - Using  union과 같은 작업을 수행하지만 Usion union으로 처리될 수 없는 경우(OR로 연결된 상대적으로 대량의 range 조건들)에 이 방식으로 처리된다. 차이점은 Using sort_union은 프라이머리 키만 먼저 읽어서 정렬하고 병한 후에야 비로소 레코드를 읽어서 반환할 수 있다는 것이다

**Using temporary**

- 쿼리를 처리하는 동안 중간 결과를 담아두기 위한 임시 테이블을 사용하는 경우 표시됨
- 임시 테이블은 메모리상에 생성될 수도, 디스크상에 생성될 수도 있음
- 임시 테이블이 메모리 또는 디스크 중 어디에 생성됐는지는 실행 계획만으로 판단 불가능
- 아래 쿼리는 GROUP BY 컬럼과 ORDER BY 컬럼이 다르기 때문에 임시 테이블이 필요한 쿼리
- GROUP BY 가 인덱스를 사용하지 못하기 때문에 Using temporary가 표시됨

```sql
EXPLAIN
SELECT * FROM employees GROUP BY gender ORDER BY MIN(emp_no);
```

- 실행 계획의 Extra 칼럼에 Using temporary가 표시되지 않아도 내부적으로 임시테이블을 사용할 때도 많다. Extra 칼럼에 Using temporary가 표시되지 않았다고 해서 임시 테이블을 사용하지 않는다라고 판단하면 안된다. 대표적으로 메모리나 디스크에 임시 테이블을 생성하는 쿼리는 다음과 같다.
    - FROM 절에 사용된 서브쿼리 (임시테이블 = 파생테이블 = Derived table)
    - COUNT(DISTINCT column) 을 포한하는 쿼리도 인덱스를 사용할 수 없는 경우 임시테이블이 만들어짐
    - UNION, UNION ALL 이 사용된 쿼리
    - 인덱스를 사용하지 못하는 정렬 작업 ( Using filsort도 결국 임시테이블을 사용하는 것)

**Using where**

- MySQL은 스토리지엔진(InnoDB, MyISAM)과 MySQL엔진이라는 2개의 레이어로 구분됨
- MySQL 엔진 레이어에서 별도의 가공을 해서 필터링 작업을 처리한 경우에 "Using where"가 표시됨
- 스토리지 엔진에서 전체 200건의 레코드를 읽었는데 MySQL엔진에서 별도의 필터링이나 가공없이 그대로 클라이언트로 전달하면 "Using where"가 표시되지 않음
- 작업 범위 제한 조건은 스토리지 엔진 레벨에서 처리되고, 체크(필터링) 조건은 MySQL 엔진레이어 에서 처리됨

```sql
EXPLAIN
SELECT * FROM employees WHERE emp_no BETWEEN 10001 AND 10100 AND gender='F';
```

- 위 쿼리에서 emp_no 는 작업범위 제한조건, gender는 체크 조건으로 사용됨
- emp_no만 만족하는 조건은 100건인데, gender 까지 만족하는 조건은 37건이라고 가정하면, 스토리지 엔진에서는 100건을 MySQL엔진으로 넘겨줬지만, MySQL엔진에서는 63건의 레코드를 필터링해서 버린 후에 클라이언트로 전달함. 이런 경우 "Using where"가 표시됨

### 6.2.11 EXPLAIN EXTENDED(Filtered 칼럼)

- 바로위에 Extra 칼럼에 표시되는 "Using where"에서 필터링으로 버려지는 레코드가 얼마나 되는지 알기 위한 추가 컬럼(MySQL v5.1.12부터 표시됨)
- `EXPLAIN EXTENDED` 명령으로 실행계획 조회 ( `EXPLAIN` 명령으로만 조회해도 가능한 버전 있음(ex v8.x))
- 실행 계획에서 filtered 칼럼에는 MySQL 엔진에 의해 필터링되어 제거된 레코드는 제외하고 최종적으로 레코드가 얼마나 남았는지의 비율이 표시됨
- ex) 실행계획에 rosw 100 filtered 20 으로 표시되는 경우, 필터링되고 남은 레코드가 20%라는 의미. 즉 100건에 대해 20%인 20건이 남았다는 의미
- 단, filtered에 표시되는 값은 예측된 값일뿐이지 정확한 값은 아니라는 것에 유의해야 함

## 6.3 MySQL의 주요 처리 방식

### 6.3.1 풀 테이블 스캔

아래 조건이 일치할 때 주로 풀 테이블 스캔을 선택한다.

- 테이블의 레코드 건수가 너무 작아서 인덱스를 통해 읽는 것보다 풀 테이블 스캔을 하는 편이 더 빠른 경우 (일반적으로 테이블이 페이지 1개로 구성된 경우)
- WHERE 절이나 ON 절에 인덱스를 이용할 수 있는 조건이 없는 경우
- 인덱스 레인지 스캔을 사용할 수 있는 쿼리라 하더라도 옵티마이저가 판단한 조건 일치 레코드 건수가 너무 많은 경우(인덱스의 B-Tree를 샘플링해서 조사한 통계 정보 기준)
- 반대로, max_seeks_for_key 변수를 특정 값(N)으로 설정하면 MySQL 옵티마이저는 인덱스의 기수성(cardinality)이나 선택도(Selectivity)를 무시하고, 최대 N건만 읽으면 된다고 판단하게 한다. 이 값을 작게 설정할수록 MySQL 서버사 인덱스를 더 사용하도록 유도함

InnoDB엔진은 특정 테이블의 연속된 데이터 페이지가 읽히면 백그라운드 스레드에 의해 `Read ahead` 작업이 자동으로 시작된다

- `Read ahead`란 어떤 영역의 데이터가 앞으로 필요해질 거라는걸 예측해서 요청이 오기 전에 미리 디스크에서 읽어 InnoDB의 버퍼 풀에 가져다 두는 것을 의미
- 즉, 풀 테이블 스캔이 실행되면 처음 몇 개의 데이터 페이지는 포그라운드 쓰레드(클라이언트 쓰레드)가 페이지 읽기를 실행하지만, 특정 시점 부터는 읽기 작업을 백그라운드 쓰레드로 넘김
- 백그라운드 쓰레드가 읽기를 넘겨받는 시점부터는 한번에 4개 또는 8개의 페이지를 읽으면서 계속 그 수를 증가시키고, 이때 한번에 최대 64개의 데이터 페이지까지 읽어서 버퍼풀에 저장해둠
- 포그라운드 쓰레드는 미리 버퍼 풀에 준비된 데이터를 가져다 사용하기만 하면 되므로 쿼리가 상당히 빨리 처리됨
- 언제 Read ahead를 시작할지 시스템 변수 `innodb_read_ahead_threshold`를 이용해 변경가능
- 시스템 변수 보기 명령어 `show variables;`

### 6.3.2 ORDER BY 처리(Using filesort)

정렬을 처리하기 위해서는 인덱스를 이용하는 방법과 쿼리가 실행될 때 "Filesort"라는 별도의 처리를 이용하는 방법으로 나뉨

MySQL이 인덱스를 이용하지 않고 별도의 정렬 처리를 수행했는지는 실행계획의 Extra 칼럼에 "Using filesort"라는 코멘트가 표시되는지로 판단가능

- 인덱스 이용
    - 장점 : INSERT, UPDATE, DELETE 쿼리가 실행될 때 이미 인덱스가 정렬돼 있어서 순서대로 읽기만 하면되므로 빠름
    - 단점 : INSERT, UPDATE, DELETE 작업 시 부가적인 인덱스 추가/삭제 작업이 필요하므로 느림. 또한 인덱스 때문에 디스크 공간이 더 많이 필요하며 인덱스 개수가 늘어날수록 innodb 버퍼 풀 메모리가 많이 필요
- Filesort 이용
    - 장점 : 인덱스를 생성하지 않아도 되므로 인덱스를 이용할 때의 단점이 장점으로 바뀜. 정렬해야 할 레코드가 많지 않으면 메모리에서 Filesort 처리가 가능하므로 충분히 빠름
    - 단점 : 정렬 작업이 쿼리 실행시 처리되므로 레코드 대상 건수가 많아질수록 쿼리의 응답속도가 느림

**소트 버퍼(Sort buffer)**

- MySQL은 정렬을 수행하기 위해 별도의 메모리 공간을 할당 받는데, 이 메모리 공간을 소트버퍼라 함
- 소트 버퍼는 정렬이 필요한 경우에만 할당되며, 버퍼의 크기는 정렬해야 할 레코드의 크기에 따라 가변적으로 증가하지만 최대 사용 가능한 소트 버퍼의 공간은 `sort_buffer_size` 라는 시스템 변수로 설정 가능
- 소트 버퍼를 이ㅜ한 메모리 공간은 쿼리의 실행이 완료되면 즉시 시스템으로 반납됨

**정렬의 문제**

- 정렬해야할 레코드가 적어서 메모리에 할당된 소트 버퍼만으로 정렬할 수 있다면 빠르게 정렬이 처리되겠지만, 레코드 건수가 소트 버퍼로 할당된 공간보다 크다면 MySQL은 정렬해야할 레코드를 여러 조각을 나눠서 처리하는데 이 과정에서 임시 저장을 위해 디스크를 사용함
- 메모리의 소트 버퍼에서 정렬을 수행하고, 그 결과를 임시로 디스크에 기록해둔 후 그다음 레코드를 가져와서 다시 정렬해서 반복적으로 디스크에 임시 저장하는 방식으로 처리됨 (각 버퍼 크기 만큼씩 정렬된 레코드를 다시 병합하면서 정렬을 수행 - 이 병합 작업을 multi-merge라고 표현)
- 이 작업들은 디스크 I/O가 발생하기 때문에 성능 저하 유발
- 소트 버퍼를 크게 설정하면 디스크를 사용하지 않아서 더 빨라질 것으로 생각할 수 있지만 실제 벤치마크 결과 거의 차이가 없었다고 함
- 소트 버퍼는 글로벌 메모리 영역이 아닌 세션(로컬) 메모리 영역이기 때문에 클라이언트가 공유해서 사용 불가함. 따라서 클라이언트 커넥션이 많을 수록 소트 버퍼로 소비되는 메모리 공간이 커짐
- 소트 버퍼를 크게 설정해서 빠른 성능을 얻을 수는 없지만 디스크 I/O 사용량은 줄일 수 있다. 그래서 MySQL 서버의 데이터가 많거나 디스크의 I/O 성능이 낮은 장비라면 소트 버퍼의 크기를 더 크게 설정하는 것이 도움될 수도 있다. 하지만 소트 버퍼를 너무 크게 설정하면 서버의 메모리가 부족해져서 MySQL 서버가 메모리 부족을 겪을 수 있기 때문에 소트 버퍼의 크기는 적절히 설정하는 것이 좋다. (일반적으로 1MB 미만)
