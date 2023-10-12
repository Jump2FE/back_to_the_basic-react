> [5회] State 관리하기 - 1

> 링크: https://bronzed-icicle-65c.notion.site/5-State-1-192fea181a2c449481458eea2d74224b?pvs=4

# [5회] State 관리하기 - 1

## state로 입력에 반응하기

### 선언형과 명령형의 차이

- 선언형
  - 원하는 결과를 명시하고 컴퓨터가 그 결과를 얻기 위해 어떻게 수행할지 결정하는 방식
  - 무엇을 원하는지를 설명하지 구체적인 지시사항은 제공하지 않음
    - 대표적으로 React, SQL
- 명령형
  - 하나부터 열까지 어떤 행동을 취해야 하는지를 지시하는 방식
    - 대표적으로는 C, C++, Java

### UI를 선언형으로 생각하기

1. 컴포넌트의 다양한 시각적 상태를 **식별**한다.
2. 상태 변화를 촉발하는 요소를 **파악**한다.
   - 사람 아니면 컴퓨터의 입력으로 촉발한다
   - 시각화하면 더 좋다
3. `useState`를 사용하여 메모리의 상태를 **표현**한다.
   - 중복이 있어도 되니 되도록 모든 상태를 표현한다
4. 비필수적인 state 변수를 **제거**한다.
   - 앞서 표현한 상태들 중 중복되거나 의미없는 state는 제거한다
5. 이벤트 핸들러를 **연결**하여 state를 설정한다.
   - 살아남은 state를 변경시키는 이벤트 핸들러에 state 변화 로직을 작성한다.

- **한번에 여러 시각적 상태 표시하기**

  - 아예 한 페이지에서 컴포넌트의 시각적 상태를 다 나열하는게 좋을 수 있음
  - storybook(그 Storybook이랑은 다른 얘기) 혹은 living styleguide라고도 한다
  - 예시

    ```tsx
    import Form from "./Form.js";

    let statuses = ["empty", "typing", "submitting", "success", "error"];

    export default function App() {
      return (
        <>
          {statuses.map((status) => (
            <section key={status}>
              <h4>Form ({status}):</h4>
              <Form status={status} />
            </section>
          ))}
        </>
      );
    }
    ```

- **reducer로 불가능한 state 제거하기**
  - 세 state 변수(`typing`, `submitting`, `sucess`)는 예제의 form의 상태를 충분히 잘 표현하지만 완전히 설명되지 않는 중간 상태가 몇 가지 있다.
    - ex) null이 아닌 `error`는 `status`가 `'success'`일 때는 의미가 없다.
  - state를 조금 더 정확하게 모델링하면 [reducer로 분리](https://react-ko.dev/learn/extracting-state-logic-into-a-reducer)하면 된다.
    - reducer를 사용하면 여러 state 변수를 하나의 객체로 통합하고 관련된 모든 로직도 합칠 수 있다

### 요약

- **선언형 프로그래밍**은 UI를 세밀하게 관리(명령형)하지 않고 **각 시각적 상태에 대해 UI를 기술하는 것**을 의미
- 컴포넌트를 개발할 때
  1. **모든 시각적 상태를 식별**
  2. 사람 및 컴퓨터가 **상태 변화를 촉발하는 요인을 결정**
  3. `useState`로 상태를 모델링
  4. 버그와 모순을 피하려면 **비필수적인 state를 제거**
  5. **이벤트 핸들러를 연결**하여 state를 설정

---

## State 구조 선택하기

### State 구조화 원칙

1. **관련 state를 그룹화한다.**
   - 항상 **두 개 이상의 state 변수를 동시에 업데이트**하는 경우 **단일 state 변수로 병합**하라
   - ex) 좌표는 x, y가 필요한데 별도의 state가 아닌 하나의 객체 state로 관리
2. **state의 모순을 피하라.**
   - 여러 state 조각이 서로 모순되거나 ‘불일치’할 수 있는 방식
   - ex) Form을 submit하는 과정에서 `isSending`과 `isSent`가 동시에 `true`가 되는 모순
3. **불필요한 state를 피하라.**
   - 렌더링 중에 컴포넌트의 props나 기존 state 변수에서 일부 정보를 계산할 수 있다면 해당 정보를 해당 컴포넌트의 state에 넣지 않아야 한다.
   - ex) `fullname`은 `firstName`과 `LastName`을 합치면 표현할 수 있으니 state로 넣지 않는다
4. **state 중복을 피하라.**
   - 동일한 데이터가 여러 state 변수 간에 또는 중첩된 객체 내에 중복되면?
     ⇒ **state의 동기화를 유지하기가 어렵다.**
   - ex) Items와 selectItem을 별도 state로 하다가 둘 중 하나의 업데이트를 놓칠 수 있음
5. **깊게 중첩된 state는 피하라**
   - 깊게 계층화된 state는 업데이트하기 쉽지 않습니다.
   - 가능하면 state를 평평한 방식으로 구성하는 것이 좋다.
     - Immer를 사용하는 방안도 있음

- **주의**
  state 변수가 객체인 경우 다른 필드를 명시적으로 복사하지 않고는 [그 안의 한 필드만 업데이트할 수는 없다](https://react-ko.dev/learn/updating-objects-in-state)
  - 전개 문법을 사용하자! ⇒ `setPosition({ ...position, x: 100 })`
- **Props를 state에 그대로 미러링 하지 말 것 (3번 예시)**
  ```tsx
  function Message({ messageColor }) {
    const [color, setColor] = useState(messageColor);
  ```
  여기서 `color` state 변수는 `messageColor` props로 초기화됩니다.
  문제는 **부모 컴포넌트가 나중에 다른 `messageColor` 값(예: `blue` 대신 `red`)을 전달하면 `color` state 변수가 업데이트되지 않는다는 것!** (위 코드에서는 state는 첫 번째 렌더링 중에만 초기화)
  그렇기 때문에 state 변수에 일부 prop을 **미러링**하면 혼란을 초래할 수 있습니다. 대신 코드에서 `messageColor` prop을 **직접 사용**하세요. 더 짧은 이름을 지정하려면 상수를 사용하세요:
  ```tsx
  function Message({ messageColor }) {
    const color = messageColor;
  ```
  이렇게 하면 부모 컴포넌트에서 전달된 prop과 동기화되지 않는다.
  **props를 state로 미러링하는 것**은 **특정 prop에 대한 모든 업데이트를 무시하려는 경우**에만 의미가 있다. (관례에 따라 prop 이름을 `initial` 또는 `default`로 시작)
  ```tsx
  function Message({ initialColor }) {
    // The `color` state variable holds the *first* value of `initialColor`.
    // Further changes to the `initialColor` prop are ignored.
    const [color, setColor] = useState(initialColor);
  ```
- **메모리 사용량 개선하기 (5번 예시)**
  - 이상적으로는 **중첩된 객체에서 삭제된 항목(및 그 하위 항목!)도 제거하여 메모리 사용량을 개선하는 것이 좋다**.
  - 추가로 [Immer를 사용](https://react-ko.dev/learn/updating-objects-in-state#write-concise-update-logic-with-immer)하여 업데이트 로직을 더욱 간결하게 만들 수 있다

### 요약

- 두 개의 state 변수가 항상 함께 업데이트되는 경우
  - **두 변수를 하나로 병합**하자
- state 변수를 신중하게 선택하여 **불가능한** state를 만들지 않도록 하라
- state를 업데이트할 때 **실수할 가능성을 줄이는 방식으로 state를 구성**하라
- 동기화 state를 유지할 필요가 없도록 **불필요 및 중복 state를 피하**라
- 특별히 업데이트를 막으려는 경우가 아니라면 **props를 state에 넣지 마라**
- 선택과 같은 UI 패턴의 경우 객체 자체 대신 **ID 또는 인덱스를 state로 유지**한다
- 깊게 중첩된 state를 업데이트하는 것이 복잡하다면 **state를 평평하게 만들어 보라**
  - 아니면 Immer를 사용하라

---

## 컴포넌트 간의 state 공유하기

- 두 컴포넌트의 state가 항상 함께 변하고 싶을 때, **가장 가까운 공통 부모 컴포넌트로 state를 끌어올려서 props로 전달**하라

### State 끌어올리기

1. 자식 컴포넌트에서 state를 **제거**
2. 공통 부모 컴포넌트에 하드 코딩된 데이터를 **전달**
3. 공통 부모 컴포넌트에 state를 **추가**하고 이벤트 핸들러와 함께 전달

- **제어(Controlled) 및 비제어(Uncontrolled) 컴포넌트**

  - 일반적으로 **일부 로컬 state를 가진** 컴포넌트를 **비제어 컴포넌트**라고 부른다.
    - 예를 들어, `isActive` state 변수가 있는 원래 `Panel` 컴포넌트는 부모가 패널의 활성화 여부에 영향을 줄 수 없기 때문에 제어되지 않는다.
  - 컴포넌트의 중요한 정보가 자체 로컬 state가 아닌 **props에 의해 구동되는** 컴포넌트를 **제어 컴포넌트**라고 부른다.

    - 이렇게 하면 부모 컴포넌트가 그 동작을 완전히 지정할 수 있다.
    - 최종 `Panel` 컴포넌트에는 `isActive` props가 있으며, `Accordion` 컴포넌트에 의해 제어된다.

  - 비제어 컴포넌트는 구성이 덜 필요하기 때문에 상위 컴포넌트 내에서 사용하기가 더 쉽다.
  - But, 함께 통합하려는 경우 유연성이 떨어진다.

    - 제어 컴포넌트는 최대한의 유연성을 제공하지만 부모 컴포넌트가 props를 사용하여 완전히 구성해야 한다.

  - 실제로 **제어**와 **비제어**는 엄격한 기술 용어가 아니며, 각 컴포넌트에는 **일반적으로 로컬 state와 props가 혼합**되어 있다.
  - But, 컴포넌트가 어떻게 설계되고 어떤 기능을 제공하는지에 대해 이야기할 때 유용

  - 컴포넌트를 작성할 때는 (props를 통해) 컴포넌트에서 어떤 정보를 제어해야 하는지,
  - (state를 통해) 어떤 정보를 제어하지 않아야 하는지 고려하라
    - But, 나중에 언제든지 마음을 바꾸고 리팩토링할 수 있다.
      **참고 (Form 요소에 대한 예시도 한번 보자)**
      [[React] 제어 컴포넌트(Controlled Component)와 비제어 컴포넌트(Uncontrolled Component)](https://dori-coding.tistory.com/entry/React-제어-컴포넌트Controlled-Component와-비제어-컴포넌트Uncontrolled-Component)

### 각 State의 단일 진실 공급원(SSOT)

- React 애플리케이션(이하 앱)에서 많은 컴포넌트는 고유한 state를 가지고 있다.
- 일부 state는 입력값과 같이 [leaf 컴포넌트](https://stackoverflow.com/questions/65278395/what-do-you-mean-by-leaf-components-in-react)(트리의 맨 아래에 있는 컴포넌트)에 가깝게 **위치**할 수 있고 다른 state는 앱의 상단에 더 가깝게 **위치**할 수 있다.

  - ex) 클라이언트 측 라우팅 라이브러리도 일반적으로 현재 경로를 React state에 저장하고 props를 통해 전달하는 방식

- **각 고유한 state들에 대해 해당 state를 “소유”하는 컴포넌트를 선택하게 된다.**
  - 이 원칙을 ”[단일 진실 공급원](https://en.wikipedia.org/wiki/Single_source_of_truth)“
  - 모든 state가 한 곳에 있다는 뜻이 아니라, 각 state마다 해당 정보를 소유하는 특정 컴포넌트가 있다는 뜻
  - 컴포넌트 간에 공유하는 state를 복제하는 대신 공통으로 공유하는 부모로 _끌어올려서_ 필요한 자식에게 전달한다.

### 요약

- 두 컴포넌트를 조정하려면,
  - 해당 컴포넌트의 state를 공통 부모로 이동(**state 끌어올리기**)
  - 그런 다음 공통 부모로부터 props를 통해 정보를 전달
  - 마지막으로 이벤트 핸들러를 전달하여 자식이 부모의 state를 변경할 수 있도록 한다.
- 컴포넌트를 **(props에 의해) 제어**할 지 **(state에 의해) 비제어**할지 고려해보라

---

## State를 보존 / 재설정하기

### UI 트리

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/693bc352-6af5-47d2-866f-3108bb9e50f3/b9ab3a3e-2fbc-4220-ad29-056a2155dc22/Untitled.png)

- React는 JSX로부터 UI 트리를 만든다.
- 이후 React DOM은 해당 UI 트리와 일치하도록 브라우저 DOM 엘리먼트를 업데이트한다

### state는 트리의 한 위치에 묶인다

- 컴포넌트에 state를 부여할 때, state가 컴포넌트 내부에 “존재”한다고 생각할 수 있다.
  - **But, state는 실제로 React 내부에서 유지된다**.
  - React는 UI 트리에서 해당 컴포넌트가 어디에 위치하는지에 따라 보유하고 있는 각 state를 올바른 컴포넌트와 연결해준다.
- React는 컴포넌트가 UI 트리의 해당 위치에서 렌더링되는 동안 컴포넌트의 state를 유지한다.

  - 컴포넌트가 제거되거나 같은 위치에 다른 컴포넌트가 렌더링되면 React는 해당 컴포넌트의 state를 삭제한다.
  - 예시

    ```tsx
    import { useState } from "react";

    export default function App() {
      const [showB, setShowB] = useState(true);
      return (
        <div>
          <Counter />
          {showB && <Counter />}
          <label>
            <input
              type="checkbox"
              checked={showB}
              onChange={(e) => {
                setShowB(e.target.checked);
              }}
            />
            Render the second counter
          </label>
        </div>
      );
    }

    function Counter() {
      const [score, setScore] = useState(0);
      const [hover, setHover] = useState(false);

      let className = "counter";
      if (hover) {
        className += " hover";
      }

      return (
        <div
          className={className}
          onPointerEnter={() => setHover(true)}
          onPointerLeave={() => setHover(false)}
        >
          <h1>{score}</h1>
          <button onClick={() => setScore(score + 1)}>Add one</button>
        </div>
      );
    }
    // 두 컴포넌트 1씩 증가후 두번째 counter를 삭제했다가 다시 렌더시키면 0으로 초기화된
    // 두번째 counter를 볼 수 있다.
    ```

- **함정**
  **React에서 중요한 것은,**

  - **JSX 마크업이 아니라 UI 트리에서의 위치라는 것을 기억하라!**
  - 예시

    ```tsx
    import { useState } from "react";

    export default function App() {
      const [isFancy, setIsFancy] = useState(false);
      if (isFancy) {
        return (
          <div>
            <Counter isFancy={true} />
            <label>
              <input
                type="checkbox"
                checked={isFancy}
                onChange={(e) => {
                  setIsFancy(e.target.checked);
                }}
              />
              Use fancy styling
            </label>
          </div>
        );
      }
      return (
        <div>
          <Counter isFancy={false} />
          <label>
            <input
              type="checkbox"
              checked={isFancy}
              onChange={(e) => {
                setIsFancy(e.target.checked);
              }}
            />
            Use fancy styling
          </label>
        </div>
      );
    }

    function Counter({ isFancy }) {
      const [score, setScore] = useState(0);
      const [hover, setHover] = useState(false);

      let className = "counter";
      if (hover) {
        className += " hover";
      }
      if (isFancy) {
        className += " fancy";
      }

      return (
        <div
          className={className}
          onPointerEnter={() => setHover(true)}
          onPointerLeave={() => setHover(false)}
        >
          <h1>{score}</h1>
          <button onClick={() => setScore(score + 1)}>Add one</button>
        </div>
      );
    }
    ```

    - checkbox를 선택하면 state가 재설정될 것으로 예상할 수 있지만 그렇지 않다!
      - 이 **두 `<Counter />` 태그가 모두 같은 위치에 렌더링되기 때문!**
    - React는 함수에서 조건을 어디에 배치했는지 알지 못합니다. 단지 여러분이 반환하는 트리만 볼 수 있다.
    - 두 경우 모두 `App` 컴포넌트는 `<Counter />`를 첫 번째 자식으로 가진 `<div>`를 반환합니다.
      - React에서 두 카운터는 **루트의 첫 번째 자식의 첫 번째 자식이라는 동일한 주소**를 갖는다.
    - React는 로직을 어떻게 구성하든 상관없이 **이전 렌더링과 다음 렌더링 사이에서 이 방법으로 이들을 일치**시킬 수 있다.

### 동일한 위치의 다른 컴포넌트는 state를 초기화 한다

- 같은 위치 다른 컴포넌트를 렌더링하면?

  - **전체 하위 트리의 state가 재설정되면서 state가 초기화**
  - 예시

    ```tsx
    import { useState } from "react";

    export default function App() {
      const [isPaused, setIsPaused] = useState(false);
      return (
        <div>
          {isPaused ? <p>See you later!</p> : <Counter />}
          <label>
            <input
              type="checkbox"
              checked={isPaused}
              onChange={(e) => {
                setIsPaused(e.target.checked);
              }}
            />
            Take a break
          </label>
        </div>
      );
    }

    function Counter() {
      const [score, setScore] = useState(0);
      const [hover, setHover] = useState(false);

      let className = "counter";
      if (hover) {
        className += " hover";
      }

      return (
        <div
          className={className}
          onPointerEnter={() => setHover(true)}
          onPointerLeave={() => setHover(false)}
        >
          <h1>{score}</h1>
          <button onClick={() => setScore(score + 1)}>Add one</button>
        </div>
      );
    }
    ```

- 리렌더링 사이에 s**tate를 유지하려면 트리의 구조가 일치**해야 한다

- **함정**

  - **컴포넌트 함수 정의를 중첩해서는 안 된다**

  ```tsx
  import { useState } from "react";

  export default function MyComponent() {
    const [counter, setCounter] = useState(0);

    function MyTextField() {
      const [text, setText] = useState("");

      return <input value={text} onChange={(e) => setText(e.target.value)} />;
    }

    return (
      <>
        <MyTextField />
        <button
          onClick={() => {
            setCounter(counter + 1);
          }}
        >
          Clicked {counter} times
        </button>
      </>
    );
  }
  ```

  - 버튼을 클릭할 때마다 입력 state가 사라진다!
    - `MyComponent`를 렌더링할 때마다 다른 `MyTextField` 함수가 생성되기 때문
    - **같은 위치 다른 컴포넌트를 렌더링**하기 때문에 React는 아래의 모든 state를 초기화
  - **항상 컴포넌트 함수를 최상위 수준에서 선언하고 정의를 중첩하지 마라**

- **제거된 컴포넌트에 대한 state 보존**
  실제 채팅 앱에서는 사용자가 이전 수신자를 다시 선택할 때 입력 state를 복구하고 싶을 것입니다. 더 이상 표시되지 않는 컴포넌트의 state를 **살아있게 유지**하는 몇 가지 방법이 있다:
  - 현재 채팅만 렌더링하는 것이 아니라 모든 채팅을 렌더링하되 다른 모든 채팅은 CSS로 숨기기면 채팅은 트리에서 제거되지 않으므로 로컬 state가 유지된다.
    - 이 솔루션은 간단한 UI에 적합하지만 숨겨진 트리가 크고 많은 DOM 노드를 포함하는 경우 속도가 매우 느려질 수 있다.
  - 부모 컴포넌트에서 각 수신자에 대한 보류 중인 메시지를 [state를 끌어올려서](https://react-ko.dev/learn/sharing-state-between-components) 보관한다
    - 이렇게 하면 자식 컴포넌트가 제거되더라도 중요한 정보를 보관하는 것은 부모 컴포넌트이므로 문제가 되지 않는다. (가장 일반적인 해결책)
  - React state 외에 다른 소스를 사용한다.
    - 예를 들어, 사용자가 실수로 페이지를 닫아도 메시지 초안이 유지되기를 원할 수 있다. 이를 구현하기 위해 `Chat` 컴포넌트가 `[localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)`에서 읽어서 state를 초기화하고 초안도 저장하도록 할 수 있다.
      어떤 전략을 선택하든 앨리스와의 채팅은 밥과의 채팅과 개념적으로 구별되므로 현재 수신자를 기준으로 `<Chat>` 트리에 `key`를 부여하는 것이 합리적이다.

### 요약

- React는 **동일한 컴포넌트**가 **동일한 위치**에서 렌더링한다?
  - **state를 유지**한다
- **state는 JSX 태그에 보관되지 않는다**
  - **JSX를 넣은 트리 위치와 연관되어 있다**
- 하위 트리에 다른 key를 지정하여 강제로 state를 재설정할 수 있다
- **컴포넌트 정의를 중첩하지 마라**
  - 실수로 state가 초기화될 수 있다
