---
layout: post
title: "MySQL DISTINCT & GROUP BY"
tags: [mysql]
comments: true
date: 2019-11-27
---


## MySQL DISTINCT
- 중복을 제거한 결과를 보기 위해 사용하는 키워드
- SELECT DISTINCT fd1 FROM tab;     // fd1 컬럼에 대해서만 중복을 제거한 결과 조회
- SELECT DISTINCT fd1, fd2 FROM tab;    // (fd1 + fd2)를 하나의 컬럼으로 보고 중복을 제거한 결과를 조회
- 위와 같이 DISTINCT 뒤에 컬럼을 여러개 나열하면, 나열된 컬럼 전부가 일치해야 중복된 값이라고 본다.

## MySQL GROUP BY
- 데이터를 그룹핑해서 조회할 때 사용하는 키워드
- SELECT fd1 FROM tab GROUP BY fd1;
- SELECT fd1, fd2 FROM tab GROUP BY fd1, fd2; // fd1으로 그룹핑한 후에 fd2로 그룹핑
- 위의 두 쿼리는 DISTINCT에서 설명한 두 쿼리와 동일한 결과물을 출력해낸다.

## MySQL DISTINCT & GROUP BY 차이
- DISTINCT는 결과를 정렬하지 않은 채로 가져오고 GROUP BY는 결과를 정렬한 다음에 가져온다.
- 따라서 정렬이 필요없을 경우에는 DISTINCT가 성능상 유리할 수 있다.
- GROUP BY를 사용하는 경우에 정렬을 하지 않도록 유도할 수 있다. (ORDER BY NULL 이용)
- DISTINCT 와 GROUP BY는 각자 고유의 기능이 있다.
  * DISTINCT : SELECT COUNT(DISTINCT fd1) FROM tab;
    - fd1 컬럼의 중복을 제거한 후 전체 카운트 개수를 조회 - 결과는 전체 카운트인 row 하나만 나온다.
    - SELECT COUNT(fd1) FROM tab GROUP BY fd1 을 하게 되면, fd1으로 그룹핑된 row들의 카운트 개수가 조회된다. - 결과 row는 그룹핑된 fd1의 개수만큼 나온다
  * GROUP BY : SELECT fd1, MIN(fd2) FROM tab GROUP BY fd1;
    - 집계함수를 사용해야 하는 경우에는 GROUP BY를 사용한다


### example
```
SELECT  count(DISTINCT A.C1)
FROM    (SELECT 'AAA' C1, '111' C2 FROM DUAL UNION ALL
        SELECT 'AAA', '222' FROM DUAL UNION ALL
        SELECT 'AAA', '222' FROM DUAL UNION ALL
        SELECT 'BBB', '333' FROM DUAL UNION ALL
        SELECT 'BBB', '444' FROM DUAL UNION ALL
        SELECT 'CCC', '444' FROM DUAL) AS A;        

# 결과 : row 1개
# 3
```

```
SELECT  count(A.C1)
FROM    (SELECT 'AAA' C1, '111' C2 FROM DUAL UNION ALL
        SELECT 'AAA', '222' FROM DUAL UNION ALL
        SELECT 'AAA', '222' FROM DUAL UNION ALL
        SELECT 'BBB', '333' FROM DUAL UNION ALL
        SELECT 'BBB', '444' FROM DUAL UNION ALL
        SELECT 'CCC', '444' FROM DUAL) AS A
group by A.C1;

# 결과 : row 3개
# 3
# 2
# 1
```


#### 참고
- <http://intomysql.blogspot.com/2011/01/distinct-group-by.html>