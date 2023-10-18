# [9회] useState ~ useImperativeHandle
> 링크:https://woozy-stool-6eb.notion.site/9-useState-useImperativeHandle-1e447f58b74943869969ede50a8cb335?pvs=4
# useState

## 참조

컴포넌트 최상위 레벨에서 useState를 호출하여 state 변수를 선언한다. 배열 구조 분해를 사용하여 state 변수의 이름을 지정하는 것이 관례이다.

**매개변수**

`initialState`: 초기 state 설정 값으로 모든 데이터 타입이 허용되며, 초기 렌더링 이후에는 무시된다.

함수를 인자로 전달하면 초기화 함수로 취급하는데, 이 함수는 순수해야 하고 인자를 받지 않아야 하며 반드시 return 값이 존재해야 한다. (return 값이 state로 저장된다.)

**반환 값**

- `state` : 현재 state 값으로 첫 번째 렌더링 중에는 전달한 initialState 와 일치한다.
- `set(설정자) 함수` : state 값을 다른 값으로 업데이트하고 리렌더링을 촉발할 수 있다.

**주의사항**

useState는 훅이므로 컴포넌트의 최상위 레벨 혹은 직접 만든 훅에서만 호출된다. 반복문이나 조건문 안에서는 호출할 수 없다. 

strict mode 에서는 초기화 함수를 두 번 호출하는데 이는 개발 환경 전용으로 상용 환경에서는 2번 호출되지 않는다. 이때 초기화 함수는 순수해야 하므로 두번 동작해도 결과에 영향을 미치지는 않는다. 

### set(설정자) 함수

useState가 반환하는 set 함수를 사용하면 state 값을 업데이트하고 리렌더링을 촉발할 수 있다. set 함수에는 **다음 state를 직접 전달**하거나, **이전 state로부터 계산하여 다음 state를 도출**하는 함수를 전달할 수 있다.

```jsx
const [name, setName] = useState('kim');

function handleClick () {
	setName('Edward');
	setAge(a => a + 1);
	...
}
```

**매개변수**

`nextState` : state가 될 값, 함수를 전달하면 업데이터 함수로 취급된다. 이 역시 순수함수여야 하고 기존의 state를 유일한 인수로 사용해야 하며 다음 state를 반환해야 한다. React는 업데이터 함수를 대기열에 넣고 컴포넌트를 리렌더링한다. **렌더링 중에 대기열에 있는 모든 업데이터 함수는 이전 state를 적용하여 다음 state를 계산한다.**

**반환 값**

set 함수는 반환 값이 없다.

**주의사항**

set 함수는 다음 렌더링에 대한 state 변수만 업데이트 하므로, set 함수를 호출한 후에도 state 변수에는 여전히 호출 전 화면에 있던 이전 값이 담겨 있다. state 변수의 값이 동일하다면 컴포넌트와 그 자식들은 리렌더링 하지 않는다. → 최적화

React는 모든 state 업데이트를 일괄처리 한다. 모든 이벤트 핸들러를 실행하고 set 함수를 호출한 후 화면을 업데이트한다. → 단일 이벤트 중에 여러번 리렌더링 되는 것을 방지할 수 있다. 

❓드물게 DOM에 접근하기 위해 React가 화면을 더 일찍 업데이트하도록 강제해야 하는 경우 flushSync를 사용할 수 있다. 

렌더링 도중 set 함수를 호출하는 것은 현재 렌더링 중인 컴포넌트 내에서만 적용된다. React는 해당 출력을 버리고 즉시 새로운 state로 다시 렌더링을 시도한다. 이 패턴은 거의 필요하지 않지만 이전 렌더링 정보를 저장하는데 사용할 수 있다. 

## 사용법

### 이전 state를 기반으로 state 업데이트 하기

> Updating state based on the previous state
> 

```jsx
function handleClick() {
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
  setAge(age + 1); // setAge(42 + 1)
}
```

set 함수가 호출될 때마다 즉각적으로 state 변수를 업데이트 하려면 업데이터 함수를 전달하면 된다.

```jsx
function handleClick() {
  setAge(a => a + 1); // setAge(42 => 43)
  setAge(a => a + 1); // setAge(43 => 44)
  setAge(a => a + 1); // setAge(44 => 45)
}
```

React는 업데이터 함수가 실행될때마다 업데이터 함수를 큐에 넣고 다음 렌더링 중에 동일한 순서로 호출한다.

<aside>
🌟 **DEEP DIVE | 항상 업데이터 함수를 사용하는 것이 좋은가요?**

이전 state를 가지고 새로운 state를 계산할 때 업데이터 사용을 권장하지만 항상 그럴 필요는 없다. 

대부분의 경우 set 함수를 사용하는 것과 업데이터를 사용하는 방식에 차이가 없다.

다만 동일한 이벤트 내에서 여러 업데이트를 수행하는 경우, state 변수 자체에 접근하는 것이 어려운 경우에는 유용하게 사용할 수 있다. 

만약 어떤 state가 다른 state 변수의 이전 state로부터 계산되는 경우라면 이를 하나의 객체로 결합하고 reducer를 사용하는 것이 좋다. 

</aside>

### 객체 및 배열 state 업데이트

> Updating objects and arrays in state
> 

```jsx
// 공식문서의 예시코드
setForm({
  ...form,
  firstName: 'Taylor'
});

// 또 다른 방법
const updatedForm = [...form];
updatedForm.firstName = 'Taylor'

setForm(updatedForm);
```

**객체 및 배열 state 예시**

> Examples of objects and arrays in state
> 

2. Form (중첩 객체) 

중첩 객체를 업데이트 할때는 업데이트 하려는 객체의 복사본 뿐만 아니라 위쪽으로 올라갈 때마다 해당 객체를 포함하는 모든 객체에 대해 복사본을 만들어야 한다.

1. List(배열)

map() 메서드와 filter() 메서드는 기존의 배열을 변이하지 않고 교체한다. (map 메서드는 새로운 배열을 반환, filter 메서드는 조건에 해당하는 배열을 얕은 복사하기 때문)

1. Immer 로 간결한 업데이트 로직 작성

Immer 라이브러리를 사용하여 변이 없이 배열과 객체를 업데이트 하기 위한 반복적인 코드를 줄일 수 있다. 

```jsx
import { useImmer } from 'use-immer';

export default function BucketList() {
	const [list, **updateList**] = useImmer(initialList);

	function handleToggle(artworkId, nextSeen) {
    **updateList(draft => {
      const artwork = draft.find(a =>
        a.id === artworkId
      );
      artwork.seen = nextSeen;
    });**
  }
...
}
```

### 초기 state 다시 생성하지 않기

> Avoiding recreating the initial state
> 

```jsx
// 🔴 Avoid
// createInitialTodos()의 결과는 초기 렌더링에만 사용되만, 모든 렌더링에서 이 함수를 호출한다.
function TodoList() {
  const [todos, setTodos] = useState(**createInitialTodos**());
	...
}

// ✅ Good
// 함수결과가 아닌 **함수 자체를 전달하면 초기화 중에만 함수를 호출**한다.
function TodoList() {
  const [todos, setTodos] = useState(**createInitialTodos**);
	...
}
```

### key로 state 재설정하기

> Resetting state with a key
> 

컴포넌트에 다른 key를 전달하여 state를 재설정 할 수 있다. key가 변경되면 React는 컴포넌트 및 그 모든 자식을 처음부터 다시 생성하므로 state가 초기화 된다.

### 이전 렌더링에서 얻은 정보 저장하기

> **Storing information from previous renders**
> 

드물게 렌더링에 대한 응답으로 state를 변경해야하는 경우, 예를 들어 props가 변경에 따라 state 변수를 변경하고 싶을 때 이전 props의 값을 추적하기 위한 state 변수를 사용할 수 있다.

공식문서의 예시 코드 참고

이 방식은 오직 현재 렌더링 중이 컴포넌트의 state만 업데이트 할 수 있으며 변이가 아닌 state 업데이트여야만 한다. 

## 문제해결

> Troubleshooting
> 

### ****state의 값으로 함수를 설정하려고 하면 설정은 안되고 대신 호출됩니다****

> **I’m trying to set state to a function, but it gets called instead**
> 

함수를 인자로 전달하면 초기화 함수로 여기고, 이벤트 핸들러에서는 업데이터 함수로 받아들인다. 이 경우에는 함수를 호출해서 그 결과를 저장하려고 한다. 만약 함수 그 자체를 저장하려고 한다면 화살표 함수로 전달하여야 한다. 그래야 전달한 함수를 값으로써 저장한다. 

```jsx
// 🔴
const [fn, setFn] = useState(someFunction);

function handleClick() {
  setFn(someOtherFunction);
}

// ✅
const [fn, setFn] = useState(() => someFunction);

function handleClick() {
  setFn(() => someOtherFunction);
}
```

# useReducer

## 참조

컴포넌트에 reducer에 컴포넌트를 추가할 수 있다. 

```jsx
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

**매개변수**

- `reducer` : state가 업데이트 되는 방식을 지정하는 함수로 state와 action을 인자로 받고 다음 state를 반환해야 한다.
- `initialArg` : 초기 state가 계산되는 값으로 모든 유형의 값일 수 있다. 초기 state를 계산하는 방법은 init 인자에 따라 달라진다.
- `init` (optional) : 초기 state 계산 방법을 지정하는 초기화 함수로 따로 값을 전달하지 않으면 초기 state는 initialArf 로 설정되고, init 값을 전달하면 초기 state는 init(initialArg) 의 결과 값으로 설정된다.

**반환 값**

- 현재 state
- state를 다른 값으로 업데이트하고 리렌더링을 촉발할 수 있는 dispatch function

### `dispatch` function

useReducer 가 반환하는 dispatch 함수를 사용하면 state를 다른 값으로 업데이트 하고 렌더링을 촉발할 수 있다. dispatch 함수에 유일한 인수로 action을 전달해야 한다.

```jsx
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
  ...
```

**매개변수**

- `action` : 사용자가 수행한 작업으로 어떤 데이터 유형이든 올 수 있다. 관용적으로 액션은 보통 이를 식별하는 type 속성이 존재하는 객체이며 선택적으로 다른 정보를 포함할 수 있다.

**반환 값**

dispatch 함수에는 반환 값이 없다. 

**주의사항**

useState 와 동일

## 사용법

### 컴포넌트에 reducer 추가하기

> Adding a reducer to a component
> 

reducer 함수는 state와 action을 인자로 받는다. 그리고 action 의 type 값에 따라 여러 동작을 한다.

reducer 함수를 호출 할때는 reducer 함수를 직접 호출하지 않고 dispatch 함수를 호출하는 대신 원하는 동작에 해당하는 type을 인자로 전달하여 원하는 action 이 실행되게 한다. 

useState와 유사하지만 이벤트 핸들러의 state 업데이트 로직을 컴포넌트 외부의 단일함수(reducer 함수)로 옮길 수 있다. 

### reducer 함수 작성하기

> Writing the reducer function
> 

일반적으로 switch 을 사용하여 action type에 따라 각 case 에 대해 state 계산하고 반환한다. action은 어떤 형태든 가질 수 있는데 관례 상 type 프로퍼티가 있는 객체를 전달하는 것이 일반적이다. 

### 초기 state 재생성 방지하기

> Avoiding recreating the initial state
> 

세번 째 인수에 초기화 함수를 전달하면 이를 해결할 수 있다. 

props 변수에 따라 state를 업데이트 하려고 한다면 props 받아와 state를 업데이트하여 새로 할당해줘야 한다. 이런 경우 useState와 마찬가지로 props를 전달받아 state를 업데이트 해주는 함수를 호출하여야 하는데 이렇게 작성하면 이 역시 모든 렌더링에서 해당 함수를 호출하게 될 것이다.

이를 예방하는 방법 역시 useState와 동일하게 세번째 인수에 초기화 함수를 전달함으로써 해결할 수 있다.

만약 초기화 함수가 초기 state를 계산하는데 아무런 정보가 필요하지 않는 경우 useReducer의 두번째 인수로 null을 전달할 수 있다. 

```jsx
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
	// 🔴
  **const [state, dispatch] = useReducer(reducer, createInitialState(username));**
	// ✅ 
	// 이렇게 작성하면 초기화 후에는 초기 state가 다시 생성되지 않는다. 
  **const [state, dispatch] = useReducer(reducer, username, createInitialState);**
  // ...
```

## 문제해결

> Troubleshooting
> 

### action을 dispatch 했지만 로깅하면 이전 state 값이 표기 된다.

> **I’ve dispatched an action, but logging gives me the old state value**
> 

useState와 마찬가지로 state가 스냅샷처럼 발생하기 때문에 발생하는 에러이다. 

useState에서 업데이터 함수를 사용하는 것처럼 useReducer에선느 reudcer를 직접 호출하여 수동으로 계산할 수 있다.

```jsx
const action = { type: 'incremented_age' };
dispatch(action);

**const nextState = reducer(state, action);**
console.log(state);     // { age: 42 }
console.log(nextState); // { age: 43 }
```

## action을 dispatch 했지만 화면은 업데이트 되지 않습니다.

> **I’ve dispatched an action, but the screen doesn’t update**
> 

기존의 객체나 배열 state를 변이하면 발생하는 문제이다. 변이가 아니라 객체 state 업데이트 및 배열 state를 업데이트 해야 한다. 

### ****dispatch하면 reducer state의 일부분이 undefined가 됩니다****

> **A part of my reducer state becomes undefined after dispatching**
> 

새로운 state를 반환할 때 모든 case 브랜치가 기존 필드를 모두 복사하는지 확인해야 한다. 

(특히 중첩된 객체나 배열을 사용할 경우)

```jsx
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        ...state, 
        user: {
					...state.user
					state.name = 'kim'
				}
      };
    }
    // ...
```

### ****dispatch하면 모든 reducer state가 undefined가 됩니다****

> **My entire reducer state becomes undefined after dispatching**
> 

state 가 예기치 않게 undefined가 되었다면 케이스 중 하나에서 state 가 반환되지 않았거나 action type이 case문의 어느 경우와도 일치하지 않을 때가 있을 수 있다. 이런 경우 switch 문 부분에서 오류를 발생시켜 문제를 파악하고 해결하자.

### ****“리렌더링이 너무 많습니다” 라는 오류가 발생합니다****

> **I’m getting an error: “Too many re-renders”**
> 

이런 경우에는 이벤트 핸들러를 지정하는 과정에서 실수하는 경우가 많다. 

렌더링 하는 동안 핸드러를 호출하지 않도록 주의하자.

### ****reducer 또는 초기화 함수가 두 번 실행됩니다****

> **My reducer or initializer function runs twice**
> 

strict mode로 설정으로 개발 환경에서 기본적으로 reducer 및 초기화 함수가 두 번 호출된다. 

# useContext

## 참조

컴폰너트 최상위 레벨에서 useContext를 호출하여 context를 읽고 구독할 수 있다.

```jsx
import { useContext } from 'react';

function MyComponent() {
  const theme = useContext(ThemeContext);
  // ...
```

**매개변수**

`context` : creatContext로 생성한 context로 자체 정보를 보유하지 않고 컴포넌트에 제공하거나 읽을 수 있는 정보의 종류를 나타낸다.

**반환 값**

호출하는 컴포넌트에 대한 context 값을 반환한다. 이 값은 호출한 컴포넌트에서 트리 상 가장 가까운 SomeContext.Provider 에 전달된 value 이다. 만약 provider 가 존재하지 않는 경우에는 해당 context에 대해 createContext에 전달한 defaultValue가 된다. 

React는 context가 변경되면 context를 읽는 컴포넌트를 자동으로 리렌더링 한다.

**주의사항**

<Context.Provider>는 반드시 useContext()를 호출하는 컴포넌트 상위에 있어야 한다.

빌드 시스템이 출력 결과에 중복 모듈을 생성하는 경우 context가 손상될 수 있다. context는 전달하고 읽어와서 사용하는 conext는 정확하게 동일한 객체일 경우에만 동작하기 때문이다.

## 사용법

### 트리 깊숙이 데이터 전달하기

> Passing data deeply into the tree
> 

useContext() 는 항상 그것을 호출하는 컴포넌트의 위에서 가장 가까운 provider를 찾는다. useContext()를 호출하는 컴포넌트와의 사이에 얼마나 많은 컴포넌트 레이어가 있는지 상관없으며 컴포넌트 내의 provider 는 고려하지 않는다. 

### context를 통해 전달된 데이터 업데이트 하기

> Updating data passed via context
> 

Context.Provider 를 통해 value 값으로 context에 사용할 state를 전달하면 해당 Provider 하위에 위치한 모든 컴포넌트에서 context 객체를 사용하면 해당 state를 사용할 수 있다. 

set함수 역시 함께 전달하여 state 업데이트 또한 Provider 하위 객체에서 가능하다.

**context 업데이트 예시** 참고

1. 단일 컴포넌트로 Provider 추출하기
    
    앱이 커짐에 따라 Provider가 중첩된다면 단일 컴포넌트로 추출할 수 있다. 
    
2. context와 reducer를 통한 확장

### fallback 기본 값 지정하기

> **Specifying a fallback default value**
> 

부모 트리에서 context의 Provider를 찾지 못하면 useContext()가 반환하는 context 값을 해당 context를 생성할 때 지정한 기본값과 동일하다.

기본 값은 절대 변경되지 않으며 업데이트를 하려면 state를 사용해야 한다. 

null 대신 의미있는 기본값을 사용하면 실수로 provider 없이 일부 컴포넌트를 렌더링 하거나 테스트 환경에서 컴포넌트가 동작하는데 도움이 된다. 

### ****트리 일부에 대한 context 재정의하기****

> **Overriding context for a part of the tree**
> 

트리의 일부분을 다른 provider로 감싸면 context를 재정의 할 수 있다.

```jsx
<ThemeContext.Provider value="dark">
  ...
  **<ThemeContext.Provider value="light">**
    <Footer />
  **</ThemeContext.Provider>**
  ...
</ThemeContext.Provider>
```

### ****객체 및 함수 전달 시 리렌더링 최적화****

> **Optimizing re-renders when passing objects and functions**
> 

context는 값 뿐만 아니라 객체와 함수 등 모든 데이터 유형을 전달할 수 있다. 

단 함수를 전달하게 되면 context.Provider 가 속한 컴포넌트가 리렌더링 될 때마다 해당 conetxt를 사용하는 트리의 모든 컴포넌트도 리렌더링이 된다. 이를 예방하기 위해서 **함수를 context에 전달할 때는 useCAllback으로 감싸고 객체 생성은 useMemo로 감싸는게 좋다.** 

## 문제해결

### 컴포넌트가 provider 의 값을 인식하지 못합니다.

> **My component doesn’t see the value from my provider**
> 
1. `<SomeContext.Provider>`를 `useContext()`를 호출하는 컴포넌트 *위와 외부*로 이동한다.
2. 컴포넌트를 `<SomeContext.Provider>`로 감싸는 것을 잊었거나 생각했던 것과 다른 트리 부분에 넣었을 수 있다. → 계층 구조를 확인해야 한다.
3. 제공하는 컴포넌트에서 보는 `SomeContext` 와 읽는 컴포넌트에서 보는 `SomeContext`가 서로 다른 두 개의 객체가 되는 빌드 문제가 있을 수 있다. 전역에서 Context를 할당하여 콘솔로 확인해보고 동일하지 않는 경우 빌드 도구 수준에서 문제를 해결해야 한다. 

### ****기본값이 다른데도 context에서 항상 `undefined`만 얻습니다**

> **I am always getting `undefined` from my context although the default value is different**
> 

트리에 value 가 없는 provider를 사용했거나 실수로 다른 prop 명을 사용했을 수 있다. 

**`[createContext(defaultValue)` 호출의 기본값](https://react-ko.dev/reference/react/useContext#specifying-a-fallback-default-value)은 오직 위쪽에 일치하는 provider가 전혀 없는 경우만 적용된다.** 

# useRef

ref의 특정 행동만 제약할 수 있다. 

## 참조

`useRef(initialValue)`

컴포넌트의 최상위 레벨에서 `useRef`를 호출하여 [ref](https://react-ko.dev/learn/referencing-values-with-refs)를 선언합니다.

```jsx
import { useRef } from 'react';

function MyComponent() {
  const intervalRef = useRef(0);
  const inputRef = useRef(null);
  // ...
```

**매개변수**

`initialValue` : ref 객체의 current 프로퍼티 초기 설정 값이다.

**반환 값**

`current` : useRef는 단일 프로퍼티를 가진 객체를 반환한다. 

**주의사항**

ref.current 프로퍼티는 변이가 가능하다. 그러나 렌더링에 사용하는 객체(e.g. state의 일부)를 포함하는 경우에는 해당 객체를 변이해서는 안된다. 

ref.current 는 프로퍼티를 변경해도 컴포넌트가 렌더링 되지 않는다. ref는 일반 JavaScript 객체이므로 React가 변경을 감지할 수 없기 때문이다.

따라서 초기화를 제외하고는 렌더링 중에 ref.current 를 사용하지 말자. 컴포넌트 동작을 예측 할 수 없게 된다.

## 사용법

### ref로 값 참조하기

> **Referencing a value with a ref**
> 

ref는 컴포넌트의 시각적 출력에 영향을 미치지 않는 정보를 저장하는데 적합하다. 

**ref의 특징**

- 리렌더링 사이에 정보를 저장할 수 있다.
- 리렌더링을 촉발하지 않는다.
- 각각의 컴포넌트에 로컬로 저장된다.

### ref로 DOM 조작하기

> **Manipulating the DOM with a ref**
> 

ref를 사용하여 DOM을 조작하는 것이 일반적이다.

1. 값이 null인 ref 객체를 선언한다.
    
    ```jsx
    import { useRef } from 'react';
    
    function MyComponent() {
      **const inputRef = useRef(null);**
      // ...
    ```
    
2. ref 객체를 ref 속성으로 조작하려는 DOM 노드의 JSX에 전달한다.
    
    ```jsx
    // ...
    return **<input ref={inputRef} />;**
    ```
    

이렇게 코드를 작성하면 React가 DOM 노드를 생성하고 화면에 그린 후, ref 객체의 current 프로퍼티를 DOM 노드로 설정한다. 그러면 이제 DOM 노드 <input>에 접근하여 focus 같은 메서드를 호출할 수 있게 된다.

```jsx
function handleClick() {
    **inputRef.current.focus();**
}
```

### ref 콘텐츠 재생성 피하기

> **Avoiding recreating the ref contents**
> 

React는 초기에 ref 값을 한 번 저장하고, 다음 렌더링부터는 이를 무시한다. 

```jsx
function Video() {
  const playerRef = **useRef(new VideoPlayer());**
  // ...
```

위 예시 코드에서 new VideoPlayer()의 결과는 초기 렌더링에만 사용되지만 호출 자체는 렌더링 후에도 계속 이루어지는데, 값 비싼 객체를 생성하는 경우 리소스 낭비일 수 있다. → **ref 초기화가 필요하다.**

```jsx
function Video() {
  const playerRef = useRef(null);
  if (playerRef.current === null) {
    playerRef.current = new VideoPlayer();
  }
  // ...
```

## 문제해결

### 커스텀 컴포넌트에 대한 ref를 얻을 수 없습니다.

> **I can’t get a ref to a custom component**
> 

기본적으로 컴포넌트는 내부의 DOM 노드에 대한 ref를 외부로 노출하지 않는다.

```jsx
export default function MyInput({ value, onChange }) {
  return (
    <input
      value={value}
      onChange={onChange}
    />
  );
}

// 이런 경우에는 **forwardRef로 감싸면 된다.**
import { forwardRef } from 'react';

const MyInput = forwardRef(({ value, onChange }, ref) => {
  return (
    <input
      value={value}
      onChange={onChange}
      ref={ref}
    />
  );
});

export default MyInput;
```

이렇게 작성하면 부모 컴포넌트가 ref를 가져올 수 있다.

# ****useImperativeHandle****

## 참조

`useImperativeHandle`은 [ref](https://react-ko.dev/learn/manipulating-the-dom-with-refs)로 노출되는 핸들을 사용자가 직접 정의할 수 있게 해준다.

**매개변수**

- `ref` : forwardRef 렌더 함수에서 두 번째 인자로 받은 ref 이다.
- `createHandle` : 인자가 없고 노출하려는 ref 핸들을 반환하는 함수이다. 일반적으로 노출하려는 메서드가 있는 객체를 반환한다.
- `dependencies`(optional) : createHandle 코드 내에서 참조하는 모든 반응형 값을 나열한 목록이다. 리렌더링으로 일부 의존성이 변경되거나 이 인수를 생략한 경우 createHandle 함수가 다시 실행되고 새로운 핸들이 ref에 할당된다.

**반환 값**

undefined를 반환한다.

## 사용법

### ****부모 컴포넌트에 커스텀 ref 핸들 노출****

> **Exposing a custom ref handle to the parent component**
> 

컴포넌트는 자식 컴포넌트의 DOM 노드를 기본적으로 부모 컴포넌트에 노출하지 않는다. 부모 컴포넌트가  자식 컴포넌트의 DOM 노드에 접근하려면 forwardRef를 사용하여 선택적으로 참조에 포함해야 한다.

```jsx
import { forwardRef } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  **return <input {...props} ref={ref} />;**
});
```

forwardRef로 감싸진 MyInput은 input 태그의 DOM 노드에 접근이 가능해졌다. (MyInput의 ref는 input 태그의 DOM 노드를 받게 되었다.) 

그러나 대신 **사용자 지정값을 노출**할 수도 있다. 노출된 핸들을 사용자 정의하려면 컴포넌트의 최상위 레벨에서 useImperativeHandle을 호출하면 된다.

```jsx
import { forwardRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  **useImperativeHandle(ref, () => {
    return {
      // 메서드 입력
    };
  }, []);**

  return <input {...props} />;
});
```

위 코드에서 <input> 에 대한 ref는 더 이상 전달되지 않는다.

input 태그의 **DOM 노드를 노출하지 않고** focus와 scrollIntoView 두 **메서드만 노출하고 싶다**고 가정해보자.

→ 실제 브라우저의 DOM을 별도의 ref에 유지하고 useImperativeHandle을 사용하여 메서드만 있는 핸들을 노출한다.

```jsx
import { forwardRef, useRef, useImperativeHandle } from 'react';

const MyInput = forwardRef(function MyInput(props, ref) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => {
    **return {
      focus() {
        inputRef.current.focus();
      },
      scrollIntoView() {
        inputRef.current.scrollIntoView();
      },
    };**
  }, []);

  return <input {...props} ref={inputRef} />;
});
```

### 사용자 정의 명령 노출

> ****Exposing your own imperative methods****
> 

자식 컴포넌트에서 노출시킬 메서드를 useImperativeHandle의 매개변수로 작성하고,

부모 컴포넌트에서도 부모의 부모 컴포넌트에ㅓ 메서드를 재차 전달할 수도 있다. 이 때도 역시 컴포넌트를 forwardRef로 감싸야 한다. 

<aside>
🔥 **Pitfall | 함정**

ref를 과도하게 사용하지 말자. props로 표현할 수 없는 필수적인 행동에만 사용해야 한다. 

</aside>