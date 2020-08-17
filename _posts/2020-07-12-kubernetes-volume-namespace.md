---
layout: post
title: "Kubernetes Volume & Namespace"
tags: [kubernetes]
comments: true
date: 2020-07-12
---

## Volume

- 컨테이너 내의 디스크에 있는 파일은 임시적이며, 컨테이너 장애로 인해 재시작시 이 파일들은 유실된다.
- Pod 에서 여러 컨테이너를 실행할 때 컨테이너 사이에 파일을 공유해야하는 경우가 자주 발생한다.
- Volume은 이런 문제를 모두 해결해준다.
- 쿠버네티스의 볼륨과 도커에서의 볼륨은 약간 다르다.
- 쿠버네티스에서의 볼륨은 Pod내에서 실행되는 모든 컨테이너보다 수명이 길기 때문에 컨테이너를 재시작해도 데이터가 보존된다. 물론 Pod가 존재하지 않으면 볼륨도 존재하지 않는다.
- 볼륨을 사용하기 위해 설정파일에서 Pod에 제공할 볼륨 컨테이너에 마운트할 위치를 지정한다.
> 도커에서 볼륨은 단순한 디스크 내 디렉터리 또는 다른 컨테이너에 있는 디렉터리다. 수명은 관리되지 않으며 최근까지는 로컬 디스크 백업 볼륨만 있었다. 도커는 이제 볼륨 드라이버를 제공하지만, 현재 기능은 매우 제한되어 있다(예: 도커 1.7부터 컨테이너 당 하나의 볼륨 드라이버만 허용되고 매개 변수를 볼륨에 전달할 방법이 없다

## NameSpace

- 네임스페이스는 쿠버네티스 클러스터 하나를 여러개의 논리적인 단위로 나눠서 사용하는 것을 말한다. 네임스페이스를 통해 여러 사용자가 하나의 클러스터에서 각각 다른 공간을 사용할 수 있다.
- 쿠버네티스에서는 기본적인 네임스페이스를 생성한다.
    - defalut, kube-system, kube-public, kube-node-lease 등
- kubectl get namespaces
    - 사내 규정 등으로 위 명령에 대한 권한이 없다면, 현재 자신이 속해있는 namespace를 아래 2개의 명령으로 확인 가능하다.
    - kubectl config current-context
    - kubectl config get-contexts ${context_name}
- 쿠버네티스 Service를 생성하면 해당 DNS엔트리가 생성된다. 이 엔트리는 \<service-name\>.\<namespace-name\>.svc.cluster.local 과 같은 형식을 갖게 된다.