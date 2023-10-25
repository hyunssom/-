## 13장 Pairwise & Nonpairwise

### 따라하기 - 부품 정보 읽기


- TEST09 
	LINT (공정) SPEC (제품사양) ITEM (단위품목) QTY (수량) <br>
	-> 특정 공정에서 특정 사양의 제품을 만들기 위해 어떤 부품이 얼마나 들어가는지 확인하며 60행이 있다
<IMG SRC="C:\Users\HP\OneDrive\바탕 화면\스샷\스크린샷 2023-10-21 094719.png">

- TEST10
	IDATE (투입일자), IN_SEQ ( 투입순서 ), LINE ( 공정 ), SPEC ( 제품사양 )
<IMG SRC="C:\Users\HP\OneDrive\바탕 화면\스샷\스크린샷 2023-10-21 094915.png">

```
1999년 2월 3일에 쓰이는 부품의 LIST 를 보고자 한다 TEST10 의 일자가 '19990203' 인 데이터 중 공정과 SPEC 에 관한 정보만 가지고 TEST09 을 가지고 온다

1. 첫번째 방법, 두 개 테이블의 조인을 이용하여 부품만을 UNIQUE 하게 읽어 처리한다
2. 두번째 방법, TEST10에서 공정별로 투입되는 SPEC 정보를 서브쿼리로 읽고 그 정보를 이용해서 공정과 SPEC 이 그 중 포함되는 데이터의 부품만을 구별하여 읽어온다 
```

```SQL
두 테이블을 조인해서 부품만을 unique 하게 읽어 처리하는 방법

SELECT DISTINCT ITEM
FROM TEST09 A, TEST10 B
WHERE B.IDATE = '19990203'
AND A.LINE = B.LINE
AND A.SPEC = B.SPEC;
```

<img src="C:\Users\HP\OneDrive\바탕 화면\스샷\스크린샷 2023-10-21 111540.png">


```sql
서브쿼리를 이용하는 방법

SELECT DISTINCT ITEM
FROM TEST09
WHERE LINE IN (SELECT LINE
				  FROM TEST10 WHERE IDATE = '19990203')
AND SPEC IN (SELECT SPEC
				FROM TEST10 WHERE IDATE = '19990203');
```

<IMG SRC="C:\Users\HP\OneDrive\바탕 화면\스샷\스크린샷 2023-10-21 112310.png">

조인을 이용했을 때는 14건이 출력되고 서브쿼리를 이용하면 15건이 출력된다

```SQL
SELECT DISTINCT ITEM
FROM TEST09
WHERE (LINE, SPEC) IN (SELECT LINE, SPEC
							  FROM TEST10
							  WHERE IDATE = '19990203');
```

<IMG SRC="C:\Users\HP\OneDrive\바탕 화면\스샷\스크린샷 2023-10-21 111540.png">

14건이 출력된다

#### 어떤 결과가 맞을까?

다른 건들과의 차이는 **P16** 이다 이 P16 에 대해서 더 알아보면,
<IMG SRC="C:\Users\HP\OneDrive\바탕 화면\스샷\스크린샷 2023-10-21 113023.png">

3번 공정에서 A002 SPEC 을 생산할 때 2개를 사용하고 있다

```SQL
SELECT SPEC
FROM TEST10
WHERE LINE = '03'
```

<IMG SRC="C:\Users\HP\OneDrive\바탕 화면\스샷\스크린샷 2023-10-21 113401.png">

그런데 해당일의 3번 공정에서는 A002 SPEC을 생산할 계획이 없다 <br>
그렇다면 서브쿼리를 사용한 방식이 왜 다른 결과를 낸 걸까? <br>
	해당일에 03번의 공정에서 생산이 있고, A002 의 사양도 생산한다 하지만, P16의 부품을 사용하는 03번 공정의 A002 사양은 생산 계획이 없다  <br>
	공정과 SPEC 이 마치 하나의 칼럼처럼 조건을 걸어야 하는데 그렇게 되지 않아 생긴 변수이다

### 13-1 Pairwise 와 NonPairwise

```
'19990203' 일자 생상되는 공정별 제품에 필요한 부품을 제외한 공정, 사양, 부품을 찾는 쿼리를 예제에서 보여준 세가지 형식을 이용해 작성하고 결과를 비교하기
```

```sql
--1. JOIN
SELECT DISTINCT A.LINE, A.SPEC, A.ITEM
FROM TEST09 A, TEST10 B
WHERE B.IDATE = '19990203'
AND A.LINE <> B.LINE
AND A.SPEC <> B.SPEC;
```

- `B.IDATE = '19990203'` 중,  <br>
	1. `AND A.LINE <> B.LINE` : A 테이블과 B 테이블의 공정이 같지 않고
	2. `AND A.SPEC <> B.SPEC` : A 테이블과 B 테이블의 SPEC 이 같지 않아야 한다 <br>
-> 공정별 제품에 필요한 부품을 제외 = `AND A.LINE <> B.LINE` 공정별 `AND A.SPEC <> B.SPEC` 제품에 필요한 부품을 제외한다

<IMG SRC="C:\Users\HP\OneDrive\바탕 화면\스샷\스크린샷 2023-10-23 190859.png">
	총 60행이 출력된다

```SQL
--2. Nonpairwise
SELECT DISTINCT LINE, SPEC, ITEM
FROM TEST09
WHERE LINE NOT IN
		(SELECT LINE
		FROM TEST10 WHERE IDATE = '19990203')
AND SPEC NOT IN
	(SELECT SPEC
	FROM TEST10 WHERE IDATE = '19990203');
```

- `NOT IN (SELECT LINE FROM TEST10 WHERE IDATE = '19990203')` <br>
	:  공정 날짜가 19990203 외의 날짜는 데이터에 없기 때문에 제대로 출력되지 않는다

<IMG SRC="C:\Users\HP\OneDrive\바탕 화면\스샷\스크린샷 2023-10-23 190932.png">
	0 행이 출력된다


```SQL
--3. Pairwise
SELECT DISTINCT LINE, SPEC, ITEM
FROM TEST09
WHERE (LINE, SPEC) NOT IN
		(SELECT LINE, SPEC
		FROM TEST10
		WHERE IDATE = '19990203');
```

- 서브쿼리로 해당되는 일자의 공정과 제품을 테이블을 작성하고 NOT IN 으로 결과를 제한한다

<IMG SRC="C:\Users\HP\OneDrive\바탕 화면\스샷\스크린샷 2023-10-23 191030.png">
	30행이 출력된다

**JOIN 과 PAIRWISE** 의 결과가 다른 이유
1. 조인 방식의 차이
	: Pairwise 쿼리에서는 서브쿼리를 사용하여 필터링하고 조인하지 않는다 <br>
	: Join 쿼리는 조인 조건을 만족하는 행들을 추려낸 후에 DISTINCT를 적용하여 중복을 제거한다 <br>
	(조인 조건 = WHERE ~ AND ~ AND)


----
