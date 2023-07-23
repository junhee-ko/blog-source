---
layout: post
title: Transaction, Lock
date: 2023-07-23
categories: MySQL
---

동시성에 영향을 미치는 다음 세 가지 내용을 정리한다.

1. 트랜잭션
2. 잠금
3. 트랜잭션의 격리 수준

## 트랜잭션

트랜잭션이란 하나의 논리적인 작업 셋에 쿼리가 하나이든 두 개 이상이든, 논리적인 작업 셋 자체가 모두 적용되거나 아무것도 적용되지 않음을 보장하는 것이다. 

MyISAM 스토리지 엔진이나 MEMORY 스토리지 엔진은 트랜잭션을 지원하지 않는다.
InnoDB 스토리지 엔진은 트랜잭션을 지원한다.

다음 예를 보자.

```mysql
create table tab_myisam ( fdpk int not null, primary key (fdpk) ) ENGINE=MyISAM;
insert into tab_myisam (fdpk) values (3);

create table tab_innodb ( fdpk int not null, primary key (fdpk) ) ENGINE=INNODB;
insert into tab_innodb (fdpk) values (3);

SET autocommit = ON;

insert into tab_myisam (fdpk) values (1), (2), (3); ## error
insert into tab_innodb (fdpk) values (1), (2), (3); ## error

select * from tab_myisam; ## 1,2,3
select * from tab_innodb; ## 3
```

두 insert 모두 primary key 중복 오류로 실패하지만, 레코드를 조회하면 결과가 다르다.

1. InnoDB: 쿼리 중 일부라도 오류가 발생하면 전체를 원 상태로 만든다. 
2. MyISAM: 부분 업데이트 (Partial Update) 가 일어난다.

## MySQL 엔진의 잠금

MySQL 의 잠금은 크게 두 가지이다.

1. MySQL 엔진 레벨 잠금: 모든 스토리지 엔진에 영향을 미침
2. 스토리지 엔진 레벨 잠금: 스토리지 엔진 간 영향을 미치지 않음

MySQL 엔진 레벨 잠금에는, 아래와 같은 락들이 있다.

1. 글로벌 락
2. 테이블 락
3. 네임드 락
4. 메타데이터 락

### 글로벌 락

아래 명령으로 획득할 수 있다.

```mysql
FLUSH TABLES WITH READ LOCK 
```

MySQL 락 중에 가장 범위가 크다. 
한 세션에서 글로벌 락을 획득하면, 다른 세션에서 SELECT 를 제외한 대부분의 DDL 이나 DML 문장을 실행하면 글로벌 락이 해제될 때 까지 대기 상태가 된다.
영향 범위는, MySQL 서버 전체이고 작업 대상 테이블이나 데이터베이스가 다르더라도 동일하게 영향을 미친다.

### 테이블 락

개별 테이블 단위로 설정되는 잠금이다.
명시적 또는 묵시적으로 특정 테이블의 락을 획득할 수 있다.

명시적으로는, 아래 명령으로 특정 테이블에 대해 락을 획득 및 해제 할 수 있다.

```mysql
LOCK TABLES table_name [ READ | WRITE ]
UNLOCK TABLES
```

묵시적으로는, 

1. MyISAM 이나 MEMORY: 테이블에 데이터를 변경하는 쿼리를 실행하면 발생한다.
2. InnoDB: 스토리지 엔진 차원에서 레코드 기반의 잠금을 제공하므로, 단순 데이터 변경으로 테이블 락이 설정되지 않는다.

### 네임드 락

"GET_LOCK()" 함수를 이용해 임의의 문자열에 대해 잠금을 설정할 수 있다.
이 잠금의 대상은 테이블이나 레코드가 아니라, 사용자가 지정한 문자열이다.

```mysql
SELECT GET_LOCK('myLock', 2)  # myLock 이라는 문자열에 대해 잠금 획득 시도, 이미 사용 중이면 2초 동안 대기
SELECT RELEASE_LOCK('myLock') # 해제 
```

### 메타데이터 락

데이터베이스 객체 (테이블이나 뷰 등등) 의 이름이나 구조를 변경하는 경우 획득하는 잠금이다.
명시적으로 획득하고 해제할 수 있는 락이 아니다.
예를 들어, "RENAME TABLE tab_a to tab_b" 와 같이 테이블의 이름을 변경하는 경우 자동으로 획득한다.

## InnoDB 스토리지 엔진 잠금

MySQL 에서 제공하는 잠금과는 별개로 스토리지 엔진 내부에서 레코드 기반의 잠금 방식을 지원한다.
InnoDB 스토리지 엔진 잠금의 종류로는 다음과 같은 것들이 있다.

1. 레코드 락
2. 갭 락
3. 넥스트 키 락
4. 자동 증가 락

### 레코드 락

레코드 자체만을 잠그는 것이다. 
정확히는, 레코드 자체가 아니라 인덱스의 레코드를 잠근다.

### 갭 락

레코드 자체가 아니라, 레코드와 바로 인접한 레코드 사이의 간격만 잠그는 것이다.
레코드와 레코드 사이의 간격에 새로운 레코드가 insert 되는 것을 제어할 수 있다.
그 자체로 사용되기 보다, 넥스트 키 락의 일부로서 자주 사용된다.

### 넥스트 키 락

레코드 락과 갭 락을 합쳐 놓은 형태의 잠금이다.
InnoDB 의 갭 락이나 넥스트 키 락은, 바이너리 로그에 기록되는 쿼리가 레플리카 서버에서 실행될 때 소스 서버에서 만들어낸 결과와 동일한 결과를 만들어내도록 보장하는 것이 주 목적이다.

### 자동 증가 락

MySQL 에서는 자동 증가하는 숫자 값을 추출하기 위해 AUTO_INCREMENT 라는 칼럼 속성을 제공한다.
AUTO_INCREMENT 칼럼이 사용된 테이블에 동시에 여러 레코드가 insert 되는 경우, 저장 되는 레코드는 중복되지 않고 저장된 순서대로 증가하는 일련 번호 값을 가져야한다.
InnoDB 스토리지 엔진에서는, 이를 위해 테이블 수준의 락인 AUTO_INCREMENT 락을 사용한다.

새로운 레코드를 저장하는 insert, replace 등의 쿼리에서만 필요하다.
트랜잭션과 관계 없이 AUTO_INCREMENT 값을 가져오는 순간만 락이 걸렸다가 즉기 해제된다.
AUTO_INCREMENT 락은 테이블에 단 하나만 존재하므로, 두 개의 insert 쿼리가 동시에 실행되는 경우 하나의 쿼리가 AUTO_INCREMENT 락을 걸면 나머지 쿼리는 AUTO_INCREMENT 락을 기다려야한다.  

## 인데스와 잠금

InnoDB 의 잠금은 레코드를 잠그는 것이 아니라, 인덱스를 잠근다.
즉, 변경해야할 레코드를 찾기 위해 검색한 인덱스의 레코드에 모두 락을 걸어야한다.

```mysql
# employees 에는 first_name 칼럼만 멤버로 담긴 ix_firstname 이라는 인덱스가 준비됨

SELECT COUNT(*) FROM employees WHERE first_name = 'Junhee'; # 253
SELECT COUNT(*) FROM employees WHERE first_name = 'Junhee' and last_name = 'Ko'; # 1

UPDATE employees SET hire_date = NOW() WHERE first_name = 'Junhee' and last_name = 'Ko';
```

위 한 건의 업데이트를 위해, first_name = 'Junhee' 인 레코드 253 건의 레코드가 모두 잠긴다.
만약 이 테이블에 인덱스가 하나도 없으면, 테이블을 풀 스캔하면서 update 작업을 하는데 이 과정에서 테이블의 모든 레코드를 잠그게 된다.
그래서, MySQL InnoDB 에서 인덱스 설계가 중요하다.

## 격리 수준

트랜잭션 격리 수준이란, 여러 트랜잭션이 동시에 처리될 때 특정 트랜잭션이 다른 트랜잭션에서 변경하거나 조회하는 데이터를 볼 수 있게 허용할지 말지를 결정하는 것이다.

크겍 네 가지로 나뉜다.
아래로 내려갈 수록, 각 트랜잭션 간의 데이터 격리 정도가 높아져서 동시 처리 성능이 떨어진다.

1. READ UNCOMMITTED
2. READ COMMITTED
3. REPEATABLE READ
4. SERIALIZABLE

### READ UNCOMMITTED

각 트랜잭션의 변경 내용이 COMMIT 이나 ROLLBACK 여부에 상관 없이 다른 트랜잭션에서 보인다.

1. 사용자 A: employees 테이블에 새로운 사원을 insert 한다. 아직 커밋하지 않았다.
2. 사용자 B: 1번에서 insert 한 사원 정보를 조회할 수 있다.

만약, 사용자 A 가 롤백을 하더라도 여전히 사용자 B 는 새로운 사원이 정상적인 사원이라고 생각하고 처리를 계속 할 수 있다.
이것이, 어떤 트랜잭션에서 처리한 작업이 완료되지 않았는데도 다른 트랜잭션에서 볼 수 있는 현상인 Dirty Read 이다.

### READ COMMITTED

오라클 DBMS 에서 기본으로 사용되는 격리 수준이고, 온라인 서비스에서 가장 많이 선택되는 격리 수준이다.
어떤 트랜잭션에서 데이터를 변경했더라도 커밋이 완료된 데이터만 다른 트랜잭션에서 조회가 가능하다.

1. 사용자 A: emp_no=123 인 사원의 first_name 을 변경. "Junhee1" -> "Junhee2". 아직 커밋하지 않음
2. 사용자 B: emp_no=123 인 사원을 조회하면 조회 결과의 first_name 값은 여전히 "Junhee1"

이렇게 가능한 이유는 다음과 같다.

1. 위 1번에서 새로운 값인 "Junhee2" 는 employees 테이블에 즉시 기록되고 이전 값인 "Junhee1" 은 언두 영역으로 백업
2. 사용자 B 는 사용자가 A 가 커밋하기 전에는 언두 영역에 백업된 레코드를 가져온다.

READ COMMITTED 격리 수준에서는, "NON-REPEATABLE READ" 라는 문제가 발생한다.

1. 사용자 B: 트랜잭션을 시작하고 first_name 이 "Junhe2" 인 사용자를 조회. 결과 없음
2. 사용자 A: 사원 번호가 123 인 사원의 이름을 변경. "Junhee1" -> "Junhee2" 
3. 사용자 B: 같은 쿼리를 실행. 결과 한 건

사용자 B 는, 하나의 트랜잭션에서 같은 SELECT 쿼리를 실행하면 항상 같은 결과를 가져와야하는 "REPEATABLE READ" 정합성에 어긋났다.  

### REPEATABLE READ

MySQL 의 InnoDB 스토리지 엔진에서 기본으로 사용하는 격리 수준이다.

모든 InnoDB 의 트랜잭션은 고유한 트랜잭션 번호 (순차 증가) 를 가지며, 언두 영역에 백업된 모든 레코드에는 변경을 발생시킨 트랜잭션의 번호가 포함된다.
언두 영역의 백업된 데이터를 InnoDB 스토리지 엔진이 불필요하다고 판단되는 시점에 주기적으로 삭제한다.

다음 예를 보자.
사용자 A 의 트랜잭션 번호는 12, 사용자 B 의 트랜잭션 번호는 10 이라고 하자.

1. 사용자 B: 트랜잭션을 시작하고 사원번호 123 인 사원 조회. 결과는 "Junhee1" 한 건 
2. 사용자 A: 사원의 이름을 변경하고 커밋한다. "Junhee1" -> "Junhee2" (이 때, 변경 전 데이터를 언두 로그로 복사)
3. 사용자 B: 다시 사원번호 123 인 사원 조회. 결과는 "Junhee1" 한 건

이렇게 "REPEATABLE READ" 가 가능한 이유는 다음과 같다.

1. 사용자 B 가 트랜잭션을 시작하면서 10 번 트랜잭션 번호를 부여 받음
2. 사용자 B 는 트랜잭션 안에서 실행되는 모든 SELECT 쿼리는 10 보다 작은 트랜잭션 번호에서 변경한 것만 볼 수 있다.

REPEATABLE READ 격리 수준에서도 "Phantom Read" 라는 부정합이 발생할 수 있다.

1. 사용자 B: 트랜잭션을 시작하고 사원번호 >= 123 인 사원 SELECT .. FOR UPDATE. 결과는 "Junhee1" 한 건
2. 사용자 A: 사원번호 124 인 사원 "Junhee2" 추가하고 커밋
3. 사용자 B: 사원번호 >= 123 인 사원 SELECT .. FOR UPDATE. 결과는 "Junhee1", "Junhee2" 두 건

두 번의 SELECT .. FOR UPDATE 결과가 다르다.
SELECT .. FOR UPDATE 쿼리는 SELECT 하는 레코드에 쓰기 잠금을 걸어야하는데 언두 레코드에는 잠금을 걸 수 없다.
그래서, SELECT .. FOR UPDATE 로 조회되는 레코드는 언두 영역의 변경 전 데이터를 조회하지 않고 현재 레코드의 값을 가져온다.

### SERIALIZABLE

가장 엄격한 수준의 격리 수준이다. 
그만큼 동시 처리 성능도 다른 격리 수준보다 떨어진다.

---

Real MySQL 8.0 <백은빈, 이성욱>
