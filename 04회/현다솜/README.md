4 - 상호작용성 더하기 (2)

# 렌더링 그리고 커밋

컴포넌트를 화면에 표시하기 이전에 React에서 렌더링을 이해해야한다. 해당 과정의 단계를 이해하면 코드가 어떻게 실해되는지 이해할 수 있고 리액트 렌더링 동작에 관해 설명하는데 도움이 된다.

### UI를 요청하고 제공하는 세 가지 단계

![i_render-and-commit1.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/f076ced0-fa44-460f-a1f6-67fa9565b764/157230bb-a107-4200-8ea1-8f7489e68356/i_render-and-commit1.png)

1. 렌더링 트리거

![i_render-and-commit2.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/f076ced0-fa44-460f-a1f6-67fa9565b764/e12cbf1f-ec3a-4f7d-8c9d-55760c58cd95/i_render-and-commit2.png)

2. 컴포넌트 렌더링

![i_render-and-commit3.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/f076ced0-fa44-460f-a1f6-67fa9565b764/f98a4b92-1c78-4668-b995-678c30dd520e/i_render-and-commit3.png)

3. DOM에 커밋

---

## 1단계: 렌더링 트리거

컴포넌트 렌더링이 일어나는 이유는 두가지가 있다.

1. 컴포넌트의 초기 렌더링인 경우
2. 컴포넌트의 state가 업데이트된 경우

### 초기 렌더링

처음 앱을 실행했을 때, 초기 렌더링을 트리거하게 된다. 프레임워크나 샌드박스는 때때로 이러한 코드가 숨어있지만 이는 DOM node에 `createRoot` 를 요청하고, `render` 메서드를 호출한다.

```jsx
import Image from "./Image.js";
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"));
root.render(<Image />);
```

### State 업데이트시 리렌더링

컴포넌트가 초기 렌더링된 이후 state 변경을 함으로 렌더링을 트리거할 수 있다. 컴포넌트 상태의 업데이트는 자동으로 렌더 큐에 쌓이게 된다.

## 2단계: React 컴포넌트 렌더링

렌더링을 트리거한 후 React는 컴포넌트를 호출하여 화면에 표시할 내용을 파악한다. `렌더링` 은 React에서 컴포넌트를 호출하는 것이다.

- 초기 렌더링: React는 루트 컴포넌트를 호출
- 이후 렌더링: React는 state 업데이트가 일어나 렌더링을 트리거한 컴포넌트를 호출

→ 재귀적 단계: 업데이트된 컴포넌트가 다른 컴포넌트를 반환하면 React는 다음으로 해당 컴포넌트를 렌더링하고, 해당 컴포넌트를 반환하면 반환된 컴포넌트를 다음에 렌더링하는 방식이다. 중첩된 컴포넌트가 더이상 없고 React가 화면에 표시되어야하는 내용을 정확히 알 때까지 이 단계는 계속된다.

```jsx
export default function Gallery() {
  return (
    <section>
      <h1>Inspiring Sculptures</h1>
      <Image />
      <Image />
      <Image />
    </section>
  );
}

function Image() {
  return (
    <img
      src="https://i.imgur.com/ZF6s192.jpg"
      alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
    />
  );
}
```

- 초기 렌더링을 하는 동안 React는 `<section>` , `<h1>` 그리고 3개의 `<img>`태그에 대한 DOM 노드를 생성한다.
- 리렌더링하는 동안 React는 이전 렌더링 이후 변경된 속성을 계산한다. 다음 단계인 커밋 단계까지는 해당 정보로 아무런 작업도 수행하지 않는다.

<aside>
💡 렌더링은 항상 순수한 계산:
- 동일한 입력에는 동일한 출력을 해야한다.
- 이전의 state를 변경해서는 안된다.(렌더링 전에 존재했던 객체나 변수를 변경해서는 안된다)
그렇지 않으면 코드 베이스가 복잡해짐에 따라 혼란스러운 버그와 예측할 수 없는 동작이 발생할 수 있다.

</aside>

<aside>
💡 Optimizing performance
업데이트된 컴포넌트 내에 중첩된 모든 컴포넌트를 렌더링하는 기본 동작은 업데이트된 컴포넌트가 트리에서 매우 높은 곳에 위치할 경우 성능이 최적화되지 않는다. (성급하게 최적화하지 말자)

</aside>

## 3단계: React가 DOM에 변경사항을 커밋

컴포넌트를 렌더링(호출)한 후 React는 DOM을 수정한다.

- 초기 렌더링의 경우: React는 `appendChild()` DOM API를 사용하여 생성한 모든 DOM노드를 화면에 표시한다.
- 리렌더링의 경우: React는 필요한 최소한의 작업(렌더링하는 동안 계산된 것)을 적용하여 DOM이 최신 렌더링 출력과 일치하도록 한다.

React는 렌더링 간 차이가 있는 경우에만 DOM 노드를 변경한다.

## 에필로그: 브라우저 페인트

렌더링이 완료되고 React가 DOM을 업데이트한 후 브라우저는 화면을 다시 그린다. 이 단계를 `브라우저 렌더링` 이라고 하지만 이 문서의 나머지 부분에서 혼동을 피하고자 `페인팅` 이라고 부를 것이다.

---

# 스냅샷으로서의 State

state 변수는 읽고 쓸 수 있는 것 처럼 보이지만 스냅샷처럼 동작한다. state 변수를 설정하여도 이미 가지고 있는 state 변수는 변경되지 않고 대신 리렌더링이 발동된다.

## state를 설정하면 렌더링이 동작한다

우리는 보통 클릭과 같은 사용자 이벤트에 반응하여 사용자 인터페이스가 직접 변경된다고 생각할 수 있다. 하지만 React에서는 이 멘탈 모델과는 조금 다르게 동작된다. 인터페이스가 이벤트에 반응하려면 state를 업데이트해야 한다.

```jsx
import { useState } from "react";

export default function Form() {
  const [isSent, setIsSent] = useState(false);
  const [message, setMessage] = useState("Hi!");
  if (isSent) {
    return <h1>Your message is on its way!</h1>;
  }
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault();
        setIsSent(true);
        sendMessage(message);
      }}
    >
      <textarea
        placeholder="Message"
        value={message}
        onChange={(e) => setMessage(e.target.value)}
      />
      <button type="submit">Send</button>
    </form>
  );
}

function sendMessage(message) {
  // ...
}
```

이 예시는 `send`버튼을 누르면 `setIsSend(true)`에서 React에서 UI를 다시 렌더링하도록 지시한다.

1. `onSubmit` 이벤트 핸들러 실행
2. `setIsSent(ture)`가 isSent를 true로 설정하고 새로운 렌더링을 큐에 넣음
3. React는 새로운 isSent 값에 따라 컴포넌트를 다시 렌더링

## 렌더링은 그 시점의 스냅샷을 찍는다

렌더링이란 React가 컴포넌트, 즉 함수를 호출한다는 뜻이다. 해당 함수에서 반환하는 JSX는 시간상 UI의 스냅샷과 같다. prop, 이벤트 핸들러, 로컬 변수 모두 렌더링 당시의 state를 사용해 계산된다.

사진이나 동영상 프레임과 달리 반환하는 UI 스냅샷은 대화형이다. 여기에는 입력에 대한 응답으로 어떤 일이 일어날지 설정하는 이벤트 핸들러와 같은 로직이 포함된다. 그러면 React는 이 스냅샷과 일치하도록 화면을 업데이트하고 이벤트 핸들러를 연결한다. 결과적으로 버튼을 누르면 JSX의 클릭 핸들러가 발동된다.

React가 컴포넌트를 다시 렌더링할 때

![i_render1.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/f076ced0-fa44-460f-a1f6-67fa9565b764/c63d1342-cb35-4d59-9a7f-9c76a539d6e7/i_render1.png)

1. React가 함수를 다시 호출

![i_render2.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/f076ced0-fa44-460f-a1f6-67fa9565b764/5b1576f7-e68b-43ea-9346-613d20631c54/i_render2.png)

2. 함수가 새로운 JSX 스냅샷을 반환

![i_render3.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/f076ced0-fa44-460f-a1f6-67fa9565b764/fcd44ffd-f0b0-443f-a141-7c3aec58606b/i_render3.png)

3. React가 반환한 스냅샷과 일치하도록 화면을 업데이트(DOM tree 업데이트)

컴포넌트의 메모리로써 state는 함수가 반환된 후 사라지는 일반 변수와 다르다. state는 실제 함수 외부에 마치 선반에 있는 것처럼 React 자체에 ‘존재’한다. React가 컴포넌트를 호출하면 특정 렌더링에 대한 state의 스냅샷을 제공한다. 컴포넌트는 해당 렌더링의 state 값을 사용해 계산된 새로운 props 세트와 이벤트 핸들러가 포함된 UI의 스냅샷을 JSX에 반환한다.

![i_state-snapshot1.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/f076ced0-fa44-460f-a1f6-67fa9565b764/0d17c400-f0c2-4e96-869f-45fdd33ba842/i_state-snapshot1.png)

React 에 state를 업데이트 하라고 명령

![i_state-snapshot2.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/f076ced0-fa44-460f-a1f6-67fa9565b764/dea5e966-7f10-42db-b6ac-47fcb98df5aa/i_state-snapshot2.png)

React가 state 값을 업데이트

![i_state-snapshot3.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/f076ced0-fa44-460f-a1f6-67fa9565b764/cb8badc5-2691-4c67-9abd-59352d653c25/i_state-snapshot3.png)

React는 상태값의 스냅샷을 컴포넌트에 전달

```jsx
import { useState } from "react";

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button
        onClick={() => {
          setNumber(number + 1);
          setNumber(number + 1);
          setNumber(number + 1);
        }}
      >
        +3
      </button>
    </>
  );
}
```

→ 이 예제는 스냅샷으로 동작한다는걸 보여준다. (실제로 number는 +3이 아닌 +1 되어 나타난다)

state를 설정하면 다음 렌더링에 대해서만 변경이된다. setNumber(number + 1)을 세번 호출했지만 이벤트 핸들러에서 number는 항상 0이기 때문에 state를 1로 세번 설정한 것이다.

## 시간 경과에 따른 State

같은 이벤트 핸들러 내에서 state의 값을 변경하고 setTimeout 같은 타이머를 이용하여 해당 상태값을 불러오더라도 상태값은 이전의 값을(스냅샷 된) 참조하고 있다.

state 변수의 값은 이벤트 핸들러의 코드가 비동기적이더라도 렌더링 내에서 절대 변경되지 않는다.

이러한 실수를 줄이는 방법을 다음장에서 알아보자.

---

# state 업데이트 큐

state 변수를 설정하면 다음 렌더링 큐에 들어간다. 하지만 때에 따라 다음 렌더링을 큐에 넣기 전에 값에 대해 여러 작업을 수행하고 싶을 때도 있다. 이를 위해서 state 업데이트를 어떻게 배치하면 좋을지 이해하는 것이 도움된다.

## React state batches 업데이트

위의 예제처럼 setNumber(number + 1)을 여러번 호출한다고 하더라도 state 값이 고정되어 있어 실제로 한번만 호출한 것 처럼 결과가 나온다. 여기는 한가지 더 요인이 있는데 React는 state를 업데이트 하기 전에 이벤트 핸들러의 모든 코드가 실행될 때 까지 기다린다는 것이다. 때문에 리렌더링은 모든 setNumber() 가 호출이 완료된 이후에만 일어난다.

우리는 이러헥 해서 너무 많은 리렌더링이 발생하지 않고도 여러컴포넌트의 다수의 state 변수를 업데이트 할 수 있다. 하지만 이벤트 핸들러와 그 안에 있는 코드가 완료될 때 까지 UI를 업데이트 하지 않는다는 의미기도하다. ‘batching’ 이라고도 하는 이 동작은 React 앱을 훨씬 빠르게 실행할 수 있게 도와준다.

React는 클릭과 같은 여러 의도적인 이벤트에 대해서는 batch 하지 않으며 각 클릭은 개별적으로 처리된다. React는 안전한 경우에만 batch 를 수행하니 안심하자.

## 다음 렌더링 전에 동일한 state 변수를 여러번 업데이트 하기

```jsx
setNumber(number + 1);
// 대신 사용하기
setNumber((n) => n + 1);
```

→ 이러한 방법으로 호출하면 n은 큐에 쌓일 때 저장되는 것이 아닌 실행될 때 결정되기 때문에 같은 이벤트 핸들러 내에서 여러번 업데이트가 가능하다.

## state를 교체한 후 업데이트하면 어떻게 될까?

```jsx
const handleClick = () => {
  setNumber(number + 5);
  setNumber((n) => n + 1);
};
```

→ +6 이 적용된다

## 업데이트 후 state를 바꾸면 어떻게 될까?

```jsx
const handleClick = () => {
  setNumber((n) => n + 1);
  setNumber(number + 5);
};
```

→ +5 가 적용된다.

## 명명 규칙

업데이트 함수 인수의 이름은 해당 state 변수의 첫글자로 지정하는 것이 일반적이다.

```jsx
setEnabled((e) => !e);
setLastName((ln) => ln.reverse());
// 혹은
setEnabled((enabled) => !enabled);
// 혹은
setEnabled((prev) => !prev);
```

### 챌린지 2

state 큐를 직접 구현해보자

```jsx
export function getFinalState(baseState, queue) {
  let finalState = baseState;

  queue.forEach((q) => {
    if (typeof q === "number") {
      finalState = q;
    } else {
      finalState = q(finalState);
    }
  });

  return finalState;
}
```

---

# 객체 State 업데이트 하기

state는 객체를 포함한 모든 종류의 자바스크립트 값을 가질 수 있다. 하지만 리액트 state가 가진 객체를 직접 변경해서는 안된다. 객체를 업데이트 하기 위해서는 새로운 객체를 생성하거나 복사본을 이용해야한다.

## 변경이란?

지금까지 state에서 숫자, 문자열, 불리언 값을 다뤘다. 이러한 자바스크립트 값들은 변경할 수 없거나 ‘읽기전용’을 의미하는 ‘불변성’을 가진다. 값을 교체하기 위해서는 리렌더링이 필요하다.

```jsx
const [x, setX) = useState(0);

setX(5);
```

x state는 5로 바뀌었지만 숫자 0 자체는 바뀌지 않았다. 숫자나 문자열, 불리언과 같이 자바스크립트에 정의되어있는 원시 값들은 변경할 수 없다.

```jsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

기술적으로 객체 자체의 내용을 바꿀수는 있다. 이것을 변경(Mutation)이라고 한다. 하지만 리액트 state의 객체들이 기술적으로 변경 가능할지라도 불변성을 가진것 처럼 다루어야 한다. → 객체를 변경하는 대신 교체해야한다.

## State를 읽기 전용인 것처럼 다루자

state에 저장한 자바스크립트 객체는 어떤 것이라도 읽기 전용인 것처럼 다루어야 한다. → 리액트에서 객체가 변경되었는지 알 수 없다.(리렌더링이 일어나지 않음)

## 전개 문법으로 객체 복사하기

```jsx
setPerson({
  ...person,
  lastName: "dasom",
});
```

객체 state의 이전 값을 쓰고싶을 때 쓰면 좋다.

- `…` 전개 문법은 “얕다”는 점을 알아두자. 이는 한 레벨 깊이의 내용만 복사한다.

<aside>
💡 여러 필드를 하나의 이벤트 핸들러로 사용하기
`[ ]` 대괄호를 이용하여 동적으로 키 값을 넣어줄 수 있다.

</aside>

## 중첩된 객체 갱신하기

```jsx
setPerson({
  ...person,
  artWork: {
    ...person.artWork,
    city: "New Delhi",
  },
});
```

코드가 길어질 수 있지만 정상적으로 동작하게 하기 위해 이용한다.

## Immer로 간결한 갱신 로직 작성하기

state가 깊이 중첩되어 있다면 평탄화(flat)를 고려해보자. 만약 구조를 변경하고 싶지 않다면 Immer를 사용해보자 (라이브러리). Immer를 이용하면 작성한 코드는 “법칙을 깨고” 객체를 변경하는 것 처럼 보일 수 있다

→ 개인적으로는 굳이 안쓰는게 좋을 것 같다

<aside>
💡 리액트는 왜 mutating state를 추천하지 않을까?
- 디버깅 : 가장 최근 값을 바뀔 때 마다 확인 가능 (console.log)
- 최적화 : state 가 이전 값과 같을 때 렌더링을 건너뛰고 아무것도 하지 않음으로 써 최적화 할 수 있다.
- 새로운 기능: 리액트 새 기능은 스냅샷에 의존한다.
- 요구사항 변화: 복사본을 이용하기 때문에 이전값으로 되돌리기 쉽다.
- 더 간단한 구현: 리액트는 변경에 의존하지 안힉 때문에 객체로 뭔가 특별한 것을 더 할 필요가 없다

</aside>

---

# 배열 State 업데이트 하기

배열도 마찬가지로 state를 업데이트 하고 싶을 때는 새 배열을 생성해서 업데이트 해야한다

## 변경하지 않고 배열 업데이트하기

filter 나 map 메서드 이용

- 새 배열을 반환하는 메서드
  concat, […arr], filter, slice, map, reduce 등

## 배열에 항목 추가하기

push()를 사용하지 않도록 조심하자.

```jsx
setArtist([...artist, { id: nextId, name: nextName }]);
// 혹은
setArtist([...artist].push({ id: nextId, name: neextName }));
```

## 배열에서 항목 제거하기

filter를 활용하

## 배열 변환하기

map

## 배열 내 항목 교체하기

map

## 배열에 항목 추가하기

`[...]` + slice

## 배열에 기타 변경 적용하기

전개구문, map, filter

reverse나 sort 의 경우 원본 배열을 변경시키기 때문에 복사한 뒤 사용하자.

## 배열 내부의 객체 업데이트하기

기존 배열을 업데이트 하지 않도록 조심하자

map 이용
