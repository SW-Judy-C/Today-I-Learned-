# 1. 기초 :relieved:
## 5. Subquery 
>Subquery는 위치에 따라 2가지 형식으로 다르게 작용함. `WHERE` 절에 사용하는 경우, `SELECT`절에 사용하는 경우. 

### 5-1. `WHERE` 절에 사용
 - 중요: `WHERE` 절에 사용시 Subquery는 오른쪽에 있어야함. 비교 연산자 (<,>,=) 통한 Subquery와 비교하기 위해서는 1가지 row만 나와야함

>Q1. salaries 테이블에서 from_date가 2000-12-31 이전인 사람들의 급여 중 하나의 급여 보다 더 적은 급여를 받은 직원의 급여 정보를 모두 출력해보세요. ((In, Any, All)사용 )

해답: 

```SQL
SELECT * FROM salaries 
WHERE salary 
< ANY (SELECT * 
FROM salaries WHERE from_date <'2000-12-31'); 
-- where절에서 salary로 비교하므로 서브쿼리의 결과에서 salary를 알아서 찾아준다면 에러가 발생하지 않을 수도 있겠지만 그런 식으로 동작하는게 아니라 
그냥 값 대 값으로 비교하는 것이기 때문에 제대로 동작하지 않습니다. 서브쿼리의 결과 값이 1 개의 값으로만 나와야지만 WHERE 절의 조건과 비교할 수 있음.
```
>Q2. students 테이블에는 학생들의 정보가, middle_test테이블에는 중간고사 점수가 기록되어 있습니다. 1학년 5반 4번인 경민이가 자신의 중간고사 수학점수보다 더 높은 점수를 기록한 사람이 1, 2, 3학년중에 있는지 보고싶다고 하네요. 서브쿼리를 이용하여 경민이보다 높은 수학 점수를 기록한 학생이 1, 2, 3학년중에 있는지 조회해볼까요?

--내 풀이 (오답)
```SQL
SELECT *
FROM students INNER JOIN middle_test
ON students.student_id=middle_test.student_id
WHERE math 
>= (SELECT math FROM middle_test WHERE math > 96); -- 비교 연산자 (>=)사용을 위해서는 row 1개 만 나와야함. 다중행이 나올 경우 In, Any, All 형식 필요 
```
--해답: 
```SQL
SELECT * 
FROM students INNER JOIN middle_test 
ON students.student_id=middle_test.student_id
WHERE math 
> (SELECT math FROM middle_test WHERE student_id=10504); -- 이렇게 작성하면 row 1개 값만 나와서 비교 연산자를 사용할 수 있음 
```

### 5-2. SELECT 절에 사용 (스칼라 Subquery) 
>Q. students 테이블 , test 테이블 에서 학생 id, 이름, 그리고 3가지 항목의 평균 점수를 출력해보세요. 

--내 풀이: 
```SQL
SELECT students.id, students.name, 
(SELECT (kor+eng+math)/3 FROM test 
WHERE test.id = students.id)AS test_average 
FROM students 
LEFT JOIN test  -- 스칼라 Subquery 형식으로 작성하면, 이미 JOIN한 것과 같은 형태로 이후에 JOIN 필요 없음 
ON students.id = test.id; 
```
-- 해답: 
```SQL
SELECT *, 
(SELECT (kor+eng+math)/3 FROM test as t WHERE s.id = t.id) AS test_average -- avg 는 한 컬럼에 대해서 전체 데이터의 평균을 나타내는 방법입니다. 따라서 국어, 영어, 수학 점수의 평균은 avg로 구할 수 없습니다.
FROM students AS s; 
```

# 2. 심화 과정 :raised_eyebrow:
## 01. 집합 연산자 & 계층형질의 

### 01-1. Standard SQL (관계형 대수)
관계형 데이터베이스에서 원하는 정보를 유도하기 위한 기본 연산 집합 (비관계형 데이터베이스 (NoSQL) - Mongo DB)

*일반 집합 연산*
-합집합 (`Union`), 교집합 (`Intersect`), 차집합 (`Except`), 카티션 곱 (`CrossJoin`)

*순수 관계 연산* 
-셀렉션 (`WHERE` 절: 특정 행 조회), 프로젝션 (`SELECT` 절: 특정 컬럼 조회), 조인 , 디비전 (사용 X. 두개의 테이블에서 두번째 테이블과 연결된 데이터만 출력) 

### 01-2. 집합 연산자 
>두개 이상의 테이블에서 조인을 사용하지 않고 연관된 데이터를 조회하는 방법 중에 하나. 테이블에서 'SELECT'한 컬럼 수와 각 컬럼의 데이터 타입이 테이블간 상호 호환 가능해야한다. 
#### 일반 집합 연산 
*`UNION`* 
>두개의 테이블을 하나로 만드는 연산. `UNION`에 사용할 컬럼 수와 데이터 형식이 일치해야하며 합친 후에 *중복된 데이터는 제거*. 이를 위해 `UNION`은 테이블을 합칠때 *정렬 과정*을 발생시킴.(하지만 최종 결과에 대해 올바른 정렬을 위해서는 `ORDER BY` 구문을 사용해야함). 관계형 대수의 일반 집합 연산에서 합집합의 역할 

```SQL
SELECT * FROM ALPHA 
UNION 
SELECT * FROM BETA;'
```

*`UNIONALL`* 
>`UNION`과 거의 같은 기능을 수행. 그러나 *중복된 데이터의 제거 및 정렬을 하지 않음*

```SQL
SELECT * FROM ALPHA 
UNIONALL 
SELECT * FROM BETA;
```

*`INTERSECT`* 
>두개의 테이블에 대해 겹치는 부분을 추출하는 연산 (교집합).추출 후에는 *중복된 결과를 제거*. 관계형 대수의 일반 집합연산에서 교집합의 역할. **OMaria DB에서는 지원, MYSQL에서는 지원되지 않기 떄문에 JOIN 사용해야함.** 

```SQL
SELECT A,B FROM ALPHA 
INTERSECT
SELECT A,B FROM BETA;
```

*`EXCEPT(MINUS)`*
>두 개의 테이블에서 겹치는 부분을 *앞의 테이블*에서 제외하여 추출하는 연산. 추출 후에는 중복된 결과를 제거. 관계형 대수의 일반 집합 연산에서 *차집합*의 역할. **Orcale Database에서는 지원, MariaDB 에서는 10.3부터 EXCEPT키워드로 지원. MYSQL에서는 지원되지 않기 때문에 JOIN 사용필요.** 

```SQL
SELECT A,B FROM ALPHA 
EXCEPT 
SELECT A,B FROM BETA;
```
#### 순수 관계 연산 
[그림]

*'WHERE'/셀릭션*
>특정 행만 호출 

*'SELECT'/프로젝션*
>특정 컬럼만 호출 

*'다양한 JOIN'/ 조인*
>두개의 테이블에서 겹치는 부분을 하나의 새로운 테이블로 만드는 연산. 

*디비전*
>두개의 테이블이 있을때, 두번째 테이블에 연관된 데이터만 추출하는 연산. 

### 01-3. 계층형 질의 -Oracle 
> 테이블의 *계층형 데이터*가 존재하는 경우, 데이터를 조회하기 위해 사용하는 것. 대표적인 데이터 베이스로는 Oracle, SQL Server. 

*계층형 데이터*
동일 테이블에 계층적으로 상위와 하위 데이터가 포함되어 있는 데이터. 
[그림]

*계층형 질의 예시 (Oracle)*

[그림]
```SQL
SELECT LEVEL, 자식 컬럼, 부모 컬럼, 원하는 컬럼 
FROM 테이블명 
START WITH 부모 컬럼 IS NULL -- 부모 컬럼이 NULL인 행이 Root (가장 상위)가 됨
CONNECT BY PRIOR 자식 컬럼= 부모 컬럼; --상위 데이터와 하위 데이터 연결 방식
```


*계층형 질의 응용 (Oracle)*
[그림]
```SQL
SELECT LEVEL, LPAD('',4*(Level-1))||사원번호, 관리자 --LPAD('',n)은 왼쪽에 n자리의 공백 추가를 의미.ROOT는 LEVEL 값이 1이기 때문에 4*(LEVEL-1)=0이 된다.
FROM 직원
START WITH 관리자 IS NULL CONNECT BY PRIOR 사원번호 = 관리자;
```

[그림]
*Connect by Keyword (Oracle)*
|키워드|설명|
|---|---|
|`LEVEL`| 검색 항목의 깊이를 의미하며,계층구조에서 루트(최상위)의 레벨1|
|`CONNECT_BY_ROOT`| 현재 전개할 데이터의 루트(최상위) 데이터 값 표기|
|`CONNECT_BY_ISLEAF`| 현재 전개할 데이터의 리프(최하위) 데이터 인지에 대한 값 표시(0 or 1)|
|`SYS_CONNECT_BY_PATH(A,B)`| 루트 데이터부터 현재까지 전개한 경로 표시 (A:컬럼명,B:구분자)

### 01-4. 계층형 질의 -SQL Server/Maria DB

*계층형 질의* 
- SQL Server 2000 이전: 저장 프로시저를 재귀 호출 / WHILE 루프문에서 임시테이블 이용 
- SQL Server 2005 이후: CTE (Common Table Expression)을 이용하여 재귀 호출 
- MariaDB 10.2 이후: CTE (Common Table Expression)을 이용하여 재귀 호출 
[그림]

## 02. JOIN 심화 
### 02-1. JOIN 
> 두 개 이상의 테이블들을 *연결* 또는 *결합*하여 데이터를 출력하는 것. 연산자에 따라 EQUI JOIN, Non EQUI JOIN 으로 분류함. 

*`EQUI JOIN` (등가 교집합)*
> 두 개의 테이블 간에 *서로 정확하게 일치하는 경우*를 활용하는 조인. 간단히 말해, 등가 연산자를 사용한 조인을 의미 `=` 대부분 기본키-외래키 관계를 기반으로 발생하나, 모든 조인이 그런 것은 아니다. 

*`Non EQUI JOIN` (비등가 교집합)*
> 두 개의 테이블 간에 *서로 정확하게 일치하지 않는 경우*를 확용하는 조인. 간단히 말해, *등가 연산자*이외의 연산자들을 사용하는 조인을 의미. `>` `>=` `<=` `<` `BETWEEN`

### 02-2. FROM 절 JOIN 형태 
*`INNER JOIN`*
> *내부 JOIN*이라고 하며 JOIN 조건에서 동일한 값이 있는 행만 반환. INNER JOIN은 *JOIN의 기본값*으로 'INNER'생략 가능 
```SQL
SELECT * FROM 테이블1 [INNER]JOIN 테이블2 -- INNER JOIN구로 테이블 정의 

ON 테이블1.컬럼명 = 테이블2.컬럼명; -- ON구를 사용해 조인 조건 지정 
```

*`USING 조건절`*
> *같은 이름을 가진 컬럼들* 중 원하는 칼럼에 대해서만 선택적으로 등가조인 가능. SQL Server에서는 지원 X 

```SQL
SELECT * FROM 테이블1 JOIN 테이블2 
USING(기준칼럼); -- USING 조건절 사용시에는 칼럼이나 테이블에 별칭을 붙일 수 없음 
```

*`NATURAL JOIN`*
> 두 테이블 간의 *동일한 이름*을 갖는 모든 칼럼들에 대해 *등가 조인*을 수행. 
```SQL
SELECT * FROM 테이블1 NATURAL JOIN 테이블2; -- 추가로 ON 조건절이나 USING 조건절, WHERE절에서 JOIN 조건 정의 불가  
```
<img width="8000" alt="2022-10-13_20-42-27" src="https://user-images.githubusercontent.com/114547060/195587269-48b97365-2c05-4706-b1a3-5f78fe797eb6.png">

*`CROSS JOIN`* 
> JOIN 조건이 없는 경우, *모든 데이터의 조합*을 조회
```SQL
SELECT * FROM PERSON
(CROSS) JOIN PUBLIC_TRANSPORT; -- CROSS 부분 생략 가능   
```
<img width="962" alt="2022-10-18_15-51-48" src="https://user-images.githubusercontent.com/114547060/196356764-affc343f-8ca0-4f0a-9816-6a915a869670.png">

*`OUTER JOIN`* 
<img width="158" alt="2022-10-18_16-06-16" src="https://user-images.githubusercontent.com/114547060/196359945-4cc22cd6-4e66-449d-b414-721a05d12519.png">
> 두개의 테이블 간의 교집합을 조회하고 *한쪽 테이블에만 있는 데이터도 포함* 시켜서 조회.빈 곳은 NULL 값으로 출력. WHERE 조건절에서 한쪽에만 있는 데이터를 *포함시킬 테이블 쪽으로 (+)* 를 위치 
```SQL
SELECT * FROM USER, CLASS  
WHERE USER.CLASS_ID (+) = CLASS.CLASS_ID; 
```
<img width="952" alt="2022-10-18_16-02-33" src="https://user-images.githubusercontent.com/114547060/196359160-f4329e92-9ddc-4331-9b1f-95adcdc1ff83.png">

*표준 OUTER JOIN (`LEFT JOIN`)*
<img width="158" alt="2022-10-18_16-06-16" src="https://user-images.githubusercontent.com/114547060/196360047-4d1d04f0-a588-4989-b6f6-77bdbda32b7b.png">
```SQL
SELECT * FROM USER LEFT[OUTER]JOIN CLASS  
WHERE USER.CLASS_ID = CLASS.CLASS_ID; 
```
<img width="952" alt="2022-10-18_16-02-33" src="https://user-images.githubusercontent.com/114547060/196360255-75fea4d7-1065-4a5b-b6bf-e2bb8e95a307.png">

*표준 OUTER JOIN (`RIGHT JOIN`)* 
<img width="168" alt="2022-10-18_16-09-39" src="https://user-images.githubusercontent.com/114547060/196360727-0fcb4f4d-1021-4c55-9d52-7baf14121adb.png">
```SQL
SELECT * FROM USER RIGHT[OUTER]JOIN CLASS -- CLASS 테이블은 모두 출력 되어야함  
WHERE USER.CLASS_ID = CLASS.CLASS_ID; 
```
<img width="938" alt="2022-10-18_16-10-28" src="https://user-images.githubusercontent.com/114547060/196360961-7d2dc299-ca41-4a15-9983-336948bfc1f2.png">
*표준 OUTER JOIN (`FULL OUTER JOIN`)* **
<img width="161" alt="2022-10-18_16-14-10" src="https://user-images.githubusercontent.com/114547060/196361763-ea41773c-a2c1-40dd-b9a7-3e72a4f8bf7a.png">
```SQL
SELECT * FROM CLASS FULL OUTER JOIN USER 
WHERE USER.CLASS_ID (+) = CLASS.CLASS_ID; 
```
<img width="940" alt="2022-10-18_16-16-08" src="https://user-images.githubusercontent.com/114547060/196362143-0ba420b7-d28e-4fd7-aec2-9d94d60b0726.png">

** `FULL OUTER JOIN`은 ORACLE DB에서만 사용, MYSQL/MARIA DB에서는 `UNION` 사용. 
예시 
```SQL
SELECT * FROM CLASS LEFT OUTER JOIN USER 
ON USER.CLASS_ID = CLASS.CLASS_ID
UNION 
SELECT * FROM CLASS RIGHT OUTER JOIN USER 
ON USER.CLASS_ID = CLASS.CLASS_ID; 
```
*INNTER JOIN*

> JOIN 활용한 쿼리에서도 WHERE 문을 이용하여 조건을 걸수 있음 
기본 형태: 
```SQL
SELECT * FROM 테이블1 [INNER] JOIN 테이블2
ON 테이블1.[컬럼명] = 테이블2.[컬럼명]
WHERE [조건]; 
```
```SQL
SELECT * FROM USER a [INNER] JOIN CLASS b
ON a.CLASS_ID = b.ID
WHERE name = '모자장수'; 
```
<img width="456" alt="2022-10-18_17-22-38" src="https://user-images.githubusercontent.com/114547060/196376848-53b80932-843c-45f9-afdf-8c9e705b4ae8.png">


### 02-3. SELF JOIN 

> 동일 테이블 사이의 조인. 동일 테이블 사이 조인을 실행하면, 테이블 및 컬럼 이름이 모두 동일하므로 별칭 사용 필수. 

```SQL
SELECT ALPHA.사원번호, ALPHA.관리자, BETA.관리자 차상위 
FROM 직원 ALPHA, 직원 BETA 
WHERE ALPHA.관리자 = BETA.관리자; 
```
<img width="784" alt="2022-10-18_17-42-21" src="https://user-images.githubusercontent.com/114547060/196381536-6e322056-d7d6-49e0-a2bd-b71fc8dd7764.png">

:exclamation:문제 예시 
> EMPLOYEE 테이블에는 사원 ID, 사원 이름, 관리자 ID 정보가 담겨있습니다. 각 사원들의 관리자가 누구인지 확인하고자 합니다. 하지만 EMPLOYEE테이블에는 관리자 ID 만 함께 존재하기 때문에, 셀프 조인 개념을 이용하여 각 사원 정보와 관리자 이름을 함께 조회해봅시다. EMPLOYEE 테이블에 대해서 SELF JOIN을 이용하여, 사원 ID(employee_id), 사원 이름(employee_name), 관리자 이름(employee_name as manager_name) 을 출력하는 쿼리를 작성해봅시다.

*<EMPLOYEE TABLE 구조, DATA >* 

```SQL
DESC EMPLOYEE; 
+---------------+-------------+------+-----+---------+-------+
| Field         | Type        | Null | Key | Default | Extra |
+---------------+-------------+------+-----+---------+-------+
| employee_id   | int(11)     | NO   | PRI | NULL    |       |
| employee_name | varchar(30) | NO   |     | NULL    |       |
| manager_id    | int(11)     | YES  |     | NULL    |       |
+---------------+-------------+------+-----+---------+-------+
SELECT * FROM EMPLOYEE; 
+-------------+---------------+------------+
| employee_id | employee_name | manager_id |
+-------------+---------------+------------+
|       10001 | Kim           |       NULL |
|       10002 | Lee           |      10001 |
|       10003 | Park          |      10001 |
|       10004 | Moon          |      10002 |
|       10005 | Choi          |      10002 |
+-------------+---------------+------------+

```
*답* 
```SQL
SELECT a.employee_id, a.employee_name, b.employee_name AS manager_name 
 FROM EMPLOYEE AS a, EMPLOYEE AS b 
 WHERE a.manager_id = b.employee_id
 ORDER BY employee_id ASC; 
 
+-------------+---------------+--------------+
| employee_id | employee_name | manager_name |
+-------------+---------------+--------------+
|       10002 | Lee           | Kim          |
|       10003 | Park          | Kim          |
|       10004 | Moon          | Lee          |
|       10005 | Choi          | Lee          |
+-------------+---------------+--------------+
```
## 3. 서브쿼리 심화 
### 03-1. 동작하는 방식에 따른 서브쿼리 분류 
> 서브쿼리에 *메인쿼리의 컬럼*이 포함되는지에 따라 분류. 

#### 연관 서브쿼리 (Correlated Subquery)
> 메인쿼리의 컬럼이 서브쿼리에 *포함되며*, 메인쿼리의 컬럼은 서브쿼리의 특정조건으로 사용된다.
<img width="960" alt="2022-10-20_11-02-29" src="https://user-images.githubusercontent.com/114547060/196840564-bc841d15-a93d-4ae9-b58f-c55a85319b56.png">

#### 비연관 서브쿼리 (Un-Correlated Subquery)
> 메인쿼리의 컬럼이 서브쿼리에 *포함되지 않으며*, 주로 메인쿼리에 특정한 값을 제공할때 사용된다. 
<img width="974" alt="2022-10-20_11-19-02" src="https://user-images.githubusercontent.com/114547060/196840715-245d1d75-7c0a-497e-9843-8335d65b0ad5.png">

### 03-2. 반환되는 방식에 따른 서브쿼리 분류 
#### 단일 행 서브쿼리 (Single Row Subquery)
> 서브쿼리의 결과가 *한 개의 행* 을 반환하며, 단일 행 비교 연산자 (=,<,>,<=,>=) 와 같이 사용된다. 
<img width="951" alt="2022-10-20_11-36-16" src="https://user-images.githubusercontent.com/114547060/196842984-b9c4ee1e-daf0-4570-9694-aeaac12cd548.png">
#### 다중 행 서브쿼리 (Multi Row Subquery) 
> 서브쿼리의 결과가 *두 개 이상의 행을* 반환 할 수 있으며, 다중 행 비교 연산자 (IN, ALL, ANY, EXISTS)와 같이 사용된다. 
<img width="952" alt="2022-10-20_16-59-50" src="https://user-images.githubusercontent.com/114547060/196891008-e439aba2-b7c0-4817-ab17-c73bfff1bc74.png">
*`IN`*
<img width="971" alt="2022-10-20_17-01-27" src="https://user-images.githubusercontent.com/114547060/196891355-57fbd7ff-0db2-4325-85e7-6b1f68db1db2.png">

*`EXISTS`*
<img width="955" alt="2022-10-20_17-03-56" src="https://user-images.githubusercontent.com/114547060/196891927-52f624cc-2d8b-480a-96e9-3f657b7ca914.png">

*`ALL`*
<img width="977" alt="2022-10-20_17-08-03" src="https://user-images.githubusercontent.com/114547060/196892770-caf506ae-0647-46b2-89bb-60be4b333246.png">

*`ANY`*
<img width="971" alt="2022-10-20_17-10-40" src="https://user-images.githubusercontent.com/114547060/196893360-64c63c91-c119-4828-af98-c10962a39674.png">

#### 다중 컬럼 서브쿼리 (Multi Column Subquery) 
> 서브쿼리의 결과가 *여러개의 컬럼*을 반환하며, 메인쿼리의 조건과 동시에 비교된다. 
<img width="956" alt="2022-10-20_17-34-53" src="https://user-images.githubusercontent.com/114547060/196898875-9f00c79d-d131-4cc4-809d-0c21a16094cc.png">

### 03-3. 스칼라 서브쿼리 
> 하나의 속성을 가지면서, 하나의 행만을 반환하는 쿼리. `SELECT`, `WHERE`, `HAVING` 절에서 사용할 수 있음
<img width="965" alt="2022-10-20_17-50-49" src="https://user-images.githubusercontent.com/114547060/196902614-236925ad-c46b-4844-93fb-da4a6b21533d.png">
<img width="985" alt="2022-10-20_17-55-18" src="https://user-images.githubusercontent.com/114547060/196903712-1009a841-49d2-4591-8aae-dcaa7380f029.png">
_MY SQL, MARIA DB에서는 임시 테이블 (예시: FROM DUAL)생략 가능_ 

### 03-4. 뷰 
> 뷰는 다른 테이블에서 파생된 테이블이다. 물리적으로 데이터가 저장되는 것이 아니라, *논리적으로만 존재*하며 뷰를 사용한 질의 시에는 DBMS에서 뷰 정의에 따라 질의를 재작성하여 수행한다. 

*뷰의 장점*
<img width="945" alt="2022-10-20_18-08-46" src="https://user-images.githubusercontent.com/114547060/196906828-8b66af29-47ec-4759-890d-c37add8456cd.png">

*뷰의 특징* 
<img width="874" alt="2022-10-20_18-14-00" src="https://user-images.githubusercontent.com/114547060/196907966-9f6e5c2f-18a6-4e25-a0b4-b2fa7585c532.png">
*뷰 정의 쿼리* 
<img width="963" alt="2022-10-20_18-19-52" src="https://user-images.githubusercontent.com/114547060/196909393-497a3c5b-dda7-42c2-944f-7fa0a68bdd33.png">
<img width="1005" alt="2022-10-20_18-20-43" src="https://user-images.githubusercontent.com/114547060/196909591-af60dd17-f52a-45ed-bf50-6c532efcde28.png">

## 4. 그룹 함수 & 윈도우 함수 
### 04-1. 데이터 분석 개요 
데이터 분석을 위한 함수 
윈도우 함수 (Window Function) -열과 열 뿐만 아니라, 행과 행도 포함 
집계 함수 (Aggregate Function) - 윈도우 함수에 포함됨. 
그룹 함수 (Window Function) 

### 04-2. 윈도우 함수
> 순위, 합계 등 *행과 행 사이의* 관계를 정의하는 함수. *`OVER` 구문을 필수*로 한다. 

```SQL
SELECT WINDOW_FUNCTION(ARGUMENTS)
OVER( [PARTITION BY 칼럼][ORDER BY 절][WINDOWING 절])FROM 테이블 명; 
```
|구조|설명|
|-|-|
|ARGUMENTS| 윈도우 함수에 따라서 필요한 인수|
|PARTITION BY|전체 집합에 대해 소그룹으로 나누는 기준|
|ORDER BY| 소그룹에 대한 정렬 기준|
|WINDOWING|행에 대한 범위 기준|

*WINDOWING 절에서 사용하는 명령어* 
|구조|설명|
|-|-|
|ROWS|물리적 단위로 행의 집합을 지정|
|UNBOUNDED PRECEDING| 윈도우 시작 위치가 첫번째 행|
|UNBOUNDED FOLLOWING| 윈도우 마지막 위치가 마지막 행|
|CURRENT ROW|윈도우 시작 위치가 현재 행|

#### 04-2-A. 순위 함수 

```SQL
RANK()OVER( [PARTITION BY 칼럼][ORDER BY 절][WINDOWING 절])
```
|함수|설명|
|-|-|
|RANK|동일한 값에는 동일한 순서를 부여|
|DENSE_RANK|RANK와 같이 같은 값에는 같은 순위를 부여하거나 한 건으로 취급|
|ROW_NUMBER|동일한 값이라도 고유한 순위를 부여|

<img width="991" alt="2022-10-20_20-22-13" src="https://user-images.githubusercontent.com/114547060/196935516-fad87ed4-69bb-4531-ac94-6fd0e564d619.png">

#### 04-2-B. 일반 집계 함수 
> 일반 집계 함수 (SUM, AVG, MAX, MIN ...)를 GROUP BY 구문 없이 사용할 수 있다.
<img width="1003" alt="2022-10-20_20-39-15" src="https://user-images.githubusercontent.com/114547060/196938573-b1710255-1dd4-42a7-8835-2f1f182ae03a.png">

#### 04-2-C. :fire: 그룹 내 행 순서 함수 

|함수|설명|
|-|-|
|FIRST_VALUE|가장 먼저 나온 값을 구한다| 
|LAST_VALUE|가장 나중에 나온 값을 구한다|
|LAG|이전 x 번째 행을 가져온다|
|LEAD|이후 x 번째 행을 가져온다| 

<img width="992" alt="2022-10-20_21-01-37" src="https://user-images.githubusercontent.com/114547060/196942973-45301022-14db-4b3d-b08c-99321de957ae.png">

<img width="1006" alt="2022-10-20_21-13-39" src="https://user-images.githubusercontent.com/114547060/196945392-741aa4d8-2443-41f7-b297-393430b0aff9.png">

#### 04-2-D. :fire: 그룹 내 비율 함수 

|함수|설명|
|-|-|
|`RATIO_TO_REPORT`|파티션 내 전체 SUM에 대한 비율을 구한다| 
|`PERCENT_RANK`|파티션 내 순위를 백분율로 구한다|
|`CUME_DIST`|파티션 내 현재 행보다 작거나 같은 건들의 수 누적 백분율을 구한다|
|`NTILE`|파티션내 행들을 N등분한 결과를 구한다| 

<img width="988" alt="2022-10-20_21-46-34" src="https://user-images.githubusercontent.com/114547060/196952583-414c6d93-094d-4be1-a4c7-3594d3f11b1f.png">

<img width="985" alt="2022-10-20_21-48-41" src="https://user-images.githubusercontent.com/114547060/196953081-313b21ce-fdf2-4a67-96fc-d3950295c092.png">
`NTILE`
```SQL
NTILE(N)OVER(ORDER BY 컬럼) - N 개의 그룹으로 분류 
```
![2022-10-20_22-16-36](https://user-images.githubusercontent.com/114547060/196959500-d0dfd448-4518-4797-a81a-e8739e4cc0ab.png)

### 04-3. 그룹 함수 
> 데이터를 통계 내기 위해서는, 전체 데이터에 대한 통계는 물론이고 데이터 일부에 대한 소계, 중계 또한 필요하다. 
<img width="860" alt="2022-10-20_23-03-33" src="https://user-images.githubusercontent.com/114547060/196970553-80d24263-766e-46e8-b3ac-7057490d50d7.png">
<img width="854" alt="2022-10-20_23-01-23" src="https://user-images.githubusercontent.com/114547060/196970002-458cf226-c633-4b90-94ea-c0138a379a76.png">

`ROLLUP`
> 그룹화하는 컬럼에 대한 부분적인 통계를 제공해준다 
<img width="946" alt="2022-10-20_23-19-15" src="https://user-images.githubusercontent.com/114547060/196974457-848b3675-ba03-41c5-870a-4e9db537eaa5.png">

`CUBE`
> `ROLLUP`함수에서 제공하는 결과를 포함해서,CUBE 함수에서는 그룹화 하는 컬럼에 대해 결한 가능한 모든 경우의 수에 대해 다차원 집계를 생성한다.(`ROLLUP`들의 합이라고 생각하면 됨) 
<img width="975" alt="2022-10-20_23-28-11" src="https://user-images.githubusercontent.com/114547060/196976782-2d512027-c66d-42ca-a77e-464dcb841dee.png">



