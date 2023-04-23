---
layout: post
title: "[도커/쿠버네티스] 7장_쿠버네티스 리소스 관리와 설정"
date: 2020-12-11
categories: K8S
---

효율적으로 애플리케이션을 관리하기 위해 사용되는
Namespace / ConfigMap / Secret 오브젝트를 정리한다.

## Namespace

네임스페이스는 포드, 레플리카셋, 디플로이먼트, 서비스 등과 같은 쿠버네티스 리소드들이 묶여 있는
하나의 가상 공간 또는 그룹이다.
하지만, 네임스페이스의 리소스들은 논리적으로 구분된 것일 뿐, 물리적으로 격리된 것은 아니다.
즉, 서로 다른 네임스페이스에 생성된 포드가 같은 노드에 존재할 수 있다.

> 리눅스의 네임스페이스와는 완전히 다르다.
> 리눅스의 네임스페이스는, 리눅스 커널 자체 기능

### Namespace 사용하기

아래와 같이 production-namespace.yaml 파일을 정의하자.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

그리고 네임스페이스를 생성하자.

```bash
kubectl apply -f production-namespace.yaml
```

특정 네임스페이스에 리소스를 생성하려면
hostname-deploy-svc-ns.yaml 파일에 아래와 같이 작성하고,

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hostname-deployment-ns
  namespace: production
spec:
  replicas: 1
  selector:
    matchLabels:
      app: webserver
  template:
    metadata:
      name: my-webserver
      labels:
        app: webserver
    spec:
      containers:
      - name: my-webserver
        image: alicek106/rr-test:echo-hostname
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: hostname-svc-clusterip-ns
  namespace: production
spec:
  ports:
    - name: web-port
      port: 8080
      targetPort: 80
  selector:
    app: webserver
  type: ClusterIP
```

아래와 같이 생성하자.

```bash
kubectl apply -f hostname-deploy-svc-ns.yaml
```

### Namespace 에 종속되는 오브젝트, 종속되지 않는 오브젝트

A 라는 네임스페이스에 포드를 만들면, A 네임스페이스에는 보이고 B 라는 네임스페이스에는 보이지 않을 것이다. 이런 경우를, 오브젝트가 네임스페이스에 속한다라고 한다. (namespaced)
하지만, 모든 오브젝트가 쿠버네티스의 네임스페이스에 의해 구분되는 것은 아니다.
예를 들면, 노드 오브젝트의 경우에 네임스페이스에 의해 구분되지 않는다.
아래 명령어로도 확인할 수 있다.

```bash
kubectl api-resources --namespaced=false
```

## ConfigMap, Sectret

개발한 애플리케이션에는 설정값을 가지고 있다.
예를 들면, LOG_LEVEL = INFO 와 같이 key-value 형태의 설정값을 사용할 수 있다.
Nginx 웹 서버가 사용하는 nginx.conf 처럼 파일을 사용할 수도 있다.
이러한 설정값을 애플리케이션에 적용하는 방법에는 뭐가 있을까 ?

1. 도커 이미지 내부에 설정값 또는 파일을 정적으로 저장
   도커 이미지는 빌드되면 불변의 상태이다. 그래서, 설정 옵션을 유연하게 변경할 수 없다.

2. 포드를 정의하는 YAML 파일에 환경 변수 설정

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: env-hard-coding-deployment
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: my-nginx
     template:
       metadata:
         name: my-nginx-pod
         labels:
           app: my-nginx
       spec:
         containers:
         - name: nginx
           env:
           - name: LOG_LEVEL # 이렇게 환경 변수를 설정
             value: INFO
           image: nginx:1.10
           ports:
           - containerPort: 80
   ```

   이렇게 하면, 운영 환경과 개발 환경에서 각각 디플로이먼트를 생성해야한다면,
   환경 변수만 다르게 설정된 두 버젼의 YAML 파일이 필요하다.
   
3. ConfigMap, Secret 오브젝트 사용
   컨피그맵에는 설정값을, 시크릿에는 노출되어서는 안되는 비밀값을 저장한다.
   한 개의 포드 YAML 파일을 사용하고,
   환경에 따라 다른 컨피그맵이나 시크릿을 생성해서 사용하면 된다.

### ConfigMap

컨피그맵은 다음 처럼 생성할 수 있다.

```bash
kubectl create log-level-configmap --from-literal LOG_LEVEL=DEBUG
kubectl create start-k8s --from-literal k8s=kubuernetes --from-literal container=docker
```

생성한 컨피그맵의 값을 포드로 가져올 때는 두 가지 방법이 있다.

1. 컨패그맵의 값을, 컨테이너의 환경 변수로 가져오기
   app-env-from-configmap.yaml 을 작성하자.

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: container-env-example
   spec:
     containers:
       - name: my-container
         image: busybox
         args: ['tail', '-f', '/dev/null']
         envFrom:
         - configMapRef:
             name: log-level-configmap # 한 개의 key-value 가 포드의 환경 변수로 등록
         - configMapRef:
             name: start-k8s # 두 개의 key-value 가 환경 포드의 환경 변수로 등록
   ```

   포드를 생성하자.

   ```bash
   kubectl apply -f app-env-from-configmap.yaml
   ```

   그리고 포드 내부에서 환경 변수를 출력하면, 설정된 세 개의 환경 변수가 조회된다.

2. 컨패그맵의 값을, 포드 내부의 파일로 마운트
   애플리케이션이 nginx.conf 같은 특정 파일로부터 설정값을 읽어온다면,
   컨피그맵의 데이터를 포드 내부의 파일로 마운트해서 사용할 수 있다.
   예를 들어,
   start-k8s 컨피그맵에 존재하는 모든 key-value 쌍을 /etc/config 디렉토리에 위치시킨다.

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: configmap-volume-pod
   spec:
     containers:
       - name: my-container
         image: busybox
         args: [ "tail", "-f", "/dev/null" ]
         volumeMounts:
         - name: configmap-volume # volumes에서 정의한 컨피그맵 볼륨 이름 
           mountPath: /etc/config # 컨피그맵의 데이터가 위치할 경로
   
     volumes:
       - name: configmap-volume   # 컨피그맵 볼륨 이름
         configMap:
           name: start-k8s
   ```

#### 파일로부터 ConfigMap 생성

위와 같이, 컨피그맵을 볼륨 파일로 포드 내부에 제공할 수 있다.
그런데, 어떻게 nginx.conf 내용을 아예 통째로 컨피그맵으로 저장할 수 있을까 ?
파일로부터 컨피그맵을 생성하려면, --from-file 옵션을 사용하면 된다. (위에서는 --frome-lietral 사용)

```bash
kubectl create configmap ngin-config-map --from-file nginx.conf
```

#### YAML 파일로부터 ConfigMap 생성

```bash
kubectl apploy -f my-configmap-yaml
```

### Secret

SSH 키, 비밀번호 등과 같은 민감 정보를 저장하기 위한 용도로 사용된다.
컨피그맵처럼 --from-literal, --from-file 을 사용할 수 있다.

```bash
kubectl create secret generic \
my-password --from-literal password=1234
```

generic 은 시크릿의 종류이다. 시크릿 종류는 사용목적에 따라 나뉜다.
그리고, 쿠버네티스는 기본적으로 key-value 의 value 를 base64 로 인코딩해서 저장한다.

---

시작하세요! 도커/쿠버네티스 <용찬호>
