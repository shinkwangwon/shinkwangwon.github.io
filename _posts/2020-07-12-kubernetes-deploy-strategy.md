---
layout: post
title: "Kubernetes 배포전략(Deploy Strategy)"
tags: [kubernetes]
comments: true
date: 2020-07-12
---

# 쿠버네티스 배포 전략(Deploy Strategy)

## Scale Up/Down

- Deployment의 Pod 개수를 늘리거나 줄여서 서비스 트래픽에 대응할 수 있다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: http-server-deploy-${USER}
  labels:
    app.kubernetes.io/instance: http-server-${USER}
spec:
  replicas: 2
  selector:
    matchLabels:
      app.kubernetes.io/instance: http-server-${USER}
  template:
    metadata:
      labels:
        app.kubernetes.io/instance: http-server-${USER}
    spec:
      containers:
      - image: #image url
        name: my-pod
        args:
        - --port=80
        ports:
        - containerPort: 80
          protocol: TCP
```

- Deployment 오브젝트를 생성하면 쿠버네티스의 deployment controller가 replicaset을 생성하고, 이를 통해 pod들이 생성된다.
- 위에서 replicas를 2로 설정했기 때문에 pod이 2개 구동된 상태인데, 아래 명령을 통해 구동중인 Deployment의 pod 개수를 변경할 수 있다.
    - kubectl scale \-\-replicas 3 deploy ${deployment_name}
    - kubectl scale \-\-replicas 1 deploy ${deployment_name}
- kubectl scale 명령은 Deployment 뿐만아니라 ReplicaSet, Replication Controller, StatefulSet에도 사용이 가능하다.
- kubectl delete 명령으로 Deployment를 삭제하면, 함께 생성된 ReplicaSet과 Pod들이 모두 삭제된다.

## Rolling Update

- 쿠버네티스에서 동작하고 있는 Pod를 하나씩 늘려나가고, 기존 버전을 하나씩 줄여가는 방식이다. 내부적으로 Deployment controller가 자동으로 replicationset을 하나 더 추가해서 Pod들을 차례로 교체한다.
- 기존 pod, 새로운 pod를 하나씩 교체하는 방식이기 때문에 무중단 배포가 가능하다. 다만, 구버전과 새버전이 공존하는 시간이 발생하여 두 버전으로 트래픽을 받는다는 단점이 있다.
    - 만약 Rest Controller의 Request Parameter가 변경되었다면, 구버전/신버전 중 한 곳에서는 요청실패가 발생할 수 있다.
- Deployment 오브젝트의 .spec.strategy.type 속성을 통해 설정이 가능하고 기본값이 RollingUpdate이다.
- .spec.strategy.type 다른 값으로 Recreate 값이 있는데, Recreate는 기존 pod가 모두 삭제된 다음 새로운 pod를 생성하기 때문에 무중단 배포를 보장할 수 없다.

![No Image](/assets/posts/20200712/rolling-update.png)

## Blue-Green

- Blue-Green 배포는 기존버전의 Deployment와 새버전의 Deployment를 구동시킨 후,  Service 오브젝트의 설정변경을 통해 두개의 Deployment간 부하를 조절하여 새로운 Deployment로 교체하는 방식이다.
- 새로운 Deployment를 구동한 후, 한 번에 부하를 전달할 수 있고 기존 Deployment와 부하를 조절하면서 점차 부하를 전달할 수도 있다.
- 따라서 Rolling Update와 마찬가지로 구버전과 새버전의 Deployment가 존재하지만 부하를 한번에 전달할지, 점차적으로 전달할지 관리자가 컨트롤할 수 있기 때문에 RollingUpdate의 단점을 커버하는 방식이다.

![No Image](/assets/posts/20200712/blue-green.png)
