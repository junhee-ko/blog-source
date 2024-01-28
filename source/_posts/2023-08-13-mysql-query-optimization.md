---
layout: post
title: "쿼리 작성 및 최적화" 
date: 2023-08-13
categories: MySQL
---

## SELECT

select 쿼리의 각 부분에 사용될 수 있는 기능을 성능 위주로 보자.

### SELECT 절의 처리 순서

1. WHERE 적용 및 조인 실행
2. GROUP BY
3. DISTINCT
4. HAVING 조건 적용
5. ORDER BY
6. LIMIT

### WHERE 절, GROUP BY 절, ORDER BY 절의 인덱스 사용

where 절의 조건뿐만 아니라 group by, order by 절도 인덱스를 이용해 빠르게 처리할 수 있다.

#### 인덱스를 사용하기 위한 기본 규칙

where 절이나 order by 또는 group by 가 인덱스를 사용하려면 기본적으로 인덱스된 칼럼의 값 자체를 변환하지 않고 그대로 사용해야한다.
만약 인덱스는 salary 칼럼으로 만들어져 있는데, 다음처럼 salary 칼럼을 가공한 후 다른 상수 값과 비교하면 이 쿼리는 인덱스를 이용하지 못한다.

```mysql
select *
from salaries
where salary * 10 > 150000;
```

또한, where 절에 사용되는 비교 조건에서 연산자 양쪽의 두 비교 대상 값은 데이터 타입이 일치해야한다.

```mysql
select *
from tb_testw
where age = 2 # age column: VARCHAR(10);
```

위 쿼리는 age 칼럼의 데이터 타입과 비교되는 값 2 의 타입이 다르기 때문에, 인덱스 풀 스캔으로 동작한다.

#### where 절의 인덱스 사용

where 조건이 인덱스를 사용하는 방법은 크게 작업 범위 결정 조건과 체크 조건의 두 가지 방식으로 구분할 수 있다.
작업 범위 결정 조건은 where 절에서 동등 비교 조건이나 in 으로 구성된 조건에 사용된 칼럼들이 인덱스의 칼럼 구성과 좌측에서부터 비교했을 때 얼마나 일치하는가에 따라 달라진다.

(col_1, col_2, col_3, col_4) 로 구성된 인덱스가 있다고 하자.
where 조건에 col_1 과 col_2 는 동등 비교 조건이고 col_3 은 범위 비교 조건이면 col_4 의 조건은 작업 범위 결정 조건으로 사용되지 못하고 체크 조건으로 사용된다.

where 조건절의 조건이 인덱스에 명시된 칼럼의 순서대로 나열될 필요는 없다.
where 조건절에 나열된 순서가 인데스와 다르더라도, MySQL 서버 옵티마이저는 인덱스를 사용할 수 있는 조건들을 뽑아서 최적화를 수행할 수 있다.
즉, 인덱스를 구성하는 칼럼에 대한 조건이 있는지 없는지가 중요하다.

#### group by 절의 인덱스 사용

group by 절의 각 칼럼은 비교 연산자를 가지지 않으므로 작업 범위 결정 조건이이나 체크 조건과 같이 구분해서 생각할 필요가 없다.
group by 절에 명시된 칼럼의 순서가 인덱스를 구성하는 칼럼의 순서와 같으면, group by 절은 일단 인덱스를 이용할 수 있다.

1. group by 절에 명시된 칼럼이 인덱스의 칼럼 순서와 위치가 같아야한다.
2. 인덱스를 구성하는 칼럼 중에 뒤쪽에 있는 칼럼은 group by 절에 명시되지 않아도 인덱스를 사용할 수 있다.
3. 인덱스의 앞쪽에 있는 칼럼이 group by 절에 명시되지 않으면 인덱스를 사용할 수 없다.
4. where 조건과 달리 group by 절에 명시된 칼럼이 하나라도 인덱스에 없으면 group by 절은 전혀 인덱스를 이용하지 못한다.

(col_1, col_2, col_3, col_4) 로 구성된 인덱스가 있다고 하자.
다음 예제는 group by 절이 인덱스를 사용할 수 있는 패턴이다.

```mysql
GROUP BY col_1
GROUP BY col_1, col_2
GROUP BY col_1, col_2, col_3
GROUP BY col_1, col_2, col_3, col_4
```

where 조건절에 col_1 이나 col_2 가 동등 비교 조건으로 사용되면 group by 절에 col_1 이나 col_2 가 빠져도 인덱스를 이용한 group by 가 가능하다.

```mysql
WHERE col_1 = '상수' GROUP BY col_2, col_3 
WHERE col_1 = '상수' and col_2 = '상수' GROUP BY col_3, col_4 
```

#### ORDER BY 절의 인덱스 사용

group by 의 요건과 비슷하다. 
조건이 하나 더 있다면, 정렬되는 각 칼럼의 오른차순 및 내림차순 옵션이 인덱스와 같거나 정반대인 경우에만 사용할 수 있다.

### WHERE 절의 비교 조건 사용시 주의사항

쿼리가 최적으로 실행되려면, 적합한 인덱스와 함께 where 절에 사용되는 비교 조건의 표현식을 적절히 사용해야한다.

#### null 비교

SQL 표준에서 null 의 정의는 "비교할 수 없는 값" 이다.
그래서, 연산이나 비교에서 한쪽이라도 null 이면 그 결과도 null 이 반환된다.
쿼리에서 null 인지 비교하려면 "is null" 이나 "<=>" 연산자를 사용하면 된다.

null 값이 포함된 레코드도 인덱스로 관리된다.
즉, 인덱스에서는 null 도 하나의 값으로 인정해서 관리한다.
아래 쿼리는 to_date 칼럼이 null 인 레코드를 조회하는 쿼리이지만, to_date 칼럼에 생성된 인덱스를 적절히 이용한다.

```mysql
select *
from titles
where to_date is null;
```

#### 문자열이나 숫자 비교

문자열 칼럼이나 숫자 칼럼을 비교할 때는 반드시 그 타입에 맞는 상숫값을 사용하자.
즉, 비교 대상 칼럼이 문자열 칼럼이면 문자열 리터럴을, 숫자 타입이면 숫자 리터럴을 사용하자.

```mysql
## 칼럼 타입과 비교하는 상숫값이 동일한 타입이므로, 인덱스 적절히 이용
select *
from employees
where emp_no = 10001;

## 칼럼 타입과 비교하는 상숫값이 동일한 타입이므로, 인덱스 적절히 이용
select *
from employees
where first_name = 'Smith';

## 문자열 상숫값을 숫자로 타입 변환해서 비교를 수행하므로, 성능 저하 발생하지 않음
select *
from employees
where emp_no = '10001';

## first_name 칼럼의 문자열열을 숫자로 변환해서 비교를 수행하므로, 인덱스를 사용하지 못함
select *
from employees
where first_name = 10001;
```

#### 날짜 비교

##### DATE 또는 DATETIME 과 문자열 비교

DATE 또는 DATETIME 타입의 값과 문자열을 비교할 때는, 문자열 값을 자동으로 DATETIME 타입의 값으로 변환해서 비교를 수행한다.
아래 두 쿼리는 모두 동일하게 처리되며, 인덱스도 효율적으로 처리된다.

```mysql
select count(*)
from employees
where hire_date > str_to_date('2023-08-12', '%y-%m-%d');

select count(*)
from employees
where hire_date > '2023-08-12';
```

위 쿼리를 다음과 같이 작성하면, hire_date 타입의 값을 강제적으로 문자열로 변경하기 때문에 인덱스를 이용하지 못한다.

```mysql
select count(*)
from employees
where date_format(hire_date, '%y-%m-%d') > '2023-08-12';  
```

#### DATE 와 DATETIME 의 비교

DATETIME 의 값에서 시간 부분만 떼어 버리고 비교하려면, 다음 예제와 같이 쿼리를 작성하면 된다.
date() 함수는 datetime 타입의 값에서 시간 부분은 버리고 날짜 부분만 반환한다.

```mysql
select count(*)
from employees
where hire_date > DATE(NOW());
```

DATETIME 타입의 값을 DATE 타입으로 만들지 않고 그냥 비교하면 MySQL 서버가 DATE 타입의 값을 DATETIME 으로 변환해서 같은 타입을 만든 다음 비교를 수행한다.

#### Short-Circuit Evaluation

선행 표현식의 결과에 따라 후행 표현식을 평가할지 말지 결정하는 최적화를 Short-Circuit Evaluation 이라고 한다.
아래 예시에서는, in_transaction 의 값이 true 이어야 has_modified() 함수가 호출된다.

```java
boolean in_transaction;

if ( in_transaction && has_modified() ) {
    commit();    
}
```

MySQL 서버에서는 Short-Circuit Evaluation 이 어떻게 쿼리의 성능에 영향을 미치는지 보자.

```mysql
select count(*)
from salaries; ## 2844047 건

select count(*)
from salaries
where convert_tz(from_date, '+00:00', '+09:00') > '1991-01-01'; ## 2442932 건 

select count(*)
from salaries
where to_date < '1985-01-01'; ## 0 건

## 0.73 sec
select count(*)
from salaries
where convert_tz(from_date, '+00:00', '+09:00') > '1991-01-01'
  and to_date < '1985-01-01';

## 0.52 sec
select count(*)
from salaries
where to_date < '1985-01-01' 
  and convert_tz(from_date, '+00:00', '+09:00') > '1991-01-01';
```

MySQL 서버는 where 절에 나열된 조건을 순서대로 Short-Circuit Evaluation 방식으로 평가한다.
그런데, where 절의 조건 중에서 인덱스를 사용할 수 있는 조건이 있으면 Short-Circuit Evaluation 와 무관하게 그 조건을 가장 최우선으로 사용한다.
그래서, where 절에 나열된 조건의 순서가 인덱스의 사용 여부를 결정하지 않는다.
아래의 경우, where 절에 나열된 순서와 무관하게 인덱스를 사용할 수 있는 조건을 먼저 평가한다.

```mysql
select *
from employees
where last_name = 'ko'       ## 인데스 사용 불가능 칼럼
  and first_name = 'junhee'; ## 인데스 사용 가능 칼럼
```

### LIMIT n

limit 이 사용된 쿼리를 보자.

```mysql
select *
from employees
where emp_no between 10001 and 10010
order by first_name
limit 0, 5;
```

1. employees 테이블에서 where 절의 검색 조건에 일치하는 레코드를 전부 읽는다.
2. first_name 칼럼 값에 따라 정렬한다.
3. 상위 5건만 사용자에게 반환한다.

limit 의 중요한 특성은, limit 에서 필요한 레코드 건수만 준비되면 즉시 쿼리를 종료한다.
즉, 위 쿼리는 모든 레코드의 정렬이 완료되지 않았어도 상위 5건 까지만 정렬되면 작업을 멈춘다.

group by 절이나 distinct 등과 같이 limit 이 사용되었을 때를 보자.

```mysql
## 풀 테이블 스캔을 수행하면서 10개의 레코드를 읽으면, 읽기 작업을 멈춤
select *
from employees
limit 0, 10;

## grouping 모두 완료되어야 limit 수행
select *
from employees
group by first_name 
limit 0, 10;

## 레코드를 읽음과 동시에 중복 제거 작업을 진행하다가 유니크한 레코드 10건을 읽으면 쿼리를 완료
select distinct first_name
from employees
limit 0, 10;
```

애플리케이션에서 테이블의 데이터를 select 할 때 paging 해서 가져가게 되는 경우를 보자.

```mysql
## 처음부터 읽으면서 2000010 건의 레코드를 읽은 후에 2000000 건은 버리고 마지막 10 건만 반환 
select *
from salaries
order by salary
limit 2000000, 10;
```

페이징이 처음 몇 개의 페이지 조회로 끝나지 않을 가능성이 있으면, where 조건 절로 읽어야할 위치를 찾고 그 위치에서 10 개만 읽는 쿼리를 사용하자.

```mysql
## 첫 번쨰 쿼리
select *
from salaries
order by salary
limit 0, 10;

## 두 번째 쿼리
## "not (salary = 38864 and emp_no <= 274021)" 조건은 이전 페이지에서 조회한 것을 제외하기 위한 조건
select *
from salaries
where salrary >= 38864 and not (salary = 38864 and emp_no <= 274021)
order by salary
limit 0, 10;
```

### COUNT()

count query 에서 가장 많이 하는 실수는, order by 구문이나 left join 같은 레코드 건수를 가져오는 것과는 무관한 작업을 포함하는 것이다.
count query 에서 order by 절은 어떤 경우에도 필요없다. 그리고, left join 또한 레코드 건수의 변화가 없거나 아우터 테이블에서 별도 체크를 하지 않아도 되는 경우에는 모두 제거하는 것이 좋다.

count() 함수에 칼럼명이나 표현식이 인자로 사용되면, 그 칼럼이나 표현식의 결과가 null 이 아닌 레코드 건수만 반환한다.
"count(column1)" 은 column1 이 null 이 아닌 레코드 건수를 가져온다. 

### JOIN

JOIN 이 어떻게 인덱스를 사용하는지에 대해 보자.

#### JOIN 의 순서와 인덱스

조인 작업에서 드라이빙 테이블을 읽을 때는 인덱스 탐색 작업을 단 한번 수행하고, 그 이후부터는 스캔만 실행하면 된다.
하지만 드리븐 테이블에서는 인덱스 탐색 작업과 스캔 작업을 드라이빙 테이블에서 읽은 레코드 건수만큼 반복한다.
드리븐 테이블을 읽는 것이 훨씬 더 큰 부하를 차지하는 것이다.

```mysql
select *
from employees e, dept_emp de
where e.emp_no = de.emp_no;
```

e.emp_no, de.emp_no 모두 인덱스가 있으면, 어느 테이블을 드라이빙으로 선택하든 드리븐 테이블의 검색 작업을 빠르게 처리할 수 있다.
옵티마이저가 통계 정보를 이용해서 적절히 드라이빙 테이블을 선택한다.

e.emp_no 에만 인덱스가 있을 떄, employees 테이블이 드라이빙 테이블로 선택되면 employees 테이블의 레코드 건수만큼 dept_emp 테이블을 풀 스캔해야 "e.emp_no = de.emp_no" 조건에 일치하는 레코드를 찾을 수 있다.
그래서 항상 dept_emp 테이블을 드라이빙 테이블로 선택한다. 

두 칼럼 모두 인덱스가 없으면, 어느 테이블을 드라이빙으로 선택해도 드리븐 테이블의 풀 스캔이 발생한다.
그래도 레코드 건수가 적은 테이블을 드라이빙 테이블로 선택하는 것이 효율적이다.

#### JOIN 칼럼의 데이터 타입

조인 칼럼 간의 비교에서 각 칼럼의 데이터 타입이 일치하지 않으면, 인덱스를 효율적으로 사용할 수 없다.

#### OUTER JOIN 의 성능과 주의사항

```mysql
select *
from employees e
  left join dept_emp de on de.emp_no = e.emp_no
  left join departments d on d.dept_no = de.dept_no and d.dept_name = 'Dev';
```

이 쿼리의 실행 게획은, 먼저 employees 테이블을 풀 스캔하고 나머지 두 테이블을 드리븐 테이블로 사용한다.
employees 테이블에 존재하는 사원 중에 dept_emp 테이블에 레코드를 갖지 않는 경우 아우터 조인이 필요하지만, 대부분 그러지 않으므로 굳이 아우터 조인을 사용할 필요가 없다.
옵티마이저는 절대 아우터로 조인된 테이블을 드라이빙 테이블로 사용하지 못하기 때문에, 풀 스캔이 필요한 employees 테이블을 드라이빙 테이블로 선택한다.
즉, 이너 조인으로 사용해도 되는 쿼리를 아우터 조인으로 작성하면 옵티마이저가 조인 순서를 변경하면서 최적화를 할 수 있는 기회를 놓친다.

또 하나의 실수는, 아우터로 조인되는 테이블에 대한 조건을 where 절에 명시하는 것이다.

```mysql
select *
from employees e
  left join dept_manager mgr on mgr.emp_no = e.emp_no
where mgr.dept_no = 'd001';
```

위 쿼리는 where 절의 조건 때문에 left join 을 inner join 으로 변환해서 실행된다.
그래서, 정상적인 아우터 조인이 되려면 where 절을 left join 의 on 절로 옮겨야한다.

#### JOIN 과 FOREIGN KEY

외래키가 생성되어 있어야만 조인을 할 수 있는 것은 아니다. 즉, 외래키는 조인과 무관하다. 
외래키를 생성하는 목적은 데이터의 무결성을 보장하기 위함이고, 외래키와 연관된 무결성을 참조 무결성이라고 한다.

#### 지연된 조인

지연된 조인이란 조인이 실행되기 전에 group by 나 order by 를 처리하는 방식이다.
조인은 실행되면 실행될 수록 레코드 건수가 늘어날 수 있으므로, 조인의 결과를 group by 하거나 order by 하면 더 많은 레코드를 처리해야한다.

```mysql
select e.*
from salareis s, employees e
where e.emp_no = s.emp_no
  and s.emp_no between 10001, 13000
group by s.emp_no
order by sum(s.salary) desc
limit 10;
```

이 쿼리의 실행 계획은,

1. employees 테이블을 드라이빙 테이블로 선택해서 where 조건을 만족하는 레코드 3000 건을 읽는다.
2. 3000 건에 대해 salareis 테이블과 조인한다. 이 때 조인 수핻 횟수는 3000 * 4 건이다.
3. 조인의 결과를 임시 테이블에 저장하고 group by 처리를 해서 3000 건으로 줄인다.
4. order by 로 10 건을 반환한다.

지연된 조인으로 변경한 쿼리를 보자.

```mysql
select *
from
    (
        select s.emp_no
        from salareis s
        where s.emp_no between 10001, 13000
        group by s.emp_no
        order by sum(s.salary) desc
        limit 10 
    ) x, employees e
where e.emp_no = x.emp_no
```

1. salareis 테이블에서 모든 처리를 수행한 다음, 결과를 임시 테이블에 저장한다.
2. 임시 테이블 결과 최종 10 건에 대해 employees 테이블과 조인한다.

조인의 횟수를 비교하면, 지연된 조인으로 변경된 쿼리의 횟숫가 훨씬 적다는 것을 알 수 있다.

### GROUP BY > WITH ROLLUP 

group by 와 함께 사용할 수 있는 WITH ROLLUP 에 대해 보자. 
그룹핑된 그룹별로 소계를 가져올 수 있는 기능이다.

```mysql
select dept_no, count(*)
from dept_emp
group by dept_no with rollup ;

## 결과
## dept_no / count(*)
## d001 / 20211
## d002 / 17323
## ...
## null / 833121  <-- 소계
```

### ORDER BY

order by 절이 없으면,

1. 인덱스를 사용한 select 의 경우, 인덱스에 정렬된 순서대로 레코드를 가져온다.
2. 인덱스를 사용하지 못하고 풀 테이블 스캔을 실행하는 select 를 가정해보자. InnoDB 의 경우 항상 프라이머리 키로 클러스터링 되어 있개 때문에 기본적으로 프라이머리 키 순서대로 레코드를 가져온다.
3. select 쿼리가 임시 테이블을 거쳐 처리되면 조회되는 레코드 순서를 예측하기 어렵다.

---

Real MySQL 8.0 <백은빈, 이성욱>
