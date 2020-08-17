---
layout: post
title: "Kubernetes ConfigMap"
tags: [kubernetes]
comments: true
date: 2020-07-26
---

## ConfigMap

- ConfigMap은 컨테이너에 필요한 환경 설정을 컨테이너와 분리해서 제공하는 기능
- 서비스 환경 별로 설정 값을 다르게 가져가기 위한 용도로 주로 사용

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-dev
data:
  DB_URL: localhost
  DB_USER: myuser
  DB_PASS: mypass
  DEBUG_INFO: debug
```

- .data 하위 필드에 실제 사용하려는 환경 설정 값 세팅
- 쿠버네티스 클러스터에 ConfigMap 적용
    - kubectl apply -f ${configmap yaml 파일명}
- 적용된 ConfigMap 확인
    - kubectl describe configmap ${configmap name}

## ConfigMap 설정 중 일부만 불러와서 사용

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: configapp
  labels:
    app: configapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: configapp
  template:
    metadata:
      labels:
        app: configapp
    spec:
      containers:
      - name: testapp
        image: arisu1000/simple-container-app:latest
        ports:        
        - containerPort: 8080
        env:
        - name: DEBUG_LEVEL
          valueFrom: 
            configMapKeyRef:
              name: config-dev
              key: DEBUG_INFO
```

- .spec.template.spec.containers[].env[].name
    - Deployment에서 사용할 환경변수명 설정
- .spec.template.spec.containers[].env[].valueFrom
    - 값을 어디에서 가져올 것인지 지정하며, 하위의 configMapKeyRef 필드는 어떤 ConfigMap의 어떤 키를 가져올 것인지 지정
    - 여기서는 앞에서 설정한 ConfigMap(name=config-dev)의 .data.DEBUG_INFO 필드만 불러옴

## ConfigMap 설정 전체를 한꺼번에 불러와서 사용

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: configapp
  labels:
    app: configapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: configapp
  template:
    metadata:
      labels:
        app: configapp
    spec:
      containers:
      - name: testapp
        image: arisu1000/simple-container-app:latest
        ports:
        - containerPort: 8080
        envFrom:
        - configMapRef:
            name: config-dev
            # name: config-prod
---
apiVersion: v1
kind: Service
metadata:
  labels:
    app: configapp
  name: configapp-svc
spec:
  ports:
  - nodePort: 30800
    port: 8080
    protocol: TCP
    targetPort: 8080
  selector:
    app: configapp
  type: NodePort
```

- .spec.template.spec.containers[].envFrom[].configMapRef.name
    - envFrom 필드와 하위의 configMapRef 필드를 통해 config-dev라는 ConfigMap을 통째로 불러옴
    - 만약 .configMapRef.key 필드까지 설정한 후 deployment 실행 불가능
- Deployment & Service 실행
    - kubectl apply -f ${deployment & service yaml 파일명}
    - nodePort가 이미 사용 중이라는 에러시 적정한 포트 번호로 변경
- 내부 클러스터에 Shell pod 생성
    - kubectl run -i -t shell-${USER} --image={images url} --restart=Never --rm --env USER=${USER} /bin/bash
- 서비스 http 요청을 통해 환경변수 확인
    - curl http://configapp-svc:8080/env
    - ConfigMap에 세팅한 4가지가 모두 환경변수로 설정되어 있음

    ![No image](/assets/posts/20200726/configmap1.png)

## ConfigMap을 볼륨에 불러와서 사용

- ConfigMap 설정을 컨테이너의 환경 변수로 설정하는 것이 아닌, 컨테이너의 볼륨 형식으로 설정해서 파일로 컨테이너에 제공할 수 있다.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: configapp
  labels:
    app: configapp
spec:
  replicas: 1
  selector:
    matchLabels:
      app: configapp
  template:
    metadata:
      labels:
        app: configapp
    spec:
      containers:
      - name: testapp
        image: arisu1000/simple-container-app:latest
        ports:
        - containerPort: 8080
        volumeMounts:
        - name: config-volume
          mountPath: /etc/config
      volumes:
      - name: config-volume
        configMap:
          name: config-dev
```

- .sepc.template.spec.volumes[]
    - .name : config-volume 이라는 이름의 볼륨 생성
    - .configMap : config-volume 볼륨은 config-dev라는 컨피그맵을 제공
- .sepc.template.spec.containers[].volumeMounts[]
    - 위에서 생성한 config-volume 이라는 이름의 볼륨을 \/etc\/config 위치에 마운트 함
    - 이 결과 config-dev 라는 ConfigMap에 설정한 .data 필드의 Key를 파일명으로, Value를 파일내용으로한 파일이 생성된다.
    - pod 접근 후 \/etc\/config 디렉터리 확인

    ```yaml
    ...
    kind: ConfigMap
    data:
      DB_URL: localhost
      DB_USER: myuser
      DB_PASS: mypass
      DEBUG_INFO: debug
    ...
    ```

    ![No image](/assets/posts/20200726/configmap2.png)
