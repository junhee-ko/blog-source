---
layout: post
title:  "High Availability"
date:   2018-12-05
categories: Database
---

##### 고가용성(HA, High Availability)이 뭔가요?

서버가 오랜 시간 동안 지속적으로 정상 운영이 가능한 성질을 말합니다.

##### HA를 이루기 위한 방법이 뭐가 있나요?

클러스터링, 이중화, RAID가 있습니다.

##### 클러스터링이 뭔가요?

마치 하나의 컴퓨터처럼 사용하기 위해, 여러 컴퓨터를 연결한 병렬 시스템입니다. 클러스터링 환경에서는 특정 장비에 문제가 생기거나 특정 장비에서 실행중인 애플리케이션에서 문제가 발생더라도 전체 서비스에 영향을 미치지 않도록 제어가 가능합니다. 

##### 클러스터링의 장점은?

1. 저렴한 비용으로 다수의 서버를 증설 할 수 있습니다.
2. 1대의 서버 장애가 발생하여도 서비스 중단없이 다른 서버로 적절히 자동 분배되어 서비스를 계속 운용할 수 있습니다.
3. 서버를 확장할 때도 서비스 중단 없이 증설이 가능합니다.

##### 이중화(replication) 는 무엇인가요?

replication은 물리적으로 다른 서버의 저장 공간에 동일한 데이터를 복사하는 기술입니다.

##### master 와 slave가 뭔가요?

원본 데이터가 위치하는 서버를 master 라고 하고 그 원본을 복제한 서버를 slave 라고 합니다. master에 쓰기 작업을 하고, 그 서버의 데이터를 복제해서 여러 대의 slave 서버를 만든 후에 slave 에서는 읽기 작업만을 수행합니다.

##### replication은 어떤 용도로 사용하나요?

1. 백업용으로 사용할 수 있습니다.

   master의 database와, 그로부터 복사된 slave의 database는 identical하기 때문에, master에 문제가 생겼을 경우에, slave가 하던 일을 떠맡도록 하면 다시 시스템을 정상적으로 가동하는 데 드는 시간을 절약할 수 있습니다.

2. 역할을 분산할 수 있습니다.

   data를 직접 변경하는 처리는 master가 하도록 하고, 변경하지 않는 SELECT 같은 처리는 slave가 맡는 형식입니다.

##### mysql 에서 replication은 어떤 방식으로 하나요?

1. Master에서 데이터 변경이 일어나면 자신의 데이터베이스에 반영합니다.
2. Master에서 변경된 이력을 Binary Log에 기록 후 관련 이벤트를 날립니다.
3. Slave IO_THREAD에서 Master 이벤트를 감지하고, Master Binary Log를 자신의 Relay Log라는 곳에 기록을 합니다.
4. Slave SQL_THREAD는 Relay Log를 읽고 자신의 데이터베이스에 기록을 합니다.

##### RAID가 무엇인가요?

한 개의 디스크에 데이터를 저장하는 방식이 아닌, 데이터 저장의 성능 및 안정성 확보를 위해 복수의 디스크를 구성하는 방식입니다.

##### 샤딩이 무엇인가요?

Horizontal Partitioning입니다. 같은 테이블 스키마를 가진 데이터를 다수의 데이터베이스에 분산하여 저장하는 방법입니다.

##### 샤딩이 항상 필요하나요?

가능하면 Sharding을 피하거나 지연시킬 수 있는 방법을 찾는 것이 우선되어야 합니다. 

##### 샤딩 이외의 방법은 뭐가 있을까요?

1. Scale in 방식

   Hardware Spec이 더 좋은 컴퓨터를 사용할 수 있습니다. 

2. Cache, Repliction

   read 부하가 크다면 

3. Vertically Partition

   Table의 일부 컬럼만 자주 사용한다면

##### Reference

<http://snthewhitehacker.tistory.com/4>

<http://gywn.net/2011/12/mysql-replication-1/>