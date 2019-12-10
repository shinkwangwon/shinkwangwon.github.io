---
layout: post
title: "AngularJS Context"
categories: angularjs
date: 2019-10-22
---


## AngularJS Context
- 브라우저는 사용자 동작에 반응하기 위해 이벤트 루프를 돌면서 이벤트에 반응한다.
- angularJS 에서는 이벤트를 관리하기 위해 angular context라는 것을 만들어서 이벤트 루프를 확장하여 사용한다.


### $watch, $digest, $apply
1. $watch
- UI와 무언가를 바인딩하게 되면 항상 $watch list에 넣고 $watch는 이 리스트에 대한 변경을 감시한다.
- 디렉티브를 만들때도 바인딩이 있으면 $watch가 생성되는데, 디렉티브의 link절차에서 $watch가 생성된다.
  > - directive compile 단계 : HTML의 DOM 엘리먼트들을 돌면서 디렉티브를 찾는다. (attribute name, tag name, comments, class name을 이용하여 디렉티브를 매칭시킨다.) 결과로 link function을 리턴한다.
  > - directive link 단계 : 디렉티브와 HTML이 상호작용(동적인 view) 할 수 있도록 디렉티브에 event listener를 등록하며 scope와 DOM 엘리먼트간에 2-way data binding을 위한 $watch를 설정한다. 위의 HTML Compiler의두 단계를 거쳐 HTML에서 디렉티브를 사용할 수 있게 됩니다.
- $scope.$watch('name', function() {} ) 을 통해, 특정 모델을 지정하여 해당 모델이 변경되었을때 호출할 함수를 지정할 수 있음

2. $digest
- 브라우저가 angluar context에 의해 관리되는 이벤트를 받게되면 $digest loop가 작동한다.
- $digest loop는 $watch list를 루핑돌면서 모델의 변경을 체크하고 $watch에 등록된 이벤트 리스너 핸들러를 수행한다.
 이때 dirty-checking이 이루어지는데, 하나가 변경되면 모든 $watch list를 루핑돌고 다시 체크해보고 변화가 없을때까지 루핑을 돈다(그러나 무한루프 방지를 위해 기본적으로 최대 10번의 루핑을 돈다고 함)
- 즉, 모델 변경 감지시 즉시 DOM반영이 아닌 $digest loop가 끝났을때 DOM을 업데이트 함
- angular context 안으로 들어간 모든 이벤트는 $digest loop를 수행함

3. $apply
- 이벤트가 발생할때 $apply가 호출되면 이벤트는 angular context 안으로 들어간다
- 만약, $apply를 호출하지 않으면 angular context 안으로 들어가지 못하고 밖에서 이벤트를 수행하게 된다
- angular context 밖에서 이벤트를 수행한다는 말은, $digest loop를 돌지않게되고 $watch도 수행되지 않으니 결국 DOM의 변경이 발생하지 않는다는 말이다.
- (angular context밖에서 이벤트를 수행하면 모델의 값은 변경되어져 있으나 DOM에는 변경된 값을 그리지 못함)
- angularJS에 이미 만들어져 있는 디렉티브들(ng-click, ng-change, ng-model 등)은 이벤트를 $apply 안에 랩핑한다. 따라서 이벤트를 맵핑만 해놓으면 자동으로 DOM이 업데이트 되는 것이다
- jQuery를 사용하는 경우 이러한 $apply를 호출하지 않기 때문에 angular context 안으로 못들어가게 되고 $digest loop가 돌지 않으니 DOM이 변경되지 않는 것이다.(모델의 값은 변경할 수 있으나 화면에만 못그리는 것임)


### 정리
- angularJS 에서는 브라우저가 이벤트에 반응하도록 하기위해 angular context라는 것을 만들어서 이벤트를 관리한다.
- 이벤트 발생시 $apply를 수행하여 이벤트가 angular context안으로 들어가서 $digest loop를 돌면서 $watch를 통해 모델에 변경된것이 있는지 확인한다.
- $apply가 수행되지 않으면 이벤트가 angular context 안으로 들어가지 못하고 $digest loop를 수행하지 못해서 DOM을 업데이트하지 못한다.
- (ex: jQuery를 사용하여 이벤트(click등)를 처리하게되면 $apply를 수행하지 못하고 DOM업데이트 못함 -> jQuery click이벤트를 통해 자바스크립트 단에서 모델의 값은 업데이트 할 수 있으나 화면에 그리지를 못하는 것뿐)
- angularJS에 이미 만들어져있는 디렉티브(ng-click 등)는 이벤트를 $apply로 랩핑하고 있기 때문에 자동으로 $apply 가 수행되고, DOM까지 업데이트 하는 것이다.    


#### 참고
- <http://www.nextree.co.kr/p8890/>
- <https://mobicon.tistory.com/328>
