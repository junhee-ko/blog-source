---
layout: post
title: "[도커/쿠버네티스] 2장_도커 엔진"
date: 2020-12-05
categories: Container
---

## 도커 이미지와 컨테이너

![](/image/docker-container.png)

1. 도커 이미지 

   여러 개의 계층으로 된 바이너리 파일로 존재하고, 컨테이너를 생성하고 실행할 때 읽기 전용으로 사용된다.

   이미지 이름은, 다음과 같은 형태로 구성된다 : [저장소 이름]/[이미지 이름]:[태그]

   ex) jko/ubuntu:14.04

   

2. 도커 컨테이너

   이미지로 컨테이너를 생성하면, 이미지 목적에 맞는 파일이 들어 있는 파일 시스템과, 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립된 공간이 생성된다.

   이것이 도커 컨테이너이다.

   컨테이너는 이미지를 읽기 전용으로 사용하되 이미지에서 변경된 사항만 컨테이너 계층에 저장하기때문에, 컨테이너에 무엇을 하든지 원래 이미지는 영향을 받지 않는다.

   또한, 컨테이너는 호스트와 분리되어있기 때문에, 어떤 애플리케이션을 설치하거나 삭제해도 다른 컨테이너와 호스트는 변화가 없다.

## 도커 컨테이너 다루기

1. 컨테이너 생성

   ```bash
   docker run -i -t ubuntu:14.04
   ```

   docker run 명령어는 컨테이너를 생성하고 실행한다. 

   -i -t 는 컨테이너와 interactive 입출력을 가능하게 한다.

   

2. 컨테이너 목록 확인

   ```bash
   docker ps
   ```

3. 컨테이너 삭제

   ```bash
   docker rm mycentos
   ```

   컨테이너의 이름은 생성한 컨테이너의 이름에 맞게.

   

4. 컨테이너를 외부에 노출

   ```bash
   docker run -i -t --name mywebserver -p 80:80 ubuntu:14.04
   ```

   컨테이너는 가상 머신처럼, 가상 IP 주소를 할당 받는다. 

   -p 옵션은 컨테이너의 포트를 호스트의 포트와 바인딩한다. ( 호스트의 포트:컨테이너의 포트 )

   

5. 컨테이너 애플리케이션 구축

   여러 애플리케이션을 한 컨테이너에 설치할 수 있다. 

   하지만, 컨테이너에 애플리케이션을 하나만 동작시키면 컨테이너 간 독립성을 보장하고 애플리케이션의 버전 관리와 소스 코드 모듈화가 쉬워진다.

   

6. 도커 볼륨

   ```bash
   docker run -d \
   --name testvolume \
   --e MYSQL_PWD=password \
   --e MYSQL_DATABASE=db \
   -v /home/test_db:/var/lib/mysql \
   mysql:5.7
   ```

   mysql 컨테이너를 삭제하면, 컨테이너 계층에 저장되어 있던 DB 정보도 삭제된다. 

   이를 방지하기 위해, 컨테이너의 데이터를 영속적으로 활용할 수 있는 방법이 볼륨을 활용하는 것이다.

   위 명령어는, 호스트의 /home/test_db 디렉터리와 컨테이너의 /var/lib/mysql 디렉토리를 공유한다는 의미이다.

## 도커 이미지

도커는 Docker Hub 이라는 중앙 이미지 저장소에서 이미지를 내려받는다. 

대부분의 이미지는 도커 허브에서 공식적으로 제공하거나 (ex. ubuntu:14.04, centos:7) 다른 사람이 도커 허브에 올려놓은 경우가 대부분이다.

도커 허브에서 어떤 이미지가 있는지 확인하기 위해, 도커 허브 사이트를 직접 접속해서 찾아보거나 아래와 같이 도커 엔진에서 찾을 수 있다.

```bash
docker search ubuntu
```

1. 이미지 생성

   다음 명령어로 이미지로 만들 컨테이너를 생성하고, 기존의 이미지로부터 변경 사항을 만들자.

   ```bash
   docker run -i -i --name commit_test ubuntu:14.04
   echo test_first!! >> first
   ```

   그리고, 호스트로 빠져나와 아래 명령어로 컨테이너를 이미지로 만들자.

   commit_test 라는 컨테이너를 commit_test:first 라는 이름의 이미지로 생성한다.

   ```bash
   docker commit \
   -a 'test_author' -m 'commit_message' \
   commit_test \
   commit_test:first
   ```

2. 이미지 구조

   이미지를 커밋할 때, 컨테이너에서 변경된 사항만 새로운 레이어로 저장하고, 그 레이어를 새로운 이미지로 생성한다.

   

3. 이미지 추출

   ```bash
   docker save -o ubuntu_14.04.tar ubuntu:14.04
   ```

   도커 이미지를 별도로 저장하거나 옮기는 등 필요에 따라 이미지를 단일 바이너리 파일로 저장해야할 때 필요하다.

   -o 옵션에는 추출될 파일명을 입력한다.

## Dockerfile

개발한 애플리케이션을 컨테이너화 하려면,

1. 아무것도 존재하지 않는 이미지 (ubuntu, centos..) 로 컨테이너 생성
2. 애플리케이션을 위한 환경 설치하고, 소스코드 복사해서 정상 동작 확인
3. 컨테이너를 이미지로 commit

위 방법은, 애플리케이션이 동작하는 환경을 일일이 설치하고 소스코드를 git 에서 복제해야한다.

도커는 위와 같은 일련의 과정을 손쉽게 수행할 수 있도록 build 명령어를 제공한다.

이미지 생성을 위해 

- 컨테이너에서 설치해야하는 패키지
- 추가해야하는 소스코드
- 실행해야하는 명령어

를 하나의 파일(= Dockerfile) 에 기록해두면, 도커는 이 파일을 읽어 컨테이너에 작업을 수행한뒤 이미지로 만든다.

1. Dockerfile 작성

   ```dockerfile
   FROM ubuntu:14.04  // 생성할 이미지의 베이스가 될 이미지
   MAINTAINER jko // 이미지 생성한 개발자의 정보
   LABEL "perpose"="practice" // 이미지의 메타데이터
   RUN apt-get update // 이미지 만들기 위해 컨테이너 내부에서 명령어 실행
   RUN apt-get install apache2 -y
   ADD test.html /var/www/html // Dockerfile 이 위치한 디렉토리에서 test.html 파일을 이미지의 /var/www/html 디렉토리에 추가
   WORKDIR /var/www/html // == cd 명령어
   RUN ["/bin/bash", "-c", "echo hello >> test2.html"]
   EXPOSE 80 // 이미지에서 노출할 포트
   CMD apachectl -DFOREGROUND // 컨테이너가 시작될 때마다 실행할 명렁어
   ```

2. Dockerfile 빌드 

   ```bash
   docker build -t mybuild:0.0 ./
   ```

   -t 옵션은 생성될 이미지의 이름이고 끝에는 Dockerfile 이 저장된 경로를 입력한다.

   Dockerfile 에서 명령어 한줄이 실행될 때마다, 이전 step 에서 생성된 이미지에 의해 새로운 컨테이너가 생성된다. 

   그래서, 이미지의 빌드가 완료되면 Dockerfile 의 명령어 수 만큼 레이어가 존재하며, 중간에 컨테이너도 같은 수만큼 생성되고 삭제된다.

   ![](/image/docker-img-build.png)

## 도커 데몬

도커의 구조는 두 가지로 나뉜다.

1. 클라이언트로서의 도커

   도커 데몬은 API 입력을 받아 도커 엔진의 기능을 수행하는데, 이 API 를 사용할수 있도록 CLI 를 제공한다.

   

2. 서버로서의 도커

   실제로 컨테이너를 생성하고 이미지를 관리하는 주체이다. dockerd 프로세스로 동작한다.

따라서, 터미널에서 도커가 설치된 호스트에 접속해서 docker 명령어를 입력하면 아래와 같은 과정으로 도커가 제어된다.

1. 사용자가 docker ps 같은 명령어 입력

   

2. /usr/bin/docker 는 /var/run/docker.sock 유닉스 소켓을 사용해서 도커 데몬에게 명령어 전달

   

3. 도커 데몬은 이 명령어를 파싱하고 명령어에 해당하는 작업 수행

   

4. 수행 결과를 도커 클라이언트게 반환하고 사용자에게 결과 출력

---

시작하세요! 도커/쿠버네티스 <용찬호>
