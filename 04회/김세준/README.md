# [4회차] - 상호작용 추가하기 (나머지)
> 링크 : https://woozy-stool-6eb.notion.site/4-40c5022af8fb4111986164270512a0cb?pvs=4

# **Render and Commit**

> **렌더링하고 커밋하기**
> 

컴포넌트를 화면에 표시하기 이전에 렌더링을 거쳐야 한다. 

<aside>
💡 학습내용

- React에서 렌더링의 의미
- React가 컴포넌트를 언제, 왜 렌더링 하는지
- 화면에 컴포넌트를 표시하는 단계
- 렌더링이 항상 DOM 업데이트를 하지 않는 이유
</aside>

React 를 웨이터에 비유하여 렌더링 과정을 3단계로 설명

![Screenshot 2023-09-12 at 6.11.31 PM.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8883249-24a5-43d5-a7b0-f77dda4453cc/7057c08c-1d9e-4dfa-aff0-e46d9d4fa2f3/Screenshot_2023-09-12_at_6.11.31_PM.png)

1. 렌더링 촉발 (손님의 주문을 주방으로 전달)
    
    렌더링이 필요하다는 신호 감지
    
2. 컴포넌트 렌더링 (주방에서 주문 받기)
    
    렌더링해야할 부분 계산
    
3. DOM에 커밋 (테이블에 주문한 요리 내놓기)
    
    실제 DOM에 렌더링
    
- React는 렌더링 결과가 이전과 같으면 DOM을 건드리지 않는다.
- 렌더링이란?
    
    현재 props 및 상태를 기반으로 리액트가 컴포넌트에게 UI 영역이 어떻게 보이길 원하는지 설명을 요청하는 일련의 프로세스
    
- 가상돔(Virtual DOM) 멀리하기
    
    https://twitter.com/dan_abramov/status/1066328666341294080?lang=en
    

## **Step 1: Trigger a render**

> **렌더링을 촉발시킵니다**
> 

렌더링이 일어나는 2가지 이유

1. 컴포넌트의 첫 랜더링
2. 컴포넌트의 state(또는 상위 요소 중 하나)가 업데이트 된 경우

### **Initial render**

> **첫 렌더링**
> 

앱을 시작하기 위해서는 첫 렌더링이 촉발되어야 한다. 

`createRoot` 를 호출한 다음 컴포넌트로 `render 메서드`를 호출하면 된다.

render 메서드는 작성한 JSX를 HTML로 변환하여 실제 브라우저에 렌더링하는 작업을 한다.

```jsx
// Image.js
export default function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    />
  );
}

// index.js
import Image from './Image.js';
import { createRoot } from 'react-dom/client';

const root = createRoot(document.getElementById('root'))
root.render(<Image />);
```

### **Re-renders when state updates**

> **state가 업데이트되면 리렌더링합니다**
> 

컴포넌트가 처음 렌더링 되면 setState 함수로 state를 업데이트 하여 추가 렌더링을 촉발할 수 있다. 

컴포넌트의 state가 업데이트 되면 자동으로 렌더링이 대기열에 추가된다. 

마치 손님이 식당에서 첫 주문 후 추가로 주문하는 로직과 같다.

![Screenshot 2023-09-12 at 6.22.39 PM.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8883249-24a5-43d5-a7b0-f77dda4453cc/c8f8093f-1afa-4262-bb05-ed0a8ac11f42/Screenshot_2023-09-12_at_6.22.39_PM.png)

## **Step 2: React renders your components**

> **React가 컴포넌트를 렌더링합니다**
> 

렌더링이 촉발되면 React는 컴포넌트를 호출하여 화면에 표시할 내용을 파악한다.

`렌더링`은 React에서 컴포넌트를 호출하는 것이다.

첫 렌더링에서 React는 루트 컴포넌트를 호출한다.

이후 렌더링에서는 React는 state 업데이트에 의해 렌더링이 발동된 함수 컴포넌트를 호출한다. (컴포넌트의 state 혹은 상위 요소 중 하나의 state가 업데이트 된 컴포넌트)

일련의 과정은 재귀적으로 발생한다. 업데이트된 컴포넌트가 다른 컴포넌트를 반환하면 React는 해당 컴포넌트를 렌더링 하고 해당 컴포넌트도 컴포넌트를 반환하면 반환된 컴포넌트를 다음에 렌더링 하는 방식이다.

중첩된 컴포넌트가 더 이상 존재하지 않고 React가 화면에 표시할 내용을 정확히 알 때까지 이 단계는 반복된다. 

- 랜더링은 항상 순수한 계산이어야 한다.
    - 동일한 입력, 동일한 출력
    - 이전의 state를 변경해서는 안된다.
    
    → 그렇지 않으면 코드베이스가 복잡해지면서 버그와 사이드 이펙트가 발생할 수 있다. strict mode 에서 개발하면 컴포넌트의 함수가 두번 호출되어 순수하지 않는 함수로 인한 실수를 포착하는데 도움이 된다.
    
- 성능측정과 성능최적화
    
    [React 렌더링 성능을 측정하고 개선해보자 (Memo, useCallback)](https://velog.io/@dy6578ekdbs/오소플-기술블로그)
    
    업데이트된 컴포넌트가 트리에서 매우 높은 곳에 위치해 있다면 중첩된 모든 컴포넌트를 렌더링 해야하므로 성능 이슈가 존재할 수 있다. → 컴포넌트 분리
    
    useMemo, useCallback, 컴포넌트 분리
    

## **Step 3: React commits changes to the DOM**

> **React가 DOM에 변경사항을 커밋**
> 

컴포넌트를 렌더링(호출)한 후 React는 DOM을 수정한다.

- 초기 렌더링의 경우 Reactsms appendChild() DOM API를 사용하여 생성한 모든 DOM 노드를 화면에 표시한다.
- 리렌더링의 경우 React는 필요한 최소한의 작업을 적용하여 DOM이 최신 렌더링 출력과 일치하도록 한다.

React는 렌더링 간에 차이가 있는 경우에만 DOM 노드를 변경한다. 

```jsx
// Clock.js
export default function Clock({ time }) {
  return (
    <>
      <h1>{time}</h1>
      <input />
    </>
  );
}
```

### 브라우저 렌더링

React가 DOM을 업데이트 하고 나면 브라우저 렌더링 과정을 거쳐 화면을 그리게 된다.

DOM Tree 생성 → CSSOM Tree 생성 → Rendering Tree 생성 → Layout → Painting

## **Epilogue: Browser paint**

> **에필로그: 브라우저 페인트**
> 

렌더링이 완료되고 React가 DOM을 업데이트한 후 브라우저는 다시 화면을 그린다. 이 단계를 ‘브라우저 렌더링’ 이라고 하지만 이 문서의 나머지 부분에서는 혼동을 피하기 위해 ‘페인팅’이라고 지칭할 것

---

# **State as a Snapshot**

> **스냅샷으로서의 state**
> 

state는 스냅샷처럼 동작한다. state 변수를 설정해도 이미 가지고 있는 **state 변수는 변경되지 않고 대신 리렌더링이 실행**된다. 

<aside>
💡 학습내용

- state 설정으로 리렌더링이 촉발되는 방식
- state 업데이트 시기 및 방법
- state를 설정한 직후에 state가 업데이트되지 않는 이유
- 이벤트 핸들러가 state의 ‘스냅샷’에 액세스하는 방법
</aside>

- 리액트의 라이프 사이클 ( 클래스형 컴포넌트 )
    
    ![React Lifecycle Cheatsheet from [reactjs.org](http://reactjs.org/)](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8883249-24a5-43d5-a7b0-f77dda4453cc/5d297555-f364-480b-b016-b7171ed2de2f/Untitled.png)
    
    React Lifecycle Cheatsheet from [reactjs.org](http://reactjs.org/)
    
    **Mount**
    
    컴포넌트가 처음 실행되는 것을 mount 된다고 표현한다. 
    
    constructor
    
    컴포넌트 생성자 메서드로 컴포넌트가 생성되면 가장 먼저 실행되는데 this.props, this.state에 접근이 가능하고 리액트 요소를 반환한다.
    
    getDerivedStateFromProps
    
    props로부터 파생된 state를 가져온다. 즉, 부모 컴포넌트로부터 props로 전달된 것을 state에 넣어주고 싶을 때 사용한다
    
    render
    
    컴포넌트 렌더링을 하는 메서드
    
    componentDidmount
    
    컴포넌트의 첫 번째 렌더링이 끝나면 호출되는 메서드로 이 메서드가 호출되는 시점에는 화면에 컴포넌트가 나타난 상태이다. 여기서 주로 DOM을 사용해야하는 라이브러리 연동, api 요청 등의 이벤트가 발생한다.
    
    **Update**
    
    컴포넌트가 업데이트와 관련된 생명주기
    
    getDerivedStateFromProps
    
    props 나 state가 바뀌었을 때 이 메서드가 호출된다.
    
    shouldComponentUpdate
    
    컴포넌트가 리렌더링 할지 말지를 결정하는 메서드, 주로 최적화 할 때 사용하는 메서드
    
    componentDidUpdate
    
    컴포넌트가 업데이트 되고 난 후 발생한다.
    
    **Unmount**
    
    컴포넌트가 화면에 사라지는 것을 의미한다. 언마운트와 관련된 생명주기 메서드는 componentWillUnmount 하나이다.
    
    componentWillUnmount
    
    컴포넌트가 화면에서 사라지기 직전에 호출되며, 컴포넌트를 더 이상 사용하지 않을때 발생하는 메서드이다. 
    

## **Setting state triggers renders**

> **state를 설정하면 렌더링이 촉발됩니다**
> 

인터페이스가 이벤트에 반응하려면 state를 업데이트 해야한다.

```jsx
// App.js
import { useState } from 'react';

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState('Hi!');
  if (isSent) {
    return <h1>Your message is on its way!</h1>
  }
  return (
    <form onSubmit={(e) => {
      e.preventDefault();
      setIsSent(true);
      sendMessage(message);
    }}>
      <textarea
        placeholder="Message"
        value={message}
        onChange={e => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

버튼을 클릭하면

1. onSubmit 이벤트 핸들러가 실행된다.
2. setIsSent(true) 가 isSent를 true 로 설정하고 새 렌더링을 큐에 대기시킨다.
3. React는 새로운 isSent 값에 따라 컴포넌트를 다시 렌더링 한다.

## **Rendering takes a snapshot in time**

> **렌더링은 그 시점의 스냅샷을 찍습니다**
> 

`렌더링` 이란 React 가 컴포넌트, 즉 함수를 호출한다는 것이다. 해당 함수에서 반환하는 JSX는 UI의 스냅샷과 같다고 할 수 있다. props, 이벤트 핸들러, 로컬 변수 모두 렌더링 당시의 state를 사용한다.

UI ‘스냅샷’은 대화형이다. input에 대한 응답으로 어떤 동작을 할지 지정하는 이벤트 핸들러와 같은 로직이 포함되어 있다. 그러면 React는 스냅샷과 일치하도록 화면을 업데이트하고 이벤트 핸들러를 연결한다. 

React가 컴포넌트를 다시 렌더링할 때

1. React 가 함수를 다시 호출한다.
2. 함수가 새로운 JSX 스냅샷을 반환한다.
3. React가 반환한 스냅샷과 일치하도록 화면을 업데이트 한다.

![*Illustrated by [Rachel Lee Nabors](http://rachelnabors.com/)*](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8883249-24a5-43d5-a7b0-f77dda4453cc/675328b3-fc0b-4b1d-b8b4-185e293ad449/Screenshot_2023-09-12_at_7.32.32_PM.png)

*Illustrated by [Rachel Lee Nabors](http://rachelnabors.com/)*

함수가 반환된 후 사라지는 일반 변수와 달리 state는 컴포넌트의 메모리로서 함수 외부, React 자체에 존재한다. React가 컴포넌트를 호출하면 특정 렌더링에 대한 state의 스냅샷을 제공하고 컴포넌트는 해당 렌더링의 state 값을 사용해 계산된 props 와 이벤트 핸들러가 포함된 UI 스냅샷을 JSX에 반환한다. 

![*Illustrated by [Rachel Lee Nabors](http://rachelnabors.com/)*](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8883249-24a5-43d5-a7b0-f77dda4453cc/ed1b3a1d-bf43-450f-a812-9875183cc2fe/Screenshot_2023-09-12_at_7.35.57_PM.png)

*Illustrated by [Rachel Lee Nabors](http://rachelnabors.com/)*

공식문서의 예시코드 일부

```jsx
<button onClick={() => {
  setNumber(number + 1);
  setNumber(number + 1);
  setNumber(number + 1);
}}>+3</button>

// 첫 번째 렌더링
<button onClick={() => {
  setNumber(0 + 1);
  setNumber(0 + 1);
  setNumber(0 + 1);
}}>+3</button>

// 두 번째 렌더링
<button onClick={() => {
  setNumber(1 + 1);
  setNumber(1 + 1);
  setNumber(1 + 1);
}}>+3</button>
```

첫 번째 렌더링이 일어날때 이벤트 핸들러가 받아온 state는 0, 해당 이벤트 핸들러의 number state가 0인 상태로 각각 setState 함수를 호출하니까 +3 이 아니라 +1 증가

두 번째 렌더링이 일어날 때의 이벤트 핸들러는 앞선 렌더링에서 업데이트된 state 값 1을 기준으로 하여 역시 +1 증가

## **State over time**

> **시간 경과에 따른 state**
> 

```jsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        alert(number);
      }}>+5</button>
    </>
  )
}
```

이 역시 위의 예시와 유사하다. 이벤트 핸들러가 실행되었을 때의 state 값은 0, setNumber 로 state가 업데이트 되었더라도 현재 렌더링에서의 nubmer는 아직 0인 상태, 외부의 state는 5가 되어 있다.

따라서 이벤트 핸들러 내부로직을 다 돌고 나면 외부 state 대로 다시 렌더링 되면서 업데이트 된 state 5를 기준으로 렌더링 한다. 

React에 저장된 state는 이벤트가 발생했을 때 변경되지만, 사용자가 상호작용한 시점에 state 스냅샷을 사용하게 되어 있기 때문이다. 

React 는 하나의 렌더링 이벤트 핸들러 내에서 state 값을 ‘고정’으로 유지한다. 따라서 코드가 실행되는 동안 state가 변경되었는지를 걱정할 필요가 없다.

만약 리렌더링 하기 전에 최신 state를 읽고 싶다면? → 다음 챕터의 state 업데이터 함수를 사용

---

# **Queueing a Series of State Updates**

> **여러 state 업데이트를 큐에 담기**
> 

state 변수를 설정하면 다음 렌더링이 큐(대기열, queue)에 들어간다. 그러나 경우에 따라 다음 렌더링을 큐에 넣기 전에 값에 대해 여러 작업을 수행하고 싶을 때가 있다. 

→ 스냅샷이 아니라 그때 그때 바로 업데이트 된 최신의 state를 사용하고 싶은 경우 ⇒ state 업데이트 사용

<aside>
💡 학습내용

- 일괄처리(배칭, batching)이란 무엇이며 React가 여러 state 업데이트를 처리하는 방법
- 동일한 state 변수에서 여러 업데이트를 적용하는 방법
</aside>

## **React batches state updates**

> **state 업데이트 일괄처리**
> 

```jsx
<button onClick={() => {
  setNumber(1 + 1);
  setNumber(1 + 1);
  setNumber(1 + 1);
}}>+3</button>
```

앞선 세션에서 확인했듯이 React 는 state 업데이트를 하기 전에 이벤트 핸들러의 모든 코드가 실행될 때까지 기다린다. 따라서 리렌더링은 모든 setNumber() 호출이 완료된 이후에만 일어난다. 

식당에서 웨이터가 첫 번째 요리를 말하자마자 바로 주방으로 달려가지 않고 주문이 끝날 때까지 기다렸다가 주문은 변경하고 테이블의 다른 사람의 주문까지 다 받는 과정과 빗대어 생각해보면 된다.

이런 동작 방식은 **너무 많은 리렌더링을 촉발하지 않고 여러 컴포넌트에서 나온 다수의 state 변수를 업데이트할 수 있다.** 하지만 이벤트 핸들러와 그 안에 있는 코드가 완료될 때까지 UI가 업데이트 되지 않는다는 의미이기도 하다. 

이와 반대로 **일괄처리(배칭, batching)**라는 동작을 통해 React 앱을 훨씬 빠르게 실행할 수도 있다. 

## **Updating the same state multiple times before the next render**

> **다음 렌더링 전에 동일한 state 변수를 여러 번 업데이트하기**
> 

동일한 state 변수를 여러 번 업데이트 하고 싶다면 setNumber(number + 1)과 같이 state 값을 그대로 전달하는 대신 setNumber(n ⇒ n + 1)과 같이 큐의 이전 state를 기반으로 다음 state를 계산하는 함수를 전달할 수 있다. 

단순히 state 값을 전달하는 것이 아니라 state 값을 어떻게 가공할지 지시하는 방법이다.

```jsx
<button onClick={() => {
  setNumber(n => n + 1);
  setNumber(n => n + 1);
  setNumber(n => n + 1);
}}>+3</button>
```

여기서 `n ⇒ n + 1` 이 **업데이터 함수(updater function)**이다.

state setter 함수에 업데이터 함수를 전달하면

1. React는 이벤트 핸들러의 다른 코드가 모두 실행된 후 이 함수가 처리되도록 큐에 넣는다.
2. 다음 렌더링 중에 React는 큐를 순회하여 최종 업데이트된 state를 제공한다.

React가 이벤트 핸들러를 수행하는 동안 여러 코드를 통해 작동하는 방식은 다음과 같다.

1. `setNumber(n => n + 1)`: `n => n + 1` 함수를 큐에 추가합니다.
2. `setNumber(n => n + 1)`: `n => n + 1` 함수를 큐에 추가합니다.
3. `setNumber(n => n + 1)`: `n => n + 1` 함수를 큐에 추가합니다.

다음 렌더링 중에 `useState` 를 호출하면 React는 큐를 순회합니다. 이전 `number` state는 `0`이었으므로 React는 이를 첫 번째 업데이터 함수에 `n` 인수로 전달합니다. 그런 다음 React는 이전 업데이터 함수의 반환값을 가져와서 다음 업데이터 함수에 `n` 으로 전달하는 식으로 반복합니다.

| queued update | n | returns |
| --- | --- | --- |
| n => n + 1 | 0 | 0 + 1 = 1 |
| n => n + 1 | 1 | 1 + 1 = 2 |
| n => n + 1 | 2 | 2 + 1 = 3 |

## **What happens if you update state after replacing it**

> **state를 교체한 후 업데이트하면 어떻게 될까요**
> 

```jsx
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
}}>
```

- 상기 코드 작동 방식
    
    1. `setNumber(number + 5)` : `number`는 `0`이므로 `setNumber(0 + 5)`입니다. 
    
    React는 큐에 “`5`로 바꾸기”를 추가합니다.
    2. `setNumber(n => n + 1)` : `n => n + 1` 는 업데이터 함수입니다. React는 해당 함수를 큐에 추가합니다.
    
    다음 렌더링 동안 React는 state 큐를 순회합니다:
    
    | queued update | n | returns |
    | --- | --- | --- |
    | “replace with 5” | 0 (unused) | 5 |
    | n => n + 1 | 5 | 5 + 1 = 6 |
    
    React는 `6`을 최종 결과로 저장하고 `useState`에서 반환합니다.
    

<aside>
🌟 `setState(x)` vs `setState(n ⇒ x)`

</aside>

## **What happens if you replace state after updating it**

> **업데이트 후 state를 바꾸면 어떻게 될까요**
> 

```jsx
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
```

- 상기 코드 작동 방식
    1. `setNumber(number + 5)`: `number` 는 `0` 이므로 `setNumber(0 + 5)`입니다. React는 *“`5`로 바꾸기”*를 큐에 추가합니다.
    2. `setNumber(n => n + 1)`: `n => n + 1` 는 업데이터 함수입니다. React는 *이 함수*를 큐에 추가합니다.
    3. `setNumber(42)`: React는 *“`42`로 바꾸기”*를 큐에 추가합니다.
    
    다음 렌더링 동안, React는 state 큐를 순회합니다:
    
    | queued update | n | returns |
    | --- | --- | --- |
    | “replace with 5” | 0 (unused) | 5 |
    | n => n + 1 | 5 | 5 + 1 = 6 |
    | “replace with 42” | 6 (unused) | 42 |

이벤트 핸들러가 완료되면 React는 리렌더링을 실행한다. 리렌더링 하는 동안 React는 큐를 처리한다.

업데이터 함수는 렌더링 중에 실행되므로 업데이터 함수는 순수해야 하며 결과만 반환해야 한다. 따라서 업데이터 함수 내부에서 state를 변경하거나 다른 사이드 이펙트를 실행하지 않는 것이 좋다. 

## **Naming conventions**

> **명명 규칙**
> 

업데이터 함수 인수의 이름은 해당 state 변수의 첫 글자로 지정하는 것이 일반적

```jsx
setEnabled(e => !e);
setLastName(ln => ln.reverse());
setFriendCount(fc => fc * 2);
```

---

# **Updating Objects in State**

> **객체 state 업데이트**
> 

state는 객체를 포함하여 어떤 JavaScript 값이든 저장할 수 있다. 하지만 React state의 객체를 직접 변이해서는 안된다. 대신 객체를 업데이트 하려면 새로운 객체를 생성하고(혹은 기존 객체의 복사본을 만들고) 해당 복사본을 사용하도록 state를 설정해야 한다. 

<aside>
💡 학습내용

- React state에서 객체를 올바르게 업데이트하는 방법
- 중첩된 객체를 변이하지 않고 업데이트하는 방법
- 불변성이란 무엇이며, 불변성을 깨뜨리지 않는 방법
- Immer로 객체 복사를 덜 반복적으로 만드는 방법
</aside>

## **What’s a mutation?**

> **변이란 무엇인가요?**
> 

state는 어떤 종류의 JavaScript 값이라도 저장할 수 있지만 그 값은 불변이다. 값이 변이하면 렌더링이 촉발된다.

```jsx
const [x, setX] = useState(0);

setX(5);
```

x state가 0에서 5로 변경되었지만 숫자 0 자체는 변경되지 않는다. JavaScript 에서는 숫자, 문자, 불리언과 같은 빌트인 원시형 자료형 값을 변경할 수 없다. 

**객체 state**

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });

position.x = 5;
```

기술적으로 객체 자체의 내용을 변경을 하는 것은 가능하다. 이를 변이(mutation)이라고 한다.

React state의 객체는 기술적으로는 변이할 수 있지만 숫자, 불리언, 문자열과 같이 불변하는 것처럼 취급해야 한다. ⇒ 객체를 직접 변이하는 대신 항상 교체해하 하는 이유

## **Treat state as read-only**

> **state를 읽기 전용으로 취급하세요**
> 

state에 넣는 모든 JavaScript 객체는 읽기 전용으로 취급해야 한다.

```jsx
import { useState } from 'react';
export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        position.x = e.clientX;
        position.y = e.clientY;
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  );
}
```

- 상기 코드 작동 원리
    
    상기 코드는 이전 렌더링에서 positoin에 할당된 객체를 수정한다. 하지만 state setting 함수를 사용하지 않으면 React는 객체의 변이를 감지하지 못한다. 따라서 아무런 반응도 하지 않는다.
    
    state 변이는 경우에 따라 작동할 수 있지만 권장되지 않으며 렌더링에 접근하기 위한 state 값은 읽기전용으로 취급해야 한다.
    

리렌더링을 촉발하려면 새로운 객체를 생성하고 state 설정자 함수에 전달해야 한다.

```jsx
onPointerMove={e => {
  setPosition({
    x: e.clientX,
    y: e.clientY
  });
}}
```

`setPosition` 은 React에게 2가지 동작을 지시한다. 

- `position`을 이 새 객체로 바꿔라.
- 이 컴포넌트를 다시 렌더링 하라.

<aside>
🌟 **DEEP DIVE: 지역 변이는 괜찮습니다.**

</aside>

- Why?
    
    

## **Copying objects with the spread syntax**

> **전개 구문을 사용하여 객체 복사하기**
> 

```jsx
const [origin, setOrigin] = useState({
	name: 'kim',
	age: 23,
	job: 'doctor',
	salary: 100000,
	unit: 'US dollar',
	province: 'San Francisco',
	city: 'San Jose',
	avenue: '351'
})
```

종종 새로운 객체에 기존 객체의 데이터에 일부만 업데이트 한 값을 전달하고 싶을 때가 있다. 이런 경우 기존 객체를 복사하여 붙여넣은 다음 업데이트 하고자 하는 데이터만 값을 업데이트 할 수 있다.

```jsx
setOrigin({
	name: 'kim',
	**age: 32**,
	job: 'doctor',
	salary: 100000,
	unit: 'US dollar',
	province: 'California',
	city: 'San Jose',
	avenue: '351'
})
```

이때 모든 속성을 개별적으로 복사하지 않고 객체 전개 구문을 사용할 수 있다.

```jsx
setOrigin({
	...origin,
	**age: 32**,
})
```

<aside>
🌟 **DEEP DIVE:** **Using a single event handler for multiple fields**

> **여러 필드에 단일 이벤트 핸들러 사용하기**
> 

객체 내에서 `[` 및 `]` 중괄호를 사용하여 동적 이름을 가진 프로퍼티를 지정할 수도 있다. 

</aside>

```jsx
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleChange(e) {
    setPerson({
      ...person,
      [e.target.name]: e.target.value
    });
  }

  return (
    <>
      <label>
        First name:
        <input
          name="firstName"
          value={person.firstName}
          onChange={handleChange}
        />
      </label>
      <label>
        Last name:
        <input
          name="lastName"
          value={person.lastName}
          onChange={handleChange}
        />
      </label>
      <label>
        Email:
        <input
          name="email"
          value={person.email}
          onChange={handleChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
```

## **Updating a nested object**

> **중첩된 객체 업데이트하기**
> 

중첩된 객체 구조 예시코드

```jsx
const [user, setUser] = useState({
  name: 'kim',
  job: 'doctor',
  age: 32,
  address: {
    state: 'California',
    city: 'San Jose',
    avenue: '312'
  }
})
```

이 때, user 가 로스앤젤레스로 이사를 가서 address 객체의 city를 업데이트 해야한다.

```jsx
user.address.city = 'Los Angeles';
```

이렇게 하면 간단하게 객체 데이터를 바꿀 수 있지만, React 는 state를 불변으로 취급해야하며 앞선 세션에서 객체를 복사하여 바꿔야 한다.

```jsx
const newAddress = {...user.address, city: 'Los Angeles'}
const udpatedUser = {...user, address: newAddress}
setUser(updatedUser);
```

혹은

```jsx
setUser({
	...user,
	address: {
		...user.address,
		city: 'Los Angeles',
	}
});
```

<aside>
🌟 **DEEP DIVE: Objects are not really nested**

> 객체는 실제로 중첩되지 않는다.
> 

위 예시코드에서

```jsx
setUser({
	...user,
	city: 'Los Angeles'
})
```

이런 방법으로 객체를 변이할 수 없다. 이 방법이 불가능한 이유는 전개구문은 `얕은 복사` 를 하므로 한 단계의 깊이만 복사하기 때문이다. 

실제 객체의 동작을 살펴보면 객체는 실제로 중첩되지 않음을 알 수 있다. 

```jsx
let user = {
	name: 'kim',
  job: 'doctor',
  age: 32,
	address: addressObj
}

let addressObj = {
	state: 'California',
  city: 'San Jose',
  avenue: '312'
}
```

기존의 user 객체는 위와 같은 형태로 다른 객체를 바라보고 있는 형태이다.

```jsx
let user2 = {
	name: 'lee',
  job: 'doctor',
  age: 32,
	address: addressObj
}
```

addressObj 객체를 바라보는 객체가 또 하나 존재한다고 할 때

addressObj 는 user 객체의 내부에 존재하지 않고, user2의 내부에도 존재하지 않지만, user와 user2는 모두 addressObj를 가리킬 수 있다. 
따라서 addressObj.city를 변이하면 user와 user2에 모두 영향을 미친다. 

**객체는 중첩된 것이 아니라 프로퍼티를 사용하여 서로를 ‘가리키는’ 별도의 객체이다.**

</aside>

## **Write concise update logic with Immer**

> **Immer로 간결한 업데이트 로직 작성**
> 

state가 깊게 중첩된 경우 객체를 평평하게 만드는 것을 고려할 수 있다. 

만약 state 구조를 변경하고 싶지 않다면 전개구문을 중첩해서 사용하는 대신에 Immer를 사용할 수 있다.

Immer 라이브러리는 변이 구문을 작성하더라도 자동으로 사본을 생성해주는 라이브러리이다. 

```jsx
updateUser(draft => {
	draft.address.city = 'Los Angeles';
});
```

마치 규칙을 깨고 state 객체를 변이하는 것처럼 보이지만 일반적인 변이와 달리 이전 state를 덮어쓰지 않는다.

<aside>
🌟 **DEEP DIVE: IMMER 동작 원리**

Immer 에서 제공하는 `draft` 는 프록시라는 특수한 객체로 사용자가 수행하는 작업을 `기록` 한다. 내부에서 자동적으로 draft의 어떤 부분이 변경되었는지 감지하고 변이된 완전히 새로운 객체를 생성해주기 때문에 규칙 얽매이지 않고 변이를 할 수 있다.

</aside>

Immer 사용방법

1. `npm install use-immer`를 실행하여 Immer를 의존성으로 추가합니다.
2. 그런 다음 `import { useState } from 'react'`를 `import { useImmer } from 'use-immer'`로 바꿉니다.
3. useState 대신 useImmer를 사용하면 된다. 

```jsx
import { useImmer } from 'use-immer'

const [user, setUser] = useImmer({
  name: 'kim',
  job: 'doctor',
  age: 32,
  address: {
    state: 'California',
    city: 'San Jose',
    avenue: '312'
  }
})

...

funcition onChange (e) {
	updateUser(draft => {
		draft.city = e.target.value;
	}
}

...
```

<aside>
🌟 DEEP DIVE: React에서 state 변이를 권장하지 않는 이유는?

- **디버깅 :** console.log 를 사용하고 state 를 변이하지 않으면 console의 기록들이 지워지지 않는다. 따라서 렌더링 사이에 state 가 어떻게 변경되었는지 명확하게 확인이 가능하다.
- **최적화 :** 일반적인 React 최적화 전략은 이전 프로퍼티 혹은 state 가 다음 프로퍼티나 state와 동일한 경우 작업을 건너 뛰는 것에 의존한다. state가 변이되지 않으면 변경이 있었는지 빠르게 확인할 수 있다. ( ex. prevObj === obj 라면, 내부에 변경이 없다는 것을 확신할 수 있음)
- **새로운 기능** : 새로운 React 기능은 스냅샷처럼 취급되는 것에 의존한다. 과거 버전의 state를 변이하는 것은 새로운 기능을 사용하지 못할 수 있다.(❓)
- **요구사항 변경 :** 일부 어플리케이션 기능은 아무것도 변이되지 않는 state에서 더 쉽게 수행될 수 있다. 과거 state의 복사본을 메모리에 보관하고 재사용할 수 있기 때문이다.
- **더 간단한 구현 :** React는 변이에 의존하지 않아 객체에 특별한 작업을 할 필요가 없다. 많은 반응형 솔루션처럼 프로퍼티를 가로채거나, 항상 프록시로 감싸거나, 초기화할 때 다른 작업을 할 필요가 없다. 추가 성능이나 정확성의 함정없이 아무리 큰 객체라도 state에 넣을 수 있는 이유이기도 하다.
</aside>

---

# **Updating Arrays in State**

> **배열 state 업데이트**
> 

객체와 마찬가지로 배열 역시 state 저장할 때는 변이가 불가능한 것으로 취급해야 한다. state에 저장된 배열을 업데이트 하려면 역시 기존의 배열을 복사하여 새로운 배열을 만들고 새 배열을 state로 사용하도록 해야한다.

<aside>
🌟 학습내용

- React state 안의 배열 항목을 추가, 제거 또는 변경하는 방법
- 배열 내부의 객체를 업데이트 하는 방법
- Immer로 덜 반복적으로 배열을 복사하는 방법
</aside>

## **Updating arrays without mutation**

> **변이 없이 배열 업데이트하기**
> 

배열 역시 객체와 마찬가지로 읽기 전용으로 취급해야 하고 배열의 재할당, push() 및 pop()과 같은 배열을 변이하는 메서드를 사용해서는 안된다.

배열 state를 업데이트 하고 싶다면 원래 배열을 복사한 후 filter() 및 map() 과 같은 비변이 메서드를 호출하여 새 배열을 만들고 새로 만들어진 새 배열을 state로 설정하면 된다.

| avoid (mutates the array)비추천 (배열 직접 변이) | prefer (returns a new array)추천 (새 배열 반환) |
| --- | --- |
| adding 추가 | push, unshift |
| removing삭제 | pop, shift, splice |
| replacing교체 | splice, arr[i] = ... assignment |
| sorting정렬 | reverse, sort |

일반적인 배열 연산에 대한 참조 표

|  | avoid (mutates the array)비추천 (배열 직접 변이) | prefer (returns a new array)추천 (새 배열 반환) |
| --- | --- | --- |
| adding추가 | push, unshift | concat, [...arr] spread syntax (https://react-ko.dev/learn/updating-arrays-in-state#adding-to-an-array) |
| removing삭제 | pop, shift, splice | filter, slice (https://react-ko.dev/learn/updating-arrays-in-state#removing-from-an-array) |
| replacing교체 | splice, arr[i] = ... assignment | map (https://react-ko.dev/learn/updating-arrays-in-state#replacing-items-in-an-array) |
| sorting정렬 | reverse, sort | copy the array first (https://react-ko.dev/learn/updating-arrays-in-state#making-other-changes-to-an-array)
배열을 복사한 다음 처리 |

또는 객체와 마찬가지로 Immer를 사용할 수도 있다. 

## **Adding to an array**

> **배열에 추가하기**
> 

가장 쉬운 방법은 객체와 마찬가지로 전개 구문을 사용하는 것

```jsx
setArtists( // Replace the state
  [ // with a new array
    ...artists, // that contains all the old items
    { id: nextId++, name: name } // and one new item at the end
  ]
);

setArtists([
  { id: nextId++, name: name },
  ...artists // Put old items at the end
]);
```

전개 구문의 사용 위치에 따라 배열을 끝에 새로운 배열을 추가할지, 시작하는 부분에 추가할지 push()와 unshift()의 기능을 모두 수행할 수 있다. 

## **Removing from an array**

> **배열에서 제거하기**
> 

가장 쉬운 방법은 filter 메서드를 사용하는 방법

```jsx
setArtists(
  artists.filter(a => a.id !== artist.id)
);
```

## **Transforming an array**

> **배열 변경하기**
> 

배열의 일부 또는 모든 항목을 변경 하려면 map()을 사용하여 새로운 배열을 만들 수 있다. 

```jsx
let initialShapes = [
  { id: 0, type: 'circle', x: 50, y: 100 },
  { id: 1, type: 'square', x: 150, y: 100 },
  { id: 2, type: 'circle', x: 250, y: 100 },
];

...
const nextShapes = shapes.map(shape => {
  if (shape.type === 'square') {
    // No change
    return shape;
  } else {
    // Return a new circle 50px below
    return {
      ...shape,
      y: shape.y + 50,
    };
  }
});
...
```

## **Replacing items in an array**

> **배열에서 항목 교체하기**
> 

```jsx
let initialCounters = [
  0, 0, 0
];

...
const nextCounters = counters.map((c, i) => {
      if (i === index) {
        // Increment the clicked counter
        return c + 1;
      } else {
        // The rest haven't changed
        return c;
      }
    });
...
```

## **Inserting into an array**

> **배열에 삽입하기**
> 

배열 중간에 특정 위치에 항목을 삽입하고 싶을 때 배열 전개 구문과 slice() 메서드를 함게 사용할 수 있다.

(배열의 맨 앞, 맨뒤에 삽입하고 싶을 때는 전개구문만으로 구현)

slice() 메서드를 사용하면 배열 ‘조각’을 잘라 낼 수 있다. 항목을 삽입하고 싶은 위치 앞뒤로 배열을 잘라내고 그 사이에 원하는 항목을 삽입한 후 잘라낸 배열과 삽입된 항목을 붙여주면 된다.

```jsx
...
function handleClick() {
  const insertAt = 1; // Could be any index
  const nextArtists = [
    // Items before the insertion point:
    **...artists.slice(0, insertAt),**
    // New item:
    **{ id: nextId++, name: name },**
    // Items after the insertion point:
    **...artists.slice(insertAt)**
  ];
  setArtists(nextArtists);
  setName('');
}
...
```

## **Making other changes to an array**

> **배열에 다른 변경 사항 적용하기**
> 

배열을 반전시키거나 정렬하고 싶을 때는 reveser() 및 sort() 메서드를 사용하여 간단하게 구현할 수 있는데, 이 역시 원래 배열을 직접 변이 하므로 전개구문을 사용하여 배열을 복사한 뒤에 변이하면 된다.

```jsx
const initialList = [
  { id: 0, title: 'Big Bellies' },
  { id: 1, title: 'Lunar Landscape' },
  { id: 2, title: 'Terracotta Army' },
];

export default function List() {
  const [list, setList] = useState(initialList);

  function handleClick() {
    const nextList = [...list];
    nextList.reverse();
    setList(nextList);
  }
...
}
```

🔥 그러나 배열을 복사하더라도 배열 내부의 기존 항목을 직접 변이할 수는 없다. 

얕은 복사가 이루어져 새 배열에는 원래 배열과 동일한 항목이 포함되기 때문이다. 이런 경우 복사된 배열 내부의 객체를 수정하면 기존 state를 변이하는 것이다. 

```jsx
const nextList = [...list];
nextList[0].seen = true; // Problem: mutates list[0]
setList(nextList);
```

nextList 와 list는 서로 다른 배열이지만 nextList[0]과 list[0]은 같은 객체를 가리킨다. 따라서 nextList[0].seen 을 변경하면 list[0].seen 도 변경하는 것이다. 이는 state 변이를 의미하므로 피해야 한다. 

⇒ 중첩된 JavaScript 객체 업데이트와 유사하게 변경하려는 개별 항목을 직접 변이하는 대신 복사하여 해결할 수 있다.

## **Updating objects inside arrays**

> **배열 내부의 객체 업데이트하기**
> 

```jsx
import { useState } from 'react';

let nextId = 3;
const initialList = [
  { id: 0, title: 'Big Bellies', seen: false },
  { id: 1, title: 'Lunar Landscape', seen: false },
  { id: 2, title: 'Terracotta Army', seen: true },
];

export default function BucketList() {
  const [myList, setMyList] = useState(initialList);
  const [yourList, setYourList] = useState(
    initialList
  );

  function handleToggleMyList(artworkId, nextSeen) {
    const myNextList = [...myList];
    const artwork = myNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen;
    setMyList(myNextList);
  }

  function handleToggleYourList(artworkId, nextSeen) {
    const yourNextList = [...yourList];
    const artwork = yourNextList.find(
      a => a.id === artworkId
    );
    artwork.seen = nextSeen;
    setYourList(yourNextList);
  }
  ...
}

```

list 배열 내부에 객체가 존재하는 것처럼 보이지만, 배열의 각 객체는 배열이 가리키는 별도의 값이다. 따라서 list[0]과 같이 중첩된 필드를 변경할 때 주의가 필요하다. 다른 배열에서 동일한 요소를 가리키고 있을 수도 있기 때문이다. 

중첩된 state를 업데이트 하기위해 업데이트 하려는 지점부터 최상위 수준까지 복사본이 필요하다. 

```jsx
setMyList(myList.map(artwork => {
  if (artwork.id === artworkId) {
    // Create a *new* object with changes
    return { ...artwork, seen: nextSeen };
  } else {
    // No changes
    return artwork;
  }
}));
```

위와 같은 접근 방식을 사용하면 기존 state 항목이 변이되지 않고, 두 list가 연동되는 버그가 수정된다. 

일반적으로 방금 만든 객체만 변이해야 한다. 새로운 artwork 를 삽입하는 경우에는 변이해도 되지만, 이미 있는 state의 artwork를 다루는 경우에는 복사본을 만드렁야 한다.

## **Write concise update logic with Immer**

> **Immer로 간결한 업데이트 로직 작성하기**
> 

중첩 배열을 변이 없이 업데이트 하는 작업은 객체를 다룰 때와 마찬가지로 반복적일 수 있다. 

Immer 를 사용하는 이유로 유사하다. 

- 일반적으로 state를 몇 레벨 이상 깊이 업데이트할 필요는 없습니다. state 객체가 매우 깊다면 [다르게 재구성](https://react-ko.dev/learn/choosing-the-state-structure#avoid-deeply-nested-state)하여 평평하게 만드는 것이 좋습니다.
- state 구조를 변경하고 싶지 않다면, [Immer](https://github.com/immerjs/use-immer)를 사용해보세요. Immer는 변이 구문을 사용하여 작성하더라도 자동으로 사본 생성을 처리해 주어 편리합니다.

```jsx
updateMyTodos(draft => {
  const artwork = draft.find(a => a.id === artworkId);
  artwork.seen = nextSeen;
});
```

Immer 가 제공하는 draft 객체를 변이하여 원본 state의 변이 없이 업데이트 로직을 작성할 수 있다. 

### 참조