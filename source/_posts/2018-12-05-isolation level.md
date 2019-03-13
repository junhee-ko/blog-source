---
layout: post
title:  "isolation level"
date:   2018-12-05
categories: Database
---

##### Isolation level이 무엇인가요 ?

트랜잭션에서 일관성이 없는 데이터를 허용하는 수준입니다.

##### 고립화 수준을 높이는 것이 좋을 까요, 낮추는 것이 좋을까요 ?

트랜잭션 고립화 수준을 높이면 일관성은 향상 되지만, 더 넓은 범위의 Lock을 더 오랫동안 유지하는 방식으로 동시성을 저하시킵니다.

##### 낮은 단계의 고립화 수준에서 발생하는 에러는 뭐가 있나요?

Dirty Read, Non Repeatable Read, Phantom Read가 있습니다.

1. Dirty Read

   데이터 캐시에는 변경이 되었지만, 아직 디스크에는 변경되지 않은 데이터(페이지)를 읽는 행위 입니다.

2. Non Repeatable Read

   트랜잭션 내에서 한 번 읽은 데이터가 트랜잭션이 끝나기 전에 변경되었다면, 다시 읽었을 때 새로운 값이 읽히는 것입니다.

3. Phantom Read

   트랜잭션 중에 없던 행이 추가되어 새로 입력된 데이터를 읽는 것입니다.

##### isolation level 에는 뭐가 있나요?

read uncommitted, read committed, repeatable read, serializable 이 있습니다.

1. read uncommitted

   고립화 수준에서 가장 낮은 수준입니다. 아직 커밋되지 않은 데이터를 다른 트랜잭션이 읽는 것을 허용합니다.세션 1)에서 price * 10을 처리하고 있으므로 price에 10 이 곱해진 결과가 세션 2)에서 보이게 됩니다. 만일 세션 1)에서 ROLLBACK 시켜버리면 세션 2)는 잘못된 결과를 보게 됩니다. 이러한 것을 Dirty Page를 읽었다고 합니다 (Dirty Read). 

   ![](/image/isolationLevel01.png)

2. read committed

   DIrty Read를 방지합니다. COMMIT 하지 않은 트랜잭션이 이용한 자원에 대해 트랜잭션이 완료되어야 (= 잠금이 해제되어야) 데이터를 읽을 수 있습니다.

3. repeatable read

   트랜잭션이 완료될 때까지 공유 잠금을 유지하여 여러번 같은 데이터를 읽어도 항상 같은 값이 얻어지도록 합니다.

   주의해야할 사항은 REPEATABLE READ가 설정되면 원래 읽었던 데이터에 대하여서는 항상 같은 값이 유지 되지만 새로 추가되는 데이터를 막을 수는 없습니다. 

   예를 들어 특정 조건에 만족한 레코드가 5개였으면 이 5개의 레코드는 언제든 같은 값을 보이게 되지만 다른 트랜잭션에 의해 이 조건을 만족하는 새로운 레코드가 추가되면 원래 없었던 새로운 레코드가 보이게 됩니다. 이러한 것을 Pantom Read라고 합니다.

   세션 1) 에서 Test 테이블을 만들어서 3개의 레코드를 추가하였습니다. 트랜잭션에서 검색의 범위가 1부터 4이므로 INSERT한 3개의 레코드가 위 빨간색 박스안의 내용처럼 표시됩니다.

   REPEATABLE READ 가 설정되었으므로 이 3개의 레코드는 트랜잭션이 끝날 때 까지 아무도 변경 할 수 없습니다. 예를 들어 세션 2)에서 col1 컬럼이 1인 레코드의 col2를 'AAAAA' 에서 'BBBBB'로 바꾸려고 하면 잠금으로 인하여 처리되지 못합니다.

   하지만 세션 2)에서는 검색 범위안에는 들어가지만 아직 존재하지 않는 레코드를 추가할 때 아무 문제 없이 처리가 됩니다. 세션 2)의 수행 결과를 보면 (1개 행 적용됨) 이 표시된 것을 볼 수 있습니다.

   이 상태에서 세션 1)에서 똑같은 검색을 수행하면 새로운 레코드가 하나 추가되어 이전의 결과와 다르게 됩니다. 이것이 Pantom Read 입니다.

   ![](/image/isolationLevel02.png)

4. serializble

   가장 높은 고립화 수준으로 REPEATABLE READ 고립화 수준이 막지 못하는 Pantom Read까지 방지해줍니다. 완벽한 데이터베이스의 일관성은 유지해 주지만 잠금을 유지하기 위하여 오버헤드는 증가하게 됩니다.바로 위의 그림에서, 만일 세션 1) 에서 SERIALIZABLE 고립화 수준이 설정되었다면 세션 2)에서의 INSERT 는 불가능하게 됩니다.

##### Reference

<http://goforaurora.tistory.com/entry/SQL-%ED%8A%B8%EB%9E%9C%EC%9E%AD%EC%85%98%EC%9D%98-%EA%B3%A0%EB%A6%BD%ED%99%94-%EC%88%98%EC%A4%80Isolation-Level>