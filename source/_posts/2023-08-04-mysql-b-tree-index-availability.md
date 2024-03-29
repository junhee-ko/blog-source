---
layout: post
title: "B-Tree Index #4 - 가용성" 
date: 2023-08-04
categories: MySQL
---

어떤 조건에서 인덱스를 사용할 수 있고 없는지를 정리해보자.

## 비교 조건의 종류와 효율성

다중 칼럼 인덱스에서 각 칼람의 순서와 그 칼럼에 사용된 조건이 동등 비교인지 아니면 범위 조건인지에 따라 각 인덱스 칼럼의 활용 형태가 달라진다.

```mysql
select *
from demt_emp
where dept_no = 'd002' and emp_no >= 10114;
```

이 쿼리를 위해 칼럼의 순서만 다른 두 가지 케이스로 인덱스를 생성했다고 하자.

1. A index: (dept_no, emp_no)
2. B index: (emp_no, dept_no)

A 인덱스는 "dept_no = 'd002' and emp_no >= 10114" 인 레코드를 찾은 뒤에 dept_no = 'd002' 가 아닐 때 까지 인덱스를 쭉 읽으면 된다.
조건을 만족하는 레코드가 5건 이라고 하면, 5건의 레코드에 대해 비교 작업만 수행하면 된다.

B 인덱스는 "emp_no >= 10114 and dept_no = 'd002'" 인 레코드를 찾은 뒤에 모든 레코드에 대해 dept_no = 'd002' 인지 비교해야한다. 

이처럼 인덱스를 통해 읽은 레코드가 나머지 조건에 맞는지 비교하면서 선택하는 작업을 필터링이라고 한다.
B index 는 최종적으로 dept_no = 'd002' 조건을 필터링하는 레코드 5건을 가져온다.

A index 는 두 번째 칼럼인 emp_no 이 비교 작업의 범위를 좁히는데 도움을 주었다. 
B index 는 두 번째 칼럼인 dept_no 비교 작업의 범위를 좁히는데 아무런 도움을 주지 못했다. 단지 쿼리의 조건에 맞는지 검사하는 용도로 사용했다.

A 인덱스에서의 두 조건과 같이 작업 범위를 결정하는 조건을 "작업 범위 결정 조건" 이라고 한다.
B 인덱스의 dept_no = 'd002' 조건과 같이 비교 작업의 범위를 줄이지 못하고 단순히 필터링 역할을 하는 조건을 "필터링 조건" or "체크 조건" 이라고 한다.

A 인덱스에서는 dept_no 과 emp_no 모두 작업 범위 결정 조건에 해당되고, B 인덱스에서는 emp_no 칼럼만 작업 범위 결정 조건이고 dept_no 칼럼은 필터링 조건으로 사용된 것이다.
작업 범위 결정 조건이 많으면 많을 수록, 쿼리의 성능 조을 높일 수 있다.

## 인덱스의 가용성

B-Tree 인덱스의 특징은, 왼쪽 값을 기준으로 오른쪽 값이 정렬되어 있다는 것이다.
여기서 왼쪽이란, 하나의 칼럼 내에서 뿐만 아니라 다중 칼럼 인덱스의 칼럼에 대해서도 함께 적용된다.

즉, 하나의 칼럼으로 검색해도 왼쪽 부분이 없으면 인덱스 레인지 스캔 방식이 불가능하다.
또한, 다중 칼럼 인덱스에서도 왼쪽 칼럼의 값을 모르면 인덱스 레인지 스캔이 불가능하다.

first_name 칼럼이 인덱스로 지정된 employees 테이블에 대해 다음 쿼리가 어떻게 실행되는지 보자.

```mysql
select *
from employees
where first_name like '%mer';
```

이 쿼리는, 인덱스 레인지 스캔 방식을 사용할 수 없다.
왜냐하면, first_name 칼럼에 저장된 값의 왼쪽부터 한 글자씩 비교하면서 일치하는 레코드를 찾아야하는데 조건절에 주어진 상숫값 ('%mer') 에는 왼쪽 부분이 고정되어 있지 않기 때문이다.

이번에는, dept_emp 테이블에 (dept_no, emp_no) 인덱스가 지정되어 있다고 하자.
다음 쿼리는 어떻게 실행되는지 보자.

```mysql
select *
from dept_emp
where emp >= 10144;
```

인덱스의 선행 칼럼인 dept_no 조건 없이 emp_no 값으로만 검색하면 인덱스를 효율적으로 사용할 수 없다.
다중 칼럼으로 구성된 인덱스이므로 dept_no 칼럼에 대해 먼저 정렬된 후, 다시 emp_no 칼럼 값을 정렬되어 있기 때문이다.

## 가용성과 효율성 판단

다음 조건에서는, 작업 범위 결정 조건으로 사용할 수 없다.

1. not-equal 로 비교된 경우: <>, not in, not between, is not null
2. like '%??' 형태로 문자열 패턴이 비교된 경우
3. 스토어드 함수나 다른 연산자로 인덱스 칼럼이 변형된 후 비교된 경우 (ex) where substring(column, 1, 1) = 'X', where dayofmonth(column) = 1
4. not-deterministic 속성의 스토어드 함수가 비교 조건에 사용된 경우
5. 데이터 타입이 다른 서로 다른 비교 (ex) where char_column = 10
6. 문자열 데이터 타입의 콜레이션이 다른 경우

다중 칼럼으로 만들어진 인덱스는 어떤 조건에서 사용될 수 있고, 없고를 보자.

인덱스: (column_1, column_2, column_3, ... column_n)

1. 작업 범위 결정 조건으로 인덱스를 사용하지 못하는 경우: column_1 칼럼에 대한 조건이 없는 경우, column_1 칼럼의 비교 조건이 위 인덱스 사용 불가 조건 중 하나인 경우
2. 작업 범위 결정 조건으로 인덱스를 사용하는 경우 (2 <= i < n)
   1. column_1 ~ column_(i-1) 칼럼까지 동등 비교 형태 (= or in) 로 비교
   2. column_i 칼럼에 대해 다음 연산자 중 하나로 비교: 동등 비교(= or in), 크다 작다 형태 (> or <), like 좌측 일치 패턴

위 두가지 조건을 모두 만족하는 쿼리는, column_1 ~ column_i 까지는 작업 범위 결정 조건으로 사용되고 column_(i+1) ~ column_n 까지의 조건은 체크 조건으로 사용된다.

---

Real MySQL 8.0 <백은빈, 이성욱>
