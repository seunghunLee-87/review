하루 30분 36강으로 배우는 완전 초보의 SQL 따라잡기
==================================================

![표지](./images/표지.png)

거의 대부분이 아는 내용들이라 아마 많은 분들은 보면서 크게 오오 하는건 없으실것 같다.  
하지만 나는 최근 2년간 ORM만 사용하다보니 실제 쿼리 작성할 일이 거의 없어서 (기획자분들께 데이터를 CSV로 뽑아드릴때나, ISMS 심사할때 심사관들 앞에서 쿼리날릴때 정도?) 그래도 기본은 알아야하지 않나 싶어서 지하철 출퇴근 시간에 읽으면서 정리중! <br/>

블로그에 바로 쓸까 Github에 쓸까 하다가 마크다운 에디터를 Visual Studio Code에서 Atom으로 환승한 기념으로!! 마크다운으로 작성한다. (정말 좋다. 나중에 시간이 되면 sublime text/ visual studio code / atom 비교글을 써야겠다.)<br/>

참고로 아래 내용은 책의 모든 내용은 아니다. 개인적으로 중요하다 생각한 부분만 정리중인거라 좋은 내용이 무지 많으니 한권쯤 사서 정리해보는 것도 좋을것 같다!

정리
----

### 기본

-	기본 예약어 명령어 처리 순서
	-	where -> select -> order by
	-	where 처리가 select보다 먼저 처리 되기 때문에 select에서 지정한 as는 where에서 사용할 수는 없다.<br/>

### 그룹화

-	COUNT(열 이름)

	-	count()는 인자에 추가한 열의 개수를 구할때 null은 제외하고 추출한다.
	-	즉, 5개의 행이 있는데 1개의 행이 name : null인 경우 count(name) 할 경우 4개 출력
	-	이외에도 SUM, AVG, MIN, MAX 등의 집계함수가 있다.
	-	집계함수들은 where에서 사용할 수 없다.

-	DISTINCT 열 이름

	-	select -> distinct
	-	ex) SELECT DISTINCT name FROM sample;
	-	name의 중복 제거 및 null을 제외한 상태에서의 행 개수를 구하면?
	-	SELECT COUNT(DISTINCT name) FROM sample;
	-	count처럼 다른 집계함수들도 동일하게 DISTINCT와 조합하여 사용 가능하다.<br/>

-	GROUP BY

	-	group by나 distinct는 집계함수를 사용하지 않는 한 큰 차이가 없다.
	-	내부처리 순서 : where -> group by -> select -> order by
	-	SELECT name, COUNT(name) FROM sample WHERE **COUNT(name)**=1 GROUP BY name;
	-	위 쿼리는 실행되지 않는다. 내부처리 순서로 인해 GROUP BY가 WHERE보다 이후에 실행되기 때문에 그룹화 된 결과에서 조건을 실행시킬 수가 없다.
	-	이렇게 그룹화 된 결과물에서 조건절 실행을 하려면 **HAVING** 응 사용한다.<br/>

-	HAVING

	-	SELECT name, COUNT(name) FROM sample GROUP BY name **HAVING COUNT(name) = 1**;
	-	내부처리 순서 : where -> group by -> having -> select -> order by<br/>

-	복수열 그룹화

	-	SELCT no, name, quantity FROM sample GROUP BY name;
	-	위 쿼리는 오류가 발생한다.
	-	name으로 그룹화 하였지만, name이 A인 그룹의 quantity는 1,2로 2행이 있으면 어느 값을 반환할지 몰라 에러가 발생한다.
	-	이럴 경우 다른 열들도 집계함수에 추가하면 에러를 피할 수 있다.
	-	SELECT **MIN(no)**, name, **SUM(quantity)** FROM sample GROUP BY name;

-	그룹화의 결과물 정렬

	-	SELECT MIN(no), name, SUM(quantity) FROM sample GROUP BY name **ORDER BY SUM(quantity) DESC**;

### 서브쿼리

-	sample 테이블에서 가장 작은 a 값을 가지고 있는 행 삭제하기

	-	대부분의 DB : DELETE FROM sample WHERE a = **(SELECT MIN(a) FROM sample)**;
	-	MySQL : DELETE FROM sample WHERE a = **(SELECT a FROM (SELECT MIN(a) AS a FROM sample) AS x)**;
	-	MySQL은 데이터를 추가/갱신 할 경우 동일 테이블을 서브쿼리에 사용할 수 없도록 되어있어, **인라인 뷰로 임시 테이블을 만들어 처리** 해야만 한다.

-	각종 상황에서 서브쿼리 사용하기

	-	set에서 서브쿼리 사용하기
		-	UPDATE sample SET a = (SELECT MAX(a) FROM sample);
	-	FROM에서 서브쿼리 사용하기
		-	SELECT * FROM (SELECT * FROM sample) sq;
	-	INSERT에서 서브쿼리 사용하기
		-	INSERT INTO sample VALUES ((SELECT COUNT(name) FROM sample));
		-	INSERT INTO sample SELECT 1,2;

### 인덱스

-	인덱스의 목적은 검색 속도 향상이다
	-	즉, SELECT & WHERE을 이용한 조건 검색시 효율적으로 검색하도록 지정하는 것
-	DB의 인덱스에서 쓰이는 대표적인 알고리즘은 **이진트리** , 해시가 있다.
	-	이진트리의 경우 데이터가 미리 정렬되어 있어야 하는데 테이블의 데이터를 언제나 정렬된 상태로 둘수는 없다.
	-	테이블에 인덱스를 작성하면 테이블 데이터와 별개로 인덱스용 데이터가 생성되는데 이때 인덱스들을 **이진트리** 구조로 작성된다.
-	사용 : CREATE INDEX 인덱스명 ON 테이블명 (열명1, 열명2)
-	모든 데이터베이스는 해당 쿼리 실행시 인덱스를 사용하는지 안하는지 확인할 수 있는 예약어가 있다. (ex : EXPLAIN)

### 조인

- Cross Join
  - SELECT * FROM sample1, sample2;
  - 2개의 테이블에서 나올 수 있는 모든 경우의 수를 나열

- INNER JOIN
  - SELECT * FROM 테이블명1 INNER JOIN 테이블명2 ON 검색조건
  - 이전에는 where에 결합조건을 지정하였지만 INNER JOIN에서는 ON을 사용하여 결합조건을 지정
  - 예제쿼리

```
SELECT product.name, stock.count
FROM product INNER JOIN stock
ON product.idx = stock.product_idx
WHERE product.category = '식료품';
```

- OUTER JOIN
  - **한 쪽에만 존재하는 데이터** 행을 다루는 JOIN
  - INNER JOIN 자리에 LEFT JOIN 혹은 RIGHT JOIN으로 변경해주면 된다.
  - JOIN의 방향은 기준이 되는 테이블 즉, NULL이 허용되지 않는 테이블쪽을 바라보면 된다.
  - 예를 들어 상품 테이블을 기준으로 재고 테이블과 JOIN을 맺지만, 재고 테이블은 데이터가 있을 수도 있고, 없을 수도 있을 경우

```
SELECT product.name, stock.count
FROM product LEFT JOIN stock
ON product.idx = stock.product_idx
WHERE product.category = '식료품';
```

### 정규화
- 결국 하나의 데이터는 한 곳에 저장되어야 한다는 것
  - 객체지향 언어에서의 하나의 객체는 하나의 책임을 져야하는 것과 비슷하다는 생각이 든다.

### 후기
> 두께에 비해 가격은 좀 비쌌지만(22000원 ㅠㅠ) 그래도 읽길 잘했던 것 같다. <br/>
처음에는 데이터 모델링 파트를 건너뛸까 했지만, 읽어보길 잘했다는 생각이 드는건, <br/>
결국 데이터베이스 모델도, 객체지향의 객체도 지향하는바는 똑같은것 같다.<br/>
하나의 요소는 하나의 책임을 가지게 하는 것 <br/>
중복을 제거할 것 <br/>
ORM에 중독된 사람들은 한번 머리를 환기 시키는 셈치고 읽어보면 좋을것 같다.
