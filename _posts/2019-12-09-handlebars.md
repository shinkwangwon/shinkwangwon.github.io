---
layout: post
title: "Handlebars"
tags: [javascript]
comments: true
date: 2019-12-09
---


## Hadlebars.js
- HTML파일을 만들기 위한 템플릿으로 사용되는 라이브러리

## 기본적인 사용법
```html
<script id="optionDivId" type="text/x-handlebars-template">
    <div>
        <select>
            <option value="">상세유형을 선택해 주세요.</option>
            {{#optionList}}
                <option value="{{optionSeqno}}" >{{optionName}}</option>
            {{/optionList}}
        </select>
    </div>
</script>
```

```javascript
// 위에 선언한 핸들바의 HTML을 가져옴
var handlebarSource = $("#optionDivId").html();

// 컴파일한 템플릿
var handlebarTemplate = Handlebars.compile(handlebarSource);

// 바인딩할 데이터 세팅
var optionList = [
    {optionSeqno: 1, optionName: "옵션1"},
    {optionSeqno: 2, optionName: "옵션2"},
    {optionSeqno: 3, optionName: "옵션3"}
];

// 핸들바 템플릿에 데이터 바인딩해서 HTML 생성 -> 위에서 만들어놓은 HTML파일에 optionList가 루프돌면서 <option>태그를 생성함
var handlebarTemplateHtml = handlebarTemplate({optionList: optionList});

// DOM의 원하는 곳에 붙임
$('body').append(handlebarTemplateHtml);

```

## Handlebars의 장점
- precompile할 수 있기 때문에 성능개선에 도움이 된다.
- Helper를 자유롭게 추가할 수 있다.
- JADE와 같은 문법에 비해 친숙하면서도 가독성이 높다.
- Grunt 등으로 빌드 자동화에 precompile 과정을 포함시키기 쉽다.
- 템플릿에는 로직 코드를 넣지 않는 것이 일반적이다.

## Handlebars의 단점
- 컴파일한 템플릿에서 오류가 발생했을 때 디버깅도 어렵다.
- 실제 문법에서도 간단한 분기문, 배열, 반복문 정도를 지원한다. -> 장점일수도 ?

 


#### 참고
- <https://ijbgo.tistory.com/8>