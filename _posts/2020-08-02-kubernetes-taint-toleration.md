---
layout: post
title: "Kubernetes Taint & Toleration"
tags: [kubernetes]
comments: true
date: 2020-08-02
---

## Taint & Toleration

- 쿠버네티스에서의 스케쥴링이란 어떠한 노드에서 pod가 실행되는 것을 말한다. 즉 pod가 스케쥴링된다는 것은 어떤 노드에 pod를 배치(실행)할 것인가를 의미한다.
- 쿠버네티스에서 특정 노드에 테인트(taint)를 설정할 수 있는데, 테인트를 설정한 노드에는 pod들이 스케쥴링 되지 않는다. 이런 노드에 특정 pod들만 실행되길 원할 때 톨러레이션(toleration)을 사용한다.
- 테인트는 노드의 자물쇠, 톨러레이션은 노드의 자물쇠를 열 수 있는 pod의 열쇠라고 보면 된다.
- 주로 특정 노드에 지정된 pod들만 수행되도록 할때 Taint & Toleration을 사용한다. 한마디로 노드가 특정 역할만 수행하도록 할때 사용한다.

## Taint

- 테인트는 key, value, effect 의 세가지 속성이 있고, 이 속성을 통해 테인트를 생성한다.
- effect
    - NoSchedule : 톨러레이션이 없으면 pod가 실행되지 않는다. 이미 실행된 pod에는 적용되지 않는다.
    - PreferNoSchedule : 톨러레이션이 없으면 pod가 실행하지 않으려고 하지만, 클러스터내의 자원이 부족한 경우에는 PreferNoSchedule 효과가 걸려있는 taint가 있더라고 pod가 실행될 수 있다.
    - NoExecute : 톨러레이션이 없으면 pod를 실행하지 않고, NoSchedule와 다르게 이미 실행된 pod도 테인트에 맞는 톨러레이션 설정이 없으면 종료시킨다.

```yaml
# 테인트 설정방식) kubectl taint nodes ${node name} ${설정할 key값}=${설정할 value값}:${설정할 effect 값}

# 노드에 테인트 설정
$ kubectl taint nodes docker-for-desktop key01=value01:NoSchedule

# 노드에 테인트 확인
$ kubectl describe nodes docker-for-desktop
...
Taints:             key01=value01:NoSchedule
...

# pod 생성 요청
$ kubectl create -f deployment-sample.yaml

# pod 생성 확인 => 테인트로 인해 pod가 생성되지 않고 pending 상태로 남아 있음
# 예제에서는 node가 1개 뿐이라고 가정. 만약 노드가 여러개면 taint가 설정되지 않은 노드에 pod가 스케쥴링 됨
$ kubectl get pods
NAME                                     READY     STATUS    RESTARTS   AGE
kubernetes-simple-app-57585656fc-vbt8x   0/1       Pending   0          53s
```

## Toleration

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    env: test
spec:
  containers:
  - name: nginx
    image: nginx
    imagePullPolicy: IfNotPresent
  tolerations:
  - key: "key01"
    operator: "Exists"
    effect: "NoSchedule"
  # tolerations:
  # - key: "key01"
  #   operator: "Equal"
  #   value: "value01"
  #   effect: "NoExecute"
  #   tolerationSeconds: 200
```

- operator
    - Equal : 테인트에 설정한 key, value, effect 와 모두 같은지 확인
    - Exists : 테인트에 설정한 key, effect가 일치하는지 확인. value를 명시하면 에러남. key, effect 값 마저 주지 않을 경우 모든 key 와 effect에 적용되서 어떤 테인트가 걸려 있던 상관없이 pod가 실행됨
- tolerationSeconds
    - 테인트의 effect가 NoExecute이고, pod에 해당 테인트와 일치하는 톨러레이션이 있다고 가정하자.
    - 이때 pod 톨러레이션 스펙에 tolerationSeconds 가 함께 설정된 경우, 테인트에 일치하는 톨러레이션이 있지만 tolerationSeconds 만큼만 노드에  남아있다가 종료된다.