---
layout: post
title: "Kubernetes Controller"
tags: [kubernetes]
comments: true
date: 2020-07-19
---

## Replication Controller

- 지정한 숫자만큼의 pod가 항상 클러스터 안에서 실행되도록 관리하는 컨트롤러
- 예를 들어 2개의 pod를 명시해 둔 Replication Controller가 있다면 장애나 다른 이유로 pod가 2개 보다 적을 때 다시 새로운 pod를 실행해서 pod개수를 2개로 맞춘다. 또한 pod가 2개보다 많아졌을 때도 pod 를 줄여서 2개만 실행되도록 조정한다.
- Replication Controller는 쿠버네티스 처음부터 있었지만 요즘은 비슷한 역할을 하는 ReplicaSet을 사용한다.앱을 배포하는 경우에는 Deployment를 주로 사용한다.

## ReplicaSet

- Replication Controller의 발전형이며 Replication Controller와 같은 동작을 하지만 집합기반(set-based)의 Selector를 지원하는 차이점이 있다.
- Replication Controller는 label을 선택할 때 등호기반(=, !=)만 지원하지만 ReplicaSet은 집합기반(in, notin, exists)연산자도 지원한다.
- ReplicaSet에서는 rolling-update 옵션을 사용할 수 없고, rolling-update옵션이 필요할 때는 Deployment를 사용해야 한다.
- Deployment는 ReplicaSet을 관리하고 pod에 대한 선언적 업데이트와 서버측 롤링 업데이트를 제공하는 상위 개념이다.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-replicaset
spec:
  template:
    metadata:
      name: nginx-replicaset
      labels:
        app: nginx-replicaset
    spec:
      containers:
      - name: nginx-replicaset
        image: nginx
        ports:
        - containerPort: 80
  replicas: 3
  selector:
    matchLabels:
      app: nginx-replicaset
```

- .metadata : ReplicaSet의 정보
- .sepc.template : ReplicaSet이 어떤 pod를 실행해야 하는지에 관한 정보를 설정
- .sepc.template.sepc.containers[] : 실행할 컨테이너의 구체적인 명세를 설정
- .spec.replicas : 몇개의 pod를 유지할지 개수를 설정(default: 1)
- .spec.selector : 어떤 레이블(.spec.template.metadata.labels)의 pod를 선택해서 관리할지 설정
    - 레이블을 기준으로 pod를 관리하므로 실행중인 pod를 재시작하지 않고 ReplicaSet이 관리하는 pod를 변경할 수 있다. ReplicaSet 설정에서 관리할 label을 변경한 후 kubectl apply -f ${yaml파일} 로 적용
    - 템플릿에 별도의 .spec.selector설정이 없으면 .spec.template.metadata.labels.app에 있는 내용을 기본값으로 설정한다.
- 실행 command : kubectl apply -f ${yaml파일명}
- 확인 command :  kubectl get replicaset
    - DESIRED : ReplicaSet에 지정한 pod 개수
    - CURRENT : ReplicaSet을 이용해 현재 클러스터에서 동작하는 실제 pod 개수

## Deployment

- Deployment는 쿠버네티스에서 상태가 없는(stateless) 앱을 배포할 때 사용하는 가장 기본적인 컨트롤러
- Deployment는 ReplicaSet을 관리하면서 앱 배포를 더 세밀하게 관리한다. 단순히 실행시켜야 할 pod개수를 유지하는 것뿐만 아니라 앱을 배포할 때 rolling-update 하거나, 앱 배포 도중 잠시 멈췄다가 다시 배포하거나, 앱 배포 후 이전 버전으로 롤백할 수 있는 기능을 제공한다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx-deployment
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 25%
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - name: nginx-deployment
        image: nginx
        ports:
        - containerPort: 80
```

- .spec.replicas :  pod를 몇개 실행할 것인지 설정
- .spec.selector : Deployment가 관리할 pod의 레이블(.spec.template.metadata.labels) 지정
- .spec.strategy : Deployment의 배포전략 설정
    - .spec.strategy.type 다른 값으로 Recreate 값이 있는데, Recreate는 기존 pod가 모두 삭제된 다음 새로운 pod를 생성하기 때문에 무중단 배포를 보장할 수 없다.
    - .spec.strategy.rollingUpdate.maxSurge : rolling update 중 정해진 Pod 수 이상으로 만들 수 있는 Pod의 최대 개수이고 기본값은 25%이다. 예를 들어 이 값을 30%로 설정하면 롤링업데이트 시작시 새로운 ReplicaSet의 크기를 즉시 조정해서 기존 및 새로운 Pod의 전체 갯수를 설정한 pod개수의 130%를 넘지 않도록 한다. 기존 pod가 죽으면 새로운 ReplicaSet은 scale-up 할 수 있으며, 업데이트하는 동안 실행하는 최대 pod의 수는 설정한 pod 개수의 130%가 되도록 보장한다
    - .spec.strategy.rollingUpdate.maxUnavailable : rolling update 중 unavailable 상태인 Pod의 최대 개수를 설정. rolling update 중 사용할 수 없는 Pod의 최대 개수이고 기본값은 25%이다. 예를 들어 이 값을 30%로 설정하면 롤링업데이트 시작시 즉시 이전 ReplicaSet의 크기를 설정한 pod 개수의 70%만큼 scale-down 할 수 있다. 새로운 pod가 준비되면 기존 ReplicaSet을 scale-down 할 수 있으며, 업데이트 중에 항상 사용 가능한 전체 pod의 수는 설정한 pod 개수의 70% 이상이 되도록 새로운 ReplicaSet을 scale-up 할 수 있다
    - 두개의 설정필드는 0보다 큰 정수를 통해 Pod의 절대 개수 설정이 가능하고, 25%와 같이 percentage 표현이 가능하다. maxSurge와 maxUnavailable 값이 동시에 0이 될 수 없다.
- 실행 command : kubectl apply -f ${yaml파일명}
- 확인 command : kubectl get deployment,replicaset,pod
    - Deployment, ReplicaSet, Pod를 모두 확인해보면 Deployment가 생성되어졌고, 이 Deployment가 관리하는 ReplicaSet이 생성되었고, ReplicaSet이 관리하는 Pod가 3개 생성되어져 있다.
- 설정 변경 command : kubectl edit deploy ${deplyment name}
    - vi 편집기가 실행되면 원하는 설정 변경 후 저장(wq)

## Deployment Pod 개수 조정

- kubectl scale deploy ${deployment name} \-\-replicas=${조정할 개수}
- kubectl get pods

## Deployment Rollback

- 컨테이너 이미지 변경 내역(REVISION) 확인
    - kubectl rollout history deploy ${deployment name}
- 특정 REVISION으로 롤백
    - kubectl rollout undo deploy ${deployment name} \-\-to-revision=${REVISION 숫자}

## Deployment 배포 정지/재개

- 배포 정지
    - kubectl rollout pause deploy ${deployment name}
- 이미지 변경
    - kubectl set image deploy/${deployment name} ${container name}=${변경할 이미지명}
    - ex) kubectl set image deploy/nginx-deployment nginx-deployment=nginx:1.11
- 배포 내역 확인 (배포정지상태이므로 배포 진행되지 않음)
    - kubectl rollout history deploy ${deployment name}
- 배포 재개
    - kubectl rollout resume deploy ${deployment name}

## DaemonSet

- DaemonSet는 클러스터 전체 노드에 특정 pod를 실행할 때 사용하는 컨트롤러
- 클러스터 안에 새롭게 노드가 추가되면 DaemonSet가 자동으로 해당 노드에 pod를 실행시킨다.
- 반대로 노드가 클러스터에서 빠졌을 때는 해당 노드에 있던 pod는 그대로 사라질 뿐 다른 곳으로 옮겨가서 실행되거나 하지 않는다.
- DaemonSet는 보통 로그 수집기를 실행하거나 노드를 모니터링하는 등, 클러스터 전체에 항상 실행시켜두어야 하는 pod에 사용한다.

## StatefulSet

- StatefulSet은 상태가 있는 pod들을 관리하는 컨트롤러
- ReplicationController, ReplicaSet, Deployment는 모두 상태가 없는 pod들을 관리하는 컨트롤러
- StatefulSet으로 생성한 pod는 각각 고유한 네트워크 식별자를 갖는다.
    - DNS 생성규칙 : \<pod name\>.\<service name\>.\<namespace\>.svc.\<cluster domain\>
- Volume을 사용해서 특정 데이터를 저장한 후 pod를 재시작했을 때 해당 데이터를 유지할 수 있으며 각 pod는  순서 및 고유성을 갖는다.
- StatefulSet이 유용한 애플리케이션
    - 고유한 네트워크 식별자가 필요한 경우
    - 지속성을 갖는 스토리지가 필요한 경우
    - 순차적인 배포와 스케일링이 필요한 경우
    - 순차적인 롤링업데이트가 필요한 경우
- 애플리케이션이 순차적인 배포와 스케일링이 필요하지 않으면 Deployment와 같은 stateless 컨트롤러가 더 적합할 수 있다.
- StatefulSet은 현재 pod의 네트워크 식별자를 책임지고 있는 Headless Service가 필요하며 이 서비스는 직접 생성해주어야 한다.
- Headless Service : 클러스터 IP를 할당하지 않고 kube-proxy가 처리하지 않으며 로드밸런싱 기능을 하지 않는다. 서비스 디스커버리를 위한 DNS용도로 쓰이는 경우에 생성한다. .sepc.clusterIP 필드에 "None"을 지정하여 헤드리스 서비스로 만들 수 있고, selector를 지정하여 엔드포인트를 위한 DNS설정을 할 수 있다.
- StatefulSet을 위한 스토리지(PersistantVolume)는 사전에 제공되어야 하며 StatefulSet을 삭제해도 연관된 볼륨이 삭제되지 않는다. StatefulSet과 연관된 리소스를 자동으로 제거하는 것보다 데이터의 안전을 보장하기 위해 이렇게 설계되었다. 즉, 관리자가 볼륨을 수동으로 관리해주어야 한다.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None # Headless Service
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  # podManagementPolicy: Parallel
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 3 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "my-storage-class"
      resources:
        requests:
          storage: 1Gi
```

- Service
    - StatefulSet에서 사용할 헤드리스 서비스 설정
    - StatefulSet의 .spec.serviceName 필드 값으로 설정할 서비스를 정의
- StatefulSet
    - [.](http://metadata.name)metadata.name : StatefulSet 이름 지정
    - .spec.podManagementPolicy
        - StatefulSet의 기본 동작은 순서대로 pod를 관리하는 것이지만 이 필드를 통해 순서를 없앨 수 있다.
        - 기본 값은 OrderedReady로 순서대로 관리하지만, Parallel로 변경하면 pod들이 순서 없이 병렬로 실행되거나 종료되도록 할 수 있다.
        - OrderedReady로 해서 실행할 경우 .metadata.name에 설정한 "web"이라는 이름으로 "web-0" , "web-1", "web-2" 을 가진 pod들이 순서대로 실행된다. "web-0"이 정상적으로 실행되지 않으면 "web-1"은 실행되지 않는다. 반대로 종료할 때는 반대로 "web-2"부터 종료된다.
        - .spec.podManagementPolicy: Parallel 을 설정하지 않으면 아래처럼 앞선 순서가 전부 running상태가 되어야 web-2 pod가 생성된다. web-0, web-1 pod가  Running 상태가 되고 web-2 pod가 실행되고 있는 것을 볼 수 있다.
        
        ![No image](/assets/posts/20200719/statefulset.png)

        - podManagementPolicy: Parallel 을 설정하면 아래처럼 이름은 0, 1, 2로 생성되지만 순서가 있지는 않고 동시에 pod가 생성되고 있는 것을 볼 수 있다. pod의 이름은 네트워크 식별자로 쓰인다.
        
        ![No image](/assets/posts/20200719/statefulset_parallel.png)

    - .sepc.template.seplc.terminationGracePeriodSeconds: gradeful의 대기시간을 설정
        - graceful 은 실행 중인 프로세스를 종료할 때 바로 종료하는 것이 아니라 하던 작업을 마무리하고 정상적으로 종료하는 것을 뜻함

## Job

- Job은 실행된 후 종료해야 하는 성격의 작업을 실행시킬 때 사용하는 컨트롤러이며, 특정 개수만큼의 pod를 정상적으로 실행 종료함을 보장
- Pod 실행 실패, 하드웨어 장애 발생, 노드 재시작 등 문제가 발생하면 다시 pod를 실행시킨다.
- Job 하나가 여러 개의 pod를 실행할 수도 있다.
- Job이 정상적으로 실행 종료되면 pod가 새로 생성되지도, 삭제되지도 않고 Job 또한 남아 있다.  pod나 Job이 남아있으면 로그를 확인할 수 있기 때문에 이를 이용해 필요한 분석을 할 수 있다. Job 삭제는 kubectl delete job 명령을 통해 사용자가 직접해야 하며 Job을 삭제하면 관련된 pod들도 같이 삭제된다.
- 메모리 사용량 제한 초과, Pod내에 컨테이너 프로세스가 0(정상)이 아닌 종료코드로 종료된 경우 등의 이유로 Job이 실패할 수 있다.
- 아래 예제에서는 job이 실행할 명령을 적어놓은 command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"] 부분에서 print라는 키워드 대신에 printa 라는 perl에 없는 키워드로 변경하여 실패하도록 했다.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: pi
spec:
  template:
    spec:
      containers:
      - name: pi
        image: perl
        command: ["perl", "-Mbignum=bpi", "-wle", "print bpi(2000)"]
      restartPolicy: Never
  backoffLimit: 4
  # completions: 2
```

- apiVersion : 배치작업을 실행하는 v1 버전 API를 사용
- .spec.template.spec.containers[].command : 단순히 원주율을 계산하는 명령 설정
- .spec.template.sepc.restartPolicy : 재시작 정책으로 Never, OnFailure 만 허용된다.
    - 만약 잡에 restartPolicy = "OnFailure" 가 있는 경우 잡 백오프 한계에 도달하면 잡을 실행 중인 컨테이너가 종료된다. 이로 인해 잡 실행 파일의 디버깅이 더 어려워질 수 있다. 디버깅하거나 로깅 시스템을 사용해서 실패한 작업의 결과를 실수로 손실되지 않도록 하려면 restartPolicy = "Never" 로 설정하는 것을 권장한다.
- .spec.backoffLimit : Job실행이 실패했을 때, 자동으로 최대 몇 번까지 재시작할 것인지 설정한다. 기본값은 6. 보통 Job컨트롤러가 재시작을 관리할 때마다 시간 간격을 늘린다. 처음 재시작 실패 후 10초기다린 후 시도하고 그 다음 20초, 그 다음 40초 이런 방식으로 계속 재시작 대기시간을 늘린다. 이렇게 재시작을 하다가 pod실행이 완료되면 재시도 횟수는 0으로 초기화 된다.
- restartPolicy=Never로 설정한 상태에서 Job 수행 실패 시, backoffLimit 횟수만큼 pod를 새로 생성해서 잡을 재수행한다. 잡 수행에 실패한 pod들은 STATUS=Error 상태로 남아 있고 kubectl logs 커맨드로 에러 로그를 확인할 수 있다.
- restartPolicy=OnFailure로 설정한 상태에서 Job 수행 실패 시, backoffLimit 횟수만큼 기존 pod를 재시작한다. backoffLimit 횟수만큼 실패하면 pod가 종료되어 버리기 때문에 log를 확인할 수 없다.

![No image](/assets/posts/20200719/job.png)

![No image](/assets/posts/20200719/job2.png)

- pod/pi1 은 단일 잡으로 수행한 pod 이고 정상적으로 종료된 케이스다.
- pod/pi3은 .spec.completions(완료된 개수 설정) 필드를 2로 설정하여 잡을 실행하는 pod를 2개 실행시킨 케이스다.
- pod/pi4 는 restartPolicy=Never, backoffLimit=4 로 설정한 상태에서 Job을 실패시킨 케이스인데, Job실패로 인해 backoffLimit 만큼 새로운 pod를 생성해서 Job수행을 시도한다. backoffLimit=4로 설정했기 때문에 최초시도 + 4번의 재시도로 총 5개의 pod가 생성된 것을 볼 수 있다.
- pod/pi6 은 restartPolicy=OnFailure,  backoffLimit=4로 설정한 상태에서 Job을 실패시킨 케이스인데, 현재 스크린샷은 재시작 횟수가 3일 때  찍어놓은 스냅샷 이긴한데, 저 pod가 한번 더 잡 수행시도를 하고 실패하게 되면 pod/pi6 은 아예 종료되어 남아있지 않게 된다. 다른 pod들 처럼 Completed, Error 상태로 남아 있지 않고 종료되어 없어진다. Job 컨트롤러(job/pi6)는 남아있고 pod만 없어진다.
