3 - UI 표현하기 (2)

# 조건부 렌더링

https://ko.react.dev/learn/conditional-rendering

컴포넌트는 조건에 따라 다른 항목을 표시해야하는 경우가 많다.

리액트는 if문, && 및 ? : 연산자와 같은 자바스크립트 문법을 사용하여 조건부로 JSX를 랜더링할 수 있다.

## 조건부로 JSX 반환하기

```jsx
const Test = (isA) => {
	if (isA) {
		return <span>A</span>
	} else {
		return <span>B</spna>
	}
};
```

## 조건부로 null을 사용하여 아무것도 반환하지 않기

```jsx
const Test = (isA) => {
  if (isA) {
    return <span>A</span>;
  } else {
    return null;
  }
};
```

## 조건부로 JSX 포함시키기

```jsx
ìf (isChecked) {
	return {checkicon}
}
```

## 삼항 조건 연산자 (? :)

```jsx
{
  isChecked ? <div>checked</div> : null;
}
```

이러한 스타일은 간단한 조건에 잘 어울리지만 적당히 사용하는 것이 좋다. 중첩된 조건부 마크업이 너무 많아서 컴포넌트가 지저분해지는 경우 자식 컴포넌트를 추출해서 정리하자.

### 논리 AND 연산자 (&&)

```jsx
{
  isChecked && { checkIcon };
}
```

<aside>
💡 && 표현식을 사용할 때는 왼쪽에 숫자를 두지 않도록 조심하자
→ 왼쪽편에는 boolean 타입이 확실할 때 쓰는 것이 좋을 듯

</aside>

## 변수에 조건부로 JSX를 할당하기

```jsx
let content = name;

if (isChecked) {
  content = name + { checkedIcon };
}
```

```jsx
<div>{content}</div>
```

---

# 리스트 렌더링

https://ko.react.dev/learn/rendering-lists

데이터 모음에서 유사한 컴포넌트를 여러개 표시하고 싶을 때 배열 메서드를 이용하여 데이터 배열을 조작할 수 있다.

## 배열을 데이터로 렌더링하기

1. 데이터를 배열로 이동
2. 배열의 요소를 새로운 JSX 노드 배열로 매핑(map사용)
3. 반환하기

## 배열의 항목들을 필터링하기

1. 배열의 특정 조건을 필터링 해서 새로운 배열을 생성
2. 요소를 매핑
3. 반환하기

<aside>
💡 화살표 함수는 암시적으로 ⇒ 바로 뒤에 식을 반환하기 때문에 return 문이 필요하지 않는다. 하지만 ⇒ 뒤에 중괄호가 오는 경우에는 return 을 명시적으로 작성해야한다.

</aside>

## key를 사용해서 리스트 항목을 순서대로 유지하기

key를 사용하지 않고 배열을 렌더링시키면 콘솔에 에러가 생긴다. 이는 각 배열 항목에 다른 항목 중에 고유하게 식별할 수 있는 문자열 또는 숫자를 key로 지정해야하기 때문이다.

<aside>
💡 각 리스트 항목에 대해 여러 DOM 노드 표시하기
<></> fragment 구문으로 key를 전달할 수 없으니 <Fragment>문법을 사용하면 된다.

</aside>

### key를 가져오는 곳

데이터 소스마다 다른 key 소스를 제공한다

- 데이터베이스의 데이터 : 고유한 데이터의 key / ID 를 사용
- 로컬에서 생성한 데이터 : uuid 패키지 사용

### key 규칙

- key는 형제간에 고유한 값을 가져야한다.
- key는 변경되어서는 안된다.

## React에 key가 필요한 이유

형제 항목간에 항목을 고유하게 식별할 수 있어야한다. 재정렬로 인해 위치가 변경되더라도 key는 React가 생명주기 내내 해당 항목을 식별할 수 있게 해준다.

<aside>
💡 배열에서 항목의 인덱스를 key로 사용하고 싶을 수 있다. 실제로 전혀 지정하지 않는 경우는 React는 인덱스를 사용한다. 하지만 항목이 삽입되거나 삭제되거나 배열의 순서가 바뀌면 시간이 지남에 따라 모든 항목을 렌더링 하는 순서가 변경된다. 인덱스를 key로 사용하면 종종 미묘하고 혼란스러운 버그가 발생할 수 있다.

</aside>

---

# 컴포넌트 순수하게 유지하기

https://ko.react.dev/learn/keeping-components-pure

순수함수는 오직 연산만을 수행한다. 컴포넌트는 엄격하게 순수함수로 작성하면 코드베이스가 점점 커지더라도 예상밖의 동작이나 당황케하는 버그를 피할 수 있다. 이러한 이점들을 취하기 위해서 몇가지 규칙을 따라야 한다.

## 순수성: 공식으로서의 컴포넌트

컴퓨터 과학에서 순수함수는 다음과 같은 특징을 지니고 있다.

- 자신의 일에 집중한다: 함수가 호출되기 전에 존재했던 어떤 객체나 변수는 변경하지 않는다.
- 같은 입력, 같은 출력: 같은 입력이 주어졌다면 순수함수는 같은 결과를 반환해야한다.

리액트는 이러한 순수함수의 컨셉 기반에 설계되었다. React는 작성되는 모든 컴포넌트가 순수함수일 거라 가정한다. 이러한 가정은 작성되는 React 컴포넌트가 같은 입력이 주어진다면 반드시 같은 JSX를 반환한다는 것을 의미한다.

## 사이드 이펙트: 의도하지않은 결과

React의 렌더링 과정은 항상 순수해야한다. 컴포넌트는 렌더링하기 전에 존재했던 객체나 변수들을 변경하지 말고 컴포넌트를 순수하지 않도록하는 JSX만 반환해야한다.

- 이를 위반하는 컴포넌트
  ```jsx
  let count = 0;

  const Content = () => {
    count = count + 1;
    return <span>{count}</span>;
  };

  const ContentSet = () => {
    return (
      <>
        <Content />
        <Content />
      </>
    );
  };
  ```
- 순수함수로 변경
  ```jsx
  const Content = ({ count }) => {
    return <span>{count}</span>;
  };

  const ContentSet = () => {
    return (
      <>
        <Content count={1} />
        <Content count={3} />
      </>
    );
  };
  ```

<aside>
💡 엄격모드로 순수하지 않은 연산을 감지

React에는 렌더링하면서 읽을 수 있는 세가지 종류의 입력 요소가 있다. props, state, context. 이러한 입력 요소는 항상 읽기전용으로 취급해야한다.
사용자의 입력에 따라 무언가를 변경하려는 경우 변수를 직접 수정하는 대신 setState를 활용해야한다. 컴포넌트가 렌더링되는 동안 기존 변수나 개체를 변경하면 안된다.
React는 개발중에 각 컴포넌트의 함수를 두 번 호출하는 ‘엄격 모드’를 제공하는데, 컴포넌트 함수를 두번 호출함으로서 엄격모드는 이러한 규칙을 위반하는 컴포넌트를 찾는데 도움을 준다.
순수함수는 연산만 하기 때문에 두 번 호출해도 아무것도 변경되지 않는다.
엄격모드는 프로덕션에 영향을 주지 않기 때문에 사용자의 앱 속도가 느려지지 않는다.

</aside>

## 지역 변형: 컴포넌트의 작은 비밀

위의 예시에서 문제는 렌더링하는 동안 컴포넌트가 기존 변수를 변경했다는 것이다. 이것은 ‘변형’으로 불리워서 조금 무섭게 들린다. 순수함수는 함수 스코프 밖의 변수나 호출 전에 생성된 객체를 변경하지 않는다.

하지만 렌더링하는 동안 그냥 만든 변수와 객체를 변경하는 것은 전혀 문제되지 않는다. 이를 ‘지역 변형’이라 불린다.

## 부작용을 일으킬 수 있는 지점

함수형 프로그래밍은 순수성에 크게 의존하지만 어떤때에는 무언가가 바뀌어야한다. 그것이 프로그래밍의 요점이다. 화면을 업데이트하고 애니메이션을 시작하고 데이터를 변경하는 변화들을 사이드 이펙트라고 한다. 렌더링 중에 발생하는 것이 아니라 “사이드에서” 발생하는 현상이다.

리액트에선 사이드 이펙트는 보통 “이벤트 핸들러”에 포함된다. 이벤트 핸들러는 리액트가 일부 작업을 수행할 때 반응하는 기능이다. 이벤트 핸들러가 컴포넌트 내부에 정의되었다 하더라도 렌더링 중에는 실행되지 않기 때문에 이벤트 핸들러는 순수할 필요가 없다.

다른 옵션을 모두 사용했지만 사이드 이펙트에 적합한 이벤트 핸들러를 찾을 수 없는 경우 useEffect를 호출하여 JSX에 해당 이벤트 핸들러를 연결할 수 있다. 이는 리액트에게 사이드 이펙트가 허용될 때 렌더링된 후 나중에 실행하도록 지시한다. → 하지만 이 접근 방식은 최후의 수단이 되어야한다.

가능한 렌더링만으로 로직을 표현해보자.

<aside>
💡 리액트는 왜 순수함을 신경쓸까?

- 컴포넌트는 다른 환경에서도 실행될 수 있다
- 입력이 변경되지 않은 컴포넌트 렌더링을 건너뛰어 성능을 향상시킬 수 있다 → 순수함수는 항상 동일결과를 반환하기에 캐시하기에 안전하다
- 깊은 컴포넌트 트리는 렌더링하는 도중에 일부 데이터가 변경되는 경우 React는 오래된 렌더링을 완료하는데 시간을 낭비하지 않고 렌더링을 다시 시작할 수 있다. 순수함은 언제든지 연산을 중단하는 것을 안전하게 한다

데이터를 가져오거나 애니메이션, 성능에 이르기 까지 컴포넌트를 순수하게 유지하면 리액트 패러다임의 힘이 발휘된다.

</aside>

---

# 상호작용 추가하기

https://ko.react.dev/learn/adding-interactivity

화면의 일부 요소는 사용자의 입력에 따라 업데이트된다. React에서는 시간에 따라 변화하는 데이터를 state라고 한다. state는 어떤 컴포넌트든 추가할 수 있으며 필요에 따라 업데이트할 수도있다.

## 이벤트에 대한 응답

React에서는 JSX에 이벤트 핸들러를 추가할 수 있다.

`<button>`과 같은 내장 컴포넌트는 `onClick`과 같은 내장 브라우저 이벤트만 지원하는 반면 사용자 정의 컴포넌트를 생성하는 경우에는 컴포넌트 이벤트 핸들러 prop의 역할에 맞는 이름을 사용할 수 있다.

## 이벤트에 응답하기

### 이벤트 핸들러 추가하기

1. `Button` 컴포넌트 내에 `handleClick` 함수를 선언.
2. 해당 함수의 내부 로직을 구현.
3. `<button>` JSX에 `onClick={handleClick}` 을 추가

### 이벤트 핸들러 함수의 특징

- 주로 컴포넌트 내부에서 정의된다.
- handle로 시작하고 그 뒤에 이벤트명을 붙인 함수명을 가진다.
  → 관습적으로 handle로 시작하여 이벤트명을 이어 붙인 명명법이 일반적이다.
- JSX 내에서 인라인으로도 정의 가능. (화살표 함수도)

<aside>
💡 이벤트 핸들러로 전달한 함수들은 호출이 아닌 전달이 되어야한다.

- 올바른 예시 : `<button onClick={handleClick} />`
- 잘못된 예시 : `<button onClick={handleClick()} />`

</aside>

### 이벤트 핸들러를 Prop으로 전달하기

종종 부모컴포넌트에서 자식 컴포넌트의 이벤트 핸들러를 지정하기 원할 수 있다. 이를 위해 컴포넌트가 그 부모 컴포넌트로부터 받은 prop을 이벤트 핸들러로 전달할 수 있다.

### 이벤트 핸들러 명명하기

보통 on 을 붙인다. 애플리케이션별 상호작용에 기반하여 명명하는 것이 나중에 어떻게 이를 이용하게 될지에 대한 유연성을 제공할 것이다.

<aside>
💡 적절한 HTML 태그를 사용하고 있는지 확인하자.
가끔 기본 브라우저 스타일링이 싫어 div로 다 통일하는 경우가 있는데 시멘틱하게 마크업을 작성하자.

</aside>

### 이벤트 전파

이벤트 핸들러는 해당 컴포넌트가 가진 어떤 자식 컴포넌트의 이벤트를 수신할 수도 있다. 이를 bubble 되거나 전파된다고 표현한다. 이때 이벤트는 발생한 지점에서 시작하여 트리를 따라 위로 전달된다.

- 부여된 JSX 태그 내에서만 실행되는 onScroll을 제외한 React 내의 모든 이벤트는 전파된다.

### 전파 멈추기

이벤트 핸들러는 이벤트 오브젝트를 유일한 매개변수로 받는다.

이를 이용하여 이벤트가 부모 컴포넌트에 닿지 못하도록 막으려면 `e.stopPropagation() `을 호출하자.

- 버튼을 누르면 이루어지는 과정
  1. React가 `<button>`에 전달된 `onClick` 핸들러를 호출한다.
  2. Button에 정으된 해당 핸들러는 다음을 수행한다
     - `e.stopPropagation()` 을 호출하여 이벤트가 더이상 bubbling 되지 않도록 방지한다
     - `onClick` 함수를 호출한다.
  3. 정의된 함수가 실행된다.
  4. 전파가 중단되었으므로 부모의 이벤트는 실행되지 않는다.

<aside>
💡 단계별 이벤트 캡쳐쳐
드물게 전파가 중단된 상황일지라도 자식 컴포넌트의 모든 이벤트를 캡쳐해 확인해야할 수도 있다. 이를 위해서는 이벤트명 마지막에 Capture 를 추가하면 된다.

</aside>

### 전파의 대안으로 핸들러를 전달하기

부모 컴포넌트에서 내려준 이벤트 핸들러를 실행하기 전에 `e.stopPropagation()`을 추가하여 작성해보자.

### 기본 동작 바잊하기

일부 브라우저 이벤트는 그와 관련된 기본 브라우저 동작을 가진다. 일례로 `<form>` 의 제출 이벤트는 그 내부의 버튼을 클릭 시 페이지 전체를 리로드하는 것이 기본 동작이다.

→ `e.preventDefault()`

### 이벤트 핸들러가 사이드 이펙트를 가질 수도 있는가?

가능하다. 이벤트 핸들러는 함수를 렌더링하는 것과 다르게 순수할 필요가 없어 무언가를 변경하는데 최적의 위치이다.

## State: 컴포넌트의 메모리

상호작용에 따라 컴포넌트는 종종 화면에 표시되는 내용을 변경해야한다. 컴포넌트는 현재 값을 기억해야하는데 React에서는 이러한 컴포넌트별 메모리를 state라고 부른다.

`useState` Hook을 사용하면 컴포넌트에 state를 추가할 수 있다.

(Hooks는 컴포넌트가 react의 주요 기능을 사용할 수 있도록 해주는 특별한 함수이다)

`useState`는 초기 state를 인자로 받으며 현재 상태와 상태를 업데이트할 수 있는 상태 설정 함수를 배열에 담아 반환한다.

```jsx
const [index, setIndex] = useState(0);
```
