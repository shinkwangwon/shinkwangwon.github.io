---
layout: post
title: "docker command 기본"
tags: [docker]
comments: true
date: 2020-06-24
---

# Docker 실행 명령어

## docker 빌드
- docker build -t {생성할 이미지 이름} {Dockerfile이 있는 경로}
- ex) docker build -t shin-image .
- 옵션 -t(또는 –tag) {생성할 이미지 이름} : 이미지 이름 지정

## docker 프로세스 확인
- docker ps -a
- 옵션 -a : 종료된 프로세스까지 확인

## docker 이미지 확인
- docker images

## docker image 삭제
- docker rmi {이미지ID}
- docker images 명령실행시 IMAGE ID에 해당하는 값

## docker image 전체 삭제
- docker rmi $(docker images -q) 또는 docker rmi \`docker images -q\`

## docker 실행
- docker run [옵션] {실행할 이미지명}
- docker images 명령 실행시 REPOSITORY에 해당하는 값
- ex) docker run -p 1234:8080 shin-image
- 옵션 -p {포트:포트}: 포트 지정 (클라이언트가 연결할 포트):(도커에서 EXPOSE로 설정한 포트)
- 옵션 -d : 백그라운드로 실행

## docker container 삭제
- docker rm {containerID}
- docker ps -a 명령 실행시 CONTAINER ID에 해당하는 값

## docker 종료된 container 한번에 삭제
- docker rm -v $(docker ps -a -q -f status=exited)

## docker container 전부 삭제
- docker rm $(docker ps -a -q) 또는 docker rm \`docker ps -a -q\`

## docker 컨테이너 중지
- docker stop {containerID}

## docker 외부에서 컨테이너 안의 명령어 실행하기
- docker exec -i -t {containerID} /bin/bash
- docker ps 명령실행시 나오는 containerID로 접근
- /bin/bash를 실행하여 도커 컨테이너 안의 Bash 셀에 연결
- 옵션 -i : 표준입출력 유지
- 옵션 -t : TTY모드 사용. Bash를 사용할때 이 옵션을 설정해야 셸과 메시지를 give&take 할 수 있음



# Dockerfile 작성시 명령어

## EXPOSE
- 도커 컨테이너를 생성하면 기본적으로 외부와 통신이 불가능한 상태다. 따라서 외부와 통신을 위해서는 컨테이너를 외부로 노출할 port를 지정해줘야 한다
- 사용법: EXPOSE 8080/tcp

## docker ADD vs COPY
- ADD {source} {destination} => destination은 이미지에서 파일이 위치할 경로를 말한다
- Dockerfile의 ADD는 source로 URL을 사용할 수 있다. ULR을 입력할 경우 다운로드 해서 이미지상의 destination위치에 추가한다.
- ADD는 source가 gzip과 같이 일반적으로 알려진 압축형태인 경우 압축을 풀어서 이미지상의 destination위치에 저장한다
- 압축된 “remote” 파일인 경우에는 압축을 풀어주지 않는다
- COPY {source} {destination} => destination은 이미지에서 파일이 위치할 경로를 말한다
- COPY는 URL을 사용할 수 없고 압축파일을 이미지에 추가할 때도 압축을 해제하지 않는다.

*ADD & copy 공통 규칙* - source의 경로는 컨텍스트 아래를 기준으로 하며 디렉토리나 절대경로는 사용할 수 없다 - 즉 /api/…. 처럼 “/”부터 시작하는 절대경로로 사용 하면 안된다 - 여기서 컨텍스트라 함은 Dockerfile과 같은 위치(디렉토리)에 있는 모든 파일을 말한다. - destination은 항상 절대경로로 설정해야한다. 그리고 마지막이 “/”로 끝나면 디렉토리가 생성되고 파일은 그 아래에 복사된다.

## WORKDIR
- 사용법 : WORKDIR <경로>
- WORKDIR 뒤에 오는 경로는 RUN, CMD, ENTRYPOINT가 수행될때 항상 적용된다.
- 주의할 점은 중간에 WORKDIR <경로> 를 다른 경로로 선언해버리면 그 라인부터 다음에 오는 명령들은 바뀐 WORKDIR 에서 명령이 수행된다
- WORKDIR 은 상대경로로 지정이 가능하기 때문에 중간에 WORKDIR을 바꾸게 된다면 주의해서 변경해야 한다.

## USER
- 사용법: USER root
- Dockerfile에 있는 RUN, CMD, ENTIRYPOINT 등의 명령을 수행할때 해당 사용자권한으로 수행한다.
- Dockerfile에서 중간에 USER를 다시 선언하면, 해당 선언 다음 명령부터는 변경된 사용자 권한으로 수행한다.

## RUN vs CMD vs ENTRYPOINT
- RUN : 도커 이미지에 다른 패키지(프로그램)를 설치하때 주로 사용한다. RUN을 수행할때마다 새로운 이미지 레이어에서 실행된다.
    - RUN 명령1 RUN 명령2 => 이렇게 2개의 명령을 각각 수행할 경우 명령1 과 명령2는 전혀 다른 레이어에서 수행되기 때문에 연관지어 수행해야하는 명령일 경우에는 RUN 명령1 && 명령2 로 묶어서 수행해야한다.
    - ex) RUN apt-get update RUN apt-get install -y ==> 이렇게 따로 수행할 경우, apt-get update 해봤자 apt-get install할때는 apt-get update 결과가 적용되어 있지 않음
- CMD : default 명령이나 파라미터를 설정한다. docker run 실행시 실행할 커맨드를 주지 않으면 이 default 명령이 실행된다. 그리고 ENTIRYPOINT의 파라미터를 설정할 수 있다. CMD의 주 용도는 컨테이너를 실행할때 사용할 default 명령을 설정하는 것이다.
    - CMD는 여러번 수행될 수 있지만, 가장 마지막에 작성한 CMD 1개만 ENTRYPOINT의 파라미터를 위해 남는다.
    - ex) CMD echo “hello” 라고 도커파일에 작성했을때 docker run 실행시 아무런 커맨드를 주지 않으면 CMD가 수행된다. ( CMD[“echo”, “hello”] 라고 작성해도 된다)
    - docker run -p 1234:8080 imageName
    - => 아무런 명령을 주지 않고 단순히 docker run만 수행 ==> echo “hello”가 수행된다
    - docker run -p 1234:8080 imageName echo “goodbye”
    - => echo “goodbye”라는 명령어를 줬기 때문에 echo “hello”는 무시되고 echo “goodbye”가 실행된다.
- ENTRYPOINT: 컨테이너 실행시 실행되는 명령이다. CMD랑 같은것 같지만 다르다.
    - ENTRYPOINT [“/bin/echo”, “Hello”] CMD [“world”]
    - 이렇게 도커파일에 적힌 경우, CMD에서 설정한 default파라미터가 ENTRYPOINT에서 사용된다.
    - 다만, docker run 실행시 파라미터를 주면 CMD에서 설정한 파라미터는 ENTRYPOINT에서 사용되지 않는다.
    - docker run [option] [imagename]
    - => 출력 Hello word ==> docker run 파라미터가 없기 때문에 CMD의 “world”가 EntryPoint에서 사용됨
    - docker run [option] [imagename] Kwang
    - => 출력 Hello Kwang ==> docker run 파라미터로 “Kwang”을 줬기 때문에 CMD의 “world”는 무시되고 “kwang”가 출력됨
