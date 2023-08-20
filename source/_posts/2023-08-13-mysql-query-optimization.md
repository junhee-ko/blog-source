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

SQL 표준에서 null 의 정의는 비교할 수 없는 값이다.
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

#### 날짜 비교

##### DATE 또는 DATETIME 문자열 비교

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

DATETIME 의 값에서 시간 부분만 떼어 버리고 비교하려면, 다음 예제와 괕이 쿼리를 작성하면 된다.
date() 함수는 datetime 타입의 값에서 시간 부분은 버리고 날짜 부분만 반환한다.

```mysql
select count(*)
from employees
where hire_date > DATE(NOW());
```

datetime 타입의 값을 date 타입으로 만들지 않고 그냥 비교하면 MySQL 서버가 date 타입의 값을 datetime 으로 변환해서 같은 타입으 만든 다음 비교를 수행한다.

### LIMIT n

```mysql
select *
from employees
where emp_no between 10001 and 10010
order by first_name
limit 0, 5;
```

1. employees 테이블에서 where 절긔 검색 조건에 일치하는 레코드를 전부 읽는다.
2. first_name 칼럼 값에 따라 정렬한다.
3. 상위 5건만 사용자에게 반환한다.

limit 의 중요한 특성은, limit 에서 필요한 레코드 건수만 준비되면 즉시 쿼리를 종료한다.
즉, 위 쿼리는 모든 레코드의 정렬이 완료되지 않았어도 상위 5건 까지만 정렬되면 작업을 멈춘다.

### JOIN

#### JOIN 의 순서와 인덱스

조인 작업에서 드라이빙 테이블을 읽을 때는 인덱스 탐색 작업을 단 한번 수행하고, 그 이후부터는 스캔만 실행하면 된다.
하지만 드리븐 테이블에서는 인덱스 탐색 작업과 스캔 작업을 드라이빙 테이블에서 읽은 레코드 건수만큼 반복한다.
드리븐 테이블을 읽는 것이 훨씬 더 큰 부하를 차지하하는 것이다.

#### JOIN 칼럼의 데이터 타입

조인 칼럼 간의 비교에서 각 칼럼의 데이터 타입이 일치하지 않으면, 인덱스를 효율적으로 사용할 수 없다.

### ORDER BY

order by 절이 없으면,

1. 인덱스를 사용한 select 의 경우, 인덱스에 정렬된 순서대로 레코드를 가져온다.
2. 인덱스를 사용하지 못하고 풀 테이블 스캔을 실행하는 select 를 가정해보자. InnoDB 의 경우 항상 프라이머리 키로 클러스터링 되어 있개 때문에 기본적으로 프라이머리 키 순서대로 레코드를 가져온다.
3. select 쿼리가 임시 테이블을 거쳐 처리되면 조회되는 레코드 순서를 예측하기 어렵다.

#### order by 사용법 및 주의사항

order by 뒤에 숫자 값이 아닌 문자열 상수를 사용하면 옵티마이저가 order by 절 자체를 무시한다.

---

Real MySQL 8.0 <백은빈, 이성욱>
