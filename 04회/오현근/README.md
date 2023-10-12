# [4회]상호작용 추가하기: 렌더링 커밋하기 ~ 배열 state 업데이트

### 링크 : https://tidal-swoop-2d1.notion.site/ff0f9adc24eb484c8bd4ce534f7c9d58?pvs=4

# 렌더링하고 커밋하기

컴포넌트를 **화면에 표시하기 이전에 React에서 렌더링**을 해야 합니다. 해당 과정의 단계를 이해하면 코드가 어떻게 실행되는지 이해할 수 있고 React 렌더링 동작에 관해 설명하는데 도움이 됩니다.

주방에서 요리사가 컴포넌트를 재료로 맛있는 요리를 한다고 상상해보세요. 리액트는 고객들의 요청을 받고 주문을 가져오는 웨이터 입니다.

1. 렌더링 **촉발** (손님의 주문을 주방으로 전달)
2. 컴포넌트 **렌더링**(주방에서 주문 받기)
3. DOM에 **커밋**(테이블에 주문한 요리 내놓기)

렌더링 촉발(state변경, props변경) → 변경 사실을 알린다( 렌더링 촉발 )→ 변경한대로 주문을 받는다 (컴포넌트 렌더링 ) → 변경된 내용을 전달한다 ( DOM에 커밋)

🤔요리를 하는건.. 누구? 리액트 아닌가 웨이터도 리액트고

-   🤔**렌더링 프로세스는 어떻게 될까?**
    1. FunctionComponent를 호출하고, 렌더링된 결과를 저장
    2. 렌더링 결과물은 jsx로 구성되어 js가 컴파일 되고 런타임 시점에 React.createElement()로 호출되어 변환
    3. 전체 컴포넌트에서 이러한 렌더링 결과물 수집
    4. 리액트는 오브젝트 트리와 비교 실제 DOM을 의도한 출력처럼 보이게 변경 수집
       **렌더와 커밋 단계**
       `Render phase`: 컴포넌트를 렌더링하고 변경사항을 계산하는 모든 작업
       `Commit phase` : 돔에 변경사항을 적용하는 과정
       **DOM을 커멋페이즈에서 업데이트**한 후 , 요청된 **DOM 노드 및컴포넌트 인스턴스를 가리키도록 모든 참조를 업데이트** 한다.
       그 다음, `useLayoutEffect` 훅 호출
       **렌더링은 DOM을 업데이트 하는 것과 같은 것 x**
       **컴포넌트는 어떠한 가시적인 변경이 없어도 컴포넌트가 렌더링 될 수 있다는 것 o**

## Step 1: Trigger a render

컴포넌트의 렌더링을 일으키는건 2가지 이유가 있습니다.

1. 컴포넌트의 첫 렌더링인 경우
2. 컴포넌트의 state(또는 상위 요소 중 하나) 가 업데이트 된 경우

-   😒props가 변경된 경우는 아닌가..? props가 state에 걸려있어야되나?

## 첫 렌더링

앱을 시작하기 위해서는 첫 렌더링을 트리거 시켜야 한다.

프레임워크와 샌드박스가 코드를 숨기지만, 대상 `DOM` 노드로 `createRoot`를 호출한 다음 컴포넌트로 `render` 메소드를 호출한다.

```tsx
import Image from "./Image.js";
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"));
root.render(<Image />);

imagae.js;
export default function Image() {
    return (
        <img
            src="https://i.imgur.com/ZF6s192.jpg"
            alt="'Floralis Genérica' by Eduardo Catalano: a gigantic metallic flower sculpture with reflective petals"
        />
    );
}
```

root.render 함수를 통해 컴포넌트를 실행시키는 걸 볼 수 있다.

## state가 업데이트 되면, 리렌더링합니다.

컴포넌트가 처음 렌더링되면, set함수로 state를 업데이트하여 추가 렌더링을 일으킬 수 있습니다.

컴포넌트의 state 업데이트 → 자동으로 렌더링이 대기열 추가

😒state 변경을 어떻게 감지할까?

-   🤔**리액트는 어떻게 렌더링을 다루는가?**
    최초 렌더링이 끝난이후에, 리액트가 리렌더링을 queueing하는 방법
    `useState`의 `set`
    `useReducer`의 `dispatch`
    일반적인 렌더링 동작은
    **부모 컴포넌트 렌더링 → 모든 자식 컴포넌트를 순차적으로 리렌더링**
    A > B > C > D 의 컴포넌트 트리가 있다고 가정하고, B에 카운터를 올리는 버튼이 있고 해당 버튼을 클릭했을때

    1. B의 setState가 호출되어 B의 리렌더링이 렌더링 큐로 들어간다.
    2. 리액트는 트리 최상단에서 부터 렌더링 패스를 시작한다.
    3. A는 업데이트가 필요하다고 체크 되어있지 않을 것이므로 지나감
    4. B는 업데이트가 필요한 컴포넌트가 체크되어 있으므로, B를 렌더링 B는 C를 리턴
    5. C는 원래 업데이트가 필요한 것으로 간주되이 있지 않는다. 그러나, 부머인 B가 리렌더 되었으므로, 리액트는 하위 컴포넌트 C를 렌더링하고 D를 리턴한다.
    6. D도 마찬가지로 렌더링에 체크되어 있지 않았지만 C가 렌더링 된 관계로 그 자식인 D 도 렌더링한다.
       여기서 A는 업데이트가 필요하다고 체크되어 있지 않을 것이므로 지나감 ?? 이부분이 너무 어색하다.

    -   🤔A의 업데이트가 왜 일어나는걸까?
        eact의 리렌더링은 일반적으로 루트 컴포넌트부터 시작되며, 컴포넌트 트리의 상단(root)에서 하위 컴포넌트로 내려가면서 진행됩니다. 이것은 React가 컴포넌트 트리를 순회하며 변경된 부분만 업데이트하는 "가상 DOM" 기반의 라이브러리로서 동작하는 방식입니다. 하지만 이것이 반드시 모든 컴포넌트를 렌더링할 때마다 루트 컴포넌트부터 시작한다는 것을 의미하지는 않습니다.
        리액트에서 컴포넌트의 리렌더링은 다음과 같이 동작합니다. 1. 초기 렌더링(initial rendering): 애플리케이션이 로드될 때 루트 컴포넌트부터 시작하여 하위 컴포넌트로 이동하며 렌더링됩니다. 2. 상태(state) 또는 프롭스(props) 변경: 컴포넌트의 상태나 프롭스가 변경되면 해당 컴포넌트와 그 하위 컴포넌트만 다시 렌더링됩니다. 이것이 React의 성능 최적화 메커니즘 중 하나입니다. 3. 이벤트 처리: 이벤트(예: 버튼 클릭)가 발생하면 해당 이벤트 핸들러가 실행되고, 상태가 변경될 수 있습니다. 이 경우 변경된 상태를 가진 컴포넌트와 그 하위 컴포넌트만 다시 렌더링됩니다.
        따라서 A > B > C > D 구조에서 B 컴포넌트의 상태(state)가 변경되면 B 컴포넌트와 그 하위인 C와 D 컴포넌트만 다시 렌더링되어야 합니다. A 컴포넌트는 B 컴포넌트의 상태 변경과 직접적인 연관이 없다면 리렌더링되지 않아야 합니다.
        만약 A 컴포넌트가 불필요하게 리렌더링된다면, 이는 일반적인 동작과는 다른 문제가 있을 수 있으며, 코드를 검토하여 원인을 찾아야 합니다.
        이 렌더링은 일반적인 렌더링이다
        **컴포넌트를 렌더링 하는 작업은, 기본적으로, 하위에 있는 모든 컴포넌트 또한 렌더링 하게 된다.**
        또한
        **일반적인 렌더링의 경우, 리액트는 `props`가 변경되어 있는지 신경쓰지 않는다. 부모 컴포넌트가 렌더링 되어 있기 때문에, 자식 컴포넌트도 무조건 리렌더링 된다.**
    -   코드로 보기

        ```jsx
        import React, { Children, useEffect, useState, cloneElement } from "react";

        function D() {
            console.log("D rerender");
            return <div style={{ width: "200px", height: "200px", background: "yellow" }}></div>;
        }

        function C({ count, children }) {
            console.log("C rerender");
            return (
                <div style={{ width: "300px", height: "300px", background: "yellow" }}>
                    <D />
                </div>
            );
        }

        function B({ children }) {
            const [count, setCount] = useState(0);
            console.log("B rerender");
            return (
                <div style={{ width: "400px", height: "400px", background: "orange" }}>
                    <button onClick={() => setCount(count + 1)}>count</button>
                    <C />
                </div>
            );
        }

        function A() {
            console.log("A rerender");

            return (
                <div style={{ width: "500px", height: "500px", background: "red" }}>
                    <B></B>
                </div>
            );
        }

        export default function List() {
            return <A />;
        }

        이렇게 넣으면 순서가
        B
        C
        D
        순서로 콘솔에 찍힌다.
        ```

    -   useEffect를 썼을때

        ```jsx
        import React, { Children, useEffect, useState, cloneElement } from "react";

        function D() {
            useEffect(() => {
                console.log("D rerender");
            }); // 빈 배열을 전달하여 한 번만 실행되도록 함
            return <div style={{ width: "200px", height: "200px", background: "yellow" }}></div>;
        }

        function C() {
            useEffect(() => {
                console.log("C rerender");
            });
            return (
                <div style={{ width: "300px", height: "300px", background: "yellow" }}>
                    <D />
                </div>
            );
        }

        function B() {
            const [count, setCount] = useState(0);
            useEffect(() => {
                console.log("B rerender");
            });
            return (
                <div style={{ width: "400px", height: "400px", background: "orange" }}>
                    <button onClick={() => setCount(count + 1)}>count</button>
                    <C />
                </div>
            );
        }

        function A() {
            useEffect(() => {
                console.log("A rerender");
            });

            return (
                <div style={{ width: "500px", height: "500px", background: "red" }}>
                    <B></B>
                </div>
            );
        }

        export default function List() {
            return <A />;
        }

        이 때 B에서의 클릭이 어떤 콘솔을 찍게 만들까?
        D
        C
        B
        순서가 된다.
        ```

    -   children을 쓸때
        ```jsx
        import React, { Children, useEffect, useState, cloneElement } from "react";

            function D() {
                console.log("D rerender");
                return <div style={{ width: "200px", height: "200px", background: "yellow" }}></div>;
            }

            function C({ count, children }) {
                console.log("C rerender");
                return (
                    <div style={{ width: "300px", height: "300px", background: "yellow" }}>
                        {children}
                    </div>
                );
            }

            function B({ children }) {
                const [count, setCount] = useState(0);
                console.log("B rerender");
                return (
                    <div style={{ width: "400px", height: "400px", background: "orange" }}>
                        <button onClick={() => setCount(count + 1)}>count</button>
                        {children}
                    </div>
                );
            }

            function A({ children }) {
                console.log("A rerender");

                return (
                    <div style={{ width: "500px", height: "500px", background: "red" }}>{children}</div>
                );
            }

            export default function List() {
                return (
                    <A>
                        <B>
                            <C>
                                <D></D>
                            </C>
                        </B>
                    </A>
                );
            }
            ```

        children을 쓰면 B만 리렌더링이 진행된다!!

## Step 2: React renders your components

렌더링을 트리거하면, 리액트는 컴포넌트를 호출하여 화면에 표시할 내용을 파악합니다. **“렌더링”은 리액트에서 컴포넌트를 호출하는 것 입니다.**

**첫 렌더링**에서 리액트는 루트 컴포넌트를 호출합니다.

**이후 렌더링**에서 React는 state 업데이트에 의해 렌더링이 발동된 함수 컴포넌트를 호출합니다.

모든 컴포넌트는 루트 컴포넌트를 시작으로 엮여있다. 컴포넌트들이 트리형태를 이루고 있음을 알 수있으니 루트 컴포넌트에서 실행하면, 전부 실행 시킬 수 있다.

근데 이후 렌더링을 state 업데이트에 의해 렌더링이 발동된 함수 컴포넌트를 호출?

이 과정은 재귀적입니다. → 업데이트가 된 컴포넌트가 다른 컴포넌트를 반환하면 리액트는 다음으로 해당 컴포넌트를 렌더링하고 해당 컴포넌트도 또 다음 반환… 중첩된 컴포넌트가 더 없고 리액트가 화면에 표시되어야 하는 내용을 정확히 알 때까지 계속..

이 과정이 재귀적이라고할 수 있을까요? 함수의호출 대상이 본인을 향하는게 아니라 아래로 내려가는 형태입니다. 내 함수를호출하는게 아니니 재귀적이란 말은 조금 이상합니다.

다음 예제에서 React는 Gallery()와 Image()르 여러 번 호출합니다.

```tsx
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

import Gallery from "./Gallery.js";
import { createRoot } from "react-dom/client";

const root = createRoot(document.getElementById("root"));
root.render(<Gallery />);
```

**첫 렌더링 동안,** <section> <h1> 3개의 <img> 태그에 대한 DOM 노드를 생성합니다.

**리렌더링하는 동안,** React는 이전 렌더링 이후 변경된 속성을 계산합니다. 다음 단계인 커밋 단계까지는 해당 정보로 아무런 작업도 수행하지 않습니다.

**함정: 렌더링은 항상 순수한계산이어야합니다.**

동일 입력 → 동일 출력 ,

이전의 state를 변경해서는 안됩니다. → 렌더링 전에 존재한 객체나 변수를 변경해서는 안됩니다.

그렇지 않다면 코드베이스가 복잡 → 혼란스러운 버그와 예측 할 수 없어집니다.

**Deep : 성능 최적화**

업데이트된 컴포넌트 내에 중첩된 모든 컴포넌트를 렌더링하는 기본 동작은 업데이트된 컴포넌트가 트리에서 매우 높은 곳에 있는 경우 성능 최적화되지 않습니다. 성능 문제가 발생하는 경우 [성능](https://reactjs.org/docs/optimizing-performance.html) 섹션에 설명된 몇 가지 옵트인 방식으로 문제를 해결 할 수 있습니다. **성급하게 최적화하지 마세요!**

## Step 3: 리액트가 DOM에 변경사항을 커밋

컴포넌트를 렌더링한 후 React는 DOM을 수정합니다.

초기 렌더링 : 리액트는 appendChild() DOM API를 사용해 생성한 모든 DOM 노드를 화면에 표시합니다.

리렌더링 : 리액트는 필요한 최소한의 작업을 적용 DOM이 최신 렌더링 출력과 일치하도록 합니다.

😒어떻게???

-   리액트가 최신 렌더링 출력을 일치시키는 방법
    **가상돔을** 활용해서 가능하게한다
    React의 리렌더링 과정은 다음과 같이 동작합니다:
    1. 컴포넌트의 상태나 프롭스가 변경되면, React는 해당 컴포넌트의 렌더링 메서드를 호출하여 **가상 DOM 트리를 생성**합니다.
    2. **가상 DOM 트리**는 이전에 렌더링된 결과와 비교되어 **변경된 부분을 식별**합니다. 이 비교과정은 효율적으로 이루어지며 **변경된 부분만을 실제 DOM에 적용**할 수 있도록 합니다.
    3. **변경된 부분만을 실제 DOM에 적용하여 화면을 업데이트**합니다.
       즉, React는 초기 렌더링에서는 가상 DOM을 생성하고, 이후의 리렌더링에서는 변경된 부분만을 업데이트하여 불필요한 작업을 최소화합니다.

리액트는 렌더링 간에 차이가 있는 경우에만 DOM 노드를 변경!

매초 부모로부터 전달된 다른 props로 다시 렌더링하는 컴포넌트

input에 텍스트를 입력해서 value를 업데이트 하지만, 컴포넌트가 리렌더링 될 때 텍스트가 사라지지 않습니다!!

마지막 단계에서 리액트가 h1의 내용만 새로운 time으로 업데이트 하기 떄문입니다.

## 에필로그: 브라우저 페인트

렌더링이 완료 → 리액트가 DOM을 업데이트 한 후 브라우저는 화면을 다시 그립니다. 이 단계를 “브라우저 렌더링”이라고 하지만 이 문서의 나머지 부분에서 혼동을 피하고자 “페인티”이라고 부를 겁니다.

state 변수는 읽고 쓸 수 있는 일반 JavaScript 변수처럼 보일 수 있습니다. 하지만 state는 **스냅샷**처럼 동작합니다. state 변수를 설정해도 **이미 가지고 있는 state 변수는 변경되지 않고, 대신 리렌더링이 실행됩니다**.

-   스냅샷
    사진을 찍듯이 특정 시간(시점)에 데이터 저장 장치(스토리지)의 파일 시스템을 포착해 별도의 파일이나 이미지로 저장, 보관하는 기술

**클릭**과 같은 **사용자 이벤트**에 **반응**하여 사용자 **인터페이스**가 **직접 변경된다고 생각**할 수 있습니다. React에서는 이 멘탈 모델과는 조금 다르게 작동합니다. 이전 페이지에서 **state**를 설정하면 **React에 리렌더링을 요청**하는 것을 보았습니다. 즉, **인터페이스가 이벤트에 반응하려면 state를 업데이트해야 합니다**.

사용자 → 이벤트 발생 → 인터페이스 변경 x

state 업데이트 → 인터페이스 변경

```tsx
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

Send 버튼이 submit 발생
isSent 가 true로 변경
sendMessage 함수 실행 (비어있음)
textarea 내부는 입력마다 변경중

isSent가 바뀌면서 재렌더링 isSent가 true가 되면서 return 문 실행
```

## 렌더링은 그 시점의 스냅샷을 찍습니다.

“렌더링”이란 React가 컴포넌트, 즉,**함수를 호출**한다는 뜻입니다. 해당 함수에서 반환하는 **JSX는 시간상 UI의 스냅샷**과 같습니다. **prop, 이벤트 핸들러, 로컬 변수**는 모두 **렌더링 당시의 state를 사용해** 계산됩니다.

렌더링 → 리액트가 컴포넌트를 호출하는 것

JSX는 시간상 UI의 스냅샷과 같다 = UI가 현재 보여주는 상태이다.

prop, 이벤트 핸들러, 로컬 변수 = 렌더링 (함수 호출 시)의 state를 사용한다.

여기서 의문인점.. 렌더링 당시의 state를 사용한다?

state변경은 render를 발생시킨다. 하지만, 한 컴포넌트 내에서 혹은 동작이 → 다른 state의 변경을 발생 시킬 수 있다.

state의존적이라면, 일반 변수로 두어도 상관없지만, 걸쳐있을 수 있고, 하나의 변수가 아닌 다른 변수를 변경시킬 가능성도 있다.

😒그때 디바운스나, 쓰로틀링을 발생시킬 것 같은데 어떻게???

-   아래에서 나오듯이
    이벤트들을 queue에 쌓아두고 렌더링시에 사용합니다.

사진이나 동영상 프레임과 달리 **반환하는 UI ‘스냅샷’은 대화형**입니다. 여기에는 input에 대한 응답으로 어떤 일이 일어날지 지정하는 **이벤트 핸들러와 같은 로직이 포함**됩니다. 그러면 React는 이 **스냅샷과 일치하도록 화면을 업데이트하고 이벤트 핸들러를 연결**합니다. 결과적으로 버튼을 누르면 JSX에서 클릭 핸들러가 발동됩니다.

return을 하는 UI 스냅샷은 대화형이다.

input에 대한 응답으로 어떤 일이 일어날지 지정하는 이벤트 핸들러

```tsx
inputHandler = (e) => setValue(e.target.value);
```

같은 로직이 포함

리액트는 이 스냅샷과 일치하도록 화면 업데이트, 이벤트 핸들러를 연결

결과적으로 버튼을 누르면 → JSX에서 클릭 핸드러가 발동?????

스냅샷과 일치하도록 화면 업데이트

예를들어 inputHandler= e ⇒ setValue(e.target.value)를 만들어 놨을때

<input onChange={inputHandler} value ={value} /> 라고 했다면,

input창에 값을 쓰면 onChange로 inputHandler가 동작할거고 setValue 로 value state값이 변경 되겠지? 그러면 리렌더링을 유발할 거고, 리렌더링이 된다는건 컴포넌트 재실행을 유발할거고, 컴포넌트의 변경된 부분을 수정할거고 그때 이벤트 핸들러는 이미 컴포넌트 내에 있는데 왜 연결을 한다는 거야? 그냥 있는거 쓰면 되는데 왜 가상 dom과 연결해?

컴포넌트 생성 → 내부에 있는변수나 함수들이 새로 호출되는게 아니고 최적화에 있어서 남아있는건 남아있는다. 메모리를 재사용한다.

리액트가 컴포넌트를 다시 렌더링할때

1. React가 함수를 다시 호출합니다.
2. 함수가 새로운 JSX 스냅샷을 반환합니다.
3. 그러면 React가 반환한 스냅샷과 일치하도록 화면을 업데이트합니다.

컴포넌트의 메모리로서 state는 함수가 반환된 후 사라지는 일반 변수와 다릅니다. state는 실제로 함수 외부에 마치 선반에 있는 것처럼 React 자체에 “**존재**”합니다. React가 컴포넌트를 호출하면 특정 렌더링에 대한 state의 스냅샷을 제공합니다. 컴포넌트는 **해당 렌더링의 state 값을 사용해** 계산된 새로운 props 세트와 이벤트 핸들러가 포함된 UI의 스냅샷을 JSX에 반환합니다!

```tsx
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);
  const [number2, setNumber2] = useState(0)
  return (
    <>
      <h1>{`${ number} ${number2}`}</h1>
      <button onClick={() => {
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber(number + 1);
        setNumber2(number2+1)
      }}>+3</button>
    </>
  )
}
setNumber에 number + 1 을 3번하는데 1번만 동작!!?
```

**state를 설정하면, 다음 렌더링에 대해서만 변경됩니다.**

물론 setNumber2도 잘 동작합니다.

왜 이런일이 이러날까요?

정말 컴포넌트를 스냅샷이라는 관점으로 바라본다면,

버튼 태그 안에 생긴 모습은

```tsx
<button
    onClick={() => {
        setNumber(0 + 1);
        setNumber(0 + 1);
        setNumber(0 + 1);
        setNumber2(0 + 1);
    }}
>
    +3
</button>
```

의 형태를 띌 것입니다.

이 버튼의 클릭 핸들러가 React에게 지시하는 작업은

1. `setNumber(number + 1)`: `number`는 `0`이므로 `setNumber(0 + 1)`입니다.
    - React는 다음 렌더링에서 `number`를 `1`로 변경할 준비를 합니다.
2. `setNumber(number + 1)`: `number`는 `0`이므로 `setNumber(0 + 1)`입니다.
    - React는 다음 렌더링에서 `number`를 `1`로 변경할 준비를 합니다.
3. `setNumber(number + 1)`: `number`는 `0`이므로 `setNumber(0 + 1)`입니다.
    - React는 다음 렌더링에서 `number`를 `1`로 변경할 준비를 합니다.

즉 3번이나 number를 1로 바꾸라고 얘기하고 있는 모습이죠

한번 더 실행했을때 모습은

```tsx
<button
    onClick={() => {
        setNumber(1 + 1);
        setNumber(1 + 1);
        setNumber(1 + 1);
        setNumber2(1 + 1);
    }}
>
    +3
</button>
```

## 시간 경과에 따른 state

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
                    alert(number);
                }}
            >
                +5
            </button>
        </>
    );
}
```

이 코드를 예측해본다면 어떻게 될까요?

알람으로 0 , 5 , 10 순으로 뜨겠죠?

그렇다면, 비동기로 호출시켜보면 어떻게 될까요?

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
```

이렇게하면 어떻게 될까요?? 3초뒤에 0이 뜨겠죠?

state가 setTimeout의 내부에 number가 변경된 값을 갖고있는게 아니기 때문에 0이라는 값이 뜰 것입니다.

그렇다면,

```tsx
import { useState } from "react";

export default function Counter() {
    // const [number, setNumber] = useState(0);
    let number = 0;

    return (
        <>
            <h1>{number}</h1>
            <button
                onClick={() => {
                    // number = number+5
                    // setNumber(number + 5);
                    number += 5;
                    setTimeout(() => {
                        alert(number);
                    }, 1000);
                }}
            >
                +5
            </button>
        </>
    );
}
```

를 1초안에 5번 누른다면 어떻게 결과가 나올까요?

리액트에 저장된 state는 알림이 실행될 때 React에 저장된 state는 알림이 실행될 때 변경되었을 수 있지만, 사용자가 상호작용한 시점에 **state 스냅샷을 사용하는 건 이미 예약되어 있던 것**입니다!

**state변수의 값은 이벤트 핸들러의 코드가 비동기적이더라고 렌더링 내에서 절대 변경되지 않습니다.**

호출할때 “고정”된 값을 쓰는 것 입니다.

다음은 이벤트 핸들러가 타이밍 실수를 줄이는 방법을 보여주는 예입니다. 아래는 5초 지연된 메시지를 보내는 양식입니다. 이 시나리오를 상상해 보세요:

1. “보내기” 버튼을 눌러 앨리스에게 “안녕하세요”를 보냅니다.
2. 5초 지연이 끝나기 전에 “받는 사람” 필드의 값을 “Bob”으로 변경합니다.

`alert`에 어떤 내용이 표시되기를 기대하나요? “앨리스에게 인사했습니다”라고 표시될까요, 아니면 “당신은 밥에게 인사했습니다”라고 표시될까요? 알고 있는 내용을 바탕으로 추측해보고, 다음을 코드를 실행해 보세요:

무조건 엘리스에게 인사했습니다. 가 뜰 것입니다.

-   예시코드

    ```tsx
    import { useState } from 'react';

    export default function Form() {
      const [to, setTo] = useState('Alice');
      const [message, setMessage] = useState('Hello');

      function handleSubmit(e) {
        e.preventDefault();
        setTimeout(() => {
          alert(`You said ${message} to ${to}`);
        }, 5000);
      }

      return (
        <form onSubmit={handleSubmit}>
          <label>
            To:{' '}
            <select
              value={to}
              onChange={e => setTo(e.target.value)}>
              <option value="Alice">Alice</option>
              <option value="Bob">Bob</option>
            </select>
          </label>
          <textarea
            placeholder="Message"
            value={message}
            onChange={e => setMessage(e.target.value)}
          />
          <button type="submit">Send</button>
        </form>
      );
    }
    바꿔도 달라지지 않는다. 바꾸려면??

    import { useState } from 'react';

    export default function Form() {
      // const [to, setTo] = useState('Alice');
      let to = 'Alice'
      const [message, setMessage] = useState('Hello');

      function handleSubmit(e) {
        e.preventDefault();
        setTimeout(() => {
          alert(`You said ${message} to ${to}`);
        }, 5000);
      }

      return (
        <form onSubmit={handleSubmit}>
          <label>
            To:{' '}
            <select
              value={to}
              onChange={e => to = e.target.value}>
              <option value="Alice">Alice</option>
              <option value="Bob">Bob</option>
            </select>
          </label>
          <textarea
            placeholder="Message"
            value={message}
            onChange={e => setMessage(e.target.value)}
          />
          <button type="submit">Send</button>
        </form>
      );
    }
    to를 let으로 바꾸면 된다. 그러면 이름이 바뀐다.

    근데 컴포넌트가 렌더링 되면서, let 변수에 있는 값이 초기화된다.
    ```

**React는 하나의 렌더링 이벤트 핸들러 내에서 state 값을 “고정”으로 유지합니다.** 코드가 실행되는 동안 state가 변경되었는지 걱정할 필요가 없습니다.

하지만 다시 렌더링하기 전에 최신 state를 읽고 싶다면 어떻게 해야 할까요? 다음 페이지에서 설명하는 [state 업데이터 함수](https://react-ko.dev/learn/queueing-a-series-of-state-updates)를 사용하면 됩니다!

## 도전 과제

-   1
    ```tsx
    import { useState } from 'react';

        export default function TrafficLight() {
          const [walk, setWalk] = useState(true);

          function handleClick() {
            setWalk(!walk);
            if(walk) {
              alert('지금은 정지입니다.')
            }
          }

          return (
            <>
              <button onClick={handleClick}>
                Change to {walk ? 'Stop' : 'Walk'}
              </button>
              <h1 style={{
                color: walk ? 'darkgreen' : 'darkred'
              }}>
                {walk ? 'Walk' : 'Stop'}
              </h1>
            </>
          );
        }

            if(walk) {
              alert('지금은 정지입니다.')
            }

        를 추가하면 됩니다.

        근데, 이 부분을 앞에넣나 뒤에 넣나 차이가 없습니다.
        ```

    **state 변수를 설정하면 다음 렌더링이 큐(대기열, queue)에 들어갑니다**. 그러나 경우에 따라 다음 렌더링을 **큐에 넣기 전에, 값에 대해 여러 작업을 수행하고 싶을 때**도 있습니다. 이를 위해서는 React가 state 업데이트를 어떻게 배치하면 좋을지 이해하는 것이 도움이 됩니다.

## state 업데이트 일괄처리

```tsx
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

예시에서 number가 1만 늘어납니다.

**리액트는 state 업데이트를 하기 전에 이벤트 핸들러의 모든 코드가 실행될 때까지 기다립니다.**

이 때문에 리렌더링은 setNumber() 호출이 완료된 이후에만 일어납니다.

웨이터로 예를 들면, 웨이터는 첫번째 요리를 말하자마자 주방으로 달려가지 않습니다. 대신, 주문이 끝날 때까지 기다렸다가 주문을 변경하고, 다른 테이블 주문까지 받고 갑니다.

**setState()를 한번에 여러개 해놔도 다 처리한 다음에 렌더링을 진행한다**

이렇게 하면 **너무 많은 리렌더링을 촉발하지 않고도 여러 컴포넌트에서 나온 다수의 state 변수를 업데이트**할 수 있습니다. **하지만 이는 이벤트 핸들러와 그 안에 있는 코드가 완료될 때까지 UI가 업데이트되지 않는다**는 의미이기도 합니다. **일괄처리(배칭, batching**)라고도 하는 이 동작은 React 앱을 훨씬 빠르게 실행할 수 있게 해줍니다. 또한 일부 변수만 업데이트된 “반쯤 완성된” 혼란스러운 렌더링을 처리하지 않아도 됩니다.

😏예를들어 이벤트 동작이 데이터 패칭을 유발하고, 해당 과정 처리 후 setValue를 한다면, 어떻게 동작하나? 예상되는 결과는 fetching된 값이 아마 들어가면서 렌더링된 값이 변할 것이라 생각됩니다.

setTimeout과 결과가 같겠죠?

-   😏배칭??
    ```jsx
    function batching() {
        // 배칭 블록 시작
        ReactDOM.unstable_batchedUpdates(() => {
            setStateA(newValueA);
            setStateB(newValueB);
            setStateC(newValueC);
            // 여러 개의 setState 호출을 한 번에 배칭하여 처리
        });
        // 배칭 블록 끝
    }
    ```
    아마 이런식의 코드가 생기지 않을까..

리액트는 클릭과 같은 여러 의도적인 이벤트에 대해 일괄 처리하지 않으며, 각 클릭은 개별적으로 처리됩니다.??

**리액트는 일반적으로 안전한 경우에만 일괄 처리를 수행하니 안심하세요.** 첫 번째 버튼 클릭으로 양식이 비활성화 된다면, 두 번째 클릭으로 양식이 다시 제출되지 않도록 보장합니다.

-   😏두번째 클릭으로 양식이 다시 제출되지 않도록 보장?

    ```tsx
    function FormExample() {
        const [formDisabled, setFormDisabled] = useState(false);

        // 양식 제출 핸들러
        const handleSubmit = (e) => {
            e.preventDefault(); // 양식 제출 이벤트의 기본 동작을 막음
            if (!formDisabled) {
                // 양식이 비활성화되지 않았을 때만 처리
                console.log("양식이 제출되었습니다.");
            }
        };

        return (
            <form onSubmit={handleSubmit}>
                <input type="text" placeholder="이름" />
                <button type="submit" disabled={formDisabled}>
                    제출
                </button>
                <button
                    type="button"
                    onClick={() => {
                        setFormDisabled(true); // 첫 번째 버튼 클릭으로 양식 비활성화
                        setTimeout(() => {
                            setFormDisabled(false); // 시간이 지나면 양식 다시 활성화
                        }, 3000);
                    }}
                >
                    양식 비활성화
                </button>
            </form>
        );
    }

    export default FormExample;
    ```

일괄처리를 수행하지 않는건 어떻게 하는걸까?

-   😏\***\*flushsync\*\***
    https://kyounghwan01.github.io/blog/React/React18/flushsync/
    setState하기 전에 강제적으로 dom을 업데이트하게 해준다

## 다음 렌더링 전에 동일한 state 변수를 여러번 업데이트하기

흔한 사례는 아니지만, 다음 렌더링 전에 동일한 state 변수를 여러 번 업데이트 하고 싶다면 `setNumber(number + 1)` 와 같은 *다음 state 값*을 전달하는 대신, `setNumber(n => n + 1)` 와 같이 큐의 이전 state를 기반으로 다음 state를 계산하는 *함수*를 전달할 수 있습니다. 이는 단순히 state 값을 대체하는 것이 아니라 React에게 “state 값으로 무언가를 하라”고 지시하는 방법입니다.

```tsx
import { useState } from "react";

export default function Counter() {
    const [number, setNumber] = useState(0);

    return (
        <>
            <h1>{number}</h1>
            <button
                onClick={() => {
                    setNumber((n) => n + 1);
                    setNumber((n) => n + 1);
                    setNumber((n) => n + 1);
                }}
            >
                +3
            </button>
        </>
    );
}
```

여기서 n ⇒ n+1 은 업데이터 함수 라고 합니다.

state 설정자 함수에 전달 할 때

1. React는 이벤트 핸들러의 다른 코드가 모두 실행된 후에 이 함수가 처리되도록 큐에 넣습니다.
2. 다음 렌더링 중에 React는 큐를 순회하여 최종 업데이트된 state를 제공합니다.

React가 이벤트 핸들러를 수행하는 동안 여러 코드를 통해 작동하는 방식은 다음과 같습니다

1. `setNumber(n => n + 1)`: `n => n + 1` 함수를 큐에 추가합니다.
2. `setNumber(n => n + 1)`: `n => n + 1` 함수를 큐에 추가합니다.
3. `setNumber(n => n + 1)`: `n => n + 1` 함수를 큐에 추가합니다.

다음 렌더링 중에 useState를 호출하면 React는 큐를 순회

queue update에 n ⇒ n+1 함수 3개가 들어가게되고,

setNumber에 값으로 n으로 들어오는 값이 0 1 2 로 커지면서 결국 3이 됩니다.

## state를 교체한 후 업데이트하면 어떻게 될까요

```tsx
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
}}>
```

값이 6씩 커지겠죠?

그렇다면 이 setNumber 함수의 위치를 바꾸면?

```tsx
<button onClick={() => {
	setNumber(n => n + 1);
  setNumber(number + 5);
}}>
```

전 사실 그래도 같을 줄 알았습니다.

update queue에 들어가니까 number+5가 먼저 실행되고, n ⇒ n+1의 결과로 6이 되고 이럴줄 알았는데 setNumber의 순서대로 동작합니다.

update queue에 들어가는건 동기적으로 들어가나 봅니다.

그럼 이제 궁금한게 생기죠!

😒비동기로 호출했을때는?

```tsx
import { useState } from "react";

export default function Counter() {
    const [number, setNumber] = useState(1);
    const asyncHandler = (n) => {
        new Promise((res, rej) => {
            console.log(n);
            setNumber(n + 1);
            res(n);
        }).then((res) => console.log(res));
    };
    return (
        <>
            <h1>{number}</h1>
            <button
                onClick={() => {
                    setNumber((n) => n + 1);
                    asyncHandler(number + 2);
                    setNumber(number + 5);
                }}
            >
                Increase the number
            </button>
        </>
    );
}
```

**Note setState(x)가 실제로는 setState(n⇒x)처럼 동작되지만 n 이 사용되지 않는다!**

## 업데이트 후 state를 바꾸면 어떻게 될까요?

```tsx
<button onClick={() => {
  setNumber(number + 5);
  setNumber(n => n + 1);
  setNumber(42);
}}>
```

어떻게 될까요? 42가 되겠죠?

동작을 살펴보면

1. `setNumber(number + 5)`: `number` 는 `0` 이므로 `setNumber(0 + 5)`입니다. React는 *“`5`로 바꾸기”*를 큐에 추가합니다.
2. `setNumber(n => n + 1)`: `n => n + 1` 는 업데이터 함수입니다. React는 *이 함수*를 큐에 추가합니다.
3. `setNumber(42)`: React는 *“`42`로 바꾸기”*를 큐에 추가합니다.

결국 42로 바꿉니다.

다음 렌더링동안

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/c25eb7fa-fce5-4067-a2c4-bec8874fa401/2487be30-1f54-4820-bd73-d92506b75ebb/Untitled.png)

를 순회합니다.

그런 다음 리액트는 42를 최종 결과로 저장하고 useState에서 반환합니다.

요약하자면, setNumber 함수에 전달할 내용은

-   **업데이터 함수** (예: **`n => n + 1`**)가 큐에 추가됩니다.
-   **다른 값** (예: 숫자 **`5`**)은 큐에 “`5`로 바꾸기”를 추가하며, 이미 큐에 대기중인 항목은 무시합니다.

이벤트 핸들러가 완료되면 React는 리렌더링을 실행합니다. 리렌더링하는 동안 React는 큐를 처리합니다. 업데이터 함수는 렌더링 중에 실행되므로, **업데이터 함수는 순수해야 하며** 결과만 *반환*해야 합니다. 업데이터 함수 내부에서 state를 변경하거나 다른 사이드 이팩트를 실행하려고 하지 마세요. Strict 모드에서 React는 각 업데이터 함수를 두 번 실행(두 번째 결과는 버림)하여 실수를 찾을 수 있도록 도와줍니다.

## 명명 규칙

업데이터 함수 인수의 이름은 state 변수의 첫 글자로 지정하는 것이 일반적입니다.

좀 더 자세한 코드를 선호하는 경우 `setEnabled(enabled => !enabled)`와 같이 전체 state 변수 이름을 반복하거나, `setEnabled(prevEnabled => !prevEnabled)`와 같은 접두사(prefix _“prev”_)를 사용하는 것이 일반적인 규칙입니다.

## 도전과제

-   요청 카운터를 고쳐보세요

    ```tsx
    import { useState } from 'react';

    export default function RequestTracker() {
      const [pending, setPending] = useState(0);
      const [completed, setCompleted] = useState(0);

      async function handleClick() {
        setPending(pending + 1);
        await delay(3000);
        setPending(pending - 1);
        setCompleted(completed + 1);
      }

      return (
        <>
          <h3>
            Pending: {pending}
          </h3>
          <h3>
            Completed: {completed}
          </h3>
          <button onClick={handleClick}>
            Buy
          </button>
        </>
      );
    }

    function delay(ms) {
      return new Promise(resolve => {
        setTimeout(resolve, ms);
      });
    }

    스냅샷임을 잘 생각해보고 구현하면,

    pending값이 1 이었다가 -1로 바뀌는걸 알 수 있죠
    값이 유기적으로 변할 수 있게 업데이트 함수로 구현합시다.

    import { useState } from 'react';

    export default function RequestTracker() {
      const [pending, setPending] = useState(0);
      const [completed, setCompleted] = useState(0);

      **async function handleClick() {
        setPending(p => p + 1);
        await delay(3000);
        setPending(p=>p - 1);
        setCompleted(c=>c+1);
      }**

      return (
        <>
          <h3>
            Pending: {pending}
          </h3>
          <h3>
            Completed: {completed}
          </h3>
          <button onClick={handleClick}>
            Buy
          </button>
        </>
      );
    }

    function delay(ms) {
      return new Promise(resolve => {
        setTimeout(resolve, ms);
      });
    }

    completed도 늘어나야하고,
    값의 변동이 렌더링 까지 동작할 수있게 바꿨습니다.
    ```

-   state 큐를 직접 구현해보세요

    ```tsx
    export function getFinalState(baseState, queue) {
      let finalState = baseState;
      let newValue = baseState
      queue.forEach(q => {
          if( typeof q === 'function'){
          newValue = q(newValue)
        } else {
            newValue = q
        }
      })

      return newValue;
    }

    q가 함수일때와 아닐때만 구분해서 값을 넣어주면 되겠죠
    ```

# 객체 state 업데이트

state는 객체를 포함해서, 어떤 종류의 JavaScript 값이든 저장할 수 있습니다. 하지만 React state에 있는 객체를 직접 변이해서는 안 됩니다. 대신 **객체를 업데이트하려면 새 객체를 생성**하고(혹은 기존 객체의 복사본을 만들고), **해당 복사본을 사용하도록 state를 설정**해야 합니다.

## 변이란 무엇인가요?

state값엔 어떤 종류던 넣을 수 있습니다.

숫자, 문자열, 불리언값은 “불변” 즉, 변이할 수 없거나 readonly입니다.

x state가 0에서 5로 변경 되었지만, 숫자 0 자체는 변경되지 않았습니다

js에서 숫자, 문자열, 불리언 같은 원시 자료형 값을 변경할 수 없습니다.

객체 state를 살펴봅시다.

```tsx
const [position, setPosition] = useState({ x: 0, y: 0 });
```

기술적으로 객체 자체의 내용을 변경하는 것은 가능합니다.

이를 **변이** 라고 합니다.

`position.x = 5`

리액트 state의 객체는 기술적으로 변이할 수 있지만, 숫자, 불리언 문자열과 같이 **불변하는 것 처럼 취급**해야 합니다. 객체를 직접 변이하는 대신, 항상 교체해야 합니다

## state를 읽기 전용으로 취급하세요

**state에 넣는 모든 js 객체를 읽기 전용으로 취급해야** 합니다.

```tsx
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

setState를 하지 않으면 렌더링 하지 않습니다.
```

이 코드는 이전 렌더링에서 `position`에 할당된 객체를 수정합니다. 하지만 **state 설정자 함수를 사용하지 않으면 React는 객체가 변이되었다는 사실을 알지 못합니다**. 그래서 React는 아무 반응도 하지 않습니다. 이미 음식을 다 먹은 후에 주문을 바꾸려고 하는 것과 같습니다. state 변이는 경우에 따라 작동할 수 있지만 권장하지 않습니다. **렌더링에서 접근할 수 있는 state 값은 읽기 전용으로 취급해**야 합니다.

실제 리렌더링을 위해선 **새 객체 생성 + state 설정자 함수에 전달**

setPosition으로 React에 지시하는 사항은

1. position을 이 새 객체로 바꿔라
2. 이 컴포넌트를 다시 렌더링 해라!

```tsx
onPointerMove={e => {
        setPosition({
          x: e.clientX,
          y: e.clientY
        });
      }}

로 수정 시 정상 동작!
```

Deep 지역 변이는 괜찮습니다.

```tsx
position.x = e.clientX;
position.y = e.clientY;
```

이런 코드는 기존 객체의 state를 수정하기 때문에 문제가 됩니다.

그러나 이런 코드는 방금 생성한 새로운 객체를 변이하는 것이기 때문에 **괜찮습니다**

```tsx
const nextPosition = {};
nextPosition.x = e.clientX;
nextPosition.y = e.clientY;
setPosition(nextPosition);

setPosition({
    x: e.clientX,
    y: e.clientY,
});
```

변이는 이미 state가 있는 기존 객체를 변경할 때만 문제가 됩니다. 방금 생성한 **객체를 변경**해도 **다른 코드가 아직 참조하지 않으므로 괜찮**습니다. **객체를 변경해도 해당 객체에 의존하는 다른 객체에 실수로 영향을 미치지 않습니다**. 이를 “**지역 변이(**local mutation)“라고 합니다. **렌더링하는 동안에도 지역 변이를 수행**할 수 있습니다. 매우 편리하고 완전 괜찮습니다!

## 전개 구문을 사용하여 객체 복사하기

이 패턴이 가장 많이 쓰이는 패턴이 아닌가 생각합니다.

모든 데이터를 다 넣을 땐 괜찮지만, 객체에서 데이터 일부분만 바꿀때 사용합니다.

-   전개구문 사용 전

    ```tsx
    import { useState } from 'react';

    export default function Form() {
      const [person, setPerson] = useState({
        firstName: 'Barbara',
        lastName: 'Hepworth',
        email: 'bhepworth@sculpture.com'
      });

      function handleFirstNameChange(e) {
        person.firstName = e.target.value;
      }

      function handleLastNameChange(e) {
        person.lastName = e.target.value;
      }

      function handleEmailChange(e) {
        person.email = e.target.value;
      }

      return (
        <>
          <label>
            First name:
            <input
              value={person.firstName}
              onChange={handleFirstNameChange}
            />
          </label>
          <label>
            Last name:
            <input
              value={person.lastName}
              onChange={handleLastNameChange}
            />
          </label>
          <label>
            Email:
            <input
              value={person.email}
              onChange={handleEmailChange}
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

    onChange가 렌더링을 유발하지 않기 때문에 동작하지 않습니다.
    ```

원하는 동작을 얻을 수 있는 가장 안정적인 방법은 새 객체를 생성 → 이를 setPerson에 전달. 하지만 여기서는 필드 중 하나만 변경되었으므로 **기존 데이터도 복사**

```tsx
setPerson({
    ...person, // Copy the old fields
    // 이전 필드를 복사합니다.
    firstName: e.target.value, // But override this one
    // 단, first name만 덮어씌웁니다.
});
```

-   사용 후

    ```tsx
    import { useState } from "react";

    export default function Form() {
        const [person, setPerson] = useState({
            firstName: "Barbara",
            lastName: "Hepworth",
            email: "bhepworth@sculpture.com",
        });

        function handleFirstNameChange(e) {
            setPerson({
                ...person,
                firstName: e.target.value,
            });
        }

        function handleLastNameChange(e) {
            setPerson({
                ...person,
                lastName: e.target.value,
            });
        }

        function handleEmailChange(e) {
            setPerson({
                ...person,
                email: e.target.value,
            });
        }

        return (
            <>
                <label>
                    First name:
                    <input value={person.firstName} onChange={handleFirstNameChange} />
                </label>
                <label>
                    Last name:
                    <input value={person.lastName} onChange={handleLastNameChange} />
                </label>
                <label>
                    Email:
                    <input value={person.email} onChange={handleEmailChange} />
                </label>
                <p>
                    {person.firstName} {person.lastName} ({person.email})
                </p>
            </>
        );
    }
    ```

각 입력값에 stat 변수를 선언하지않습니다. 양식이 크다면 업데이트가 이상하지만 않다면 그룹화하여 보관하기 편합니다.

… 전개 구문은 “얕은” 구문으로, 한 단계 깊이만 복사한다는 점에 유의합시다. 속도는 빠르지만 중첩된 프로퍼티를 업데이트 하려면 두 번 이상 사용해야 한다는 뜻입니다.

**Deep 여러 필드에 단일 이벤트 핸들러 사용하기**

객체 내에서 [] 및 중괄호를 이용해서 동적 이름을 가진 프로퍼티를 지정할 수 있습니다.

-   중괄호로 동적 이름으로 접근

    ```tsx
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
    여기선 input이라는 태그를 사용하니 쉽게 접근할 수 있습니다.
    ```

## 중첩된 객체 업데이트하기

```tsx
const [person, setPerson] = useState({
    name: "Niki de Saint Phalle",
    artwork: {
        title: "Blue Nana",
        city: "Hamburg",
        image: "https://i.imgur.com/Sd1AgUOm.jpg",
    },
});
```

person.artwork.city를 업데이트 하려면

해당 값을 바꿔줘도 됩니다.

하지만, 리액트에서 state는 불변입니다. city를 변경하려면 먼저 artwork 객체를 새 객체에 담고, person 객체를 새 person에 담아서 담아야합니다.

```tsx
const nextArtwork = { ...person.artwork, city: "New Delhi" };
const nextPerson = { ...person, artwork: nextArtwork };
setPerson(nextPerson);
```

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

😒 undefined일땐?? 어떻게될까? 합쳐지는 로직

```jsx
개인적으로 만들어본 로직이다.
a에 b를 합치는 로직이다.
mergeNotUndefined(a: Object, b?: Object) {
    const result = {}
    const keys = Object.keys(a)
    if (b === undefined) return a
    keys.forEach((key) => {
      if (a[key] !== undefined || b[key] !== undefined) {
        result[key] = b[key] || a[key]
      }
    })
    return result
  }
```

-   합쳐진 경우

    ```tsx
    import { useState } from "react";

    export default function Form() {
        const [person, setPerson] = useState({
            name: "Niki de Saint Phalle",
            artwork: {
                title: "Blue Nana",
                city: "Hamburg",
                image: "https://i.imgur.com/Sd1AgUOm.jpg",
            },
        });

        function handleNameChange(e) {
            setPerson({
                ...person,
                name: e.target.value,
            });
        }

        function handleTitleChange(e) {
            setPerson({
                ...person,
                artwork: {
                    ...person.artwork,
                    title: e.target.value,
                },
            });
        }

        function handleCityChange(e) {
            setPerson({
                ...person,
                artwork: {
                    ...person.artwork,
                    city: e.target.value,
                },
            });
        }

        function handleImageChange(e) {
            setPerson({
                ...person,
                artwork: {
                    ...person.artwork,
                    image: e.target.value,
                },
            });
        }

        return (
            <>
                <label>
                    Name:
                    <input value={person.name} onChange={handleNameChange} />
                </label>
                <label>
                    Title:
                    <input value={person.artwork.title} onChange={handleTitleChange} />
                </label>
                <label>
                    City:
                    <input value={person.artwork.city} onChange={handleCityChange} />
                </label>
                <label>
                    Image:
                    <input value={person.artwork.image} onChange={handleImageChange} />
                </label>
                <p>
                    <i>{person.artwork.title}</i>
                    {" by "}
                    {person.name}
                    <br />
                    (located in {person.artwork.city})
                </p>
                <img src={person.artwork.image} alt={person.artwork.title} />
            </>
        );
    }
    ```

**Deep 객체는 실제로 중첩되지 않습니다.**

```tsx
let obj = {
    name: "Niki de Saint Phalle",
    artwork: {
        title: "Blue Nana",
        city: "Hamburg",
        image: "https://i.imgur.com/Sd1AgUOm.jpg",
    },
};
```

```tsx
let obj1 = {
    title: "Blue Nana",
    city: "Hamburg",
    image: "https://i.imgur.com/Sd1AgUOm.jpg",
};

let obj2 = {
    name: "Niki de Saint Phalle",
    artwork: obj1,
};

let obj3 = {
    name: "Copycat",
    artwork: obj1,
};
```

이렇게 했을때 객체가 중첩된 게 아니라, obj2와 obj3의 artwork가 obj1을 가리킬 뿐입니다. 중첩이 아닙니다.

## Immer로 간결한 업데이트 로직 작성

state가 깊게 중첩된 경우 그것을 평평하게 만드는 것을 고려할 수 있습니다. 하지만 state 구조를 변경하고 싶지 않다면 중첩된 전개 구문보다 더 간편한 방법을 선호할 수 있습니다. Immer는 변이 구문을 사용하여 작성하더라도 자동으로 사본을 생성해주는 편리한 인기 라이브러리입니다. Immer를 사용하면 작성하는 코드가 “규칙을 깨고” 객체를 변이하는 것처럼 보입니다:

**Deep Immer는 어떻게 동작하나요**

Immer에서 제공하는 `draft`는 프록시라는 특수한 유형의 객체로, 사용자가 수행하는 작업을 “기록”합니다. 그렇기 때문에 원하는 만큼 자유롭게 수정할 수 있습니다! Immer는 내부적으로 `draft`의 어떤 부분이 변경되었는지 파악하고 편집 내용이 포함된 완전히 새로운 객체를 생성합니다.

1. `npm install use-immer`를 실행하여 Immer를 의존성으로 추가합니다.
2. 그런 다음 `import { useState } from 'react'`를 `import { useImmer } from 'use-immer'`로 바꿉니다.

-   immer

    ```tsx
    import { useImmer } from "use-immer";

    export default function Form() {
        const [person, updatePerson] = useImmer({
            name: "Niki de Saint Phalle",
            artwork: {
                title: "Blue Nana",
                city: "Hamburg",
                image: "https://i.imgur.com/Sd1AgUOm.jpg",
            },
        });

        function handleNameChange(e) {
            updatePerson((draft) => {
                draft.name = e.target.value;
            });
        }

        function handleTitleChange(e) {
            updatePerson((draft) => {
                draft.artwork.title = e.target.value;
            });
        }

        function handleCityChange(e) {
            updatePerson((draft) => {
                draft.artwork.city = e.target.value;
            });
        }

        function handleImageChange(e) {
            updatePerson((draft) => {
                draft.artwork.image = e.target.value;
            });
        }

        return (
            <>
                <label>
                    Name:
                    <input value={person.name} onChange={handleNameChange} />
                </label>
                <label>
                    Title:
                    <input value={person.artwork.title} onChange={handleTitleChange} />
                </label>
                <label>
                    City:
                    <input value={person.artwork.city} onChange={handleCityChange} />
                </label>
                <label>
                    Image:
                    <input value={person.artwork.image} onChange={handleImageChange} />
                </label>
                <p>
                    <i>{person.artwork.title}</i>
                    {" by "}
                    {person.name}
                    <br />
                    (located in {person.artwork.city})
                </p>
                <img src={person.artwork.image} alt={person.artwork.title} />
            </>
        );
    }
    ```

    이벤트 핸들러가 얼마나 간결해졌는지 주목하세요. 단일 컴포넌트에서 `useState`와 `useImmer`를 원하는 만큼 맞춰 사용할 수 있습니다. 특히 state에 중첩이 있고 객체를 복사하면 코드가 반복되는 경우 업데이트 핸들러를 간결하게 유지하는 데 Immer는 좋은 방법입니다.

**Deep : React에서 state 변이를 권장하지 않는 이유가 무엇인가?**

**디버깅**: console.log를 사용하고 state를 변이하지 않으면, 과거의 기록이 최근 state 변이에 의해 지워지지 않습니다. 따라서 렌더링 사이에 state가 어떻게 변경되었는지 명확하게 확인할 수 있습니다.

**최적화**: 일반적인 React 최적화 전략은 이전 프로퍼티나 state가 다음 프로퍼티나 state와 동일한 경우 작업을 건너뛰는 것에 의존합니다. state를 변이하지 않는다면 변경이 있었는지 확인하는 것이 매우 빠릅니다. 만약 `prevObj === obj` 라면, 내부에 변경된 것이 없다는 것을 확신할 수 있습니다.

**새로운 기능**: 우리가 개발 중인 새로운 React 기능은 state가 [스냅샷처럼 취급](https://react-ko.dev/learn/state-as-a-snapshot)되는 것에 의존합니다. 과거 버전의 state를 변이하는 경우 새로운 기능을 사용하지 못할 수 있습니다.

**요구 사항 변경**: 실행 취소/다시 실행 구현, 변경 내역 표시, 사용자가 양식을 이전 값으로 재설정할 수 있도록 하는 것과 같은 일부 애플리케이션 기능은 아무것도 변이되지 않은 state에서 더 쉽게 수행할 수 있습니다. 과거의 state 복사본을 메모리에 보관하고 필요할 때 재사용할 수 있기 때문입니다. 변경 접근 방식으로 시작하면 나중에 이와 같은 기능을 추가하기 어려울 수 있습니다.

**더 간단한 구현:** React는 변이에 의존하지 않기 때문에 객체에 특별한 작업을 할 필요가 없습니다. 많은 “반응형” 솔루션처럼 프로퍼티를 가로채거나, 항상 프록시로 감싸거나, 초기화할 때 다른 작업을 할 필요가 없습니다. 이것이 바로 React를 사용하면 추가 성능이나 정확성의 함정 없이 아무리 큰 객체라도 state에 넣을 수 있는 이유이기도 합니다.

Immer를 써본적은 없지만… 꼭 써야겠다!

## 도전과제

-   1

    ```tsx
    import { useState } from 'react';

    export default function Scoreboard() {
      const [player, setPlayer] = useState({
        firstName: 'Ranjani',
        lastName: 'Shettar',
        score: 10,
      });

      function handlePlusClick(e) {
        setPlayer({
          ...player,
          score: player.score+1,
        });
      }

      function handleFirstNameChange(e) {
        setPlayer({
          ...player,
          firstName: e.target.value,
        });
      }

      function handleLastNameChange(e) {
        setPlayer({
          ...player,
          lastName: e.target.value
        });
      }

      return (
        <>
          <label>
            Score: <b>{player.score}</b>
            {' '}
            <button onClick={handlePlusClick}>
              +1
            </button>
          </label>
          <label>
            First name:
            <input
              value={player.firstName}
              onChange={handleFirstNameChange}
            />
          </label>
          <label>
            Last name:
            <input
              value={player.lastName}
              onChange={handleLastNameChange}
            />
          </label>
        </>
      );
    }

    전개구문으로 잘 풀어줍시다.
    ```

-   2

    ```tsx
    import { useState } from "react";
    import Background from "./Background.js";
    import Box from "./Box.js";

    const initialPosition = {
        x: 0,
        y: 0,
    };

    export default function Canvas() {
        const [shape, setShape] = useState({
            color: "orange",
            position: initialPosition,
        });

        function handleMove(dx, dy) {
            setShape({
                color: shape.color,
                position: {
                    x: shape.position.x + dx,
                    y: shape.position.y + dy,
                },
            });
        }

        function handleColorChange(e) {
            setShape({
                ...shape,
                color: e.target.value,
            });
        }

        return (
            <>
                <select value={shape.color} onChange={handleColorChange}>
                    <option value="orange">orange</option>
                    <option value="lightpink">lightpink</option>
                    <option value="aliceblue">aliceblue</option>
                </select>
                <Background position={initialPosition} />
                <Box color={shape.color} position={shape.position} onMove={handleMove}>
                    Drag me!
                </Box>
            </>
        );
    }
    ```

-   3

    ```tsx
    import { useImmer } from "use-immer";
    import Background from "./Background.js";
    import Box from "./Box.js";

    const initialPosition = {
        x: 0,
        y: 0,
    };

    export default function Canvas() {
        const [shape, updateShape] = useImmer({
            color: "orange",
            position: initialPosition,
        });

        function handleMove(dx, dy) {
            updateShape((draft) => {
                draft.position.x += dx;
                draft.position.y += dy;
            });
        }

        function handleColorChange(e) {
            updateShape((draft) => {
                draft.color = e.target.value;
            });
        }

        return (
            <>
                <select value={shape.color} onChange={handleColorChange}>
                    <option value="orange">orange</option>
                    <option value="lightpink">lightpink</option>
                    <option value="aliceblue">aliceblue</option>
                </select>
                <Background position={initialPosition} />
                <Box color={shape.color} position={shape.position} onMove={handleMove}>
                    Drag me!
                </Box>
            </>
        );
    }
    ```

JavaScript에서 배열은 변경 가능하지만 state에 저장할 때는 변경이 불가능한 것으로 취급해야합니다. 객체와 마찬가지로, state에 저장된 배열을 업데이트하려면, 새로운 배열을 만들고(또는 기존 배열을 복사본을 만듦) 새 배열을 사용하도록 state를 설정해야합니다.

## 변이 없이 배열 업데이트하기

JavaScript에서 배열은 객체의 또 다른 종류일 뿐입니다. 객체와 마찬가지로, **React state의 배열은 읽기 전용으로 취급해야 합니다.** 즉, `arr[0] = 'bird'`와 같이 배열 내부의 항목을 재할당해서는 안 되며, `push()` 및 `pop()`과 같이 배열을 변이하는 메서드도 사용해서는 안 됩니다.

대신 배열을 업데이트할 때마다 state 설정 함수에 새 배열을 전달하고 싶을 것입니다. 그렇게 하려면 state의 원래 배열에서 `filter()` 및 `map()`과 같은 비변이 메서드를 호출하여 새 배열을 만들면 됩니다. 그렇게 만들어진 새 배열을 state로 설정할 수 있습니다.

다음은 일반적인 배열 연산에 대한 참조 표입니다. React state 내에서 배열을 다룰 때는 왼쪽 열의 메서드를 피하고 대신 오른쪽 열의 메서드를 선호해야 합니다:

또는 두 열의 메서드를 모두 사용할 수 있는 Immer를 사용할 수도 있습니다.

**함정: slice와 splice는 매우 다릅니다.**

slice는 배열 또는 배열의 일부를 복사할 수 있습니다.

splice는 배열의 항목을 삽입하거나, 삭제하기 위해 **변이**합니다.

React에서는 state의 객체나 배열을 변이하고 싶지 않기 때문에 `slice`()를 훨씬 더 자주 사용하게 될 것입니다. 객체 state 업데이트에서 변이가 무엇이고 왜 state에 권장되지 않는지에 대해 설명합니다.

😒slice와 spread 문법의 속도차이

일단 slice 는 배열에만 쓸 수 있고

slice가 배열일땐 더 빠른 **것** 같다

## 배열에 추가하기

`push`는 배열을 변이합니다.

```jsx
import { useState } from 'react';

let nextId = 0;

export default function List() {
  const [name, setName] = useState('');
  const [artists, setArtists] = useState([]);

  return (
    <>
      <h1>Inspiring sculptors:</h1>
      <input
        value={name}
        onChange={e => setName(e.target.value)}
      />
      <button onClick={() => {
        artists.push({
          id: nextId++,
          name: name,
        });
      }}>Add</button>
      <ul>
        {artists.map(artist => (
          <li key={artist.id}>{artist.name}</li>
        ))}
      </ul>
    </>
  );
}

add 버튼을 눌러도 동작하지 않습니다.
그러나 add버튼을 누르고, input값을 변경하면 값이 추가됩니다.
state변경이 렌더링을 촉발기 때문입니다.

name의 state도 바뀌었지만, artists또한 바뀌었습니다.
근데 왜 state 변경이 렌더링을 촉발하지 않은건 물론이고, 값이 변경된걸 유지하고 있을까요?
setState를 하지 않았기 때문입니다.
```

```jsx
setArtists( // Replace the state
  [ // with a new array
    ...artists, // that contains all the old items
    { id: nextId++, name: name } // and one new item at the end
  ]
);
이렇게 전개구문 + set구문을 사용하면?
쉽죠?
```

물론 앞으로 추가하고싶다면

```jsx
setArtists(
    // Replace the state
    [
        // with a new array
        { id: nextId++, name: name },
        ...artists, // that contains all the old items
        // and one new item at the end
    ]
);
```

요렇게 하면 됩니다.

## 배열 제거하기

```jsx
import { useState } from "react";

let initialArtists = [
    { id: 0, name: "Marta Colvin Andrade" },
    { id: 1, name: "Lamidi Olonade Fakeye" },
    { id: 2, name: "Louise Nevelson" },
];

export default function List() {
    const [artists, setArtists] = useState(initialArtists);

    return (
        <>
            <h1>Inspiring sculptors:</h1>
            <ul>
                {artists.map((artist) => (
                    <li key={artist.id}>
                        {artist.name}{" "}
                        <button
                            onClick={() => {
                                setArtists(artists.filter((a) => a.id !== artist.id));
                            }}
                        >
                            Delete
                        </button>
                    </li>
                ))}
            </ul>
        </>
    );
}
```

가장 간단한 방법은 각 이벤트 리스너에 해당 값을 필터링 할 수 있도록 함수를 구현합니다.

근데 이렇게하면 메모리상 손해가 아닐까요?

모든 button에 이벤트 리스너가 다리니깐

```jsx
import { useState } from "react";

let initialArtists = [
    { id: 0, name: "Marta Colvin Andrade" },
    { id: 1, name: "Lamidi Olonade Fakeye" },
    { id: 2, name: "Louise Nevelson" },
];

export default function List() {
    const [artists, setArtists] = useState(initialArtists);
    const handleDeleteArtist = (id) => {
        setArtists(artists.filter((artist) => artist.id !== id));
    };

    return (
        <>
            <h1>Inspiring sculptors:</h1>
            <ul>
                {artists.map((artist) => (
                    <li key={artist.id}>
                        {artist.name}{" "}
                        <button onClick={() => handleDeleteArtist(artist.id)}>Delete</button>
                    </li>
                ))}
            </ul>
        </>
    );
}
```

## 배열 변경하기

배열의 일부 또는 모든 항목을 변경하려면 `map()`을 사용하여 **새로운** 배열을 만들 수 있습니다. `map`에 전달할 함수는 데이터 또는 인덱스(또는 둘 다)에 따라 각 항목에 대해 수행할 작업을 결정할 수 있습니다.

이 예제에서 배열은 두 개의 원과 하나의 정사각형 좌표를 포함합니다. 버튼을 누르면 원만 50픽셀 아래로 이동합니다. 이 과은 `map()`을 사용하여 새로운 데이터 배열을 생성하여 수행됩니다

-   사각형만 안내려가는 예시 코드

    ```tsx
    import { useState } from 'react';

    let initialShapes = [
      { id: 0, type: 'circle', x: 50, y: 100 },
      { id: 1, type: 'square', x: 150, y: 100 },
      { id: 2, type: 'circle', x: 250, y: 100 },
    ];

    export default function ShapeEditor() {
      const [shapes, setShapes] = useState(
        initialShapes
      );

      function handleClick() {
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
        // Re-render with the new array
        setShapes(nextShapes);
      }

      return (
        <>
          <button onClick={handleClick}>
            Move circles down!
          </button>
          {shapes.map(shape => (
            <div
              key={shape.id}
              style={{
              background: 'purple',
              position: 'absolute',
              left: shape.x,
              top: shape.y,
              borderRadius:
                shape.type === 'circle'
                  ? '50%' : '',
              width: 20,
              height: 20,
            }} />
          ))}
        </>
      );
    }

    function Shape ({shape}) {
      return (
          <div
              style={{
              background: 'purple',
              position: 'absolute',
              left: shape.x,
              top: shape.y,
              borderRadius:
                shape.type === 'circle'
                  ? '50%' : '',
              width: 20,
              height: 20,
            }}
          />
      )
    }
    이런식으로 컴포넌트를 분리하는 게 더 좋아보인다.
    ```

## 배열에서 항목 교체하기

배열에서 하나 이상의 항목을 바꾸고 싶은 경우가 특히 흔합니다. `ar[0] = 'bird'`와 같은 할당은 원래 배열을 변이하는 것이므로 이 경우에도 **`map`**을 사용하는 것이 좋습니다.

항목을 바꾸려면 `**map**`으로 새 배열을 만듭니다. `**map`\*\* 호출 내에서 두 번째 인수로 항목의 인덱스를 받게 됩니다. 이를 사용하여 원래 항목(첫 번째 인수)을 반환할지 아니면 다른 것을 반환할지 결정할 수 있습니다

🤔이건 좀 의외입니다. 좀 더 좋은 방법이 없을까..

-   항목에 값 추가하면서, 할당 교체

    ```tsx
    import { useState } from "react";

    let initialCounters = [0, 0, 0];

    export default function CounterList() {
        const [counters, setCounters] = useState(initialCounters);

        function handleIncrementClick(index) {
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
        }

        return (
            <ul>
                {counters.map((counter, i) => (
                    <li key={i}>
                        {counter}
                        <button
                            onClick={() => {
                                handleIncrementClick(i);
                            }}
                        >
                            +1
                        </button>
                    </li>
                ))}
            </ul>
        );
    }
    ```

## 배열에 삽입하기

때로는 시작도 끝도 아닌 특정 위치에 항목을 삽입하고 싶을 때가 있습니다. 이를 위해 `...` 배열 전개 구문과 `slice()` 메서드를 함께 사용할 수 있습니다. `slice()` 메서드를 사용하면 배열의 “조각”을 잘라낼 수 있습니다. 항목을 삽입하려면 삽입 지점 앞에 slice를 spread한 다음 새 항목과 나머지 원래 배열을 펼치는 배열을 만듭니다.

-   항상 인덱스 1에 삽입되는 예시

    ```tsx
    import { useState } from "react";

    let nextId = 3;
    const initialArtists = [
        { id: 0, name: "Marta Colvin Andrade" },
        { id: 1, name: "Lamidi Olonade Fakeye" },
        { id: 2, name: "Louise Nevelson" },
    ];

    export default function List() {
        const [name, setName] = useState("");
        const [artists, setArtists] = useState(initialArtists);

        function handleClick() {
            const insertAt = 1; // Could be any index
            const nextArtists = [
                // Items before the insertion point:
                ...artists.slice(0, insertAt),
                // New item:
                { id: nextId++, name: name },
                // Items after the insertion point:
                ...artists.slice(insertAt),
            ];
            setArtists(nextArtists);
            setName("");
        }

        return (
            <>
                <h1>Inspiring sculptors:</h1>
                <input value={name} onChange={(e) => setName(e.target.value)} />
                <button onClick={handleClick}>Insert</button>
                <ul>
                    {artists.map((artist) => (
                        <li key={artist.id}>{artist.name}</li>
                    ))}
                </ul>
            </>
        );
    }
    ```

-   splice를 사용하는 예시

    ```tsx
    function handleClick() {
        const insertAt = 1; // Could be any index
        const nextArtists = [...artists];
        nextArtists.splice(insertAt, 0, { id: nextId++, name: name });
        setArtists(nextArtists);
        setName('');
      }

    splice를 쓰면 원본을 그대로 쓸 수 있다.
    ```

slice보다 좋은 방법은 없을까?

## 배열에 다른 변경 사항 적용하기

전개 구문과 `map()` 및 `filter()`와 같은 비변이 메서드만으로는 할 수 없는 일이 몇 가지 있습니다. 예를 들어, 배열을 반전시키거나 정렬하고 싶을 수 있습니다. JavaScript `reverse()` 및 `sort()` 메서드는 원래 배열을 변이하므로 직접 사용할 수 없습니다.

대신, **배열을 먼저 복사한 다음 변이하면 됩니다.**

-   반전 예시

    ```tsx
    import { useState } from "react";

    let nextId = 3;
    const initialList = [
        { id: 0, title: "Big Bellies" },
        { id: 1, title: "Lunar Landscape" },
        { id: 2, title: "Terracotta Army" },
    ];

    export default function List() {
        const [list, setList] = useState(initialList);

        function handleClick() {
            const nextList = [...list];
            nextList.reverse();
            setList(nextList);
        }

        return (
            <>
                <button onClick={handleClick}>Reverse</button>
                <ul>
                    {list.map((artwork) => (
                        <li key={artwork.id}>{artwork.title}</li>
                    ))}
                </ul>
            </>
        );
    }
    ```

여기서 […list] 를 사용하여 먼저 원본 배열의 복사본을 만들었고, 이제 복사본이 생겼으므로 nextList.reverse() 또는 nextList.sort() 같은 변이 메서드를 사용하거나 개별 항목을 할당한 후 set해주면됩니다.

그러나 **배열을 복사하더라도 배열 내부의 기존 항목을 직접 변이할 수는 없습니다.** 이는 얕은 복사가 이루어져 새 배열에는 원래 배열과 동일한 항목이 포함되기 때문입니다. 따라서 복사된 배열 내부의 객체를 수정하면 기존 state를 변이하는 것입니다. 예를 들어, 다음과 같은 코드가 문제가 됩니다.

2차원 이내라면 한번으로 가능하지만 2차원부터는 안된다.

```tsx
const nextList = [...list];
nextList[0].seen = true; // Problem: mutates list[0]
setList(nextList);
```

를 하면, list의 0번째 seen도 true로 바뀐다.

`nextList` 와 `list`는 다른 배열이지만, 각 객체 내부는 같은 곳을 바라본다.

이전 회차에서 보듯이

```tsx
const list = [
  { id: 0, title: 'Big Bellies' },
  { id: 1, title: 'Lunar Landscape' },
  { id: 2, title: 'Terracotta Army' },
];

nextList = [...list] 라면
nextList와 list는 다른 배열이다.
그러나 list[0] = { id: 0, title: 'Big Bellies' }를 가르키고 있고
이걸 좀 더 추상화해보면
list[0] = obj1 이고 이 메모리 참조를 끊지 않았다.

spread가 일어날때 해당 객체는 분해된다.
obj1,obj2,obj3 가 될 것이다.
그 상태에서 새로운 배열에 할당 했으니
[obj1, obj2, obj3]
list와 nextList는 내부가 같은 값이고 같은 값을 바라보지만
다른값이다.
할당 될 때 다른 값이 되기 때문이다.
그러나 각 obj1이라는 객체를 참조하고 있는게 같기 때문에
내부의 값을 찍어보면 같다고 나온다.
```

-   같지만 다름
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/c25eb7fa-fce5-4067-a2c4-bec8874fa401/5176af98-7e77-44c5-8431-fa54b362cf5e/Untitled.png)

그래서 중첩된 객체 업데이트는 좀 더 신중히 이루어져야한다.

## 배열 내부의 객체 업데이트기

객체는 실제로 내부에 위치하지 않습니다. 내부에 있는 것처럼 보이지만, **가르키고** 있는 별도의 값입니다.

**중첩된 state를 업데이트할 때는 업데이트하려는 지점부터 최상위 수준까지 복사본을 만들어야 합니다.** 어떻게 작동하는지 살펴보겠습니다.

-   중첩 객체를 분해하지 않았을때

    ```tsx
    import { useState } from "react";

    let nextId = 3;
    const initialList = [
        { id: 0, title: "Big Bellies", seen: false },
        { id: 1, title: "Lunar Landscape", seen: false },
        { id: 2, title: "Terracotta Army", seen: true },
    ];

    export default function BucketList() {
        const [myList, setMyList] = useState(initialList);
        const [yourList, setYourList] = useState(initialList);

        function handleToggleMyList(artworkId, nextSeen) {
            const myNextList = [...myList];
            const artwork = myNextList.find((a) => a.id === artworkId);
            artwork.seen = nextSeen;
            setMyList(myNextList);
        }

        function handleToggleYourList(artworkId, nextSeen) {
            const yourNextList = [...yourList];
            const artwork = yourNextList.find((a) => a.id === artworkId);
            artwork.seen = nextSeen;
            setYourList(yourNextList);
        }

        return (
            <>
                <h1>Art Bucket List</h1>
                <h2>My list of art to see:</h2>
                <ItemList artworks={myList} onToggle={handleToggleMyList} />
                <h2>Your list of art to see:</h2>
                <ItemList artworks={yourList} onToggle={handleToggleYourList} />
            </>
        );
    }

    function ItemList({ artworks, onToggle }) {
        return (
            <ul>
                {artworks.map((artwork) => (
                    <li key={artwork.id}>
                        <label>
                            <input
                                type="checkbox"
                                checked={artwork.seen}
                                onChange={(e) => {
                                    onToggle(artwork.id, e.target.checked);
                                }}
                            />
                            {artwork.title}
                        </label>
                    </li>
                ))}
            </ul>
        );
    }
    ```

```tsx
function handleToggleMyList(artworkId, nextSeen) {
    const myNextList = [...myList];

    setMyList(
        myNextList.map((link) => {
            if (link.id === artworkId) {
                return { ...link, seen: nextSeen };
            }
            return link;
        })
    );
}
```

이렇게 2중으로 분해할당을 하면 해결됩니다.

일반적으로 **방금 만든 객체만 변이해야 합니다.** _새로운_ artwork을 삽입하는 경우에는 변이해도 되지만, 이미 있는 state의 artwork을 다루는 경우에는 복사본을 만들어야 할 겁니다.

## Immer로 간결한 업데이트 로직 작성하기

중첩객체는 조금 반복도 해야하고, 가끔은 너무 햇갈릴 수 있습니다.

-   일반적으로 state를 몇 레벨 이상 깊이 업데이트할 필요는 없습니다. **state 객체가 매우 깊다면 다르게 재구성하여 평평하게 만드는 것**이 좋습니다.
-   state 구조를 변경하고 싶지 않다면, Immer를 사용해보세요. Immer는 변이 구문을 사용하여 작성하더라도 자동으로 사본 생성을 처리해 주어 편리합니다.

```tsx
updateMyTodos((draft) => {
    const artwork = draft.find((a) => a.id === artworkId);
    artwork.seen = nextSeen;
});
```

이는 _원본_ state를 변이하는 것이 아니라 Immer가 제공하는 특별한 `draft`객체를 변이하기 때문입니다. 마찬가지로 `push()` 및 `pop()`과 같은 변이 메서드를 `draft`의 콘텐츠에 적용할 수 있습니다.

-   장바구니 품목 업데이트하기

    ```tsx
    import { useState } from "react";

    const initialProducts = [
        {
            id: 0,
            name: "Baklava",
            count: 1,
        },
        {
            id: 1,
            name: "Cheese",
            count: 5,
        },
        {
            id: 2,
            name: "Spaghetti",
            count: 2,
        },
    ];

    export default function ShoppingCart() {
        const [products, setProducts] = useState(initialProducts);

        function handleIncreaseClick(productId) {
            const newProducts = [...products];
            setProducts(
                newProducts.map((product) => {
                    if (product.id === productId) {
                        return { ...product, count: product.count + 1 };
                    }
                    return product;
                })
            );
        }

        return (
            <ul>
                {products.map((product) => (
                    <li key={product.id}>
                        {product.name} (<b>{product.count}</b>)
                        <button
                            onClick={() => {
                                handleIncreaseClick(product.id);
                            }}
                        >
                            +
                        </button>
                    </li>
                ))}
            </ul>
        );
    }
    ```

-   장바구니 품목 제거하기
    ```tsx
    function handleDecreaseCount(productId) {
        const newProducts = [
            ...products.map((product) => {
                if (product.id === productId) {
                    return {
                        ...product,
                        count: product.count - 1,
                    };
                } else {
                    return product;
                }
            }),
        ].filter((product) => product.count > 0);
        setProducts(newProducts);
    }
    ```
-   비변이 메서드를 사용하도록 수정하기

    ```tsx
    function handleAddTodo(title) {
        const newTodos = [
            ...todos,
            {
                id: nextId++,
                title: title,
                done: false,
            },
        ];

        setTodos(newTodos);
    }

    function handleChangeTodo(nextTodo) {
        const todo = [
            ...todos.map((t) => {
                if (t.id === nextTodo.id) {
                    return {
                        ...nextTodo,
                    };
                }
                return t;
            }),
        ];
        setTodos(todo);
    }

    function handleDeleteTodo(todoId) {
        setTodos([...todos.filter((t) => t.id !== todoId)]);
    }
    ```
