---
layout: post
title: "React 요약"
tags: [react]
comments: true
date: 2020-01-28
---

## React
- 컴포넌트라는 것에 집중되어 있는 라이브러리 (프레임워크가 아님)
- 사용자에게 전달되는 뷰만 신경쓰고, 나머지 기능은 서드파티 라이브러리에서 처리함(redux, react-router 등)
- 공식 라이브러리 개념이 없음, 한가지 문제를 해결하기 위해 여러 방식이 있음
- 공식 라이브러리 개념이 없기에 redux, react-router 등 서드파티 라이브러리를 이용하는데 redux, react-router 이러한 것들이 여러종류임


## 리액트의 Virtual DOM
- 모델에 변화가 일어나면 브라우저의 DOM에 새로운 것을 넣는 것이 아니라, 자바스크립트로 이루어진 가상의 DOM에 한번 렌더링하고 기존의 DOM과 비교한 후, 변화가 필요한 곳만 업데이트 하도록 함
- 참고 : ( [React VirtualDOM](https://shinkwangwon.github.io/react-virtual-dom/)


## JSX 문법 알아보기
- 참고 JSX 문법 규칙 https://react-anyone.vlpt.us/03.html
- HTML이랑 비슷해보이지만 javascript로 변환됨
- 규칙이 몇가지 있음
  * 태그는 꼭 닫혀있어야 함 ex: <div> 태그를 열었으면 반드시 </div> 로 닫아주어야 함. <input type="text"> 에서 실수를 많이함 <input type="text"/> 이렇게 self closingTag 를 붙이도록 함
  * 두개 이상의 엘리먼트는 무조건 하나의 엘리먼트로 감싸져 있어야 함
  * JSX 안에서 자바스크립트 값 사용할 때는 {} 를 이용
  * render() 메소드에서는 반드시 jsx를 리턴해줘야함
  * Style 을 사용할 때는 Style안의 속성명은 camelCase로 작성해야함

```javascript
import React, { Component } from 'react';

// React.Component를 상속하는 클래스는 반드시 render()메소드를 정의해야 함
class App extends Component {
    render() {
        {/* style를 사용할때는 속성명을 camelCase로 작성해야함 */}
        const style = {
            backgroundColor: 'black',
            color: 'white',
            padding: '16px',
            fontSize: '36px'
        };
        const name = 'shin';
        const value = 2;
        
        return ( // rendor() 메소드 안에서는 반드시 JSX 리턴 해주어야 함
            <div style={style}>
                {/* if 없이 바로 사용할 때 */} // jsx 안에서의 주석은 {/* */} 와 같이 사용
                {name === 'shin' && <div>hihi</div>} // 자바스크립트를 사용할때는 { } 안에서 사용
                
                {/* IIFE 방식 - 메소드 만들자마자 바로 호출 */}
                {(function() {
                    if (value === 1) return <div>1이다</div>;
                    if (value === 2) return <div>2이다</div>;
                    return <div>없다</div>;
                    })()
                }
            </div>
        );
    }
}

export default App;
```

## Props 란 ?
- props란 부모컴포넌트가 자식컴포넌트한테 값을 전달할 때 사용
- props는 읽기 전용
- 함수형 컴포넌트와 클래스형 컴포넌트의 차이점

  * 함수형 컴포넌트
    - 함수형 컴포넌트는 초기 마운트 속도가 미세하게 조금 빠름
    - 불필요한 기능이 없기 때문에 메모리 자원도 덜 사용함
    - 컴포넌트가 단순히 값을 받아와서 보여주기만 하는 용도일 때 주로 함수형 컴포넌트로 만들어 사용
    - 컴포넌트 내의 state를 생성해서 관리하거나 react life cycle을 사용하는 것이 불가능 => react 16.8 이상버전의 hooks 기능을 이용해 가능하단다.
  * 클래스형 컴포넌트
    - React.Component를 상속해야하며 render() 를 반드시 구현해야 함
    - state와 life cycle 사용 가능

```javascript
import React, { Component } from 'react';

//// 클래스형 컴포넌트
class MyName extends Component {
    // 기본 props 설정 - 반드시 defaultProps 라는 이름을 가져야함
    static defaultProps = {
        name: '기본이름1'
    };
    render() {
        return <div>Hi! {this.props.name}</div>;
    }
}


//// 함수형 컴포넌트
// const MyName = ({ name }) => {
//      return <div>Bye! {name}</div>;
// };
// MyName.defaultProps = {
//      name: '기본이름2'
// };

export default MyName;
```

## State 란 ?
- state는 컴포넌트 자기 자신이 들고 있는 것
  * state는 반드시 객체형태 이어야 함
  * state가 바뀔때마다 컴포넌트는 리렌더링이 됨
  * state는 내부에서 변경할 수 있는데, 변경할 때는 setState라는 함수를 사용한다 -> setState를 사용하지 않고 직접변경하면 객체의 값을 바꿀 수는 있으나 리렌더링이 안됨. 즉, 화면에 바뀐 값으로 다시 그리지 않음.
  * props 는 읽기전용이고, state는 변경가능한 것

```javascript
import React, { Component } from 'react';

class Counter extends Component {
    // state는 반드시 객체이어야 함 - 문자열 안됨
    state = {
        number: 0
    };
    
    handleIncrease = () => {
        // state 변경은 반드시 setState를 이용
        this.setState({
            number: this.state.number + 1
        });
    };


    // arrow 메소드를 이용하지 않으면 메소드 내부에서 this가 뭔지 모름 (ES6 arrow this 참고 : https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Functions/%EC%95%A0%EB%A1%9C%EC%9A%B0_%ED%8E%91%EC%85%98
    // arrow 메소드를 사용하게 되면 this 는 해당 arrow메소드를 가진 클래스(여기선 Counter)를 this로 알고 있음
    handleDecrease = () => {
        this.setState({
            number: this.state.number - 1
        });
    };
    
    // arrow 방식을 안쓸거면 아래와 같이 사용
    // constructor(props) {
    //      super(props);
    //      this.handleDecrease = this.handleDecrease.bind(this); // arrow방식 안쓰면 this.handleDecrease을 사용할때 this가 뭔지 모르기때문에 handleDecrease에 this를 바인딩해줘야함
    // }
    // handleDecrease() {
    //      this.setState({
    //          number: this.state.number - 1
    //      });
    // }
    
    render() {
        return (
            <div>
                <h1>카운터</h1>
                <div>값: {this.state.number}</div>
                <button onClick={this.handleIncrease}>+</button>
                <button onClick={this.handleDecrease}>-</button>
            </div>
        );
    }
}

export default Counter;
```


## React LifeCycle
1. Mounting : 컴포넌트가 처음 브라우저 상에 나타날 때 실행
- constructor : 컴포넌트가 처음 브라우저 상에 나타날때 실행됨
- getDerivedStateFromProps : props로 받은 값을 state로 동기화시킬 때 사용(state 상태를 변경하고 싶을 때)
  * 여기서는 setState를 하는 것이 아니라 props가 바뀔때 설정하고 싶은 값을 리턴하기만하면 자동으로 state에 들어감. null을 리턴하면 업데이트 할것은 없다라는 것을 의미함.

2. Updating : 컴포넌트에 변화가 일어났을 때 수행
- **shouldComponentUpdate : 중요!** 컴포넌트가 업데이트되는 성능을 최적화시킬때 사용. 부모컴포넌트가 렌더링 되면 자식컴포넌트도 렌더링되게 되어 있으나, shouldComponentUpdate를 잘 사용하면 렌더링하지 않도록 설정할 수 있음
- shouldComponentUpdate를 잘 사용하면 Virtual DOM 에 그리는 것조차 막을 수 있음 - true를 반환하면 렌더링 프로세스를 거치고 false를 반환하면 렌더링 프로세스를 수행하지 않음
- render : 브라우저 보여주기 위해 렌더링 결과물을 만들어내는 함수
- componentDidMount : 외부 라이브러리를 사용하게 될 때(차트 등) 또는 컴포넌트가 나타난 후에 어떠한 작업을 하고 싶을 때(Ajax요청, 이벤트리스너 설정 등), DOM관련설정(스크롤위치)등에 사용. 즉 컴포넌트가 브라우저에 나타난 시점에 처리할 작업 수행
- getSnapshotBeforeUpdate : 렌더링을 한다음에 결과를 브라우저상에 반영되기 직전에 호출되는 함수. 렌더링 후 현재 스크롤의 위치 등을 지정하고 싶을 때 사용
- componentDidUpdate : state가 바뀌었을 때, 직전 state의 상태와 바뀐 state의 상태를 비교하여 어떤 작업을 수행할 때 주로 사용

3. Unmounting : 컴퍼넌트가 브라우저에서 사라질때 수행
- componentWillUnmount : 컴포넌트가 사라질때 수행되는 함수 - componentDidMount에서 설정한 이벤트리스너를 제거하거나 하는 작업할때 주로 사용

4. 예외
- componentDidCatch : 컴포넌트에 에러가 발생할 때 수행됨. 에러가 발생할 수 있는 컴포넌트의 부모 컴포넌트에서 componentDidCatch로 처리해줘야함
- 참고 : https://ko.reactjs.org/docs/react-component.html



## 액션
- 액션이란 작업에대한 정보를 지니고 있는 '객체'
- 액션이름은 대문자와 언더바를 이용해 작성
- 액션객체가 필수로 가지고 있어야 하는 속성은 type 속성임

```javascript
{
    type: "SET_COLOR", // 필수
    color: [200, 200, 200] // 선택
}
```

## 액션 생성자
- 액션생성자를 사용하면 위와 같이 일일이 설정해줄 필요 없이 편리하게 액션생성 가능
- 액션생성자는 함수
  * 1) 먼저 액션이름을 상수화 함 (ActionTypes.js) : export const SET_COLOR = "SET_COLOR";
  * 2) 액션이름을 import 해서 액션생성자 작성 (index.js)

```javascript
import * as types from './ActionTypes';

export function setColor(color) { // 외부에서 액션을 가져다 쓰도록 export 함
    return {
        type: types.SET_COLOR,
        color // 이렇게만 쓰면 color: color 과 동일한 의미. 오른쪽 color이 파라미터로 넘어온 값임
    };
}
```



## 리듀서 (리듀서는 함수임)
- 비동기 작업을 하면 안되고, 인수변경하면 안되고, 동일한 인수를 받으면 동일한 결과를 리턴해야함
- 이전 상태와 액션을 받아서 다음 상태를 반환함 (prevState, action) => newState
- 기존 상태를 복사하고 변화를 준 다음에 반환
- 리듀서 정의

1. 리듀서의 초기상태를 정의

```javascript
const initialState = { // 상수 형태로 정의해놓음
    number: 0,
    name: 'shin'
};
```

2. 리듀서를 정의 (옛날방식임 - 새로운방식으로는 redux-actions의 createAction과 handleActions를 이용 - 아래 redux-action 참고)
```javascript
export default function counter(state = initialState, action) { // 이전상태값인 state와 action을 파라미터로 받음

switch(action.type) {
    case types.INCREMENT :
        // spread 연산을 이용하면 일단 state객체 안에 있는 값들을 다 가져옴. number, name을 일단 모두 가져온 후에, number 값 업데이트 - number값만 업데이트하고 객체를 return하면 name속성을 잃게 됨
        return {...state, number: state.number + 1};
    }
}
```

3. 리듀서를 합침 : 여러개의 리듀서를 정의했으면 리듀서를 하나로 합침 (redux의 combineReducers를 이용)
```javascript
import { combineReducers } from 'redux';
const reducers = combineReducers({정의한 리듀서들 나열});
export default reducers;
```



## Store
- 스토어가 하는일은 dispatch(action) - 액션을 리듀서로 보내는 것
- dispatch가 실행되면 스토어는 리듀서에 현재 자신의 상태와 전달받은 액션을 리듀서로 보냄 -> 그럼 리듀서가 어떤 변화가 필요한지 보고 새 상태를 리턴해줌
1. store.dispatch() 메소드 인자로 액션을 넘길 경우, 스토어는 액션을 리듀서로 보내서 실행하게 함

```javascript
// setMyAgentInfo는 액션
export const setMyAgentInfo = createAction(types.SET_BASE_MY_AGENT_INFO);

// store.dispatch() 인자로 액션을 넘김
store.dispatch(baseAction.setMyAgentInfo(data));

// 스토어가 리듀서로 액션을 넘겨서 리듀서가 수행됨
export default handleActions({
    [types.SET_BASE_MY_AGENT_INFO]: (state, { payload: myInfo }) => {
        //console.log('reducer : base : SET_BASE_MY_INFO', myInfo);
        return state.set('myAgentInfo', Map({
            myAgentId: myInfo.userId,
            myAgentMaxChatCnt: myInfo.maxChatCnt,
            ...
        }))
    },
}, initialState)
```

2. store.dispatch() 메소드 인자로 함수를 넘길 경우, redux-chunk 미들웨어가 함수인것을 인지하여 dispatch, getState를 넣어서 실행해줌

```javascript
// store.dispatch() 인자로 함수를 넘김
store.dispatch(baseAction.getBaseApi('agents'));

// dispatch전에 redux-chunk미들웨어가 호출되고, 이 redux-chunk는 인자로 받은 함수(getBaseApi)를 수행할때 dispatch, getState를 인자로 넣어서 실행해 줌
export const getBaseApi = (code) => {
    return (dispatch, getState) => { // dispatch, getState 사용 가능해짐
        const baseApi = BASE_API_TYPES.get(code);
        apiGet(baseApi.url)
            .then((response) => {
                const { httpStatus, data } = response.data;
                if (httpStatus === 'OK') {
                    dispatch({ // 조건에 따라 dispatch할지 말지 결정할 수 있음
                    type: baseApi.type,
                    payload: data
                    });
                } else {
                    alert(`${baseApi.name} 정보 로딩 실패!!\n페이지 새로 고침 해주세요(Ctrl + F5)`);
                }
            }).catch((err) => {
                console.error(baseApi.url, err);
                alert(`${baseApi.name} 정보 로딩 실패!!\n페이지 새로 고침 해주세요(Ctrl + F5)`);
            }
        );
    };
};
```

## FLUX 란 ?
- FLUX : 추상적인 개념 (라이브러리나 프레임워크나 이런게 아님)
  * Action -> Dispatcher -> Store -> View
  * Action이 발생하면 Dispatcher가 받아서 Store를 업데이트함. 변동된 데이터가 있으면 View를 리렌더링함
  * 그리고 View에서 Store에 직접 접근하지 않고, View에서 Dispatcher로 Action을 보내고, Dispatcher은 작업이 중첩되지 않게 해줌. 즉 Dispatcher은 이미 어떤 Action이 처리중이면 새로 들어온 Action을 대기시킴
- 액션생성자(Action Creator)
  * 액션은 어떤 부분이 업데이트되어야 하는지에 대한 액션을 포맷에 맞게 만들어서 디스패쳐에 넘겨줌 (디스패쳐가 이해할 수 있는 액션으로 정의해줌 )
- 디스패쳐(Dispatcher)
  * 교환원과 같음 -> 액션을 받으면 액션을 읽고 업데이트해야 할 부분을 정하고 Store로 알려줌
  * 디스패쳐는 동기적으로 실행됨 -> 여러 액션이 들어오면 우선적으로 처리되어야 하는 액션부터 순서적으로 처리함(그래서 꼬이지 않음)
- 스토어(Store)
  * 정부관료와 같음 -> 모든 상태와 그와 관련된 로직을 지니고 있음
- 뷰(View)
  * 발표자


## Redux란 ?
- Redux란 FLUX 아키텍쳐를 구현한 라이브러리
- 컴포넌트들끼리 데이터 교류 및 state관리를 쉽고 효율적으로 하기 위한 라이브러리
- dispatch란 store를 업데이트하는것
- subscribe란 store를 지켜보고 있다가 store가 업데이트 되면 컴포넌트에 바로 반영시킴
- redux가 있으면 store를 업데이트하면 바로 컴포넌트 업데이트가능 (만약 redux가 없다면 다른 컴포넌트한테 일일이 store에 변동이 일어났다는 것을 알려줘야함)
- Redux의 3가지 원칙
  * Single Source of Truth
    - 어플리케이션 state를 위해 단 하나의 store만 사용
    - 반면 FLUX는 여러개의 store를 사용
  * State is Read-only
    - 어플리케이션에서 store의 state를 직접 변경할 수 없음
    - state를 변경하기 위해선 무조건 action이 dispatch되어야 함
  * Changes are made with pure Functions
    - action객체를 처리하는 함수를 reducer라고 부름
    - reducer는 정보를 받아서 상태를 어떻게 업데이트할지 정의함
    - reducer는 '순수함수'로 작성되어야 함
    - 즉, 네트워크 및 데이터베이스 접근X, 인수변경X 해야함. 또한 같은 인수로 실행된 함수는 언제나 같은 결과를 반환해야하고, reducer안에서 순수하지 않은 API는 사용불가(Date.now(), Math.random()등)
- 액션 생성자 : 액션 정의 (액션이란 작업에대한 정보를 지니고 있는 객체)
- 스토어 : 단 하나의 스토어를 가짐
- 리듀서 : 변화를 일으키는 함수를 일컬음. 리듀서는 여러개 있을 수도 있는데, 모든 리듀서를 관리하는 root reducer가 존재함
  * 리듀서는 예전 상태를 변경하지 않고, 새로운 복사본을 만든 후 거기에 모든 변경사항을 적용함
- 뷰 : 영민한 뷰, 멍청한 뷰
- 데이터 흐름
  * 1) 뷰가 액션을 요청하면 액션 생성자가 액션을 만들어줌
  * 2) 뷰는 스토어한테 액션을 전달
  * 3) 스토어는 루트 리듀서한테 액션을 전달
  * 4) 루트 리듀서는 서브 리듀서들한테 액션을 전달하고, 서브 리듀서들은 새로운 상태를 만들고 거기에 변경사항을 적용한 후 루트리듀서로 리턴함
  * 5) 루트 리듀서는 서브 리듀서들로부터 결과물을 받아 스토어에 전달
  * 6) 스토어는 뷰 레이어 바인딩에게 데이터가 변경된 것을 알리고 데이터를 전달함 (뷰 레이어 바인딩은 react-redux를 말함)
  * 7) 뷰 레이어 바인딩은 뷰에게 화면을 업데이트하도록 요청함

## react-redux
- react-redux가 제공하는 기능중 하나인 Provider 컴포넌트
- <Provider> : provider 컴포넌트는 connect() 함수를 사용하여 "연결(connect)"할 수 있도록 앱의 store를 "제공(provide)"함. (Provider 는 컴포넌트에서 리덕스를 사용하도록 서비스를 제공해줌)

```javascript
ReactDOM.render(
    <Provider store={store}> // Provider의 props로 store를 작성
        <ConnectedRouter history={history}>
            <Route path="/" component={App} /> // react-dom에 렌더링할 App 컴포넌트를 Provider컴포넌트로 감싸주도록 함
        </ConnectedRouter>
    </Provider>,
    document.getElementById('root')
);
```

- connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])(연결시킬 컴포넌트) : connet메소드는 react component를 redux store에 연결함.
  * mapStateToProps : 컴포넌트가 변경될 때마다 state의 특정값을 현재 컴포넌트의 props로 가져오는 함수 지정 가능 (여기서의 인자 state는 store를 뜻함)
  * mapDispatchToProps : 컴포넌트가 변경될 때마다 지정한 action을 dispatch하는 함수 지정 가능
  * bindActionCreators(액션생성자, dispatch) : 첫번째 인자로 액션생성자 혹은 액션생성자들이 정의된 모듈을 넣고, 두번째 인자로 dispatch를 넣으면 첫번째인자로 넣어준 모든 액션을 불러와서 이를 dispatch하는 함수로 만들고 난 후, 해당 컴포넌트의 props로 넣어줌

```javascript
...
handleAssignToMe = (sessionId) => {
    const { SendBoxAction } = this.props;
    //console.log('handleAssignToMe', sessionId);
    SendBoxAction.assignToMe(sessionId); // SendBoxAction의 assingnToMe 메소드 호출하면 dispatch까지 수행됨 -> mapDispatchToProps에 bindActionCreators를 사용했기 때문
};

// 여기서 state는 store 객체임
const mapStateToProps = (state) => {
    return {
        // state.base 는 store/reducers/base.js
        myAgentInfo: state.base.get('myAgentInfo'),
        selectedChat: getSelectedChat(state),
        dialog: state.base.get('dialog'),
    };
};

const mapDispatchToProps = (dispatch) => ({
    SendBoxAction: bindActionCreators(sendBoxAction, dispatch), // actionCreator들이 들어있는 객체(여기선 SendBoxAction)를 전달하면 SendBoxAction에 들어있는 모든 액션함수를 불러와서 이를 dispatch하는 함수로 만들어서 컴포넌트의 props의 SendBoxAction라는 객체에 넣어줌
    BaseAction: bindActionCreators(baseAction, dispatch),
});

// mapStateToProps로 인해 redux의 state가 컴포넌트의 props로 매핑되었기 때문에 컴포넌트에서 this.props.myAgentInfo로 사용하면 state의 myAgentInfo를 바로 가져다 쓸 수 있음
// mapDispatchToProps로 인해 bindActionCreators로 인해 this.props.BaseAction로 사용가능
// withStyles은 styles객체를 포함한 classes를 가진 컴포넌트를 리턴해줌 -> styles는 컴포넌트에 연결됨, withTheme를 true로 설정하면 theme object를 컴포넌트의 property로 제공함
export default connect(mapStateToProps, mapDispatchToProps)(withStyles(styles, { withTheme: true })(SendBox));
```

## redux-actions
- createAction : 액션 생성 자동화
- createAction(type, payloadCreator) : 두번째 파라미터인 payloadCreator에 원하는 payload값을 추출하는 메소드를 넣으면 해당 액션에서 필요한 payload만을 설정할 수 있음
  * payload : createAction을 이용해 액션을 생성하면 액션으로 전달받은 파라미터는 모두 payload라는 객체로 전달되도록 되어 있음
- handleActions : 액션핸들러 작성
- 즉, createAction을 통해 액션을 생성하고, handleActions를 이용해 액션에 따른 리듀서를 작성
- 참고 : <https://velopert.com/3358>

```javascript
// 1. 액션 상수 정의 파일
export const SET_SOCKET_STATUS = 'SET_SOCKET_STATUS'; // 액션 상수 정의
...


// 2. 액션 생성 파일 
import { createAction } from 'redux-actions';
import * as types from 'constants/actionTypes';
...
/** 소켓 접속 상태 설정 */
export const setSocketStatus = createAction(types.SET_SOCKET_STATUS); // setSocketStatus라는 이름으로 액션을 생성
...


// 3. 액션 핸들러 생성 파일 
...
const initialState = Map({
    socketStatus: 'ready',
    ...
});
export default handleActions({ // 액션 핸들러 작성
    [types.SET_SOCKET_STATUS]: (state, { payload }) => { // 메소드 파라미터에는 (state, action)이 넘어오는데 action 중에 payload 값만 추출해서 사용
        //console.log('reducer : base : SET_SOCKET_STATUS', payload);
        return state.set('socketStatus', payload);
    },
    ...
}, initialState); // 초기 상태값을 지정해줘야 함


// 4. 액션 호출 
...
socket.on('connect', message =>
    store.dispatch(baseAction.setSocketStatus('connected')) // setSocketStatus액션을 수행하도록 할때 파라미터로 'connected' 를 넘겨주면 위에서 작성한 액션핸들러 안의 payload객체에 connected라는 값이 들어감
);
...
```

## localStorage
- HTML5의 localStorage는 브라우저에 데이터를 저장해놓고 쓰는 것 (서버로 전송하는 것이 아님)
- localStorage에는 텍스트만 저장할 수 있음. 따라서 객체로 저장을 못하기 때문에 localStorage.stateData = JSON.stringfy(object)를 통해 객체를 텍스트형태로 저장하고, 가져다 쓸때 JSON.parse(localStorage.stateData); 이렇게 꺼내씀
- localStorage.clear(); // localStorage 초기화하는 메소드


## webpack
- webpack : 브라우저 위에서 import(또는 require)를 할 수 있게 해주고 자바스크립트 파일들을 하나로 합쳐줌
- webpack-dev-server : 별도의 서버를 구축하지 않고도 static 파일을 다루는 웹 서버를 열 수 있으며 hot-loader를 통하여 코드가 수정될 때마다 자동으로 리로드 되게 할 수 있음 (production으로 사용되지 않음)
- npm init 명령어를 수행하면 package.json 파일이 생성됨
- npm install --save-dev 로 설치하면 개발할때만 사용하는 개발 의존 모듈설치를 함
- webpack.config.dev.js : 개발환경에서 사용되는 webpack 설정파일
- webpack.config.prod.js : 실환경에서 사용되는 webpack 설정파일
- entry : webpack으로 번들링할 시작점을 말함. 즉 entry에 기재한 파일로부터 import된 파일들을 재귀적으로 따라가면서 번들링함
- output : 번들링한 결과물로 생성해낼 파일을 말함
- module : 웹팩을 통해 번들링할 때, 처리해야하는 loader를 지정할 수 있음
  * loader : 번들링을 진행하며 수행시키는 일종의 함수역할.
- test : loader을 적용시킬 파일을 정규식으로 표현
- loader : 사용할 로더를 명시
- exclude : loader가 진행될 때 배제할 파일 지정
- plugins : 추가적으로 사용할 플러그인을 설정
  * HotModuleReplacementPlugin : 개발환경에서 컴포넌트의 변경이 일어났을때, 새로고침하지 않아도 바로 리렌더링되도록 해주는 플러그인
- resolve : 경로나 확장자에 대한 처리할수 있게 도와주는 옵션


## create-react-app
- webpack, babel 등등에 대한 여러 설정을 하지 않고 react개발환경을 생성해주는 역할을 하는 도구 (create-react-app "project명")
- create-react-app으로 react 환경을 만들면, 기본적으로 설정파일들이 node_modules/react-scripts 에 위치하게 됨
- 따라서 설정파일들을 커스터마이징하기 위해 eject 라는 것을 사용 ( npm run eject )
- eject를 사용하면 설정파일들을 node_modules디렉토리에 있는 설정/스크립트 파일들을 현재 프로젝트로 꺼내줌 -> 커스터마이징이 가능해짐
- 참고 : https://velopert.com/2037


## package.json
```json
"scripts": {
    "local": "cross-env NODE_PATH=src node scripts/watch.js", // npm script 설정, 커스터마이징한 스크립트가 있는 watch.js 파일 수행
    "build": "cross-env NODE_PATH=src node scripts/build.js",
    "start": "cross-env NODE_PATH=src node scripts/start.js",
    "test": "node scripts/test.js --env=jsdom"
},
"dependencies": {
    "@material-ui/core": "^1.5.1", // AppBar, ToolBar, Avatar 등 React 컴포넌트
    "@material-ui/icons": "^2.0.3", // Icon 관련 컴포넌트
    "axios": "^0.18.0", // Http통신을 위한 라이브러리
    "classnames": "^2.2.6", // HTML 태그에 편리하게 클래스 적용할 때 사용
    "history": "^4.7.2", // 클라이언트의 브라우저 이동경로 히스토리 기억하는 라이브러리
    "immutable": "^4.0.0-rc.9", // Map, List, fromJS, setIn, getIn 등의 JS문법 사용하기 위한 라이브러리
    "moment": "^2.22.2", // 날짜 시간 라이브러리
    "qs": "^6.5.2", // QueryString parsing 할때 사용
    "react": "^16.4.2",
    "react-cookies": "^0.1.0", // 쿠키 저장 및 사용
    "react-copy-to-clipboard": "^5.0.1", // 클립보드로 COPY 가능
    "react-dom": "^16.4.2", // React의 DOM에 렌더링할 최상위 레벨의 진입점 역할을 함
    "react-number-format": "^3.5.1", // 숫자 포맷터
    "react-redux": "^5.0.7", // 리액트에서 redux를 사용하기 위해 필요 ( <Provider> 컴포넌트를 통해 redux를 react에 바인딩해서 store에 접근가능 하도록 함)
    "react-router-dom": "^4.3.1", // 브라우저에서 사용되는 리액트 라우터
    "react-router-redux": "^5.0.0-alpha.9", // Store와 redux의 상태를 동기화 (?)
    "react-select": "^1.2.1", // <Select> 컴포넌트 사용하기 위한 라이브러리
    "redux": "^4.0.0", // 컴포넌트의 상태관리를 쉽게 관리하기 위한 라이브러리 (createStore 메소드 사용)
    "redux-actions": "^2.6.1", // createAction, handleActions 를 통해 액션을 편리하게 다루기 위한 라이브러리
    "redux-thunk": "^2.3.0", // 비동기 작업을 처리할 때 주로 사용하는 미들웨어. redux-thunk 미들웨어에서 전달받은 액션이 함수형태이면 dispatch를 넣어서 실행해줌
    "reselect": "^3.0.1", // Memoization을 통한 성능 최적화. reselect의 createSelector메소드를 이용하여 selector을 만들면, 해당 selector의 인자가 변경되지 않으면 재계산을 수행하지 않도록 함. (selector는 store로부터 데이터를 가져오거나 계산하는 역할을 수행)
    "socket.io-client": "1.4.8", // socket IO
    "uuid": "^3.2.1", // 네트워크 상에서 개체를 구별하기 위한 식별자 역할
    "valid-url": "^1.0.9" // URL 유효성 검사
},
...
```


## socketClient.js - chatMiddleware
- 소켓과 관련된 작업을 하기 위한 미들웨어
- 미들웨어로 등록되면 액션이 디스패치되어서 리듀서에서 이를 처리하기 전에 지정된 작업을 먼저 수행
- 아래 메소드에서 next는 store.dispatch 와 비슷한 역할을 하는데 next(action)을 하면 바로 리듀서로 넘기거나, 혹은 또다른 미들웨어가 있다면 그 미들웨어가 처리되도록 진행
- 반면, store.dispatch는 처음부터 다시 액션이 디스패치되는 것이기때문에 현재 미들웨어를 다시 한번 처리하게 됨
- 미들웨어 등록은 createStore 할 때 applyMiddleware를 이용하여 등록 ( createStore && combineReducers 참고)

```javascript
// socketClient.js
...
export const chatMiddleware = (store) => (next) => (action) => {
    // 액션type에 "SEND__"가 포함되면 노드서버로 소켓이벤트 보냄
    if (action.type.includes('SEND__')) {
        const { event, message = {} } = action.payload;
        socket.emit(event, message);
    }
    return next(action);
};
...
```


## createStore && combineReducers
- combineReducers를 이용해 리듀서들을 합치면 해당 리듀서들이 state로 등록됨
- 따로 이름을 지정하지 않았으면, 각 리듀서를 export 할때 지정함 이름으로 state에 합쳐짐
  * ex) export {default as customer} from './customer'; 이렇게 리듀서를 export하고, combineReducers({...reducers})처럼 특정 이름을 지정하지 않으면 state.customer로 등록됨
- combineReducers({...reducers}) 를 쓰면 {customer: customer} 이렇게 세팅되는 것임 --> ES6의 속성 단축 표기
- 만약 state의 이름을 지정하고 싶다면 combineReducers({newCustomerName: customer, newChatsName: chats}) 이런식으로 이름 변경하면됨

```javascript
// configureStor.js
...
const configureStore = () => {
    let middlewares = [applyMiddleware( // 미들웨어 등록
        thunkMiddleware,
        chatMiddleware,
        routerMiddleware(history),
    )];

    if (isDevtools) {
        middlewares.push(REDUX_DEVTOOLS_EXTENSION); // queryString에 devtools를 쓰게되면 브라우저에서 redux Tool 사용 가능
    }

    // combineReducers({...reducers}) 를 쓰면 {base: base, agents: agetns, chats: chats, ..} 이렇게 세팅되는 것임 --> ES6의 속성 단축 표기
    return compose(...middlewares)(createStore)(combineReducers({...reducers, router: routerReducer}));
};

export default configureStore;


// index.js
export {default as base} from './base';
export {default as agents} from './agents';
export {default as chats} from './chats';
export {default as messages} from './messages';
export {default as sessions} from './sessions';
export {default as customers} from './customers'; // 아래 customers의 리듀서를 customers라는 이름으로 export


// customers.js
...
export default handleActions({ // 리듀서 등록
    [types.SET_CUSTOMER_INFO]: (state, { payload }) => {
        //console.log('reducer : customer : SET_CUSTOMER_INFO', payload);
        return initialState.set(payload.userId, fromJS(payload))
    },
}, initialState)
```


## createSelector (reselect 안에 있음 - 메모이제이션 역활)
- Reselect의 createSelector는 첫번째 파라미터에는 메소드로 이루어진 배열, 두번째 파라미터에는 실행할 메소드를 넘겨줌
- 만약 첫번째 파라미터로 들어온 메소드 배열들이 반환하는 값이 이전과 같으면 두번째 파라미터의 메소드를 실행하지 않음

```javascript
import { createSelector } from 'reselect';
...
/** 내가 배정 받은 채팅 */
const getChatListByMy = createSelector(
    [getChatListForConversations, getMyAgentInfoHelper], // getChatListForConversations, getMyAgentInfoHelper 함수의 반환값이 이전상태 값과 같으면
    (chats, myAgentInfo) => chats.filter(chat => chat.get('allocatedAgentId') === myAgentInfo.get('myAgentId')) // 이 메소드 수행하지 않음
);
...
```

## 템플릿 리터럴
- {/* 템플릿 리터럴 사용 - https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Template_literals */}