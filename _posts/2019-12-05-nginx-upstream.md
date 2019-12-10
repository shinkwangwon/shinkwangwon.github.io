---
layout: post
title: "Nginx Upstream"
tags: [nginx]
comments: true
date: 2019-12-05
---


## NginX Upstream
- NginX Upstream란 proxy_pass 지시자를 통해 nginx가 받은 리퀘스트를 넘겨 줄 서버를 정의하는 지시자.
- Upstream의 기본적인 LoadBalancing 알고리즘은 Round Robin 방식이다. 따라서 서로 다른 2개의 클라이언트에서 요청이 온다고해도 같은 톰캣으로 붙을 수 도 있다.
- Nginx 의 vhost.conf 파일의 server{} 블록안에 return 301 redirect주소; 를 쓰게 되면 해당 서버로 접속한 요청을 리다이렉트 시킬 수 있음
    - 이 방식으로 http 요청을 https 로 리다이렉트 시킴
    ```bash
    server {
        listen 80;
        server_name test.co.kr
        return 301 https://$server_name$request_uri
    }
    ```
- document.location.href = "https://$server_name$request_uri"; 처럼 스크립트에서 리다이렉트를 해줄 수 도 있지만, 이런 경우 페이지 이동과정에 사용자에게 순간적으로 노출되고, 스팸 사이트로 분류될 수 도 있음. 따라서 이동된 페이지에 대해서는 HTTP 상태 코드를 사용해서 정확히 무슨일이 일어났는지 클라이언트에게 알려주어야 함.


## HTTP 상태 코드
- 301 : 페이지 영구 이동 (검색엔진의 indexing된 주소를 수정함)
- 302 : 페이지 임시 이동 