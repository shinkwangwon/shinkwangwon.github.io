---
layout: post
title: "Kubernetes Pod"
tags: [kubernetes]
comments: true
date: 2020-07-05
---

# Kubernetes Pod
- 쿠버네티스는 pod라는 단위로 컨테이너를 묶어서 관리한다.
- pod안에 여러개의 컨테이너를 생성하고 관리할 때 컨테이너들은 IP하나를 공유해서 사용한다. pod안의 컨테이너와 통신할 때는 포트번호를 다르게 설정하여 통신한다.

```yaml
apiVersion: v1
kind: Pod   # 오브젝트 type
metadata:
  name: label-demo   # pod 이름
  labels:
    environment: production  # {key=environment, value=production} label 생성 
    app: MyApp
spec:
  containers:   # 컨테이너 리스트 정의. 하위에 "-"(하이픈)을 통해 컨테이너를 정의한다.
  - name: nginx   # 컨테이너 이름. 
    image: nginx:1.14.2  # 컨테이너에서 사용할 이미지 지정
    ports:
    - containerPort: 80  # 컨테이너에 접속할 포트 지정 
  - name: test  # 이렇게 "-"(하이픈)을 통해 또다른 컨테이너를 정의할 수 있다.
  #...
```

- .metadata.labels
    - 이 필드는 pod을 식별하는 label을 설정한다. label은 key&value로 이루어져 있고 service, delployment 와 같은 오브젝트에서 selector 설정을 통해 관리할 pod 을 지정할 때 사용한다.

    ```yaml
    ...
    kind: Service
    ...
    spec:
      selector:
        app: MyApp  # key="app"이고, value="MyApp"인 label을 가진 Pod를 이 서비스에 참여시킴
    ...
    ```

## Pod 생명주기
- command : kubectl get pods
- Pending / Running / Succeeded / Failed / Unknown
    - Pending : 쿠버네티스 시스템에 파드를 생성하는 중. 이 상태는 컨테이너 이미지를 다운로드한 후 전체 컨테이너를 실행하는 중임을 뜻함
    - Running : pod안의 모든 컨테이너들이 생성완료 되었고, 적어도 하나의 컨테이너가 동작중이거나 시작 또는 재시작 중에 있다.
    - Succeeded : pod안의 모든 컨테이너가 정상 종료된 상태로 재시작되지 않는다.
    - Failed : pod안의 모든 컨테이너가 종료되었고, 하나 이상의 컨테이너가 실패로 종료되었다.
    - Unknown : 어떤 이유에 의해 pod의 상태를 확인할 수 없는 상태이다.

## 컨테이너 상태
- kubectl describe pod ${pod_name}
- Waiting / Running / Terminated
    - Waiting : 컨테이너의 기본 상태. Waiting 상태의 컨테이너는 이미지를 pull, secret적용 등의 필요한 작업들을 수행중인 상태이다.
    - Running : 컨테이너가 이슈 없이 동작하고 있는 상태
    - Terminated : 컨테이너가 성공적으로든, 실패적으로든 종료된 상태.

## 컨테이너 Health Check 방식
- ExecAction
    - 컨테이너 안에 지정된 명령을 실행하고, 종료 코드가 0인 경우 Success로 판단
- TCPSocketAction
    - 컨테이너 안에 지정된 IP와 포트로 TCP 상태를 확인하고 포트가 열려 있으면 Success로 판단
- HTTPGetAction
    - 컨테이너 안에 지정된 IP, 포트, URL로 HTTP GET 요청을 보내고, 응답코드가 200~400사이면 Success라고 판단

## 컨테이너 Health Check
- kubelet
    - kubelet은 클러스터내의 모든 노드에서 실행되는 에이전트이다. Pod내의 컨테이너들에 대해 health check를 수행하여 컨테이너들의 실행을 관리한다.
- livenessProbe
    - 컨테이너가 동작 중 인지  확인한다. 이 체크에 실패하면 kubelet은 컨테이너를 종료시키고, 재시작 정책에 따라 컨테이너를 재시작한다. 컨테이너에 livenessProbe를 설정하지 않았다면 기본 상태 값은 Success 이다.
- readinessProbe
    - 컨테이너가 실행된 후 실제로 서비스 요청에 응답할 수 있는지를 체크한다. 이 체크에 실패하면 endpoint controller는 해당 pod에 연결된 모든 서비스에서 pod를 제외시켜 트래픽을 받지 않도록 한다.
    - readinessProbe가 실패한 경우 pod는 재시작되지는 않고 추후 정상화가 되면 다시 서비스에 투입되어 트래픽을 받는다.
    - 최초 readinessProbe를 하기 전까지의 기본 상태값은 Failure이다. readinessProbe를 지원하지 않는 컨테이너라면 기본 상태값은 Success이다.
    - readinessProbe를 지원하면 기본 상태값이 Failure이기 때문에 컨테이너가 실행된 다음 바로 서비스에 투입되서 트래픽을 받지 않는다. 트래픽을 받을 준비가 되었음을 확인한 후 트래픽을 받는다.
    - 애플리케이션이 외부 서비스에 의존하는 경우, 외부 서비스로 인해 일시적으로 트래픽을 처리할 수 없는 경우가 있다. 이런 경우에 해당 readinessProbe를 통해 컨테이너가 트래픽을 받지 않도록 서비스에서 제외시킬 수 있다.
    - livenessProbe를 통해 health check가 실패하는 경우, 재시작 정책에 따라 컨테이너가 재시작된다. 근데, 이때 livenessProbe health check 실패사유가 서비스가 불가능한 것이라면 재시작을 하더라도 또 컨테이너는 죽을 것이다. 이런 상황을 위해 서비스가 불가능한 경우에는 컨테이너를 재시작할 필요없이 트래픽을 받지 않도록 하기 위해 readinessProbe를 사용한다.
- startupProbe
    - 컨테이너 내의 어플리케이션이 시작됐는지를 체크한다. startupProbe가 설정된 경우, 성공할 때 까지 나머지 Probe들은 활성화 되지 않는다.  startupProbe 가 실패하면 kubelet은 컨테이너를 다운시키고 재시작 정책에 따라 처리된다. startupProbe 이 없는 경우 기본상태는 Success이다.
    - 컨테이너가 정상 동작하기까지 initialDelaySeconds + failureThreshold * periodSeconds 만큼의 시간을 벌 수 있다. 오래된 legacy 시스템으로 애플리케이션이 구동하는데 오래걸리는 서비스의 경우 startupProbe를 통해 컨테이너가 구동되기 까지의 시간을 벌 수 있다.

    ```yaml
    apiVersion: v1
    kind: Pod
    ...
    spec:
      containers:
      - name: liveness
        ...
        args:
        - /bin/sh
        - -c
        - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
        livenessProbe:
          exec:   # health check를 위해 ExecAction 방식 사용
            command:
            - cat
            - /tmp/healthy

    #   livenessProbe:   # HttpGetAction 방식
    #      httpGet:
    #        path: /healthz
    #        port: 8080
    #        httpHeaders:
    #        - name: Custom-Header
    #          value: Awesome

    #   readinessProbe:  # TCPSocketAction 방식
    #      tcpSocket:
    #        port: 8080
    ```
- probe 설정시 사용가능한 옵션들
    1. **initialDelaySeconds**: 컨테이너가 시작되고 나서 대기가 필요한 경우 필요. 설정한 초 만큼 대기한 이후에 liveness 와 readiness probe 가 수행됨. 디폴트 값은 0초. 최소값은 0초
    2. **periodSeconds**: probe 의 주기. 디폴트 값은 10초. 최소 값은 0초.
    3. **timeoutSeconds**: probe 의 타임아웃 값. 디폴트 값은 1초. 최소 값은 1초.
    4. **successThreshold**: probe 가 성공되어야 할 연속 성공 최소 임계치. 디폴트 값은 1. 최소 값은 1.
    5. **failureThreshold**: probe 가 실패 했을 때, 쿠버네티스는 이 수치만큼 시도함. 디폴트 값은 3. 최소 값은 1. 이 수치 만큼 시도해도 성공하지 못한다면, liveness는 컨테이너를 재시작하고 readiness는 Pod 을 unready 상태로 마킹함.
  - 참고 : https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/#configure-probes

## 재시작 정책
- 컨테이너 health check에 대해 실패한 경우, 비정상 종료된 경우에 따른 재시작 정책
- restartPolicy (default: Always)
    - Always : 컨테이너가 성공적으로든, 실패적으로든 종료되면 항상 재시작
    - OnFailure : 컨테이너가 실패적으로 종료된 경우 재시작
    - Never : 재시작하지 않음
- 참고 : [https://kubernetes.io/ko/docs/concepts/workloads/pods/pod-lifecycle/#상태-예제](https://kubernetes.io/ko/docs/concepts/workloads/pods/pod-lifecycle/#%EC%83%81%ED%83%9C-%EC%98%88%EC%A0%9C)

## 초기화 컨테이너
- 초기화 컨테이너는 앱 컨테이너가 실행되기 전 Pod를 초기화한다.
- 초기화 컨테이너는 여러 개 구성 가능하며 Pod 템플릿에 명시한 순서대로 초기화 컨테이너가 실행된다.
- 초기화 컨테이너가 모두 실행된 후에 앱 컨테이너가 실행된다.
- 초기화 컨테이너는 readinessProbe를 지원하지 않는다.
    - 한번 실행된 후 종료되기 때문에, 서비스할 준비가 되었는지 체크하는 readinessProbe Health Check가 필요없다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  initContainers:  # 초기화 컨테이너 정의 
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'sleep 2; echo init !!;']
  containers:   # 앱 컨테이너 정의
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
```

## Pod Resource
- 특정 노드에 사용량이 많은 pod가 모여있는 경우 pod들의 성능이 나빠질뿐더러 전체 클러스터의 자원 사용효율도 낮아진다. 어떤 노드에는 pod가 없어서 자원이 남고, 어떤 노드에는 pod가 많아서 자원이 부족한 현상이 발생할 수 있다.
- 쿠버네티스는 이런 상황을 막기 위해 Pod별로 사용할 Resource량을 직접 지정할 수 있다.
- requests 필드만 지정하고 limits 필드를 지정하지 않으면 해당 pod가 자신이 속해있는 노드의 모든 자원을 사용하게 될 수도 있다. 이런 경우, 같은 노드의 속해있는 다른 pod들의 자원 할당량과 겹치면서 out of memory 를 발생시킬 수도 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: MyApp
spec:
  containers:
  - name: db
    image: mysql
    resources:
      requests:   # 최소 Resource량 설정
        memory: "64Mi"
        cpu: "250m"
      limits:     # 최대 사용가능한 Resource량 설정
        memory: "128Mi"
        cpu: "500m"
```
- SI prefix
    - 1,000 (10^3) 배수가 기준
    - E, P, T, G, M, K
    - 현재 SI , IEC 표기가 뒤엉켜 쓰이고 있다. 정석인 표현으로 2^10을 나타내는 것은 명확하게 i 를 붙여 표기해야한다.
- IEC prefix
    - 1,024 (2^10) 배수가 기준
    - Ei, Pi, Ti, Gi, Mi, Ki
- ex) 128,974,848 Byte, 129M, 123Mi 세가지는 대략적으로 동일한 값을 나타낸다.


## Pod 환경변수 설정

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: MyApp
spec:
  containers:
  - name: simple-pod
    env:       # 환경변수 필드 정의 
    - name: TEST_ENV     # 사용할 환경변수 이름 설정
      value: "testvalue" # 문자열이나 숫자형식의 값 설정
    - name: POD_NAME
      valueFrom:  # 직접할당이 아닌 참조를 위한 설정
        fieldRef: # pod의 현재 설정 내용을 값으로 사용하겠다는 설정
          fieldPath: metadata.name  # 값을 참조하려는 위치를 지정
    - name: CPU_REQ
      valueFrom:
        resourceFieldRef:  # 컨테이너에 Resource 할당량에 대한 정보를 가져오는 설정 
          containerName: simple-pod  # 컨테이너 지정
          resource: requests.cpu     # 가져올 resource 지정
```

- 실행중인 pod의 환경설정을 바로 적용하는 것은 불가하다. 기존 pod를 삭제한 후 재실행 해야한다.
- command
    - kubectl get pods    # pod 리스트 확인
    - kubectl delete pod ${pod_name}    # 기존 pod 삭제
    - kubectl apply -f ${yaml 파일이름}    # pod 실행
    - kubectl exec -it ${pod_name} /bin/bash  # 컨테이너 접속
    - env  # 환경변수 확인 명령 실행

## Static Pod

- kube-apiserver를 통하지 않고 kubelet이 직접 실행하는 pod
- kubelet이 직접 관리하면서 이상이 생기면 재시작 한다.
- 보통 스태틱 pod는 kube-apiserver라던가 etcd 같은 시스템pod를 실행하는 용도로 사용한다.


## Pod-Container 상태 예시
1. 파드가 동작 중이고 하나의 컨테이너를 가지고 있다. 컨테이너는 성공으로 종료됐다.
    - 완료 이벤트를 기록한다.
    - 만약 `restartPolicy`가 :
        - 항상(Always)이면: 컨테이너는 재시작되고, 파드의 `phase`는 Running으로 유지된다.
        - 실패 시(OnFailure)이면: 파드의 `phase`는 Succeeded가 된다.
        - 절대 안 함(Never)이면: 파드의 `phase`는 Succeeded가 된다.
2. 파드가 동작 중이고 하나의 컨테이너를 가지고 있다. 컨테이너는 실패로 종료됐다.
    - 실패 이벤트를 기록한다.
    - 만약 `restartPolicy`가 :
        - 항상(Always)이면: 컨테이너는 재시작되고, 파드의 `phase`는 Running으로 유지된다.
        - 실패 시(OnFailure)이면: 컨테이너는 재시작되고, 파드의 `phase`는 Running으로 유지된다.
        - 절대 안 함(Never)이면: 파드의 `phase`는 Failed가 된다.
3. 파드가 동작 중이고 두 개의 컨테이너를 가지고 있다. 컨테이너 1이 실패로 종료됐다.
    - 실패 이벤트를 기록한다.
    - 만약 `restartPolicy`가 :
        - 항상(Always)이면: 컨테이너는 재시작되고, 파드의 `phase`는 Running으로 유지된다.
        - 실패 시(OnFailure)이면: 컨테이너는 재시작되고, 파드의 `phase`는 Running으로 유지된다.
        - 절대 안 함(Never)이면: 컨테이너는 재시작되지 않고, 파드의 `phase`는 Running으로 유지된다.
    - 만약 컨테이너 1이 동작 중이 아니고, 컨테이너 2가 종료됐다면 :
        - 실패 이벤트를 기록한다.
        - 만약 `restartPolicy`가 :
            - 항상(Always)이면: 컨테이너는 재시작되고, 파드의 `phase`는 Running으로 유지된다.
            - 실패 시(OnFailure)이면: 컨테이너는 재시작되고, 파드의 `phase`는 Running으로 유지된다.
            - 절대 안 함(Never)이면: 파드의 `phase`는 Failed가 된다.
4. 파드는 동작 중이고 하나의 컨테이너를 가지고 있다. 컨테이너의 메모리가 부족하다.
    - 컨테이너는 실패로 종료된다.
    - 메모리 부족(OOM) 이벤트를 기록한다.
    - 만약 `restartPolicy`가 :
        - 항상(Always)이면: 컨테이너는 재시작되고, 파드의 `phase`는 Running으로 유지된다.
        - 실패 시(OnFailure)이면: 컨테이너는 재시작되고, 파드의 `phase`는 Running으로 유지된다.
        - 절대 안 함(Never)이면: 로그 실패 이벤트가 발생되고, 파드의 `phase`는 Failed가 된다.
5. 파드 동작 중에, 디스크가 죽었다.
    - 모든 컨테이너들을 죽인다.
    - 적절한 이벤트를 기록한다.
    - 파드의 `phase`는 Failed가 된다.
    - 만약 컨트롤러로 실행되었다면, 파드는 어딘가에서 재생성된다.
6. 파드 동작 중에, 파드가 있는 노드가 세그먼티드 아웃되었다.
    - 노드 컨트롤러가 타임아웃을 기다린다.
    - 노드 컨트롤러가 파드의 `phase`를 Failed로 설정한다.
    - 만약 컨트롤러로 실행되었다면, 파드는 어딘가에서 재생성된다.

상태예시 참고 : [https://kubernetes.io/ko/docs/concepts/workloads/pods/pod-lifecycle/#상태-예제](https://kubernetes.io/ko/docs/concepts/workloads/pods/pod-lifecycle/#%EC%83%81%ED%83%9C-%EC%98%88%EC%A0%9C)
