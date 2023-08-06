---
layout: post
title: "실행 계획" 
date: 2023-08-06
categories: MySQL
---

옵티마이저가 사용자 개입 없이 항상 좋은 실행 계획을 만들어 낼 수 있는 것은 아니다.
사용자가 보완할 수 있도록 EXPLAIN 명령으로 옵티마이저가 수립한 실행 계획을 확인할 수 있다.

## 실행 계획 분석

EXPLAIN 명령을 실행하면 쿼리 문장의 특성에 따라 표 형태로 된 1줄 이상의 결과가 표시된다.
표의 각 라인은 쿼리 문장에서 사용된 테이블의 개수만큼 출력된다.
이 때, 서브 쿼리로 임시 테이블을 생성하면, 임시 테이블까지 포함된다.

실행 계획에 표시되는 각 칼럼에 대해 정리하자.

### id

하나의 select 문장은 다시 1개 이상의 sub select 문장을 포함할 수 있다.

```mysql
select *
from (select * from tb_test1) tb1, tb_test2 tb2
where tb1.id = tb2.id;
```

위 쿼리는 다음과 같이 분리해서 생각할 수 있다.

```mysql
select *
from tb_test1

select *
from tb1, tb_test2 tb2
where tb1.id = tb2.id;
```

이렇게 select 키워드 단위로 구분한 것을 단위 select 쿼리라고 할 수 있다.
id 칼럼은 단위 select 쿼리 별로 부여되는 식별자 값이다.
이 쿼리의 경우 최소 2개의 id 값이 표시될 것이다.

만약, 하나의 select 문장안에서 여러 개의 테이블을 조인하면 조인되는 테이블 개수만큼 실헹 계획 레코드가 출력되지만 같은 id 값이 부여된다.
아래 쿼리는, 실행 계획에 두 개의 레코드가 표시되지만 id 값이 증가되지 않고 같은 id 값이 부여된다.  

```mysql
explain
select *
from employees e, salaries s
where e.emp_no = s.emp_no
```

다음 쿼리의 실행 계획에서는 각 레코드가 서로 다른 id 값을 가진 3개의 레코드가 표시된다.

```mysql
explain
select 
( (select count(*) from employees) + (select count(*) from departments) ) as total_count;
```

여기서 주의할 점은, 실행 계획의 id 칼럼이 테이블의 접근 순서를 의미하지 않는다.

### select_type

각 단위 select 쿼리가 어떤 타입의 쿼리인지 표시한다.
select_type 에 표시 될 수 있는 값에는,

- SIMPLE
- PRIMARY
- UNION
- DEPENDENT UNION
- UNION RESULT
- SUBQUERY
- DEPENDENT SUBQUERY
- DERIVED
- DEPENDENT DERIVED
- UNCACHEABLE SUBQUERY
- UNCACHEABLE UNION
- MATERIALIZED

#### SIMPLE

UNION 이나 서브쿼리를 사용하지 않는 단순한 select 쿼리인 경우 표시된다.
쿼리 문장이 아무리 복합해도 select_type 이 SIMPLE 인 쿼리는 단 하나만 존재한다.
일반적으로 제일 바깥 select 쿼리에 표시된다.

#### PRIMARY

UNION 이나 서브쿼리를 가지는 select 쿼리의 가장 바깥쪽에 있는 단위 쿼리는 PRIMARY 로 표시된다.
SIMPLE 과 마찬가지로, select_type 이 PRIMARY 인 쿼리는 단 하나만 존재한다.

#### UNION

UNION 으로 결합하는 단위 select 쿼리 가운데 첫 번째를 제외한 두 번째 이후 단위 select 쿼리의 select_type 은 UNION 으로 표시된다.
UNION 의 첫 번째 단위 select 는 UNION 되는 쿼리 결과들을 모아서 저장하는 임시 테이블(DERIVED) 가 select_type 으로 표시된다.

```mysql
explain
select *                                                 ## PRIMARY
from (
    (select emp_no from employees e1 limit 10) union all ## DERIVED
    (select emp_no from employees e2 limit 10) union all ## UNION
    (select emp_no from employees e3 limit 10)           ## UNION
) tb;
```

#### SUBQUERY

select_type 의 SUBQUERY 는 from 절 이외에서 사용되는 서브쿼리만을 의미한다.
from 절에서 사용된 서브쿼리는 select_type 이 DERIVED 로 표시된다.

### table

실행 계획은 단위 select 쿼리 기준이 아니라, 테이블 기준으로 표시된다.
테이블 이름에 별칭이 부여되는 경우 별칭이 표시된다.
다음과 같이 별도의 테이블을 사용하지 않는 쿼리이기 때문에, table 칼럼에 null 이 표시된다.

```mysql
explain select now();
```

table 칼럼에 <derived N> 또는 <union M,N> 같이 명시되는 경우가 많은데, 이 테이블은 임시 테이블을 의미한다.
<> 안에 항상 표시되는 순자는 단위 select 쿼리의 id 값이다.

### partitions

파티션 관련 실행 계획을 확인하기 전에, 간단한 파티션 테이블을 생성하고, employees 테이블의 모든 레코드를 복사하자.
employees_2 테이블은 hire_date 칼럼 값을 기준으로 5년 단위 기준으로 나누어진 파티션을 가진다.

```mysql
create table employees_2 (
    emp_no int not null,
    first_name varchar(14) not null,
    hire_date date not null,
    primary key (emp_no, hire_date)
) partition by range columns (hire_date)
(
    partition p1986_1990 values less than ('1990-01-01'),
    partition p1991_1995 values less than ('1996-01-01'),
    partition p1996_2000 values less than ('2000-01-01'),
    partition p2001_2005 values less than ('2006-01-01'),
)

insert into employees_2 select * from employees;
```

이제 hire_date 칼럼의 값이 1999년 11월 15일과 2000년 01월 15일 사이의 레코드를 검색하는 쿼리를 보자.

```mysql
explain
select *
from employees_2
where hire_date between '1999-11-15' and '2000-01-15'
```

위 테이블 생성 구문에서 파티션 목록을 보면, 이 쿼리에서 조회하는 데이터는 p1996_2000 와 p2001_2005 파티션에 저장되어 있다.
실제로 옵티마이저는 쿼리의 hire_date 칼럼 조건을 보고 이 쿼리에서 필요로 하는 데이터는 p1996_2000 와 p2001_2005 파티션에만 있다는 것을 알 수 있다.

### type

MySQL 서버가 각 테이블의 레코드를 어떤 방식으로 읽었는지를 나타낸다.
type 칼럼에 표시될 수 있는 값은,

- system
- const
- eq_ref
- ref
- full_text
- ref_or_null
- unique_subquery
- index_subquery
- range
- index_merge
- index
- ALL

#### system

레코드가 1건 or 0건 존재하는 테이블을 참조하는 형태의 접근 방법이다.

#### const

테이블의 레코드 건수와 관계 없이, 쿼리가 프라이머리 키나 유니크 키 칼럼의 값을 이용하는 where 조건절을 가지고 있으며, 반드시 1건을 반환하는 쿼리의 처리 방식이다.
다중 칼럼으로 구성된 프라이머리 키나 유니크 키 중에서 인덱스 일부 칼럼만 조건으로 사용하면 const 타입의 접근 방법을 사용할 수 없다.
모든 칼럼을 where 절에 명시하면, const 접근 방법을 사용한다.

#### eq_ref

여러 테이블이 조인되는 쿼리의 실행 계획에서만 표시된다.
조인에서 처음 읽은 테이블의 칼럼 값을, 그 다음 읽어야할 테이블의 프라이머리 키나 유니크 키 칼럼의 검색 조건에 사용할 때를 가리켜 eq_ref 라고 한다.
다중 칼럼으로 만들어진 프라이머리 키나 유니크 인덱스라면 인덱스의 모든 칼럼이 비교 조건에 사용되어야한다.

```mysql
explain
select *
from from demp_emp de, employees e
where e.emp_no = de.emp_no and de.dept_no = 'd005';
```

employees 테이블의 emp_no 는 프라이머키 키 라서, employees 테이블을 읽는 실행계획의 레코드 type 칼럼이 eq_ref 로 표시된다.

#### ref

인덱스의 종류와 관계 없이 equal 조건으로 검색할 때 사용된다.

```mysql
explain
select *
from dept_emp
where dept_no = 'd005';
```

dept_emp 테이블의 프라이머리 키를 구성하는 칼럼이 (dept_no, emp_no) 이라고 하자.
일부만 동등 조건으로 명시되었기 때문에, 레코드가 1건이라는 보장이 없다. 그래서 const 가 아닌 ref 접근 방법이 사용되었다.

#### index_merge

2개 이상의 인덱스를 이용해 각각의 검색 결과를 만들어낸 후, 그 결과를 병합해서 처리하는 방식이다.

#### index

인덱스를 처음부터 끝까지 읽는 인덱스 풀 스캔을 의미한다.

다음 쿼리는, 아무런 where 조건이 없으므로 range 나 const 나 ref 접근 방법을 사용할 수 없다.
하지만, 정렬하려는 칼럼은 인덱스가 있으므로 별도 정렬 처리를 피하려고 index 접근 방법을 사용한다.

```mysql
explain
select *
from departments
order by dept_name desc limit 10;
```

단순히 인덱스를 거꾸로 읽어서 10개만 가져오면 되기 때문에 효율적이다.
하지만, limit 조건이 없거나 가져와야할 레코드 건수가 많으면 느릴 수 있다.

#### ALL

풀 테이블 스캔 접근 방법이다.
테이블을 처음부터 끝까지 읽어서 불필요한 레코드를 제거하고 반환한다.

InnoDB 에서는 풀 테이블 스캔이나 인덱스 풀 스캔과 같은 대량 디스크 I/O 를 유발하는 작업을 위해 한꺼번에 많은 페이지를 읽어 들이는 기능을 제공한다.
이를 Read Ahead 기능이라고 한다.

### possible_keys

옵티마이저가 최적의 실행 계획을 만들기 위해 후보로 선정했던 접근 방법에서 사용되는 인덱스의 목록이다.
즉, "사용될 법했던 인덱스의 목록" 일 뿐이다.

### key

최종 선택된 실행 계획에서 사용하는 인덱스이다.
key 칼럼에 표시되는 값이 PRIMARY 이면 프라이머리 키를 사용한다는 의미이고, 그 이외의 값은 모두 테이블이나 인덱스를 생성할 때 부여했던 고유 이름이다.

실행 계획의 type 칼럼이 index_merge 가 아닌 경우 반드시 테이블 하나다 하나의 인덱스만 이용할 수 있다.
index_merge 실행계획이 사용될 때는, 2개 이상의 인덱스가 사용되는데 이 때 key 칼럼에 여러 개의 인덱스가 "," 로 구분되어 표시된다.

### key_len

key_len 칼럼의 값은 쿼리를 처리하기 위해, 다중 칼럼으로 구성된 인덱스에서 몇 개의 칼럼까지 사용했는지 알 수 있다.

아래 쿼리는, (dept_no, emp_no) 으로 구성된 프라이머리 키를 가지는 dept_emp 테이블을 조회한다.

```mysql
explain
select *
from dept_emp
where dept_no = 'd005';
```

실행 계획 결과는, key_len 칼럼의 값이 16 으로 표시된다.
dept_no 칼럼의 타입이 CHAR(4) 이기 때문에 프라이머리 키에서 앞 쪽 16 바이트만 유효하게 사용했다는 의미이다.

MySQL 서버는 utf8mb4 문자를 위해 메모리 공간을 할당 할때, 4바이트로 계산한다.
그래서 key_len 칼럼의 값으로 4*4 바이트가 표시된다.

다음 쿼리를 보자.

```mysql
explain
select *
from dept_emp
```

emp_no 칼럼 타입은 INTEGER 이며, INTEGER 타입은 4 바이트를 차지한다.
그래서 key_len 칼럼의 값으로 dept_no 칼럼의 길이와 emp_no 칼럼의 길이 합인 20이 표시된다.

### ref 

equal 비교 조건으로 어떤 값이 제공되었는지 표시된다.
상숫값을 지정했으면 ref 칼럼의 값으로 const, 다른 테이블의 칼럼 값이면 그 테이블 명과 칼럼명이 표시된다.

```mysql
explain
select *
from employees e, dept_emp de
where e.emp_no = de.emp_no;
```

이 쿼리는 employees 테이블과 dept_emp 테이블을 조인하는데, 조언 조건에 사용된 emp_no 칼럼의 값에 아무런 변환이나 가공이 없다.
그래서 ref 칼럼에는 조인 대상의 칼럼 이름인 "employees.de.emp_no" 이 그대로 표시된다.

### rows

옵티마이저는 각 조건에 대해 처리 방식을 나열하고, 각 처리 방식의 비용을 비교해 최종적으로 하나의 실행 계획을 수립한다.
이 때 각 처리 방식이 얼마나 많은 레코드를 읽고 비교해야 하는지 예측해서 비용을 산정한다.

실행 계획의 rows 칼럼은 실행 계획의 효율성 판단을 위해 예측했던 레코드 건수를 표시한다.
이 값은 통계 정보를 참고해서 MySQL 옵티마이저가 산출해 낸 예상값이라서 정확하지 않다.

```mysql
explain
select *
from dept_emp
where from_date >= '1985-01-01';
```

이 쿼리는 ix_fromdate 인덱스를 이용해서 처리할 수 있지만 풀 테이블 스캔(ALL) 을 선택했다.
rows 칼럼에는 대략 331,143 건의 레코드를 읽어야 할것이라고 예측했다.
dept_emp 테이블의 전체 레코드가 331,143 건 이므로, 테이블의 모든 레코드를 비교해야한다고 판단한 것이다.

범위를 줄인 다음 쿼리를 보자.

```mysql
explain
select *
from dept_emp
where from_date >= '2002-07-01';
```

rows 칼럼에는 292 건의 레코드만 읽으면 원하는 결과를 가져올 수 있을 것으로 예측했다.
물론 그래서, 실행 계획도 풀 테이블 스캔이 아니라 range 로 인덱스 레인지 스캔을 사용한다.
옵티마이저는 from_date 칼럼의 값이 '2002-07-01' 보다 크거나 같은 레코드가 292 건만 존재한다고 예측했고, 이는 전체 테이블 건수와 비교하면 8.8% 밖에 되지 않는다.
그래서 최종적으로 range 방식으로 처리한 것이다.

### filtered

실행 계획의 rows 칼럼의 값은 인덱스를 사용하는 조건에만 일치하는 건수를 예측한 것이다.
하지만 대부분의 쿼리에서 where 절에 사용되는 조건이 모두 인덱스를 사용할 수 있는 것은 아니다.

```mysql
explain
select *
from employees e, salaries s
where e.first_name = 'jko'                                  # 인덱스 사용
    and e.hire_date between '1990-01-01' and '1991-01-01'
    and s.emp_no = e.emp_no
    and s.fro_date between '1990-01-01' and '1991-01-01'    # 인덱스 사용
    and s.salary between 50000 and 60000;
```

employees 테이블과 salaries 테이블 중에서 조건들을 모두 고려해서 최종적으로 일치하는 레코드 건수가 적은 테이블이 드라이빙 테이블로 선정될 것이다.

employees 테이블에 대한 실행 계획이 아래와 같다고 하자.

1. rows: 233
2. filtered: 16.03

인덱스 조건에 일치하는 레코드 건수는 233 건이고, 이 중에서 16.03% 만 인덱스를 사용하지 못하는 "e.hire_date between '1990-01-01' and '1991-01-01'" 조건에 일치한다는 것을 의미한다.
그래서, salaries 테이블로 조인을 수행하는 레코드 건수는 대략 233 * 0.1603 건 정도였다는 것을 알 수 있다.

---

Real MySQL 8.0 <백은빈, 이성욱>
