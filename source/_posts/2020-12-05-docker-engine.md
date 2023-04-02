---
layout: post
title: 도커 엔진
date: 2020-12-05
categories: Container
---

## 도커 이미지와 컨테이너

도커 엔진에서 사용하는 기본 단위와 핵심은 이미지와 컨테이너이다.

### 도커 이미지

이미지는 컨테이너를 생성할 때 필요한 요소이다. 가상 머신을 생성할 때 사용되는 iso 파일과 비슷하다.
이미지는 여러 개의 계층으로 된 바이너리 파일로 존재하고, 컨테이너를 생성하고 실행할 때 읽기 전용으로 사용된다.

이미지 이름은, 다음과 같은 형태로 구성된다 : [저장소 이름]/[이미지 이름]:[태그]
ex) jko/ubuntu:14.04

### 도커 컨테이너

![](/image/docker-container.png)

이미지로 컨테이너를 생성하면, 이미지 목적에 맞는 파일이 들어 있는 파일 시스템과, 격리된 시스템 자원 및 네트워크를 사용할 수 있는 독립된 공간이 생성된다.

컨테이너는 이미지를 읽기 전용으로 사용하되 이미지에서 변경된 사항만 컨테이너 계층에 저장하기때문에, 컨테이너에 무엇을 하든지 원래 이미지는 영향을 받지 않는다.
또한, 컨테이너는 호스트와 분리되어있기 때문에, 어떤 애플리케이션을 설치하거나 삭제해도 다른 컨테이너와 호스트는 변화가 없다.

예를 들어,

1. 우분투 도커 이미지로 두 개의 컨테이를 생성한 뒤
2. A 컨테이너에 MySQL 을, B 컨테이너에 아파치 웹 서버를 설치해도

각 컨테이너는 서로 영향을 주지 않고 호스트에도 아무런 영향을 주지 않는다.

## 도커 이미지

- 데비안 운영체제에서 apt-get install 을 실행하면, apt repository 에서 패키지를 내려받고
- 레드헷 운영체제에서 yum install 을 실행하면, yum repository 에서 패키지를 내려받듯이

도커는 Docker Hub 이라는 중앙 이미지 저장소에서 이미지를 내려받는다.

### 도커 이미지 생성

docker search 를 통해 검색한 이미지를 pull 명령어로 내려받아 사용할 수 있다.
하지만, 컨테이너에 애플리케이션을 위한 특정 개발 환경을 직접 구축한 뒤 사용자만의 이미지를 직접 생성해야하는 경우가 있다.
이를 위해, 컨테이너 안에서 작업한 내용을 이미지로 만드는 방법을 먼저 정리하자.

다음 명령어를 입력해, 이미지로 만들 컨테이너를 생성하고, 컨테이너 내부에 first 라는 이름의 파일을 하나 생성해서 기존의 이미지로부터 변경 사항을 만들자.

```shell
# docker run -i -t --name commit_test ubuntu:14.04
root@acc147398731924:/# echo test_first!! >> first
```

그리고, 호스트로 빠져나와 아래 명령어로 컨테이너를 이미지로 만들자.
commit_test 라는 컨테이너를 commit_test:first 라는 이름의 이미지로 생성한다.

```shell
# docker commit \
-a 'author' -m 'commit_message' \
commit_test \
commit_test:first
```

이번에는, commit_test:first 이미지로 컨테이너를 생성한 뒤 second 라는 파일을 추가해서 commit_test:second 라는 이미지를 생성하자.

```shell
# docker run -i -t --name commit_test2 commit_test:first
root@9fdcers32:/# echo test_second!! >> second

# docker commit \
-a 'author' -m 'commit_message' \
commit_test2 \
commit_test:second
```

### 도커 이미지 구조

다음 명령어로 이미지의 자세한 정보를 확인하자.

```shell
# docker inspect ubuntu:14.04
# docker inspect commit_test:first
# docker inspect commit_test:second
```

Layers 항목을 확인해보면 다음과 같다.

![](/image/docker-image-inspect-test.png)

이미지를 커밋할 때 컨테이너에서 변경된 사항만 새로운 레이어로 저장하고, 그 레이어를 포함해 새로운 이미지를 생성한다.

이번에는 생성한 이미지를 삭제해보자.

```shell
# docker rmi commit_test:first
Untagged: commit_test:first
```

commit_test:first 이미지를 삭제했다고 해서 실제로 해당 이미지의 레이어 파일이 삭제되지는 않는다.
commit_test:first 이미지를 기반으로 하는 commit_test:second 이미지가 존재하기 때문이다.
그래서, 이미지 파일을 삭제하지 않고 레이어에 부여된 이름만 삭제한다.
"Untagged..." 라는 출력 결과가 그 의미이다.

## Dockerfile

개발한 애플리케이션을 컨테이너화 하려면,

1. 아무것도 존재하지 않는 이미지 (ubuntu, centos..) 로 컨테이너 생성
2. 애플리케이션을 위한 환경 설치하고, 소스코드 복사해서 정상 동작 확인
3. 컨테이너를 이미지로 commit

위 방법은, 애플리케이션이 동작하는 환경을 일일이 설치하고 소스코드를 git 에서 복제해야한다.

도커는 위와 같은 일련의 과정을 손쉽게 수행할 수 있도록 build 명령어를 제공한다.
이미지 생성을 위해,

- 컨테이너에서 설치해야하는 패키지
- 추가해야하는 소스코드
- 실행해야하는 명령어와 shell script

를 하나의 파일(= Dockerfile) 에 기록해두면, 도커는 이 파일을 읽어 컨테이너에 작업을 수행한뒤 이미지로 만든다.

Dockerfile 의 명령어 한 줄이 실행될 때마다, 이전 step 에서 생성된 이미지에 의해 새로운 컨테이너가 생성된다.
그래서, 이미지의 빌드가 완료되면 Dockerfile 의 명령어 수 만큼 레이어가 존재하며, 중간에 컨테이너도 같은 수만큼 생성되고 삭제된다.

![](/image/docker-img-build.png)

## 도커 데몬

터미널에서 도커가 설치된 호스트에 접속해서 docker 명령어를 입력하면 아래 과정으로 도커가 제어된다.

1. 사용자가 docker ps 같은 명령어 입력
2. /usr/bin/docker 는 /var/run/docker.sock 유닉스 소켓을 사용해서 도커 데몬에게 명령어 전달
3. 도커 데몬은 이 명령어를 파싱하고 명령어에 해당하는 작업 수행
4. 수행 결과를 도커 클라이언트게 반환하고 사용자에게 결과 출력

이는, 도커의 구조는 두 가지로 나뉘기 때문이다: 도커 클라이언트 / 도커 데몬

### 도커 클라이어트

도커 데몬은 API 입력을 받아 도커 엔진의 기능을 수행하는데, 이 API 를 사용할수 있도록 CLI 를 제공하는 것이 도커 클라이언트이다.
사용자가 docker 로 시작하는 명령어를 입력하면 도커 클라이언트를 사용하는 것이다.
도커 클아이언트는 입력된 명령어를 로컬에 존재하는 도커 데몬에게 API 로 전달한다.
이 때, 도커 클라이언트는 /var/run/docker.sock 에 위치한 유닉스 소켓을 통해 도커 데몬의 API 를 호출한다.

### 도커 데몬

실제로 컨테이너를 생성하고 이미지를 관리하는 주체이다. dockerd 프로세스로 동작한다.
외부에서 API 입력을 받아 도커 엔진의 기능을 수행한다.
도커 프로세스가 서버로서 입력을 받을 준비가 된 상태를 도커 데몬이라고 부른다.

---

시작하세요! 도커/쿠버네티스 <용찬호>
