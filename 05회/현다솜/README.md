애플리케이션이 커짐에 따라 state가 어떻게 구성되는지, 데이터가 컴포넌트 간에 어떻게 흐르는지에 대해 의식적으로 파악하면 도움이 된다. 불필요하거나 중복된 state는 버그의 흔한 원인이다.

이 장에서는 state를 잘 구성하는 방법, 업데이트 로직을 유지보수 가능하게 관리하는 방법, 멀리있는 컴포넌트 간에 state를 공유하는 방법에 대해 알아본다.

# State를 사용해 input 다루기

https://ko.react.dev/learn/reacting-to-input-with-state

React를 사용하면 코드에서 직접 UI를 수정하지 않는다. 예로 버튼 비활성화, 활성화, 성공 메세지 표시 등의 명령을 작성하지 않는다. 대신 컴포넌트의 여러 시각적 상태(초기 상태, 입력 상태, 성공 상태)에 대해 보고싶은 UI를 설명하고 유저 입력에 따라 state 변경을 유발한다. 이는 디자이너가 UI를 바라보는 방식과 비슷하다.

## 선언형 UI와 명령형 UI 비교

UI 인터렉션을 디자인할 때 유저가 액션을 하면 어떻게 UI를 변경해야할지 고민해본 적이 있을것이다. 유저가 폼을 제출한다고 생각해보자.

- 명령형의 경우
  - 폼에 무언가 입력하면 ‘제출’ 버튼이 활성화 될것이다.
  - ‘제출’버튼을 누르면 폼과 버튼이 비활성화되고 스피너가 나타날 것이다.
  - 네트워크 요청이 성공하면 폼은 숨겨지고 ‘감사합니다’ 메세지가 나타날 것이다.
  - 네트워크 요청이 실패하면 오류 메세지가 보일 것이고 폼은 다시 활성화될 것이다.

명령형 프로그래밍의 경우 우리는 컴퓨터에게 스피너에서부터 버튼까지 각각의 요소에 UI를 어떻게 업데이트 해야할지 ‘명령’을 내릴 뿐이다. 이는 간단해보일 수 있지만 더 복잡한 시스템에서는 난이도가 기하급수적으로 올라간다.

리액트는 이러한 문제점을 해결하기 위해 만들어졌다.

리액트에서는 직접 UI를 조작할 필요가 없다. 컴포넌트를 직접 비활성화하거나 보여주거나 숨길 필요가 없다. 대신 무엇을 보여주고 싶은지 선언하기만 하면 된다.

### UI를 선언적인 방식으로 생각하기

UI를 리액트에서 다시 구현하는 과정

1. 컴포넌트의 다양한 시각적 state를 확인
2. 무엇이 state 변화를 트리거하는지 알아내기
3. useState를 사용해서 메모리의 state를 표현하기
4. 불필요한 state 변수를 제거
5. state 설정을 위해 이벤트 핸들러 연결

## 첫번째: 컴포넌트의 다양한 시각적 state 확인하기

리액트는 디자인과 컴퓨터 과학의 사이에 이씩 때문에 두 아이디어 모두 영감을 받는다. 사용자가 볼 수 있는 UI의 모든 state를 시각화 해보자

- Empty : 비활성화된 제출버튼
- Typing : 활성화된 제출 버튼
- Submitting : 폼 비활성화, 스피너가 표시
- Success: 폼 대신 ‘감사합니다’ 메세지 표시
- Error: Typing과 비슷하지만 오류메세지 표시

<aside>
💡 다양한 상태의 디자인을 확인하기위해서 Storybook 을 이용하면 좋다. (living styleguide)

</aside>

## 두번째: 무엇이 state 변화를 트리거하는지 알아내기

두 종류의 인풋 유형으로 state 변경을 트리거할 수 있다.

- 버튼을 누르거나, 필드를 입력하거나, 링크를 이동하는 `휴먼 인풋`
- 네트워크 응답이 오거나, 타임아웃이 되거나, 이미지를 로딩하는 등의 `컴퓨터 인풋`

두 경우 모두 UI를 업데이트하기 위해서는 state 변수를 설정해야한다. 지금 만들고 있는 폼의 경우 몇가지의 입력에 따라 state를 변경해야한다.

- 텍스트 인풋: (휴먼)인풋이 비어있는지 여부에 따라 state를 empty에서 typing으로 혹은 그 반대로 변경
- 제출 버튼: (휴먼)클릭하면 subitting 상태로 변경
- 네트워크 응답이 성공: (컴퓨터) success 상태로 변경
- 네트워크 응답이 실패: (컴퓨터) 오류 메세지와 함께 error 상태로 변경

이러한 흐름은 종이에 그려 시각화할 수도 있다. 이러한 상태를 라벨링한 원으로 그리고 각각의 state 변화를 화살표로 이어보면 상태의 변화 흐름을 파악할 수 있을 뿐 아니라 구현 전에 버그를 찾을 수 도 있다.

## 세번째: 메모리의 state를 useState로 표현하기

다음으로 useState를 사용하여 컴포넌트의 시각적 state를 표현해야한다. 이 과정은 단순함이 핵심이다. 각각의 state는 ‘움직이는 조각’이고, 적을수록 좋다. 복잡한건 버그를 일으키기 마련이기 때문이다.

가장 필요한 상태를 가지고 시작해보자. 예를들어 인풋의 응답은 반드시 저장되어야한다. 그리고 가장 최근 발생한 오류도 저장해야할 것이다.

그리고 나서 앞서 필요하다고 나열했던 나머지 시각적 state도 살펴보자. 보통은 어떤 state 변수를 사용할지에 대한 방법이 여러가지기 때문에 이것저것 실험해볼 필요가 있다.

좋은 방법이 바로 떠오르지 않는다면 가능한 모든 시각적 state를 커버할 수 있는 확실한 것을 먼저 추가하는 방식으로 시작해보자.

## 네번째: 불필요한 state 변수를 제거하기

state의 중복은 피하고 필요한 state만 남겨두고 싶을 것이다. 상태의 구조를 리팩토링하는데에 시간을 조금만 투자하면 컴포넌트는 더 이해하기 쉬워질것이다. 리팩토링의 목표는 state가 사용자에게 유효한 UI를 보여주지 않는 경우를 방지하는 것이다.

state 변수에 대한 몇가지 질문을 참고해보자

- state가 역설을 일으키지 않았는가?
  isTyping과 isSubmitting 이 동시에 true일 수 없다. 이런 불가능한 상태를 제거하기 위해 세가지 값(typing, submitting, success)을 하나의 상태로 합칠 수 있다.
- 다른 state 변수에 이미 같은 정보가 담겨있지 않는가?
  isEmpty, isTyping은 동시에 true가 될 수 없다. 이 경우 운좋게도 isEmpty는 answer.length === 0 으로 체크할 수 있다.
- 다른 변수를 뒤집었을 때 같은 정보를 얻을 수 있지 않은가?
  isError는 error !== null로도 대신 확인할 수 있다

이렇게 정말 필요한 상태만 남겨둘 수 있고, 어느 하나를 지웠을 때 정상적으로 작동하지 않는다는 점에서 남은 것들이 모두 필수라는 것을 알 수 있다.

<aside>
💡 불가능한 상태는 Reducer로 해결해보자.

</aside>

## 다섯번째: state 설정을 위해 이벤트 핸들러 연결하기

onSubmit, onClick 등 연결

이러한 코드가 기존의 명령형 프로그래밍 예제보다는 길지만 좀 더 견고하다. 모든 인터랙션을 state로 표현하게 되면 이후에 새로운 시각적 state가 추가되더라도 기존의 로직이 손상되는 것을 막을 수 있다. 또한 인터랙션 로직을 변경하지 않고도 각각의 state에 표시되는 항목을 변경할 수도 있다.

---

# State 구조 선택하기

https://react-ko.dev/learn/choosing-the-state-structure

State를 잘 구조화한다면 지속적인 버그의 원인이 되는 컴포넌트가 아닌 수정과 디버깅이 용이한 컴포넌트를 만들 수 있다. 가장 중요한 원칙은 state가 중복되거나 불필요한 정보를 포함하지 않는 것이다.

## state 구조화 원칙

1. 관련 state를 그룹화

   항상 두개 이상의 상태 변수를 동시에 업데이트 하는 경우 단일 상태 변수로 병합하는 것이 좋음

2. state의 모순을 피하기

   여러 상태 조각이 서로 모순되거나 불일치 할 수 있는 방식으로 상태를 구성하면 실수가 발생할 여지가 생기기 쉬우므로 피하자.

3. 불필요한 state 피하기

   렌더링 중 컴포넌트의 props나 기존 상태 변수에서 일부 정보를 계산할 수 있다면 해당 정보를 상태에 넣지 말아야한다.

4. state 중복 피하기

   동일 데이터가 여러 상태 변수 혹은 중첩된 객체 내에 중복되면 동기화 상태를 유지하기 힘들기 때문에 가능한 중복을 줄이자.

5. 깊게 중첩된 state 피하기

   깊게 계층화된 상태는 업데이트하기 쉽지 않다. 가능하면 평평한 방식으로 구성하는 것이 좋다.

이러한 원칙의 목표는 실수없이 state를 쉽게 업데이트할 수 있도록 하는 것이다. 이는 데이터베이스 엔지니어가 버그 발생 가능성을 줄이기위해 데이터베이스 구조를 정규화 하는 것과 유사하다.

## 관련 state 그룹화하기

```jsx
const [x, setX] = useState(0);
const [y, setY] = useState(0);

const [position, setPosition] = useState({ x: 0, y: 0 });
```

기술적으로 두가지 다 사용할 수 있다. 하지만 두개의 state 변수가 항상 함께 변경되는 경우에는 통합하는 쪽이 좋다.

데이터를 객체나 배열로 그룹화하는건 필요한 state의 조각수를 모를 때(사용자가 사용자 정의 필드를 추가할 수 있는 양식이 있을 때) 유용하다

<aside>
💡 그룹화 하는 경우 전개구문으로 복사하는 것을 잊지 말자!

</aside>

## state의 모순을 피하자

여러 상태들이 같이 있을 때 예를들어 isSending, isSent는 동시에 true 일 수 없는 상황을 만들지 않도록 상태를 구성하는 것이 좋다.

(가독성을 위해 상수화 하는것도 좋다)

## 불필요한 state를 피하자

렌더링 중 컴포넌트의 props나 기존 state 변수에서 일부 정보를 계산할 수 있는 경우 해당 정보를 state에 넣지 않아야 한다

```jsx
const [firstName, setFirstName] = useState("");
const [lastName, setLastName] = useState("");

const fullName = firstName + " " + lastName;
```

여기서 fullName은 state 변수가 아니다.

<aside>
💡 props를 state에 그대로 미러링하지 말자.
props에서 받아온걸 state의 초기값으로 넣는 경우, 부모 컴포넌트가 props에 보내는 변수를 바꾸는 경우 적용되지 않을 수 있다.

</aside>

## state 중복을 피하자

같은 정보를 담는 state를 줄이는 것이 좋다.

## 깊게 중첩된 state는 피하자

깊게 중첩된 경우 상태를 변경하기 복잡해진다.

너무 깊게 중첩되어 업데이트하기 어려운 경우 flat 하게 만드는 것을 고려해보자.

---

# 컴포넌트 간 state 공유하기

https://ko.react.dev/learn/sharing-state-between-components

때때로 두 컴포넌트의 state가 항상 함께 변경되길 원할 수 있다. 이를 위해서 각 컴포넌트에서 state를 제거하고 가장 가까운 공통 부모 컴포넌트로 옮긴 후 props로 자식들에게 전달해야한다. 이러한 방법을 ‘state 끌어올리기’ 라고 하며 React 코드를 작성할 때 가장 흔히 하는 일 중 하나이다.

## state 끌어올리기

```jsx
import { useState } from "react";

function Panel({ title, children }) {
  const [isActive, setIsActive] = useState(false);
  return (
    <section className="panel">
      <h3>{title}</h3>
      {isActive ? (
        <p>{children}</p>
      ) : (
        <button onClick={() => setIsActive(true)}>Show</button>
      )}
    </section>
  );
}

export default function Accordion() {
  return (
    <>
      <h2>Almaty, Kazakhstan</h2>
      <Panel title="About">
        With a population of about 2 million, Almaty is Kazakhstan's largest
        city. From 1929 to 1997, it was its capital city.
      </Panel>
      <Panel title="Etymology">
        The name comes from <span lang="kk-KZ">алма</span>, the Kazakh word for
        "apple" and is often translated as "full of apples". In fact, the region
        surrounding Almaty is thought to be the ancestral home of the apple, and
        the wild <i lang="la">Malus sieversii</i> is considered a likely
        candidate for the ancestor of the modern domestic apple.
      </Panel>
    </>
  );
}
```

여기서 한번에 하나의 패널만 열리도록 수정해보자

- 끌어올리기 과정
  1. 자식 컴포넌트의 state 제거
  2. 하드코딩된 값을 공통의 부모로부터 전달
  3. 공통 부모에 state를 추가하고 이벤트핸들러와 함께 전달

## 스텝1: 자식 컴포넌트의 state 제거

상태를 제거 후 부모로부터 isActive 를 props로 받아오는 구조로 변경

## 스텝2: 하드 코딩된 데이터를 부모 컴포넌트로 전달하기

isActive의 활성화 여부로 컴포넌트를 제어한다.

## 스텝3: 공통 부모에 state 추가하기

어떤 자식 컴포넌트가 active인지 관리하는 상태를 추가한다.

또한 활성화 될 컴포넌트를 클릭시 해당 상태를 업데이트하도록 하는 이벤트핸들러를 만들어 자식 컴포넌트에 전달해준다.

<aside>
💡 제어와 비제어 컴포넌트
자식컴포넌트가 따로 isActive의 상태를 가져 부모 패널의 활성화 여부에 영향을 줄 수 없는 경우 비제어 컴포넌트라 부른다.
반대로 부모 컴포넌트에서 isActive를 관리하여 props에 의해 컴포넌트가 제어되는 경우 제어컴포넌트라 부른다.
비제어 컴포넌트는 설정할 것이 적어 부모 컴포넌트에서 사용하기 쉽다. 하지만 여러 컴포넌트를 함께 조정하려고 할 때는 비제어 컴포넌트는 덜 유연하다. 제어컴포넌트는 이런 면에서 유연하지만 부모 컴포넌트에서 props로 충분히 설정해주어야 한다.
엄격하게 구분된 개념은 아니고 일반적으로는 혼합해서 사용하는 경우가 많다. 하지만 이런 구분은 컴포넌트의 설계와 제공하는 기능을 설명하는데에 유용하다.
컴포넌트를 작성하는데 어떤 정보가 제어되어야하는지, 제어되지 않아야하는지 고려해보자. 이는 언제든지 리팩토링 할 수 있다.

</aside>

## 각 state의 단일 진리의 원천

리액트 애플리케이션에서 많은 컴포넌트는 자체 state를 가진다. 일부 상태는 입력처럼 리프 컴포넌트에 가깝게 생존하고 다른 상태는 앱의 상단에 더 가깝게 생존할 수 있다.

각각의 고유한 state에 대해 어떤 컴포넌트가 ‘소유’할지 고를수가 있다. 이 원칙은 ‘[단일 진리의 원천](https://en.m.wikipedia.org/wiki/Single_source_of_truth)’ 을 갖는 것으로 알려져있다. 이는 모든 state가 한 곳에 존재한다는 의미가 아니라 그 정보를 가지고 있는 특정 컴포넌트가 있다는 것을 말한다. 컴포넌트간에 공유된 state를 중복하는 대신 그들의 공통 부모로 끌어올리고 필요한 자식에게 전달하자.

작업이 진행되면서 애플리케이션은 계속 변해간다. 각 state가 어디에 ‘생존’해야할지 고민하는 동안 state를 아래로 이동하거나 다시 올리는 것도 흔히 있는 일이다.

---

# State를 보존하고 초기화하기

https://ko.react.dev/learn/preserving-and-resetting-state

컴포넌트가 리렌더링될 때 React는 트리에서 유지(및 업데이트)할 부분과 버리거나 다시 생성할 부분을 결정해야한다. 대부분의 경우 React의 자동화된 동작이 충분히 잘 작동한다. 기본적으로 React는 기존에 렌더링된 컴포넌트 트리와 일치하는 트리부분을 보존한다. 하지만 때로는 이게 바람직한 동작이 아닐 수 있다.

## UI 트리

브라우저는 UI 모델링을 할 때 여러 트리 구조를 사용한다. DOM은 HTML 엘리먼트를 나타내고, CSSOM은 CSS를 나타낸다. 또한 [접근성 트리](https://developer.mozilla.org/ko/docs/Glossary/Accessibility_tree) 라는 것도 있다.

또한 React는 당신이 만드는 UI를 관리하고 모델링하기 위한 트리 구조도 가지고 있다. React는 JSX로부터 UI 트리를 만든다. 그리고 React DOM은 UI트리와 일치하도록 브라우저 DOM을 업데이트 한다.

![image.webp](https://prod-files-secure.s3.us-west-2.amazonaws.com/f076ced0-fa44-460f-a1f6-67fa9565b764/fba86377-6e80-4d96-b191-14ca22895436/image.webp)

## State는 트리에서의 위치에 묶여있다.

컴포넌트에 state를 줄 때 state가 컴포넌트 안에 ‘살고’있다고 생각할 수도있다. 하지만 사실 state는 React 안에 있다. React는 컴포넌트가 UI트리에 있는 위치를 이용해 React가 가지고 있는 각 state를 알맞은 컴포넌트와 연결한다.

![image.webp](https://prod-files-secure.s3.us-west-2.amazonaws.com/f076ced0-fa44-460f-a1f6-67fa9565b764/f6eea76e-c619-4dab-8015-01911763327e/image.webp)

이러한 구조는 각각 트리에서 자기 고유의 위치에 렌더링되어 있으므로 분리되어있는 카운터이다. 일반적으로는 React를 사용할 때 위치에 대해 생각할 필요는 없지만 React가 어떻게 작동하는지 이해할 때 유용하다.

React에서 화면의 각 컴포넌트는 완전히 분리된 state를 가진다. 예를 들어 두 카운터 컴포넌트를 나란히 렌더링하면 그들은 각각 자신만의 독립된 score, hover 상태를 가지게 된다.

특정 카운터가 갱신되면 해당 컴포넌트의 상태만 갱신되는 것을 확인할 수 있다.

만약 두개의 컴포넌트를 렌더링했다가 하나를 렌더링하지 않도록 숨겼다가 다시 재 렌더링하는 경우 그 컴포넌트가 가졌던 상태는 초기화 된다.

React는 컴포넌트가 UI트리에서 그 자리에 렌더링되는 한 state를 유지한다. 하지만 그걸 제거하거나 같은 자리에 다른 컴포넌트가 렌더링되면 리액트는 그 state를 버린다.

## 같은 자리의 같은 컴포넌트는 state를 보존한다

```jsx
{
  isFancy ? <Counter isFancy={true} /> : <Counter isFancy={false} />;
}
```

이러한 경우에는 같은 자리에 같은 컴포넌트가 있기 때문에 state가 초기화되지 않고 유지된다.

<aside>
💡 React는 JSX 마크업에서가 아닌 UI 트리에서의 위치에 관심이 있다는 것을 기억하자!

</aside>

## 같은 위치의 다른 컴포넌트는 state를 초기화한다

같은 위치에 다른 컴포넌트를 렌더링할 때 그 컴포넌트는 그의 전체 서브 트리의 state를 초기화한다.

```jsx
import { useState } from "react";

export default function App() {
  const [isFancy, setIsFancy] = useState(false);
  return (
    <div>
      {isFancy ? (
        <div>
          <Counter isFancy={true} />
        </div>
      ) : (
        <section>
          <Counter isFancy={false} />
        </section>
      )}
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

이 경우 처음엔 `<section>` 하위에 카운터 컴포넌트가 있었다가 체크박스를 체크하는 경우 `<div>` 컴포넌트 하위에 카운터 컴포넌트가 있도록 변경된다. 이런식으로 section 이 div 로 바뀌면서 section이 DOM에서 제거 될 때 그 하위 트리가 제거되고 새로 div로 교체되기 때문에 초기화된다.

경험상 리렌더링할 때 state를 유지하고싶다면 트리구조가 “같아야” 한다. 만약 구조가 다르다면 React가 트리에서 컴포넌트를 지울 때 state가 제거되기때문에 state가 유지되지 않는다.

<aside>
💡 이것이 컴포넌트 함수를 중첩해서 정의하면 안되는 이유이다.
컴포넌트가 렌더링할 때마다 중첩된 함수가 새로 만들어지기 때문에 같은 함수에서 다른 컴포넌트를 렌더링할 때 그 하위의 state를 초기화한다. 이런 문제를 피하려면 항상 컴포넌트를 중첩해서 정의하지 않고 최상위 범위에서 정의해야한다.

</aside>

## 같은 위치에서 state를 초기화하기

일반적으로 같은 위치에 같은 컴포넌트를 두면 그 하위의 컴포넌트의 state는 유지된다. 하지만 때로 초기화하고 싶은 경우가 있는데 초기화를 하기 위한 두가지 방법이 있다.

1. 다른 위치에 컴포넌트를 렌더링하기
2. 각 컴포넌트에 key로 명시적인 식별자를 제공하기

### 다른위치에서 컴포넌트 렌더링하기

```jsx
{
  isA && <A />;
}
{
  isB && <A />;
}
```

이 경우 서로 다른 위치에 있기 때문에 초기화된다.

## key를 이용해 state 초기화 하기

```jsx
{
  isA ? <A key="a" /> : <A key="b" />;
}
```

key는 배열을 위한것만은 아니다. React가 컴포넌트를 구별할 수 있도록 key를 사용할 수 있다. key를 이용하지 않는다면 첫번째 컴포넌트, 두번째 컴포넌트 이런식으로 순서를 이용하지만 key를 이용한다면 특정한 컴포넌트라고 말해줄 수 있다.

<aside>
💡 key는 전역적으로 유일하지 않는다. key는 오직 부모 안에서만 자리를 명시한다

</aside>

## key를 이용해 폼을 초기화하기

채팅앱같은 경우 폼이 유지되지 않길 바랄 수 있는데 이럴 때 key를 사용하면 유용하다.

<aside>
💡 제거된 컴포넌트의 state 유지하기
1. 모두 렌더링 후 CSS로 안보이게 하기
2. state를 상위로 올리기
3. localStorage 이용하기

</aside>
