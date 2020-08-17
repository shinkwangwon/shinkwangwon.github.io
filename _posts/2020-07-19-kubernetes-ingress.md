---
layout: post
title: "Kubernetes Ingress"
tags: [kubernetes]
comments: true
date: 2020-07-19
---


## Ingress

![No image](/assets/posts/20200719/ingress.png)

- 인그레스는 클러스터 외부에서 안으로 접근하는 요청들을 어떻게 처리할지 정의해둔 규칙 모음
- 클러스터 외부에서 접근해야 할 URL을 사용할 수 있도록 하고, 트래픽 로드밸런싱, SSL 인증서 처리, 도메인 기반 가상 호스팅도 제공
- 인그레스 자체는 이런 규칙들을 정의해 둔 자원이며(Ingress Resource로 표현), 실제로 동작시키는 것은 인그레스 컨틀롤러(Ingress Controller)이다. 즉, Ingress Controller가 외부로부터 요청을 받았을 때 Ingress Resource를 기반으로 해당 요청을 어떻게 처리할지 결정한다.
- 클라우드 서비스를 사용하면 별다른 설정없이 자체 로드밸런서 서비스와 연동해서 인그레스를 사용할 수 있으나 클라우드 서비스를 사용하지 않고 직접 쿠버네티스 클러스터를 구축해서 사용한다면 인그레스 컨트롤러를 직접 인그레스와 연동해야 한다. 인그레스 컨트롤러를 위한 여러 solution 들이 있다
    - ingress-nginx : 쿠버네티스에서 제공
    - ingress-gce : Google Compute Engine. 쿠버네티스에서 제공
    - HAProxy : 외부 솔루션
    - traefix : 외부 솔루션
    - ...
- 쿠버네티스의 Service Object를 사용하면 L4 Layer단의 로드밸런싱까지만 가능하다. 인그레스는 L7 Layer단의 로드밸런서 역할을 수행할 수 있다.

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: foo.bar.com
    http:
      paths:
      - path: /foos1
        backend:
          serviceName: s1
          servicePort: 80
      - path: /bars2
        backend:
          serviceName: s2
          servicePort: 80
  - host: bar.foo.com
    http:
      paths:
      - backend:
          serviceName: s2
          servicePort: 80
```

- .metadata.annotations : 인그레스를 설정할 때는annotations의 하위 필드를 사용하는데, 하위 필드의 설정은 인그레스 컨트롤러마다 다르다. 여기서는 ingress-nginx컨트롤러를 사용하는데 rewrite-target을 / 로 설정했다. 이는 / 경로로 리다이렉트 하라는 뜻이다
- .sepc.rules[] : rules의 하위 필드에는 요청에 대한 규칙을 설정한다. .sepc.rules[].http.paths[] 의 하위 필드는 HTTP요청이 어떤 경로로 들어오는지 뜻한다. foo.bar.com/foo1 로 들어오는 요청을 s1이라는 서비스의 80포트로 보내라는 뜻이다.
- 인그레스 실행 command : kubectl apply -f ${yaml 파일명}
- 인그레스 확인 command : kubectl describe ingress ${ingress name}   여기서 ingress name은 test
