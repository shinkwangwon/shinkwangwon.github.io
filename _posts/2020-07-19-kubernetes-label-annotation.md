---
layout: post
title: "Kubernetes Label & Annotation"
tags: [kubernetes]
comments: true
date: 2020-07-19
---


## Label

- Label은 key & Value 쌍으로 구성하며, 사용자가 클러스터 안에 오브젝트를 만들 때 메타데이터로 설정
- 레이블의 key는 쿠버네티스 안에서 컨트롤러들이 pod를 관리할 때 자신이 관리해야 할 pod를 구분하는 역할을 한다. 쿠버네티스는 레이블만으로 관리 대상을 구분하기 때문에 특정 컨트롤러가 만든 pod라도 레이블을 변경하면 인식할 수 없다.
- 컨트롤러는 selector를 이용해 관리할 pod의 label을 설정한다.
- Label 규칙
    - 63글자 이하
    - 시작과 끝 문자는 알파벳 대소문자 및 숫자([a-z0-9A-Z])
    - 중간에는 대시(-), 언더바(_), 점(.), 등이 가능

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-label01
  labels:
    app: nginx
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
        environment: develop
        release: beta
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: label-develop-service
spec:
  type: ClusterIP
  selector:
    environment: develop
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

- Deployment
    - .spec.teamplate.metadata.labels : Pod의 Label 설정
    - .spec.selector.matchLabels : Deployment가 관리할 pod Label 설정
- Service
    - .spec.selector : Service에 연결할 Pod 들의 Label 설정

## Annotation

- 레이블과 마찬가지로 Key & Value로 구성
- 레이블이 사용자가 설정한 특정 레이블의 오브젝트를 선택하는 역할을 한다면, Annotation은 쿠버네티스 시스템이 필요한 정보들을 담았으며 쿠버네티스 클라이언트나 라이브러리가 자원을 관리하는데 활용한다.
- 그래서 애너테이션의 key는 쿠버네티스 시스템이 인식할 수 있는 값을 사용하거나 그외 사용자에게 필요한 정보들을 메모하는 용도로도 사용할 수 있다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: annotation
  labels:
    app: nginx
  annotations:
    # nginx.ingress.kubernetes.io/rewrite-target: /
    manager: "myadmin"
    contact: "010-0000-0000"
    release-version: "v1.0"
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
```

- .metadata.annotations : 사용자 메모 용도로 사용할 annotation을 적용했다.
    - nginx.ingress.kubernetes.io/rewrite-target: /  와 같이 ingress를 설정할 때 사용하는 rewriter-target처럼 쿠버네티스 시스템이 인식할 수 있는 값을 사용할 수도 있다
- kubectl describe deploy ${deployment name} 명령시 annotation 확인이 가능
