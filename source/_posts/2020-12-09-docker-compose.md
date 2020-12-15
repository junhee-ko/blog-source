---
layout: post
title: "[도커/쿠버네티스] 4장_도커 컴포즈"
date: 2020-12-09
categories: Container
---

## 도커 컴포즈 사용 이유

여러 개의 컨테이너로 구성된 웹 애플리케이션을 테스트하기 위해,
아래처럼 웹 서버 컨테이너와 데이터베이스 컨테이너를 각각 생성해야한다.

```bash
docker run --name mysql -d jko/composetest:mysql mysqld

docker run -d -p 80:80 \
--link mysql:db --name web \
jko/composetest:web apachectl -DFOREGROUND
```

여러 개의 컨테이너를 하나의 서비스로 정의해 컨테이너 묶음으로 관리하면 편리할 것이다.
그래서, Docker Compose 가 필요하다. 

## 도커 컴포즈 사용

도커 컴포즈는 컨테이너의 설정이 정의된 YAML 파일을 읽어 도커 엔진을 통해 컨테이너를 생성한다.
그래서, YAML 파일을 아래처럼 작성해야한다.

```yaml
version: '3.0'
services: # 생성될 컨테이너를 묶어놓은 단위
  web:
    image: jko/composetest:web
    ports:
      - "80:80"
    links:
      - mysql:db
    command: apachectl -DFOREGROUND
  mysql:
    image: jko/composetest:mysql
    command: mysqld
```

그리고 아래 명령어로, 컨테이너를 생성한다.

```bash
docker-comose up -d
```

## 도커 컴포즈 구성 단위

![](/image/docker-compose-units.png)

도커 컴포즈는, 컨테이너를 프로젝트 및 서비스 단위로 구분한다.
프로젝트 이름은 기본적으로, docke-compose.yml 파일이 위치한 디렉토리 이름이다.
현재 디렉토리 이름으로 된 프로젝트를 제어하는데,
-p 옵션에 프로젝트 이름을 사용해 프로젝트 이름을 명시해서 제어할 수 있다.

```bash
docker-compose -p myproject up -d
```

---

시작하세요! 도커/쿠버네티스 <용찬호>
