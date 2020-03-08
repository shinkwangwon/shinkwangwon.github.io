---
layout: post
title: "MacBook 초기 개발환경 설정"
tags: [etc]
comments: true
date: 2020-03-03
---

### MacBook 초기설정
> 매번 초기에 개발환경할 때마다 헷갈려서 애먹는 것 같다. 이참에 그냥 쭈욱 정리해보자.

## Xcode
- AppStore => Xcode 설치 및 Xcode 최초 한번 실행
- Xcode를 설치했으면 꼭 한번 최초 실행을 해줘야 개발에 필요한 이것저것 세팅을 자동으로 해준다.

## git 설치
- Xcode 깔면 자동으로 깔린다
- 터미널창 열어서 git \-\-version 명령어로 git 설치되어 있는지 먼저 확인하고 안깔려 있으면 <https://12bme.tistory.com/164> 에서 다운로드
- 깃 계정 설정
- git config \-\-global user.name "your name"
- git config \-\-global user.email "your email"
- 확인 git config \-\-list


## IntelliJ
- IntelliJ download <https://www.jetbrains.com/ko-kr/idea/download/#section=mac>
- GitHub 연동
- IntelliJ IDEA -> Preference -> Version Controll -> GitHub -> 계정연동
- VCS - Git - Clone
- 계정먼저 연동하면 계정에 있는 repository 리스트 전부 다 뜬다. 선택하면 Clone 받아진다.
- 혹시 인텔리제이 버전이 너무 낮아서 repository 리스트가 안뜬다면 Clone받을 URL 입력해서 클론받자.

## jdk
- jdk 11 download (나는 jdk11 버전으로 깔았다)
- <https://www.oracle.com/java/technologies/javase-jdk11-downloads.html>
- 위 사이트에서 macOS installer 다운 (jdk-11.0.6_osx-x64_bin.dmg)
- 다운받고 설치하면 자동으로 /Library/Java/JavaVirtualMachines 이 위치에 jdk가 설치될 때도 있고 아닐 때도 있다.
- jdk가 다운로드받은 폴더에 그대로 있다면 이 위치로 옮겨주자
> mv ~/Download/jdk-11.0.6.jdk /Library/Java/JavaVirtualMachines/
- 설치확인 :  java \-\-version
- 만약 제대로 명령이 실행이 안되면 환경변수 잡아줘보자
- 1) 사용자위치에 .bash_profile 생성 
![No image](/assets/posts/20200303/jdk1.png)

- 2) .bash_profile 작성
![No image](/assets/posts/20200303/jdk2.png)

- 3) .bash_profile 실행 후 jdk 확인
![No image](/assets/posts/20200303/jdk3.png)


## Maven
- 메이븐 bin.tar.gz 으로 다운로드한다. <https://maven.apache.org/download.cgi#>
- 적당한 곳에 압축 풀자. 나는 /Users/shinkwangwon/apache/maven-3.6.3 여기에 압축풀고 이름을 maven-3.6.3 으로 짧게 변경했다.
- 그리고 ~/.bash_profile에 환경변수 설정하자
- vi ~/.bash_profile 열어서 아래 두줄 추가해주자.
- export MVN_HOME=/Users/shinkwangwon/apache/maven-3.6.3
- export PATH=${PATH}:$MVN_HOME/bin
- source ~/.bash_profile
- mvn -version 


## Tomcat
- 톰캣 다운로드. <https://tomcat.apache.org/>
- 다운로드할 톰캣 버전 선택한 다음 Core에 있는 tar.gz 파일을 다운로드한다.
- 굳이 터미널에서 압축푹지 말고 파인더에서 더블클릭해서 압축 풀어버리자
- 이번엔 터미널에서 톰캣 설치한 디렉토리의 bin까지 이동하자. 나는 압축 푼 톰캣디렉토리를 통째로 홈디렉토리안에 apache라는 디렉토리만들고 거기에다가 옮겼다.
- cd ~/apache/apache-tomcat-9.0.31/bin
- ./startup.sh  ==> startup.sh 파일이 제대로 실행되면 웹페이지에서 localhost:8080 으로 접근하면 끝
- ./shutdown.sh   ==> 톰캣 종료

## MySQL
- mysql download 페이지에서 MySQL Community Server 다운로드 한다.
- MySQL Community Server 링크는 여기다. <https://dev.mysql.com/downloads/mysql/>
- macOS 10.15 (x86, 64-bit), DMG Archive 버전으로 설치한다.
- 설치시 별다른 행동을 안했다면 아마도 /usr/local/mysql/ 에 설치됐을 것이다.
- cd /usr/local/mysql/bin
- sudo ./mysql -u root -p 
- 아, 만약에 실행이 안되면 mysql이 stop 상태일 수도 있다. mac 시스템 환경설정 들어가면 MySQL 아이콘이 보일 것이다. 실행시켜주자

## MySQL Workbench
- Workbench 다운로드. <https://dev.mysql.com/downloads/workbench/>
- 설치 후 hostname: localhost , port: 3306 으로 접근하면 끝
- MySQL 설치할 때 설정했던, root 계정 패스워드 까먹지 맙시다.

## node
- nvm을 이용해서 node를 설치하고, nvm을 이용해서 node 버전관리를 한다.
- 이 URL에서 nvm 설치 스크립트를 가져올 수 있다. <https://github.com/nvm-sh/nvm#install--update-script>
- 터미널창에서 아래 명령어들을 하나씩 수행하자. 물론 다 위의 사이트에서 가져온 스크립트다.
- 1) nvm 설치스크립트 수행)
> curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh \| bash

- 2) nvm 스크립트 자동 삽입 
  * 만약 ~/.bash_profile 또는 ~/.profile  또는 ~/.bashrc 의 파일이 이미 존재할 경우에 자동으로 nvm관련 스크립트를 해당 파일중 한곳으로 넣어준다. 나같은 경우는 위에서 jdk 설치하면서 ~/.bash_profile 파일을 생성했기 때문에 자동으로 nvm관련 스크립트가 ~/.bash_profile에 들어와 있다. 

- 3) 2번에 해당하는 파일이 없다면 스크립트 수동삽입한다. 위 쉘파일중 하나 만들고 vi로 아래 두줄 내용 집어넣는다.
> export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" \|\| printf %s "${XDG_CONFIG_HOME}/nvm")"  
> [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm

- 4) 쉘스크립트 재시작
> source ~/.bash_profile

- 5) 확인
> nvm ls

- 6) 노드설치
> nvm install stable  ==> 특정 버전 명시해도 되고 그냥 stable 버전으로 설치해도 된다.

- 7) 확인
> node -v  그리고  npm -v  그리고  nvm ls 명령어로 어떤 버전쓰고 있는지까지 확인 가능하다.

- 8) node 다른 버전 설치하고 설치한 버전으로 변경하고 싶으면
> nvm install "버전" 으로 node 설치 후에 nvm use "버전" 으로 사용할 버전 변경 가능하다. 확인은 node -v 또는 nvm ls

- 참고 : 이분이 정리 잘해놨다. 단 nvm, node 버전들은 하도 많이 생기니까 버전확인은 꼭 해보자. <https://gist.github.com/falsy/8aa42ae311a9adb50e2ca7d8702c9af1>

## docker
- docker 다운로드하고 설치하면 끝이다. <https://hub.docker.com/editions/community/docker-ce-desktop-mac>
- 설치하다 보면 아래 그림처럼 starting 중이라고 뜬다. 조금 기다리면 running으로 바뀐다.
- 그럼 터미널창 열어서 docker -v 명령어로 제대로 설치됐는지 확인하면 된다.
![No image](/assets/posts/20200303/docker1.png)

## 그외 설치할만 한 것들
- VSCode 설치: <https://code.visualstudio.com/download>
- sublimeText: <https://www.sublimetext.com/3>
- Alfred: <https://www.macupdate.com/app/mac/34344/alfred/download>
- insomnia: <https://insomnia.rest/download/#mac>
