---
layout: post
title: "쿠버네티스(Kubernetes)"
tags: [kubernetes]
comments: true
date: 2020-07-01
---

## Docker Orchestration
- 여러 호스트에서 다수의 어플리케이션을 운영/관리할 수 있게 해주는 기술이다.
- Scheduling
    - 여러 호스트에게 컨테이너를 분배하고 컨테이너나 호스트의 장애시 재분배 한다.
- Networking
    - 여러 호스트에 분산된 컨테이너 간의 네트워크와 L4/L7 등을 지원한다.
- Logging
    - 동적으로 분산되어 뜨고 내리는 컨테이너들의 로그를 조회할 수 있게 해준다.
- Monitoring
    - 동적으로 분산되어 뜨고 내리는 컨테이너들의 모니터링 정보를 수집한다.
- Storage
    - 여러 호스트간을 이동하며 운영되는 컨테이너가 접근할 리모트 스토리지를 제공한다.


## Kubernetes
- 쿠버네티스는 애플리케이션을 실행하는 컨테이너를 쉽고 빠르게 배포/확장하고, 장애조치 등 관리를 자동화 해주는 오픈소스 플랫폼이다.

## Kubernetes 특징
- 선언적 API
    - 쿠버네티스의 설계 원칙은 API가 '선언적'이라는 것이다. 컨테이너가 어떤 상태이길 원하는지만 쿠버네티스에 설정하면 지속해서 컨테이너의 상태를 확인하고, 설정한 상태에서 벗어나면 다시 설정한 상태로 맞춰간다는 개념이다.
    - 이런 '선언적'이라는 특징이 없다면 사용자가 직접 컨테이너가 원하는 개수만큼 정상 실행중인지 확인해야하고, 원하는 개수가 아니라면 직접 컨테이너를 추가하는 작업까지 수동으로 해줘야 한다.
- 워크로드 분리
    - 분산된 시스템에서 쿠버네티스는 운영체제처럼 분산된 프로세스의 관리를 추상화하는 레이어가 되므로 시스템 운영에 관한 고민을 많이 덜어준다.

## Kubernetes 제공 기능
- Service discovery and load balancing
    - 쿠버네티스는 DNS이름을 사용하거나 자체 IP주소를 사용하여 컨테이너를 노출할 수 있다. 컨테이너에 대한 트래픽이 많으면, 쿠버네티스는 네트워크 트래픽을 로드밸런싱하고 배포하여 배포가 안정적으로 이루어질 수 있다.
- Storage Orchestration
    - 쿠버네티스를 사용하면 로컬 스토리지, 공용 클라우드 등과 같이 원하는 스토리지 시스템을 자동으로 마운트 할 수 있다.
- Automated rollouts and rollbacks
    - 쿠버네티스를 사용하여 배포된 컨테이너의 상태를 확인할 수 있으며 현재 상태를 원하는 상태로 따라 변경할 수 있다. 예를 들어 쿠버네티스를 자동화해서 배포용 새 컨테이너를 만들고, 기존 컨테이너를 제거하고 모든 리소스를 새 컨테이너에 적용할 수 있다.
- Automatic bin packing
    - 컨테이너화된 작업을 실행하는데 사용할수 있는 쿠버네티스 클러스토 노드를 제공한다. 쿠버네티스에 각 컨테이너에서 필요한 CPU 및 메모리를 알려준다. 쿠버네티스는 컨테이너에서 리소스를 최대한 활용할 수 있도록 도와준다.
- Self-healing
    - 쿠버네티스는 실패한 컨테이너를 다시 시작하고, 컨테이너를 교체하고, health-check에 응답하지 않는 컨테이너를 종료한다. 이렇게 서비스를 자동으로 복구하는 기능을 가지고 있다.
- Secret and configuration management
    - 쿠버네티스를 사용하면 패스워드, OAuth 토큰, SSH key 와 같은 민감한 정보를 저장하고 관리할 수 있다. 컨테이너 이미지를 재구성하지 않고 스택 구성에 시크릿을 노출하지 않고도 secret 및 애플리케이션 구성을 배포 및 업데이트 할 수 있다.

## Kubernetes Objects
- 쿠버네티스 시스템의 상태를 표현하는 구성단위 이며 Basic과 Controller 타입이 있다.
- 사용자는 yaml 파일 형식의 템플릿 등으로 쿠버네티스 자원의 "원하는 상태"를 정의하고 컨트롤러는 그 상태를 유지하도록 오브젝트들을 생성/삭제하는 작업을 처리한다.
- Basic 타입 (기본 오브젝트)
    - 쿠버네티스에 의해 배포 및 관리되는 가장 기본적은 오브젝트로 Pod, Service, Volume, Namespace 4가지가 있다.
- Controller 타입
    - 기본 오브젝트를 생성하고 관리하는 역할을 하는 오브젝트
    - Deployment, Replication, Controller, RelicationSet, DeamonSet, Job, StatefulSet 등이 있다.

## 템플릿
- 기본형식

```yaml
---
apiVersion: v1
kind: Pod
metadata:
spec: 
```

- apiVersion : 사용하려는 쿠버네티스 API버전 명시
    - 쿠버네티스는 버전 변경이 빠른 편이므로 API버전을 정확하게 지정해야 함
    - kubectl api-versions 명령으로 현재 클러스터에서 사용가능한 API버전 확인 가능
- kind : 어떤 종류의 오브젝트 혹은 컨트롤러의 작업인지 명시
    - Pod, Deployment, Ingress 등의 오브젝트/컨트롤러 설정 가능
- metadata : 해당 오브젝트의 이름이나 Label 등을 설정
- spec : Pod의 스펙을 정의. Pod가 어떤 컨테이너를 갖고 실행하며, 실행할 때 어떻게 동작해야 할지 명시
- metadata 와 spec에는 다양한 하위 필드가 있다. 어떤 하위필드가 있고 그 하위필드가 어떤 역할을 하는지 명령어를 통해 확인할 수 있다
    - kubectl explain pods : pod의 바로 하위 필드종류와 역할
    - kubectl explain pods.metadata : pod의 하위인 metadata 필드의 하위필드 종류와 역할
    - kubectl explain pods.metadata.annotation : pod의 하위인 metadata 필드의 하위인 annotation의 하위필드 종류와 역할