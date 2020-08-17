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

## When?

- 컨테이너는 기본적으로 상태가 없는(stateless) 앱 컨테이너를 사용
- 컨테이너는 상태가 없기 때문에 노드 장애 발생 시 다른 노드로 자유롭게 옮길 수 있지만, 현재까지 저장된 데이터가 사라진다는 문제가 있다.
- 컨테이너에 문제가 생기더라도 데이터를 보존하기 위해 Volume을 사용

## emptyDir

- emptyDir은 Pod가 실행되는 호스트의 디스크를 임시로 컨테이너에 볼륨으로 할당해서 사용하는 방법
- Pod가 사라지면 emptyDir에 할당해서 사용했던 볼륨의 데이터도 함께 사라진다.
- 주로 메모리와 디스크를 함께 이용하는 대용량 데이터 계산에 사용하거나 오랜 연산 시간이 걸릴 때 중간 데이터 저장용으로 쓰인다. 연산 중 컨테이너에 문제가 발생해서 재시작 되더라도 Pod는 살아 있으므로 emptyDir에 저장해 둔 데이터를 계속 이용 가능하다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-simple-pod
spec:
  containers:
  - name: kubernetes-simple-pod
    image: arisu1000/simple-container-app:latest
    volumeMounts:
    - mountPath: /emptydir
      name: emptydir-vol
  volumes:
  - name: emptydir-vol
    emptyDir: {}
```

- .spec.volumes[]
    - emptydir-vol 이라는 이름으로 emptyDir 생성
- .spec.containers[].volumeMounts
    - 위에서 생성한 볼륨을 컨테이너에서 마운트해서 사용

## hostPath

- hostPath는 Pod가 실행된 호스트의 파일이나 디렉터리를 Pod에 마운트 한다.
- emptyDir이 임시 디렉터리를 마운트하는 것이라면 hostPath는 호스트에 있는 실제 파일이나 디렉터리를 마운트 한다.
- emptyDir이 컨테이너를 재시작했을 때 데이터를 보존하는 역할이라면, hostPath는 Pod를 재시작했을 때도 호스트에 데이터가 남아 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: kubernetes-hostpath-pod
spec:
  containers:
  - name: kubernetes-hostpath-pod
    image: arisu1000/simple-container-app:latest
    volumeMounts:
    - mountPath: /test-volume
      name: hostpath-vol
    ports:
    - containerPort: 8080
  volumes:
  - name: hostpath-vol
    hostPath:
      path: /tmp
      type: Directory
```

- .spec.volumes[].hostPath.type
    - 미설정 : hostPath 볼륨을 마운트하기 전 아무것도 확인하지 않음
    - DirectoryOrCreate : 설정한 경로에 디렉터리가 없으면 permission 755인 빈 디렉토리 생성
    - Directory : 설정한 경로에 디렉터리가 존재해야 함. 디렉터리가 없으면 Pod는 ContainerCreateing상태로 남고 생성이 안됨
    - FileOrCreate : 설정한 경로에 파일이 없으면 permission 644인 빈 파일 생성
    - File : 설정한 경로에 파일이 있는지 확인. 파일이 없으면 Pod생성 안됨
    - Socket : 설정한 경로에 유닉스 소켓 파일이 있어야 함
    - CharDevice : 설정한 경로에 문자 디바이스가 있는지 확인
    - BlockDevice : 설정한 경로에 블록 디바이스가 있는지 확인

## NFS (Network File System)

- nfs 볼륨은 기존에 사용하는 nfs 서버를 이용해서 pod에 마운트하는 것이다. NFS 클라이언트 역할이라고 생각하면 된다.
- 여러 개 pod에서 볼륨 하나를 공유해 읽기/쓰기를 동시에 할 때도 사용한다.
- pod 하나에 안정성이 높은 외부 스토리지를 볼륨으로 설정한 후 해당 pod에 NFS서버를 설정한다. 다른 pod는 해당 pod의 NFS서버를 nfs 볼륨으로 마운트한다.

## NameSpace

- 네임스페이스는 쿠버네티스 클러스터 하나를 여러개의 논리적인 단위로 나눠서 사용하는 것을 말한다. 네임스페이스를 통해 여러 사용자가 하나의 클러스터에서 각각 다른 공간을 사용할 수 있다.
- 쿠버네티스에서는 기본적인 네임스페이스를 생성한다.
    - defalut, kube-system, kube-public, kube-node-lease 등
- kubectl get namespaces
    - 사내 규정 등으로 위 명령에 대한 권한이 없다면, 현재 자신이 속해있는 namespace를 아래 2개의 명령으로 확인 가능하다.
    - kubectl config current-context
    - kubectl config get-contexts ${context_name}
- 쿠버네티스 Service를 생성하면 해당 DNS엔트리가 생성된다. 이 엔트리는 \<service-name\>.\<namespace-name\>.svc.cluster.local 과 같은 형식을 갖게 된다.