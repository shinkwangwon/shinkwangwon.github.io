---
layout: post
title: "Spring 커맨드 객체(Command Object)"
tags: [spring]
comments: true
date: 2019-11-26
---

## 스프링 커맨드 객체 (Command Object)
1. 스프링 컨트롤러 단에서 커맨드 객체 사용
    - 스프링에서 커맨드 객체란 HTTP 요청에 들어오는 속성값들을 자동으로 객체에 매핑해준 객체를 말한다.
    - 즉, @RequestParam 을 이용해 각 속성값들을 하나씩 일일이 명시하는게 아니라 해당 컨트롤러에서 받을 수 있는 속성들을 하나의 클래스에 모아놓고 객체로 선언해 두는 것이다.
    - 그럼 스프링은 요청들을 자동으로 커맨드 객체에 바인딩 해준다. 단, "set" + 속성명으로 이루어진 메소드를 기준으로 하기 때문에 속성마다 setter가 꼭 있어야 한다.
2. 뷰단에서 커맨드 객체 사용
    - RestController에서 어떤 클래스의 객체를 통째로 리턴했을때, 뷰단에서는 클래스명의 첫글자만 소문자로 바꾼 이름으로 사용가능하다.
    - 해당 클래스에는 getter가 설정되어 있어야 하며, getter의 이름으로 뷰단에 내려감

## @ModelAttribute
- Spring의 컨트롤러단에서 Request Parameter로 @ModelAttribute 와 @RequestParam, @RequestBody 등을 주로 쓴다.
- 여기서 @ModelAttribute는 객체타입을 요청 파라미터로 받을 경우에 사용하는데 이 @ModelAttribute에 선언한 객체를 커맨드 객체라고 말한다.
- @RequestParam는 요청파라미터와 메소드의 파라미터가 1:1로 매핑되는 경우에 사용되고, 오브젝트 프로퍼티에 한번에 바인딩해서 요청파라미터를 받을 경우 @ModelAttribute를 사용한다.
- @RequestParam, @ModelAttribute 어노테이션은 생략이 가능한데, 단일 변수인 경우에는 @RequestParam으로 간주되고 오브젝트형태인 경우에는 @ModelAttribute로 간주된다. 하지만 명확히 나타내기 위해 쓰는게 낫다.
- @RequestBody는 요청 본문(body)를 통해 요청 파라미터를 받을 때 사용한다. @RequestParam 와 @ModelAttribute는 요청파라미터를 URL 쿼리스트링에 전달한다.

## @RequestMapping 
- 속성 종류
    - String[] value : URL패턴을 지정하는 속성, String배열로 여러개 지정 가능
    - RequestMethod[] method : HTTP 메서드 정의 GET, POST, HEAD, OPTIONS, PUT, DELETE, TRACE 중 배열로 사용 가능
    - String[] params : Request의 파라미터를 매핑조건으로 부여
    - String[] produces : Response의 Content-Type을 제어할 수 있는 속성  

    ```java
    @RequestMapping(value = "/test", method = RequestMethod.POST, produces = "text/html;charset=UTF-8")

    // POST방식으로 /test1, /test2 로 오는 요청을 둘다 받음 
    @RequestMapping(value = {"/test1", "/test2"}, method = RequestMethod.POST, produces = "text/html;charset=UTF-8")

    // GET 방식의 요청 파리미터에 useYn 값이 Y 인 경우에만 아래 URL이 호출됨
    @RequestMapping(value = "/test", method = RequestMethod.GET, params="useYn=Y")


    @RequestMapping(value = "/test", method = RequestMethod.GET)
    public Person getPerson(PersonRequest personRequest) {  // personRequest가 command Object 이다. (@ModelAttribute 생략)
        // personRequest객체를 통해 Person 조회 후 리턴
    }

    ```

