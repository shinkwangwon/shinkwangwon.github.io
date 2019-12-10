---
layout: post
title: "HTML 렌더링"
tags: [html]
comments: true
date: 2019-11-18
---


## 브라우저가 화면그리는 과정
1. HTML 읽는다
2. HTML 파싱한다(DOM트리를 구축하기 위함))
3. DOM 트리를 생성한다
4. Render 트리를 구축
5. Render 트리를 통해 화면에 그림


## HTML 렌더링
- 화면단 개발을 하다보면 html파일에 javascript를 로딩해야 하는 경우가 많다.
- 이때 script 태그의 위치에 따라 렌더링하는 속도가 달라질 수 있다.
- html head 영역 안에 넣으면 브라우저가 html을 파싱하다가 멈추고 스크립트를 해석하거나 다운로드 받는 일을 해버린다. 그리고나서 HTML을 다시 파싱하고 파싱이 완료되면 DOM트리를 만들고 화면에 렌더링한다. 때문에 화면이 느리게 그려 질 수 있다.
- html body 태그의 끝나는 부분에 넣으면 브라우저가 렌더링을 한 후에 스크립트를 해석하기 때문에 사용자 입장에서는 조금이라도 더 빠르게 화면을 볼 수 있다.
- 문서의 DOM(Document Object Model) 구조가 필요한 스크립트의 경우 document.onload와 같은 로드 이벤트가 추가되어야 에러없이 작동된다.


## jQuery의 document.ready(), window.load() 실행 순서
- $(document).ready(function(){}) (= 생략해서 쓰면 $(function(){})로 사용 가능하지만 명확하게 하기 위해 비추천 
  * 외부 리소스, 이미지와 상관없이 DOM Tree만 생성되면 바로 실행됨
  * 중복으로 사용 가능, 선언한 순서대로 실행됨
- $(window).load(function(){}) => jQuery에서 지원하는 window.onload 방식. window.onload = function(){} 와 동일
  * 화면에 필요한 모든 요소(css, js, image, iframe 등등)들이 메모리에 모두 올려진 다음에 실행됨
  * 한번만 사용 가능, 외부 리소스에도 $(window).load()가 있을 경우 둘중 하나만 실행됨


- window객체는 브라우저의 최상위 객체
- document 객체는 window의 자식객체
- 쉽게 말해 window객체는 브라우저의 주소창, 즐겨찾기 등등까지 모두 포함(당연히 document객체 포함)하고 있고 document 객체는 웹페이지만 담당하고 있음