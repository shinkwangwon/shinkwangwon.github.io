---
layout: post
title: "React VirtualDOM"
tags: [react]
comments: true
date: 2019-12-12
---

## 브라우저의 Workflow
1. DOM Tree 생성
- 브라우저가 HTML을 전달 받으면, 브라우저의 렌더 엔진이 이를 파싱하고 DOM노드로 이루어진 DOM Tree를 만든다.
> 각 DOM노드는 HTML 엘리먼트와 연관되어져 있다.

2. Render Tree 생성
- 외부 CSS파일과 각 HTML 엘리먼트의 inline 스타일을 파싱한다. 스타일 정보를 사용하여 Render Tree를 생성한다.
- Webkit에서는 스타일을 처리하는 과정을 attachment라고 부른다. DOM트리의 모든 노드들은 'attach'라는 메소드가 있고 이 메소드는 스타일 정보를 계산해서 객체 형태로 반환한다.
- 이 과정은 동기적으로 이루어지는 작업이고 DOM트리에 새로운 노드가 추가되면 그 노드의 attach메소드가 실행된다.
- Render Tree를 만드는 과정에서는 각 요소들의 스타일이 계싼되고 이 과정에서 다른 요소들의 스타일 속성을 참고한다.
> Webket 이란 HTML, CSS, 자바스크립트로 이루어진 웹컨텐트를 렌더링하는 엔진을 말한다.

3. Layout (reflow 라고도 부름)
- Render Tree가 다 만들어지고 나면 각 노드들의 스크린좌표가 주어지고 어디에 나타내야할지 결정하는 작업을 수행한다.
- 웹브라우저에 렌더링을 한 후에, 노드의 레이아웃 변경시 영향 받은 모든 노드의 수치를 다시 계산하여 렌더트리를 재생성하는 과정을 말하기도 한다. (reflow)

4. Painting
- CSS를 연산해서 렌더링된 요소들에 색을 입히는 과정이다.


## 일반 DOM의 성능저하
- DOM에 변화가 생기면 Render Tree를 재생성하고 모든 요소들의 스타일이 다시 계산되고, 레이아웃 만들고 페인팅하는 과정을 반복하게 된다.
- 많은 DOM요소가 있는 페이지에서는 DOM조작이 많이 발생하게 되고 그 변화를 적용하기 위해 브라우저가 많은 연산을 해야하고 그만큼 전체적인 프로세스를 비효율적으로 만든다.
- 일반적인 DOM 조작은 트리변화, 레이아웃 변화와 리렌더링을 일으킨다. 예를 들어, 30개의 노드를 하나하나 수정하면 30번의 레이아웃 재계산과 30번의 리렌더링을 초래한다. 즉 Reflow과정과 Repaint 연산이 빈번해지기 때문에 성능을 저하 시킨다.
> repaint는 색상이 변경되거나 글자의 내용이 바뀌었을 때 발생되는 연산이다. reflow는 하나의 DOM객체의 크기나 위치가 변경되었을 때(레이아웃의 변경작업), 다른 DOM객체들의 위치와 크기가 함께 변경하는것을 말한다.


## VirtualDOM 이란
- VirtualDOM을 이용하면 실제 DOM에 접근하여 조작하는 대신에, 이를 추상화시킨 자바스크립트 객체를 구성하여 사용하여 DOM의 상태를 메모리에 저장하고 변경 전과 변경 후의 상태를 비교한 뒤 최소한의 내용만 반영 하는 기능
- VirtualDOM은 DOM에 변화가 일어났을 때 변화된 것을 Virtual DOM에 먼저 적용시킨 후에 연산이 모두 끝나면 최종적인 변화를 실제 DOM에 던져주는데 이러한 과정을 딱 한번만 수행한다.
- 즉, 일련의 변화를 하나로 묶어서 Virtual DOM에 적용시킨 후에, 기존의 DOM과 비교해서 변화된 부분만 리렌더링하도록 하는 작업을 **딱 한번만** 수행하는 것이다.
- VirtualDOM은 DOM의 상태를 메모리 위에 계속 올려두고 DOM에 변경이 있을 때만 해당 변경을 반영한다.

## VirtualDOM 장점
- 위에서 말한 것처럼 Virtual DOM에 변화를 먼저 적용시킨 후에 최종적인 변화를 한번만 딱 실제 DOM에 던져주는 것이 가장 큰 장점이다.
- 이러한 작업은 DOM fragment를 이용하여 비슷하게 구현이 가능하다. 하지만 DOM fragment를 관리하는 것을 수동으로 작업을 해주어야 한다.
- VirtualDOM은 이러한 과정을 자동화 해준다. 또한 DOM fragment를 관리하려면 기존 값에서 어떤 값이 바뀌었는지 안바뀌었는지 계속 파악하고 있어야 하는데 Virtual DOM은 이러한 것 또한 자동화 해준다.
- DOM관리를 VirtualDOM이 하도록 함으로써, 컴포넌트가 DOM조작 요청을 할 때 다른 컴포넌트들과 상호작용을 하지 않아도 되고(특정 DOM을 조작할거다 라는 정보 공유가 필요없음), 각 변화들의 동기화 작업을 
> DocumentFragment 란, 웹 페이지에 객체를 생성할 때 생성 객체를 Body 에 넣기 전에 미리 만들어 두는 것을 말한다.
> DocumentFragment는 일반적으로 DOM 서브트리를 조립해서 DOM에 삽입하기 위한 용도로 사용되고, 이 때 appendChild()나 insertBefore() 같은 Node 인터페이스의 메소드가 사용됩니다. 이 작업은 fragment의 노드들을 DOM으로 이동시키고, DocumentFragment를 비웁니다. 모든 노드들이 한꺼번에 문서에 삽입되기 때문에, 한 번의 리플로우와 렌더링만 일어납니다. 노드를 개별적으로 삽입할 때 매 노드마다 한 번씩 일어나는 것과는 대조적입니다.
> https://developer.mozilla.org/ko/docs/Web/API/DocumentFragment

--------

# React VirtualDOM

## ReactDOM.render()
- VirtualDOM은 React객체의 트리다. ReactDOM.render() 함수를 호출하면 VirtualDOM을 만들기 시작한다.
- 이때 생성하는 컴포넌트는 주로 ReactCompositeComponent 객체와 ReactDOMComponent 객체다.
- ReactCompositeComponent 객체는 **DOM이 아닌 컴포넌트**를 생성할 때 사용된다.
- ReactDOMComponent 객체는 **DOM을 만들때** 생성하는 컴포넌트다.
- render()함수가 생성한 컴포넌트를 React컴포넌트에 마운트하기 위해 ReactReconciler.mountComponent() 를 호출한다.
- ReactReconciler.mountComponent() 메서드에는 실제 ReactCompositeComponent객체와 ReactDOMComponent객체의 mountComponent()메소드를 호출하고 이 시점에 주요 작업이 시작된다.

## VirtualDOM 생성과정(렌더링 과정)
1. ReactCompositeComponent
- constructor() 메소드실행 -> getDerivedStateFromProps() 메소드 실행 -> 그 후에 렌더링을 수행한다.
- 배치 처리 작업에 메서드나 속성을 등록한다. componentDidMount()메소드가 있으면 이 메소드를 등록하고 ref 속성이 있으면 attachRefs속성을 등록한다.
- 하위에 ReactComponent 객체가 있으면 ReactComponent 객체를 생성하고 다시 ReactReconciler.mountComponent()메서드를 실행한다.

2. ReactDOMComponent
- 컴포넌트가 생성되면 실제 DOM과 함께 ReactDOMComponent객체가 생성된다. 실제 DOM이 생성된다는 말이다.
- style 속성과 attr 속성을 추가한다.
- 배치 처리 작업에 사용자 이벤트를 등록한다.
- 하위에 ReactComponent 객체가 있으면 ReactComponent 객체를 생성하고 다시 ReactReconciler.mountComponent()메서드를 실행한다.
- 최상위 DOM에 DOM을 추가한다.

> componentDidMount() 메소드 같은 경우 항상 모든 컴포넌트의 작업이 끝난 후 발생하도록 되어 있기 때문에 큐(queue)에 등록하고 나중에 한번에 발생시킨다. 이러한 작업을 배치 처리작업에서 수행한다.

## Reconciliation 작업
- VirtualDOM을 만든 후 Virtual DOM을 갱신할 때 Reconciliation 작업을 한다. 이 작업은 VirtualDOM과 실제 DOM을 비교해 실제 DOM을 갱신하는 작업이다.

## VirtualDOM 갱신 방법
1. setState()
- setState() 메소드는 해당 컴포넌트를 변경대상 컴포넌트로 등록해서 갱신하는 방법이다. setState() 메소드를 호출하는 컴포넌트를 기준으로 갱신 한다.
- this.state="AA" 방식으로 state를 변경할 경우에는 변경대상 컴포넌트로 등록되지 않기 때문에 컴포넌트가 갱신되지 않는다.

2. render()
- Redux에서처럼 스토어가 변할 때 다시 최상위 컴포넌트의 render() 메소드를 호출해 최상위 컴포넌트를 변경 대상 컴포넌트로 등록해서 갱신하는 방법이다. 최상위에서 시작해서 갱신이 필요한지 확인한다.
- render() 메소드는 store가 변경됐을 때 Virtual DOM을 갱신하는 방법이다.
- Redux를 사용할 경우 하나의 store를 가지고 있기 때문에 store가 변경됐을 때 다시 최상위 컴포넌트의 render()함수를 호출해 Virtual DOM을 갱신하는 작업을 한다. setState()와의 차이점은 render()는 최상위 컴포넌트부터 비교한다는 점이 다르다.


## VirtualDOM 갱신 유형
1. ReactComponent 객체의 상태가 변경될 때 하위에 있는 ReactComponent객체를 새로 만들지 않고 속성만 갱신하는 유형
- ReactCompositeComponent 객체는 VirtualDOM이 같다면 속성만 갱신하고, 새로운 VirtualDOM이거나 VirtualDOM이 다르다면 기존의 VirtualDOM의 마운트를 해제하고 새로운 Virtual DOM을 만드는 작업을한다.
- ReactDOMComponent 객체는 실제 DOM에서 변경이 필요한 부분을 갱신하는 작업을 한다. 변경된 ReactComponen 객체와 VirtualDOM이 같다면 속성만 갱신한다. props를 비교해 style속성과 event속성등을 비교해서 갱신한다.

2. 비교하는(=변경된) ReactComponent객체가 Virtual DOM이 다르다면 변경된 Virtual DOM의 마운트를 해제하고 모두 새로 만들어 갱신하는 유형
- 먼저 기존 Virtual DOM의 마운트를 해제하고 Virtual DOM을 생성하며 자식 Virtual DOM이 있다면 Virtual DOM을 새로 생성해서 갱신한다.



#### 참고
- <https://velopert.com/3236>
- <https://linuxism.ustd.ip.or.kr/1571>
- <https://d2.naver.com/helloworld/9297403>