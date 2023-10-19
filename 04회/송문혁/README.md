> [4회] 상호작용성 더하기 - 2

> 링크: https://bronzed-icicle-65c.notion.site/4-2-351ef77079f74fd582a87406809dfae3?pvs=4

# [4회] 상호작용성 더하기 - 2

## 렌더링 그리고 커밋

- 컴포넌트는 화면에 표시되기전에 React에서 렌더링 해야한다.
  ⇒ 코드가 어떻게 실행되고, React의 렌더링 동작하는지 알게 됨

### ex) 주방 → 고객까지 가는 과정

!https://react-ko.dev/images/docs/illustrations/i_render-and-commit1.png

!https://react-ko.dev/images/docs/illustrations/i_render-and-commit2.png

!https://react-ko.dev/images/docs/illustrations/i_render-and-commit3.png

1. 손님의 주문을 주방에 전달 (**렌더링 트리거**)
2. 주방에서 주문받은걸 준비 (**컴포넌트 렌더링**)
3. 주문한 요리를 손님 테이블에 두기 (**DOM에 커밋**)

### 1단계: 렌더링 트리거

렌더링 발생하는데는 두 가지 이유가 있음

1. **컴포넌트가 처음 렌더링 하는 경우**

   - createRoot + render를 사용하면 컴포넌트의 첫 렌더링이 실행된다.

     ```tsx
     import App from "./App";
     import { createRoot } from "react-dom/client";

     const root = createRoot(document.getElementById("root"));
     root.render(<App />);
     ```

2. **컴포넌트의 State(혹은 상위 요소 중 하나)가 업데이트 되는 경우 (리렌더링)**
   - useState의 두번째 인자인 set함수로 state를 업데이트해서 리렌더링을 발생시킨다.
     - state가 업데이트되면 자동으로 렌더링 동작이 대기열에 추가된다

### 2단계: React 컴포넌트 렌더링

- 렌더링을 실행하면, React는 컴포넌트를 호출해 화면에 표시할 내용을 확인한다.

  - **‘렌더링’: React에서 컴포넌트를 호출하는 것 (**⭐**중요**⭐)

- 첫 렌더링에서 React는 **root 컴포넌트를 호출**한다 (위 예시에서)
- 이후 React는 **state 업데이트에 의해 렌더링이 발동된 함수 컴포넌트를 호출**한다.

- 이 과정은 재귀

  - 업데이트된 컴포넌트가 다른 컴포넌트를 반환 ⇒ 그 컴포넌트를 렌더링하고 해당 컴포넌트도 다른 컴포넌트를 반환 ⇒ …
  - **중첩된 컴포넌트가 없고 React가 화면에 표시해야할 내용을 알 때까지 계속 들어간다**.

    <aside>
    💡 더 이상 들어갈 컴포넌트가 없다 === 제일 마지막에 있는 자식 컴포넌트
    ⇒ 자식 컴포넌트부터 렌더링되면서 상위 컴포넌트로 올라온다

    </aside>

- 함정

  렌더링은 **순수한 계산**이야 한다

  - 동일한 입력 ⇒ 동일한 출력이어야 한다
  - 이전의 state를 변경해선 안된다 (렌더링 전에 존재했던 객체, 변수를 변경X)

  strict mode 개발 목적 : React가 각 컴포넌트의 함수를 두번 호출해 순수하지 않은 함수가 있는지 확인하는 용도

- 성능 최적화
  업데이트된 컴포넌트 내에 중첩된 모든 컴포넌트를 렌더링하는 기본 동작은 업데이트된 컴포넌트가 트리에서 매우 높은 곳에 있는 경우 성능 최적화되지 않습니다.
  성능 문제가 발생하는 경우 [성능](https://reactjs.org/docs/optimizing-performance.html) 섹션에 설명된 몇 가지 옵트인 방식으로 문제를 해결 할 수 있습니다.
  **성급하게 최적화하지 마세요!**

### 3단계: React가 DOM에 변경사항을 커밋

컴포넌트 렌더링 후, React는 DOM을 수정한다

- 첫 렌더링일 땐, React는 appendChild() DOM API를 사용. 생성한 모든 DOM 노드를 화면에 표시
- 리렌더링일 땐, React는 필요한 최소한의 작업(렌더링하는 동안 계산된 것)을 적용하여 DOM이 최신 렌더링 출력과 일치하도록 함

> **React는 렌더링 간에 차이가 있을 때만 DOM을 변경한다**

### 에필로그: 브라우저 페인트

렌더링이 완료되고 React가 DOM 업데이트한 후, 브라우저는 화면을 다시 그립니다(**Repaint**)

이 과정을 **브라우저 렌더링**이라 한다

But, **React 컴포넌트 렌더링과의 혼동을 피하고자 Painting이라 부름**

### 요약

- React앱에서의 화면 업데이트
  - 트리거
  - 렌더링
  - 커밋
- strict mode는 컴포넌트가 순수한지를 체크하기 위한 목적을 지님
- 렌더링 결과가 이전과 같으면 React는 DOM을 건드리지 않음

---

## 스냅샷으로서의 State

- state 변수는 마치 javascript 변수처럼 보이지만 **스냅샷**처럼 동작한다
- staet 변수를 설정해도 이미 갖고 있는 state 변수는 변경되지 않고 리렌더링이 실행된다.

### state 설정으로 렌더링을 촉발

- 클릭 같은 사용자 이벤트에 반응해 UI가 직접 변경된다고 생각하면 됨
- React에서는 멘탈 모델과는 조금 다르게 동작 (?)

  - 인터페이스가 이벤트에 반응하기 위해선 state를 업데이트 해야 한다

    ```tsx
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
            e.preventDefault(); // 기존 form의 동작을 막아준다
            setIsSent(true); // isSent라는 State를 업데이트 해준다
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

    버튼 클릭시

    1. onSubmit 이벤트 핸들러가 실행
    2. setIsSent(true)로 인해 isSent가 true로 설정하고 새 렌더링을 큐에 대기 시킴
    3. React는 새로운 isSent 값에 따라 컴포넌트를 다시 렌더링

### 렌더링은 해당 시점의 스냅샷을 찍는다

- ‘**렌더링**’ : React가 컴포넌트(함수)를 호출한다는 뜻

  - 함수에서 반환하는 JSX는 시간상 UI의 스냅샷과 같다
  - prop, 이벤트 핸들러, 로컬 변수는 모두 렌더링 당시의 state를 사용해 계산

- 사진, 동영상 프레임과 달리 **UI 스냅샷은 대화형**이다
  - input에 대한 응답으로 어떤일이 일어날지 지정하는 이벤트 핸들러와 같은 로직이 포함됨
- React는 이 스냅샷과 일치하도록 화면을 업데이트하고 이벤트 핸들러를 연결합니다

  - 결과적으로 버튼을 누르면 JSX에서 클릭 핸들러가 발동됩니다

- **React가 컴포넌트를 다시 렌더링할 때**

  1. React가 함수를 다시 호출
  2. 함수가 새로운 JSX 스냅샷을 반환
  3. React가 반환한 스냅샷과 일치하도록 화면을 업데이트

- st**ate는 컴포넌트의 메모리**라서 함수 반환 후 사라지는 일반 변수와 다르다
  - state는 마치 React 자체에 존재함
  - R**eact가 컴포넌트를 호출하면 특정 렌더링에 대한 state의 스냅샷을 제공**
- 컴포넌트는 해당 렌더링의 state를 사용, 계산된 props 세트와 이벤트 핸들러가 포함된 UI 스냅샷을 JSX에 반환한다

- 업데이트 함수를 쓰지 않는다면, 동일한 set함수를 여러 번 호출해도 제일 마지막 한번만 적용된다.

  - 아래 예시에서 number는 마지막 set함수로 인해 `0 + 1 ⇒ 1`로 설정됨

  ```tsx
  // 예시
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

### 시간 경과에 따른 state

<aside>
💡 state는 javascript 변수처럼 보이지만 **스냅샷**처럼 동작한다는 것을 기억하자!

</aside>

- 예시

  ```tsx
  import { useState } from "react";

  export default function Counter() {
    const [number, setNumber] = useState(0);

    return (
      <>
        <h1>{number}</h1>
        <button
          onClick={() => {
            setNumber(number + 5);
            setTimeout(() => {
              alert(number);
            }, 3000);
          }}
        >
          +5
        </button>
      </>
    );
  }

  ////// timeout을 줘서 5가 나올 것 같지만 스냅샷임을 기억하자!
  setNumber(0 + 5);
  setTimeout(() => {
    alert(0);
  }, 3000);
  ```

- state 변수의 값은 이벤트 핸들러의 코드가 비동기적이더라도 **렌더링 내에서 절대 변경되지 않는다.**

- But, 렌더링하기 전에 최신 State을 읽고 싶다면 **state 업데이트 함수**를 사용하면 된다\

  - **💡참고**

    ```tsx
    function useState<S = undefined>(): [
      S | undefined,
      Dispatch<SetStateAction<S | undefined>>
    ];

    type SetStateAction<S> = S | ((prevState: S) => S); // 첫 번째가 단순 대체, 두 번째가 엡데이트 함수

    type Dispatch<A> = (value: A) => void;
    ```

### 요약

- state를 설정하면 새 렌더링을 요청한다.
- React는 컴포넌트 외부에 **마치 선반에 보관**하듯 state를 저장한다.
- `useState`를 호출하면 React는 해당 렌더링에 대한 **state의 스냅샷을 제공**한다.
- 변수와 이벤트 핸들러는 다시 렌더링해도 **남아남지** 않는다.
  - 모든 렌더링에는 자체 이벤트 핸들러가 있습니다.
- 모든 렌더링(과 그 안에 있는 함수)은 항상 React가 *렌더링*에 제공한 state의 스냅샷을 **보게**된다.
- 렌더링된 JSX에 대해 생각하는 것처럼 이벤트 핸들러에서 state를 정신적으로 대체할 수 있다.
- 과거에 생성된 **이벤트 핸들러는 그것이 생성된 렌더링 시점의 state 값을 갖는다**.

---

## 여러 State 업데이트를 큐에 담기

- state 변수를 설정하면 다음 렌더링이 큐에 들어간다
  - But, 렌더링을 큐에 넣기 전에 값에 대해 여러 작업일 해야 할 때가 있음
  - 이를 위해 state 업데이트를 어떻게 배치하면 좋을지 이해하는 것

### React Batch State 업데이트

```tsx
return (
  <>
    <h1>{number}</h1>
    <button onClick={() => {
      setNumber(number + 1);
      setNumber(number + 1);
      setNumber(number + 1);
    }}>+3</button>
  </>
)
////////
각 렌더링의 state는 고정(스냅샷!)되어 있으므로 위 코드에서 number는 0이다.
```

- **React는 state 업데이트를 하기 전에 이벤트 핸들러의 모든 코드가 실행될 때까지 대기한다**

  - 즉, 모든 set함수가 호출된 이후에 리렌더링이 발생한다. **(일괄처리(Batching))**
  - **너무 많은 리렌더링을 촉발하지 않게 하기 위해**
  - But, 이벤트 핸들러와 그 안의 코드가 다 실행되기 전에는 UI가 업데이트가 안된다는 의미

- **React는 클릭과 같은 _여러_ 의도적인 이벤트에 대해 일괄 처리하지 않는다**
  - 개별적으로 처리한다

### \***\*다음 렌더링 전에 동일한 state 변수를 여러 번 업데이트하기\*\***

- 다음 렌더링 전에 동일한 state 변수를 여러 번 업데이트 하고 싶다?

  - 다음 *state 값*을 전달하는 대신, 큐의 이전 state를 기반으로 다음 state를 계산하는 *함수*를 전달할 수 있다.
  - `setNumber(number + 1)` ⇒ `setNumber(prev ⇒ prev + 1)`
  - 이는 **React에게 state값으로 무언가를 하라** 라는 의미다

    <aside>
    💡 그래서 setState가 `value(대체 되는 것)`와 `(value) ⇒ void(갱신 되는 것)`를 타입을 갖는거구만

    </aside>

- 여기서 set함수의 입력 형식이 이전 state를 받아서 계산하는 형태 `setNumber(**prev ⇒ prev + 1**)`
  - **업데이터 함수(updater function)**라 부름

### \***\*state를 교체한 후 업데이트하면 어떻게 될까요\*\***

```tsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
      }}>Increase the number</button>
    </>
  )
}
/// 큐의 이전 state를 가져오기(n)에 6이 출력

Note: setState(x)가 실제로는 setState(n => x) 처럼 동작하는 것처럼 보이지만 n을 사용하지 않음!
```

### \***\*업데이트 후 state를 바꾸면 어떻게 되나요?\*\***

```tsx
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
////
0(초기값) => 5 => 6 => 42
```

- 이벤트 핸들러가 완료되면 React는 **리렌더링을 실행**합니다.
  - 리렌더링하는 동안 **React는 큐를 처리**합니다. (💡이 때 연산값인 42를 number에 갱신한다)
- 업데이터 함수는 렌더링 중에 실행되므로, **업데이터 함수는 [순수](https://react-ko.dev/learn/keeping-components-pure)해야 하며** 결과만 _반환_
- 업데이터 함수 내부에서 state를 변경하거나 다른 사이드 이팩트를 실행하려고 하지 말자
  - Strict 모드에서 React는 각 업데이터 함수를 두 번 실행(두 번째 결과는 버림)하여 실수를 찾을 수 있도록 도와준다.

### \***\*명명 규칙\*\***

- 보통 업데이터 함수의 인자는 해당 state변수의 첫 자를 지정하는게 일반적
  - 안 지켜도 상관없다.

### 요약

- state를 설정하더라도 **기존 렌더링의 변수는 변경되지 않음, 대신 새로운 렌더링을 요청**
- React는 **이벤트 핸들러가 실행을 마친 후 state 업데이트를 처리**합니다.
  - 이를 일괄처리(배칭, **batching**)라고 합니다. (**[참고](https://immigration9.github.io/react/2021/06/12/automatic-batching-react.html)**)
- 하나의 이벤트에서 일부 state를 여러 번 업데이트하려면 `setNumber(n => n + 1)` 업데이터 함수를 사용

---

## 객체 State 업데이트하기

- State는 객체 포함, javascript의 모든 종류의 값을 저장할 수 있음
  - 단, React state에 있는 객체 직접 변이는 X
  - **업데이트를 하려면 새 객체를 생성해서 복사본을 state에 넣어라**

### \***\*변이(mutation)이란?\*\***

- javascript 값은 불변(**변경이 불가**하거나 **읽기 전용**)
  - 원시 자료형(숫자, 문자열, 불리언)을 변경할 수 없음
- 단, 객체의 내용을 변경하는 건 가능 ⇒ 변이(mutation)
  ```tsx
  const [pos, setPos] = useState({x: 0, y: 0});
  ...
  pos.x = 5; // possible!
  ```
  - **다른 원시 자료형들과 동일하게 불변하는 것처럼 취급해야 함**
  - 고로, **새로운 객체를 생성**해서 **통째로 교체한다**

### \***\*State를 읽기 전용인 것처럼 다뤄라\*\***

- set함수를 사용하지 않으면 React는 객체가 변이됐다는걸 알지 못한다.
  - **객체를 수정하려면?**
    - **새로운 객체를 생성, 통째로 교체해야지 React가 변이됐음을 인지**한다.

```tsx
const [position, setPosition] = useState({
  x: 0,
  y: 0
});
return (
  <div
    onPointerMove={e => {
      setPosition({ // 새로운 객체로 대체(교체)해야 리렌더링이 촉발, 불변성이 유지된다
				x: e.clientX,
        y: e.clientY
			});
    }}
/////////
```

- **지역 변이는 괜찮다**

  ```tsx
  const nextPosition = {};
  nextPosition.x = e.clientX;
  nextPosition.y = e.clientY;
  setPosition(nextPosition);

  ///// 사실 아래와 똑같다
  setPosition({
    x: e.clientX,
    y: e.clientY,
  });
  ```

  - 방금 생성한 객체를 변경해도 아직 해당 객체를 다른 코드에서 참고하지 않았으니 괜찮다!
    - 이를 **지역 변이(local mutation)**이라 부름

### \***\*전개 문법(…objectName)으로 객체 복사하기\*\***

- 이전 예제에서 `position` 객체는 항상 현재 커서 위치에서 새로 만들어졌다.
- But, 종종 _기존_ 데이터를 새로 만드는 객체의 일부로 포함시키고 싶을 때가 있다.

  - 예를 들어, form에 있는 _하나의_ 필드만 업데이트하고 다른 모든 필드는 이전 값을 유지

    ```tsx
    import { useState } from 'react';

    export default function Form() {
      const [person, setPerson] = useState({
        firstName: 'Barbara',
        lastName: 'Hepworth',
        email: 'bhepworth@sculpture.com'
      });

      function handleFirstNameChange(e) {
        **setPerson({
          ...person, // person에 저장된 이전 값들 그대로 반영
          firstName: e.target.value
        });**
      }

      function handleLastNameChange(e) {
        **setPerson({
          ...person, // person에 저장된 이전 값들 그대로 반영
          lastName: e.target.value
        });**
      }

      function handleEmailChange(e) {
        **setPerson({
          ...person, // person에 저장된 이전 값들 그대로 반영
          email: e.target.value
        });**
      }
    ...
    ```

- 전개 문법은 얕기(Shallow) 때문에 객체에서 한 단계 깊이에서만 복사

  - 깊이가 깊을 때 **Immer**를 사용하면 편하다.

- **여러 필드에 단일 이벤트 핸들러 사용하기**

  - 객체 내에서 `[` 및 `]` 중괄호를 사용하여 [동적 이름을 가진 프로퍼티](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Object_initializer)를 지정할 수도 있다.
  - 세 개의 다른 이벤트 핸들러 대신 단일 이벤트 핸들러를 사용 (DOM요소의 name을 참고)

    ```tsx
    import { useState } from "react";

    export default function Form() {
      const [person, setPerson] = useState({
        firstName: "Barbara",
        lastName: "Hepworth",
        email: "bhepworth@sculpture.com",
      });

      function handleChange(e) {
        setPerson({
          ...person,
          [e.target.name]: e.target.value, //
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
            <input name="email" value={person.email} onChange={handleChange} />
          </label>
          <p>
            {person.firstName} {person.lastName} ({person.email})
          </p>
        </>
      );
    }
    ```

### \***\*중첩된 객체 갱신하기\*\***

- 다시 한번, **전개 문법은 얕기(Shallow) 때문에 객체에서 한 단계 깊이에서만 복사**
  - 깊이가 깊을 때는 **Immer**를 사용하면 편하다.

```tsx
setPerson({
  ...person, // Copy other fields
  artwork: {
    // but replace the artwork
    // 대체할 artwork를 제외한 다른 필드를 복사합니다.
    ...person.artwork, // with the same one
    city: "New Delhi", // but in New Delhi!
    // New Delhi'는 덮어씌운 채로 나머지 artwork 필드를 복사합니다!
  },
});
```

- **객체는 사실 중첩되지 않는다!**

  - 프로퍼티를 사용하여 다른 객체를 가리키는 거라 생각하자

  ```tsx
  let obj = {
    name: "Niki de Saint Phalle",
    artwork: {
      title: "Blue Nana",
      city: "Hamburg",
      image: "https://i.imgur.com/Sd1AgUOm.jpg",
    },
  };

  //////// 사실은 아래처럼 다른 두 객체를 바라보는 꼴이다

  let obj1 = {
    title: "Blue Nana",
    city: "Hamburg",
    image: "https://i.imgur.com/Sd1AgUOm.jpg",
  };

  let obj2 = {
    name: "Niki de Saint Phalle",
    artwork: obj1,
  };
  ```

### \***\*Immer로 간결한 업데이트 로직 작성하기\*\***

- state가 깊게 중첩된 경우 plat하게 만드는 걸 고려할 수 있지만,

  - Immer에서의 변이는 일반 변이와 달리 이전 state를 덮어쓰진 않음
  - Immer를 쓰면 간편하게 업데이트 로직 작성이 가능하다

    ```tsx
    import { useImmer } from 'use-immer';

    export default function Form() {
      **const [person, updatePerson] = useImmer({
        name: 'Niki de Saint Phalle',
        artwork: {
          title: 'Blue Nana',
          city: 'Hamburg',
          image: 'https://i.imgur.com/Sd1AgUOm.jpg',
        }
      });**

      function handleNameChange(e) {
        **updatePerson(draft => {
          draft.name = e.target.value;
        });**
      }

      function handleTitleChange(e) {
        **updatePerson(draft => {
          draft.artwork.title = e.target.value;
        });**
      }

      function handleCityChange(e) {
        **updatePerson(draft => {
          draft.artwork.city = e.target.value;
        });**
      }

      function handleImageChange(e) {
        **updatePerson(draft => {
          draft.artwork.image = e.target.value;
        });**
      }

      return (
        <>
          <label>
            Name:
            <input
              value={person.name}
              onChange={handleNameChange}
            />
          </label>
          <label>
            Title:
            <input
              value={person.artwork.title}
              onChange={handleTitleChange}
            />
          </label>
          <label>
            City:
            <input
              value={person.artwork.city}
              onChange={handleCityChange}
            />
          </label>
          <label>
            Image:
            <input
              value={person.artwork.image}
              onChange={handleImageChange}
            />
          </label>
          <p>
            <i>{person.artwork.title}</i>
            {' by '}
            {person.name}
            <br />
            (located in {person.artwork.city})
          </p>
          <img
            src={person.artwork.image}
            alt={person.artwork.title}
          />
        </>
      );
    }
    ```

- **Immer의 동작방식**

  - Immer에서 제공하는 `draft`는 [프록시](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy)라는 특수한 유형의 객체로, 사용자가 수행하는 작업을 **기록**한다.
    - 원하는 만큼 자유롭게 수정할 수 있다.
  - Immer는 **내부적으로 `draft`의 어떤 부분이 변경되었는지 파악하고 편집 내용이 포함된 새로운 객체를 생성**한다.

- **React에서 State 변이를 권장하지 않는 이유는?**
  **디버깅**
  - console.log를 사용하고 state를 변이하지 않으면, 과거의 기록이 최근 state 변이에 의해 지워지지 않는다.
  - 렌더링 사이에 state가 어떻게 변경되었는지 명확하게 확인할 수 있다.
    **최적화**
  - 일반적인 React [최적화 전략](https://react-ko.dev/reference/react/memo)은 이전 프로퍼티나 state가 다음 프로퍼티나 state와 동일한 경우 작업을 건너뛰는 것에 의존한다.
  - state를 변이하지 않는다면 변경이 있었는지 확인하는 것이 매우 빠르다.
    - 만약 `prevObj === obj` 라면, 내부에 변경된 것이 없다는 것을 확신할 수 있다.
      **새로운 기능**
  - 우리가 개발 중인 새로운 React 기능은 state가 [스냅샷처럼 취급](https://react-ko.dev/learn/state-as-a-snapshot)되는 것에 의존한다.
  - 과거 버전의 state를 변이하는 경우, 새로운 기능을 사용하지 못할 수 있다.
    **요구 사항 변경**
  - 실행 취소/다시 실행 구현, 변경 내역 표시, 사용자가 양식을 이전 값으로 재설정할 수 있도록 하는 것과 같은 일부 애플리케이션 기능은 아무것도 변이되지 않은 state에서 더 쉽게 수행할 수 있다.
  - 과거의 state 복사본을 메모리에 보관하고 필요할 때 재사용할 수 있기 때문
    - 변경 접근 방식으로 시작하면 나중에 이와 같은 기능을 추가하기 어려울 수 있다.
      더 간단한 구현
  - React는 변이에 의존하지 않기 때문에 객체에 특별한 작업을 할 필요가 없다.
  - 많은 “반응형” 솔루션처럼 프로퍼티를 가로채거나, 항상 프록시로 감싸거나, 초기화할 때 다른 작업을 할 필요가 없다.
    - 이것이 바로 React를 사용하면 추가 성능이나 정확성의 함정 없이 아무리 큰 객체라도 state에 넣을 수 있는 이유이기도 하다.
  - 실제로는 React에서 state를 변이해서라도 잘 **빠져나갈 수** 있겠지만,
  - state의 불변성을 유지하는 접근 방식을 염두에 두고 개발된 새로운 React 기능을 잘 사용할 수 있기 위해서, 그렇게 하지 말 것을 강력히 권장한다.

### 요약

- **React의 모든 state를 불변**으로 취급하자.
- **state에 객체를 저장**하면 객체를 변이해도 렌더링을 촉발하지 않고 **이전 렌더링 “스냅샷”의 state가 변경**된다.
- 객체를 변이하는 대신 **객체의 *새로운* 버전을 생성**하고 state를 설정하여 **다시 렌더링을 촉발**
- 객체 전개 구문 `**{...obj, something: 'newValue'}**`를 사용하여 객체 사본을 만들 수 있습니다.
- **전개 구문은 한 수준 깊이만 복사하는 얕은 구문이다**
- 중첩된 객체를 업데이트하려면 **업데이트하려는 위치에서 가장 위쪽까지 복사본을 만들어야 한다.**
- ~~반복적인 코드 복사를 줄이려면~~ 귀찮으면 **Immer를 사용**하자.

---

## 배열 State 업데이트하기

- 객체와 마찬가지로 **새로운 배열을 만들어서 복사본을 state에 넣어야 함**

### \***\*변경하지 않고 배열 업데이트하기\*\***

- 배열을 업데이트할 때마다 새로운 배열을 만들어서 전달하는 데

  - `filter`, `map`, `concat` 등의 배열 메소드를 활용

- **함정**
  slice: 배열 또는 배열의 일부를 복사할 수 있다. **(얘를 자주 씀)**
  splice: 배열에 항목을 삽입하거나 삭제하기 위해 **변이**한다

### \***\*배열에 항목 추가하기\*\***

- `push`, `unshift` 는 배열을 변이하기에 새 배열에 전개 구문(`…`)을 활용해서 항목을 추가하자
  ```tsx
  setArtists(
    // Replace the state
    [
      // with a new array
      ...artists, // that contains all the old items
      { id: nextId++, name: name }, // and one new item at the end
    ]
  );
  ////
  setArtists([
    { id: nextId++, name: name },
    ...artists, // Put old items at the end
  ]);
  ```

### \***\*배열에서 항목 제거하기\*\***

- `filter` 로 필터링해서 새로운 배열을 생성하자
  ```tsx
  setArtists(artists.filter((a) => a.id !== artist.id));
  ```

### \***\*배열 변경하기\*\***

- `map`을 사용해서 모든 또는 일부 항목을 변경해 새로운 배열을 반환받고 이를 set함수에 넣자
  ```tsx
  const **nextShapes** = shapes.**map**(shape => {
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
  // Re-render with the new array
  setShapes(nextShapes);
  ```

### \***\*배열 내 항목 교체하기\*\***

- 이 또한 `map`을 사용해서 수정하자
  - 여기서 `map`의 두번째 인수인 i를 활용해 목표 항목을 찾는다.
  ```tsx
  const nextCounters = counters.map((c, i) => {
    if (i === index) {
      // Increment the clicked counter
      return c + 1;
    } else {
      // The rest haven't changed
      return c;
    }
  });
  setCounters(nextCounters);
  ```

### \***\*배열에 항목 삽입하기\*\***

- `slice`와 `...`를 사용하자
  ```tsx
  const nextArtists = [
    // Items before the insertion point:
    ...artists.slice(0, insertAt),
    // New item:
    { id: nextId++, name: name },
    // Items after the insertion point:
    ...artists.slice(insertAt),
  ];
  setArtists(nextArtists);
  ```

### \***\*배열에 다른 변경 사항 적용하기\*\***

- `reverse`와 `sort`를 사용해 배열 반전, 정렬 가능
  - 단, **원래 배열을 변이 X**, 새로운 배열을 생성해서 진행
    ```tsx
    const nextList = [...list];
    nextList.reverse();
    setList(nextList);
    ```
  - **배열을 복사하더라도 배열 내부의 기존 항목을 직접 변이할 수는 없다.**
    - 이는 **얕은 복사(shallow copy)로 배열이 생성**되었기 때문
    - **복사된 배열 내부의 객체를 수정하면 기존 state를 변이**한다.
    ```tsx
    // 아래처럼 내부 항목을 직접접근해서 변이하려고 하면 실제 state의 값도 변이 => 불변성 X
    const nextList = [...list];
    nextList[0].seen = true; // Problem: mutates list[0]
    setList(nextList);
    ```

### \***\*배열 내부의 객체 업데이트하기\*\***

- **객체는 _실제로_ 배열 “내부”에 위치하지 않다.**

  - 코드에서는 “내부”에 있는 것처럼 보일 수 있지만 **배열의 각 객체는 배열이 “가리키는” 별도의 값**이다.

    <aside>
    💡 c / c++를 공부했다면 **pointer**를 떠올리면 될 것 같다.

    </aside>

- **중첩된 state를 업데이트할 때는**

  - **업데이트하려는 지점부터 최상위 수준까지 복사본을 만들어야 한다**

    ```tsx
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
        **setMyList(myList.map(artwork => {
          if (artwork.id === artworkId) {
            // Create a *new* object with changes
            return { ...artwork, seen: nextSeen };
          } else {
            // No changes
            return artwork;
          }
        }));**
      }

      function handleToggleYourList(artworkId, nextSeen) {
        **setYourList(yourList.map(artwork => {
          if (artwork.id === artworkId) {
            // Create a *new* object with changes
            return { ...artwork, seen: nextSeen };
          } else {
            // No changes
            return artwork;
          }
        }));**
      }
    ...
    }
    ```

### \***\*Immer로 간결한 업데이트 로직 작성하기\*\***

- 객체 state를 업데이트할 때 Immer를 쓴 케이스와 동일하다

  - Immer를 사용한다면 `push`, `pop`같은 변이 메서드를 draft에 적용할 수 있다.

  ```tsx
  const [myList, updateMyList] = useImmer(initialList);
  const [yourList, updateYourList] = useImmer(initialList);

  function handleToggleMyList(id, nextSeen) {
    updateMyList((draft) => {
      const artwork = draft.find((a) => a.id === id);
      artwork.seen = nextSeen;
    });
  }
  ```

### 요약

- 배열을 state에 넣을 수는 있지만 변경할 수는 없다.
- 배열을 변이하는 대신 배열의 *새로운* 버전을 생성하고 state를 업데이트.
- 배열 전개 구문 `[...arr, newItem]`을 사용하여 새 항목으로 배열을 만들 수 있다.
- `filter()` 및 `map()`을 사용하여 필터링되거나 변형된 항목으로 새 배열을 만들 수 있다.
- Immer를 사용하면 코드를 간결하게 유지할 수 있다.

---

## 4회차때 얻은 것

- 렌더링과 페인팅 차이
- 렌더링 과정
- setState의 동작 방식
- setState의 입력 인자가 value인 것과 function인 것의 차이
- 객체와 배열을 set할 때 권장 하는 방식
