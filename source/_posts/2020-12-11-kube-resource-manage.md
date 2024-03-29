---
layout: post
title: "쿠버네티스 리소스 관리와 설정"
date: 2020-12-11
categories: Infrastructure
---

효율적으로 애플리케이션을 관리하기 위해 사용되는 다음 오브젝트를 정리한다.

- Namespace
- ConfigMap, Secret

## Namespace: 리소스를 논리적으로 구분

네임스페이스는 포드, 레플리카셋, 디플로이먼트, 서비스 등과 같은 쿠버네티스 리소드들이 묶여 있는 하나의 가상 공간 또는 그룹이다.
예를 들어, 

- 모니터링을 위한 리소스들은 monitoring 이라는 namespace 에, 
- 테스트를 위한 리소스들은 testbed 라는 namespace 에 생성할 수 있다.
- 또는, 여러 개발 조직이 하나의 쿠버네니스 클러스터를 공유해야한다면 조직별로 네임스페이스를 구성할 수도 있다.

### 기본 개념

네임스페이스는 논리적인 리소스 공간이므로 포드, 레플리카셋, 서비스 같은 리소스들이 따로 존재한다.
네임스페이스를 생성하지 않았어도, 기본적으로 다음 네 개의 네임스페이스가 존재한다.

1. default
2. kube-node-lease
3. kube-public
4. kube-system

kube-system 네임스페이스는 쿠버네티스 클러스터 구성에 필수적인 컴포넌트들과 설정 값등이 존재한다.
예를 들어, 쿠버네티스의 포드, 서비스등을 이름으로 찾을 수 있도록 하는 DNS 서버의 서비스가 생성되어 있다.

네임스페이스의 리소스들은 논리적으로 구분된 것일 뿐, 물리적으로 격리된 것은 아니다.
즉, 서로 다른 네임스페이스에 생성된 포드가 같은 노드에 존재할 수 있다.

### Namespace 사용

아래처럼 YAML 파일을 작성해서 네임스페이스를 생성할 수 있다.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: production
```

```bash
kubectl apply -f production-namespace.yaml
```

특정 네임스페이스에 리소스를 생성하는 방법은 아래와 같다.

```yaml
apiVersion: v1
kind: Deployment
metadata:
  name: hostname-deployment-ns
  namespace: production # 네임스페이스를 이렇게 명시한다.
```

```bash
kubectl apply -f hostname-deploy-svc-ns.yaml
```

### Namespace 의 서비스에 접근

쿠버네티스 클러스터 내부에서는, 서비스 이름을 통해 포드에 접근할 수 있다. 
정확히 말하면, 같은 네임스페이스 내의 서비스에 접근할 때, 서비스 이름만으로 접근할 수 있다.

다른 네임스페이스의 서비스에 접근하려면, 서비스 이름 뒤에 네임스페이스 이름을 붙이면 된다.
"<서비스 이름>.<네임스페이스 이름>.svc"  

### Namespace 에 종속되는 오브젝트, 종속되지 않는 오브젝트

모든 리소스가 네임스페이스에 의해 구분되지 않는다.

A 라는 네임스페이스에 포드를 만들면, A 네임스페이스에는 보이고 B 라는 네임스페이스에는 보이지 않을 것이다. 
이런 경우를, "오브젝트가 네임스페이스에 속한다" 라고 한다. (= namespaced)

아래 오브젝트들은 네임스페이스에 속하지 않는다.

- namespaces
- nodes
- ...

## Configmap, Secret: 설정 값을 포드에 전달

개발한 애플리케이션에는 설정값을 가지고 있다. 예를 들면, 

- LOG_LEVEL = INFO 와 같이 key-value 형태의 설정값을 사용할 수 있다.
- Nginx 웹 서버가 사용하는 nginx.conf 처럼 파일을 사용할 수도 있다.

이러한 설정값을 애플리케이션에 적용하는 방법으로, 도커 이미지 내부에 설정값 또는 파일을 정적으로 저장할 수 있다.
하지만, 도커 이미지는 빌드되면 불변의 상태이기 때문에 설정 옵션을 유연하게 변경할 수 없다.

다른 방법으로, 포드를 정의하는 YAML 파일에 환경 변수를 아래 처럼 직접 하드 코딩할 수 있다.
이 방법의 단점은, 운영 환경과 개발 환경에서 각각 디플로이먼트를 생성해야한다면 환경 변수만 다르게 설정된 두 버젼의 YAML 파일이 각각 필요하다는 것이다.

```yaml
...
  spec:
    containers:
    - name: nginx
      env:
      - name: LOG_LEVEL
        value: INFO
      image: nginx:1.10
...
```

쿠버네티스에서는, YAML 파일과 설정 값을 분리할 수 있는 Configmap 과 Secret 이라는 오브젝트를 제공한다.
컨피그맵에는 설정값을, 시크릿에는 노출되어서는 안되는 비밀값을 저장한다.
한 개의 포드 YAML 파일을 사용하고, 환경에 따라 다른 컨피그맵이나 시크릿을 생성해서 사용하면 된다.

---

시작하세요! 도커/쿠버네티스 <용찬호>
