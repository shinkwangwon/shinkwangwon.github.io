---
layout: post
title: "Kubernetes Service"
tags: [kubernetes]
comments: true
date: 2020-07-12
---

# Kubernetes Service

- pod는 컨트롤러가 관리하므로 한군데에 고정해서 실행되지 않고, 클러스터 안을 옮겨 다닌다. 이 과정에서 노드를 옮기면서 실행되기도 하고 클러스터 안에서 pod의 IP가 변경되기도 한다. 이렇게 동적으로 변하는 pod들에 고정적으로 접근할 때 사용하는 방법이 Service 다.
- Service를 사용하면 pod가 클러스터 안에 어디에 있든 고정 주소를 이용해 접근이 가능하다. Service는 Layer4 영역에서 통신할 때 사용된다. 즉, L4단 로드밸런싱 역할을 수행한다.
- pod의 IP가 변하더라도 서비스가 자동으로 변경된 pod와 통신하므로 사용자는 서비스만 이용해서 변경된 파드를 사용할 수 있다.

## Service Template


```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP
  selector:
    app: MyApp
  ports:
    - name: http
      protocol: TCP
      port: 80
      targetPort: 9376
    - name: https
      protocol: TCP
      port: 443
      targetPort: 9377
```

- .spec.type
    - 서비스 타입. 설정하지 않으면 기본 ClusterIP
- .spec.clusterIP
    - 직접 클러스터 IP지정 가능. 설정하지 않으면 자동으로 IP 할당
- .spec.selector
    - 서비스와 연결할 pod에 설정한 label 값 설정
- .spec.ports[]
    - 외부에 노출할 포트번호. 멀티포트를 노출해야할 경우가 있기에 ports는 배열형태이다.
    - 멀티포트를 사용하는 경우 모든 포트에 name속성을 명확하게 지정해야 한다.
    - 서비스는 port로 들어온 요청을 "app: MyApp" label을 가진 pod에 targetPort로 매핑한다.

## Service Type

- ClusterIP
    - 기본 서비스 타입이며 쿠버네티스 클러스터 안에서만 사용할 수 있다. 클러스터 외부에서는 접근할 수 없다.
- NodePort
    - 모든 노드의 지정된 포트를 서비스에 할당한다. node1:8080, node2:8080 처럼 노드에 상관없이 서비스에 지정된 포트번호만 사용하면 모든 노드는 해당 서비스로 프록시 한다.  \<NodeIP\>:\<NodePort\>로 요청하여 클러스터 외부에서 서비스로 접근 가능하다.

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      type: NodePort
      selector:
        app: MyApp
      ports:
          # 기본적으로 그리고 편의상 `targetPort` 는 `port` 필드와 동일한 값으로 설정된다.
        - port: 80
          targetPort: 80
          nodePort: 30007
    ```

- LoadBalancer
    - ClusterIP와 마찬가지로 내부에서 접근가능한 IP를 포함해 외부에서 접근 가능한 External-IP까지 부여 된다.
    - LoadBalancer 서비스 타입은 두개 이상의 포트가 정의된 경우, 모든 포트의 프로토콜이 동일해야하고 프로토콜은 TCP, UDP, SCTP 중 하나여야 한다.

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
    spec:
      selector:
        app: MyApp
      ports:
        - protocol: TCP
          port: 80
          targetPort: 9376
      type: LoadBalancer
    ```

- ExternalName
    - .spec.externalName 필드에 설정한 값과 서비스를 연결한다. 클러스터 안에서 외부에 접근할 때 주로 사용되고, 클러스터 안 pod에서 이 서비스에 접근하면 서비스는 .spec.externalName에 설정한 CNAME 값을 반환해주고 pod는 CNAME을 통해 외부에 접근한다.
    - CNAME : 하나의 도메인에 다른 이름을 부여하는 것을 의미

    ```yaml
    apiVersion: v1
    kind: Service
    metadata:
      name: my-service
      namespace: prod
    spec:
      type: ExternalName
      externalName: my.database.example.com
    ```

## Kube proxy

- 쿠버네티스에서 서비스를 만들었을 때 클러스터 IP나 노드 포트로 접근할 수 있게 만드는 역할을 하는 컴포넌트로써, 쿠버네티스 클러스터의 노드마다 실행되면서 클러스터 내부 IP로 연결하려는 요청을 적절한 pod로 전달한다.
- userspace 모드
    - 클라이언트에서 서비스의 클러스터 IP를 통해 어떤 요청을 하면 iptables를 거쳐서 kube-proxy가 요청을 받는다. 그리고 서비스의 클러스터 IP는 연결되어야 하는 적절한 pod로 연결한다. 이때, kube-proxy는 Round-Robbin 방식으로 pod에 요청을 전달한다.

    ![No image](/assets/posts/20200712/userspace.png)

- iptables 모드
    - userspace 모드와 다른 점은 kube-proxy가 iptables를 관리하는 역할만 하고 직접 클라이언트에서 트래픽을 받지 않는다는 것이다. 기본적으로 iptables 모드는 요청을 임의의 pod로 전달하는 방식을 사용한다.
    - 클라이언트에서 오는 모든 요청은 iptables를 거쳐 pod로 직접 전달된다. 그래서 userspace모드보다 요청 처리 성능이 좋다.
    - userspcae 모드에서는 pod 하나로 연결 요청이 실패하면 자동으로 다른 pod에 연결을 재시도한다. 하지만 iptables 모드에서는 pod 하나로의 연결 요청이 실패하면 재시도하지 않고 그냥 요청이 실패한다. 컨테이너에 readinessProbe를 통해 pod가 정상 작동하는지 확인할 수 있으므로 iptables 모드의 kube-proxy는 정상적으로 활성화된 pod로만 요청을 보낼 수 있다.

    ![No image](/assets/posts/20200712/iptables.png)

- IPVS 모드
    - IP Virtual Server 모드는 리눅스 커널에 있는 L4 로드밸런싱 기술이다. 리눅스 커널 안 네트워크 관련 프레임워크인 netfilter에 포함되어 있기 때문에 노드에 IPVS커널 모듈이 설치되어 있어야 한다.
    - IPVS 모드는 커널 스페이스에서 동작하고 데이터 구조를 해시 테이블로 저장하기 때문에 iptables 모드보다 빠른 성능을 보이고 더 많은 로드밸런싱 알고리즘을 사용할 수 있다.
    - 참고 : [https://kubernetes.io/ko/docs/concepts/services-networking/service/#proxy-mode-ipvs](https://kubernetes.io/ko/docs/concepts/services-networking/service/#proxy-mode-ipvs)

    ![No image](/assets/posts/20200712/IPVS.png)

