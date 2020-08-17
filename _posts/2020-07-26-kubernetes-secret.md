---
layout: post
title: "Kubernetes Secret"
tags: [kubernetes]
comments: true
date: 2020-07-26
---


## Secret

- 내장(built-in) Secret
    - 쿠버네티스 클러스터 안에서 쿠버네티스 API에 접근할 때 사용
    - ServiceAccount 는 클러스터 안에서 apiserver와 통신하기 위해 사용된다.
- 사용자 정의 시크릿

## 명령으로 Secret 만들기

- kubectl create secret 명령으로 시크릿 생성시 username, password 등은 자동으로 base64 방식으로 인코딩 되어 저장된다.
- echo ${인코딩된 username 또는 password} \| base64 \-\-decode  명령으로 원래 값 확인 가능

```yaml
$ echo -n 'shin' > ./username.txt
$ echo -n 'shinpassword' > ./password.txt
$ kubectl create secret generic user-pass-secret --from-file=./username.txt --from-file=./password.txt
$ kubectl get secret user-pass-secret -o yaml
```

![No image](/assets/posts/20200726/secret1.png)

## 템플릿으로 Secret 만들기

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: user-pass-yaml
type: Opaque
data:
  username: c2hpbgo=  # shin
  password: c2hpbnBhc3N3b3JkCg==  # shinpassword
```

- .type
    - Opaque : 기본값이며 key & value 형식으로 임의의 데이터 설정할 수 있음
    - kubernetes.io/service-account-token : 쿠버네티스 인증 토큰을 저장함
    - kubernetes.io/dockerconfigjson : 도커 저장소 인증 정보를 저장함
    - http://kubernetes.io/tls : TLS 인증서를 저장함
- .data
    - .data.username 과 .data.password 필드 값을 설정할 때는 base64 인코딩된 값을 설정해야 한다.
    - echo -n "원본" \| base64 명령을 통해 인코딩된 값 구할 수 있다.

## 시크릿 사용하기

- 시크릿은 Pod의 환경변수나 볼륨을 이용한 파일 형식으로 사용할 수 있다.

### Pod의 환경 변수로 Secret 사용하기

```yaml
apiVersion: apps/v1
kind: Deployment
...
spec:
...
  template:
    spec:
      containers:
      - name: testapp
        env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: user-pass-yaml
              key: username
        - name: SECRET_PASSWORD
          valueFrom:
            secretKeyRef:
              name: user-pass-yaml
              key: password
```

- .spec.template.spec.containers[].env[]
    - SECRET_USERNAME 이라는 환경변수에 user-pass-yaml 이름을 가진 secret에서 key가 username인 것을 설정

### 볼륨 형식으로 Pod에 Secret 제공하기

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secretapp
  labels:
    app: secretapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: secretapp
  template:
    metadata:
      labels:
        app: secretapp
    spec:
      containers:
      - name: testapp
        image: arisu1000/simple-container-app:latest
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: volume-secret
          mountPath: "/etc/volume-secret"
          readOnly: true
      volumes:
      - name: volume-secret
        secret:
          secretName: user-pass-yaml
```

- .spec.template.spec.volumes[]
    - volume-secret라는 볼륨을 만들고 이 볼륨은 user-pass-yaml 이라는 시크릿을 제공
- .spec.template.spec.containers[].volumeMounts[]
    - 위에서 만든 volume-secret 라는 볼륨을 \/etc\/volume-secret 위치에 읽기 전용으로 마운트 함
    - 이 결과 Secret 의 .data 필드에 설정한 key&value에서 key값이 파일명으로, value값이 파일내용인 파일들이 생성된다.
    - value 값은 base64 디코딩된 값으로 파일에 저장된다.

    ![No image](/assets/posts/20200726/secret2.png)

### Private Container 이미지 가져올 때 Secret 사용

- Secret을 private container 이미지를 사용할 때 필요한 인증 정보로 사용할 수 있다.
- 쿠버네티스에는 kubectl create secret의 하위 명령으로 도커 컨테이너 이미지 저장소용 시크릿을 만드는 docker-registry가 있다.
    - kubectl create secret docker-registry ${secret name} \-\-docker-username=${username} \-\-docker-password=${password} \-\-docker-email=${email} \-\-docker-server=${docker registry server}
- docker-registry용 시크릿을 만들면 시크릿 type은 kubernetes.io/dockerconfigjson 이다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secretapp
  labels:
    app: secretapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: secretapp
  template:
    metadata:
      labels:
        app: secretapp
    spec:
      containers:
      - name: testapp
        image: # private 이미지라고 가정
        ports:
        - containerPort: 8080
      imagePullSecrets:
      - name: dockersecret
```

- .spec.template.spec.imagePullSecrets.name
    - 컨테이너 이미지가 private docker-registry용으로 생성한 Secret 이름을 넣어 주어야 한다
    - 유효하지 않은 시크릿을 설정하게 되면 pod이 "ErrImagePull" 이라는 상태 값을 가진 채로 정상적으로 실행되지 않는다.

### 시크릿으로 TLS 인증서를 저장해 사용

- HTTPS 인증서를 저장하는 용도로 시크릿을 사용할 수 있다.
- 여기서는 테스트 용도로 공인 기관의 인증서가 아닌 사설 인증서를 만들어 테스트한다.

```yaml
# tls.key 파일 & tls.crt 파일 생성
$ openssl req -x509 -nodes -days 365 -newkey rsa:2049 -keyout tls.key -out tls.crt -subj "/CN=example.com"
Generating a 2049 bit RSA private key

# tlssecret라는 이름으로 시크릿 생성
$ kubectl create secret tls tlssecret --key tls.key --cert tls.crt

# 확인
$ kubectl get secret tlssecret -o yaml
```

- 위에서 만든 tlssecret을 사용하는 pod 생성 (시크릿을 볼륨으로 마운트)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: secretapp
  labels:
    app: secretapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: secretapp
  template:
    metadata:
      labels:
        app: secretapp
    spec:
      containers:
      - name: testapp
        image: arisu1000/simple-container-app:latest
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: volume-secret
          mountPath: "/etc/volume-secret"
          readOnly: true
      volumes:
      - name: volume-secret
        secret:
          secretName: tlssecret
```

- 확인

![No image](/assets/posts/20200726/secret3.png)

## Secret 데이터 용량 제한

- Secret 데이터는 etcd에 암호화하지 않은 텍스트로 저장된다. Secret이 너무 큰 용량을 차지 않도록 개별 시크릿 데이터의 최대 용량은 1MB 이다. 크기가 작은 시크릿을 너무 많이 만들어도 문제가 될 수 있다.
- etcd : 쿠베네티스에서 필요한 모든 데이터를 저장하는 실질적인 데이터베이스
- etcd에 암호화되지 않은 채로 저장되기 때문에 누군가 직접 etcd에 접근한다면 시크릿을 확인할 수 있다. 따라서 etcd 접근을 제한해야 하며, 기본적으로 etcd를 실행할 때 etcd 관련 명령을 사용하는 API 통신에 TLS 인증이 적용되어 있으므로 인증서가 있는 사용자만 etcd에 접근해 관련 명령을 사용할 수 있다.
- 그 외에 etcd가 실행 중인 마스터 자체에 접근할 수 없도록 사용자 계정 기반이나 IP 기반 접근 제어 등으로 접근을 제한할 수 있다. etcd에 저장되는 시크릿 데이터를 암호화해서 사용할 수 있으나 쿠버네티스 클러스터를 직접 설치해서 암호화하는 옵션을 지정해야 한다.
