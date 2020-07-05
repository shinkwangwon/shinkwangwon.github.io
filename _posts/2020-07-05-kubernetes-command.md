---
layout: post
title: "Kubernetes command"
tags: [kubernetes]
comments: true
date: 2020-07-05
---

# kubectl 기본 사용법

## kubectl [command] [TYPE] [NAME] [flags]
- command : 자원에 실행하려는 동작 - create, get, delete 등
- TYPE : 자원 타입 - pod, service, ingress 등
- NAME : 자원 이름
- FLAG : 부가적으로 설정할 옵션

## 자원 확인
- pod, service, deployment 등 확인 : kubectl get { pods \| services \| deployments }
- pod, service 등 동시에 확인 : kubectl get pods,services
- pod, service 등 설정 내용 확인
    - yaml 포맷으로 확인 : kubectl get pods ${pod_name} -o yaml
    - json 포맷으로 확인 : kubectl get pods ${pod_name} -o json
- pod, service 등 상태 확인
    - kubectl describe pods ${pod_name}
- pod 조회 속성
    ![No image](/assets/posts/20200705/pod-command.png)
    - NAME : pod 이름
    - READY : pod의 준비상태. 준비완료된 컨테이너 개수/생성된 컨테이너 개수
    - STATUS: Running, Terminating, ContainerCreating 등
    - RESTARTS : 해당 pod가 몇번 재시작했는지
    - AGE : pod를 생성한 후 얼마나 시간이 지났는지
- service 조회 속성
    ![No image](/assets/posts/20200705/service-command.png)

    - NAME : 서비스 이름
    - TYPE : 서비스 타입. LoadBalancer, ClusterIP 등
    - CLUSTER-IP : 현재 쿠버네티스 클러스터 내부에서 사용되는 IP
    - EXTERNAL-IP : 클러스터 외부에서 접속가능한 IP
    - PORT : 해당 서비스로 접속하기 위한 포트. 쿠버네티스 내부의 80포트가 33543이라는 외부 포트와 연결되었다는 뜻
    - AGE : 서비스를 생성한 후 얼마나 시간이 지났는지
- deployment 조회 속성
    ![No image](/assets/posts/20200705/deployment-command.png)

    - NAME : deployment 이름
    - READY : 사용자가 최종 배포한 pod개수/실제로 동작시킨 pod 개수
    - UP-TO-DATE : deployment 설정에 정의한대로 동작중인 신규 pod 개수
    - AVAILABLE : 서비스 가능한 pod 개수. pod 실행 후 설정된 health check로 서비스 가능한 상태라고 판단하면 AVAILABLE 항목에 pod 개수에 포함

## 자원 삭제

- kubetctl delete pod ${pod name}
- kubectl delete service ${service_name}

## 컨테이너 실행
- kubectl run ${deployment_name} \-\-image ${image_name} \-\-port=${port_num}
    - kubectl run으로 pod를 실행시킬 때 기본 컨트롤러는 deployment이다.
- kubectl { apply \| create } -f ${yaml template file}
    - yaml 템플릿 파일을 먼저 생성한 후 템플릿을 통한 컨테이너 실행

## Context

- kubectl config current-context
    - 현재 context 확인
- kubectl config get-contexts ${context_name}
    - context_name에 해당하는 컨텍스트 정보 확인
    - 결과에 namespace가 비어있다면 해당 context의 기본 네임스페이스는 쿠버네티스에서 제공하는 "default" namespace를 사용하고 있다는 뜻
- kubectl config set-context ${context_name} \-\-namespace=kube-system
    - context_name에 해당하는 컨텍스트의 기본 네임스페이스를 kube-system 네임스페이스로 변경하는 예시
