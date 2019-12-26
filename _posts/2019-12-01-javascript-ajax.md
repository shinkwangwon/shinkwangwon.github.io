---
layout: post
title: "Javscript Ajax"
tags: [javascript]
comments: true
date: 2019-12-01
---

## Ajax (Asynchronous JavaScript and XML)
- 비동기적으로 자바스크립트와 XML을 이용한 데이터 교환방식이다. (페이지를 리로딩하지 않고 데이터만 교환함)
- JSON 형태로 작업을 하는 경우가 많아 XML보단 Json으로 데이터를 교환한다.
- 참고 : <https://victorydntmd.tistory.com/172>
- 대부분의 브라우저에는 XMLHttpRequest 라는 것이 내장되어 있다.
- 초기에는 XML방식으로 HTTP 통신만 담당했다 (화면을 그리는 용도가 아님)
- 그런데 XML을 조작하는것이 JS이기 때문에 XML을 다시 JS로 파싱해야하는 작업이 필요했다.
- 그래서 처음부터 JS객체형식(JSON)으로 데이터를 주고 받을 수 있는 형태가 나온것이다!!
- Form 태그의 submit은 동기식이고, 페이지 리로딩이 발생함


## Content-Type
- 클라이언트와 서버간에 데이터(body)를 전달할때 Http Header에 Content-Type을 지정함으로써 데이터 교환 타입을 맞추는 것을 말한다.


## Content-Disposition
- 서버에서 내려주는 Response Header의 하나로써 파일 다운로드 처리하는 용으로 쓰임
- dispositon-type
    - inline : 파일을 웹 브라우저 내에서 볼 수 있도록 하는 속성
    - attachment : 파일을 다운로드 할 수 있게 하는 속성
- disposition-param 
    - filename : 다운로드할 때 쓰일 파일 이름
```java
public void writeResponse(String fileName, HttpServletResponse response) throws IOException {
    response.setContentType("application/msexcel");
    response.setHeader("Content-Disposition", String.format("attachment; filename=\"%s\"", URLEncoder.encode(fileName,"UTF-8")));
}
```