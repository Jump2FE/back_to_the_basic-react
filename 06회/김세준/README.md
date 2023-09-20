# [6회차] State 관리하기(**State 로직을 리듀서로 작성하기** ~ 나머지)(2), EscapeHatches (~Ref로 DOM 조작하기)
> 링크 : https://woozy-stool-6eb.notion.site/6-State-State-2-EscapeHatches-Ref-DOM-951584dcac2a41ef8a38fb0b7bb5f1a5?pvs=4

# **Extracting State Logic into a Reducer**

> **State로직을 Reducer로 추출하기**
> 

reducer 함수는 컴포넌트 외부의 모든 state 업데이트 로직을 통합할 수 있는 단일함수이다. 

reducer 함수를 사용하면 state 업데이트가 여러 이벤트 핸들러에 분산되어 있는 경우 발생할 수 있는 컴포넌트 과부화를 해결할 수 있다. 

<aside>
💡 학습내용

- reducer 함수란 무엇인가
- `useState`를 `useReducer`로 리팩토링 하는 방법
- reducer를 사용해야 하는 경우
- reducer를 잘 작성하는 방법
</aside>

## **Consolidate state logic with a reducer**

> **reducer로 state 로직 통합하기**
> 

컴포넌트가 복잡해지면 컴포넌트의 state가 업데이트 되는 경우를 한눈에 파악하기 어려워질 수 있다. 

컴포넌트가 커질수록 state 로직이 여기저기 흩어지게 되는데, 이렇게 흩어진 state 로직들을 컴포넌트 외부의 reducer 라는 단일 함수로 옮겨서 복잡성을 줄이고 모든 로직에 접근이 용이한 구조를 만들 수 있다. 

Reducer는 state를 관리하는 방법 중 하나의 옵션

**useState → useReducer 마이크레이션 방법**

1. state를 설정하는 것에서 action들을 전달하는 것으로 변경하기
2. reducer 함수 작성하기
3. 컴포넌트에서 reducer 사용하기

### **Step 1: Move from setting state to dispatching actions**

> **state 설정을 action들의 전달로 바꾸기**
> 
- state 로직 : state를 설정하여 React에게 ‘무엇을 할 지’를 지시

```jsx
function handleAddTask(text) {
  setTasks([
    ...tasks,
    {
      id: nextId++,
      text: text,
      done: false,
    },
  ]);
}
```

- reducer : 이벤트 핸들러(dispatch)에서 ‘action’을 전달하여 ‘사용자가 방금 한 일’을 지정 (state 업데이트 로직은 다른 곳에..)

```jsx
function handleAddTask(text) {
	// dispatch 함수 내부의 객체를 action 이라고 한다. 
  dispatch({
    type: 'added',
    id: nextId++,
    text: text,
  });
}
```

이벤트 핸들러를 통해 ‘tasks를 설정’하는 대신 ‘tasks를 추가/변경/삭제’ 하는 action을 전달

<aside>
🔥 **Note**

action 객체는 일반적인 JavaScript 객체로 어떤 형태든 될 수 있다. 

그렇지만 일반적인 사용방법은 무슨 일이 일어나는지를 설명하는 문자열 타입의 type을 지정하고 다른 필드를 통해 추가적인 정보를 전달하는게 일반적이다. 

</aside>

### **Step 2: Write a reducer function**

> **reducer 함수 작성하기**
> 

reducer 함수에는 state 로직을 둘 수 있는데, React는 reducer 함수로부터 return 된 값을 state로 설정한다. 

reducer 함수는 두 개의 매개변수를 갖는데 하나는 현재의 state이고, 하나는 action 객체이다. 그리고 이 함수가 state를 return 한다. 

1. 현재의 state(`tasks`)를 첫 번째 매개변수로 선언하세요.
2. `action` 객체를 두 번째 매개변수로 선언하세요.
3. *다음* state를 reducer 함수에서 반환하세요. (React가 state로 설정할 것입니다.)

```jsx
function tasksReducer(tasks, action) {
  if (action.type === 'added') {
    return [
      ...tasks,
      {
        id: action.id,
        text: action.text,
        done: false,
      },
    ];
  } else if (action.type === 'changed') {
    return tasks.map((t) => {
      if (t.id === action.task.id) {
        return action.task;
      } else {
        return t;
      }
    });
  } else if (action.type === 'deleted') {
    return tasks.filter((t) => t.id !== action.id);
  } else {
    throw Error('Unknown action: ' + action.type);
  }
}
```

reducer 함수는 state를 매개변수로 갖기 때문에 컴포넌트 밖에서 reducer 함수를 선언할 수 있다. 

<aside>
🔥 **Note**

위의 코드에서는 if/else 구문을 사용하였지만 보통 reducer 함수는 swtich 문을 사용하여 작성하는게 일반적이다. 

```jsx
function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [
        ...tasks,
        {
          id: action.id,
          text: action.text,
          done: false,
        },
      ];
    }
    case 'changed': {
      return tasks.map((t) => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

switch 문을 사용할 때는 **case 블럭을 모두 중괄호로 감싸는 것을 추천**한다. 이렇게 하면 다**양한 case 들 안에서 선언된 변수들이 서로 충돌하지 않는다.** 또한 하나의 case는 보통 return 으로 끝나야 한다. return 을 하지 않으면 다음 case 로 코드가 넘어가게 된다. 

</aside>

<aside>
🌟 DEEP DIVE : 왜 reducer라고 부를까요?

`reduce()` 연산에서 따온 이름, reduce 연산은 배열을 가지고 많은 값들을 하나의 값으로 ‘누적’할 수 있다. 

React reducer 역시 지금까지의 state 와 action을 가지고 다음 state를 반환하며 이런 방식으로 시간이 지나면서 action들을 state로 모으기 때문이다. 

reduce 메서드를 사용하여 initialState와 actions 배열을 사용하여 reducer의 동작 원리를 구현할 수 도 있다. (공식 문서의 예시 코드 참조)

</aside>

### **Step 3: Use the reducer from your component**

> **컴포넌트에서 reducer 사용하기**
> 

마지막으로 useReducer Hook을 import 하여 컴포넌트에 reducer를 연결해야 한다. 

```jsx
import { useReducer } from 'react';

const [ task, dispatch ] = useReducer(taskReducer, initialTasks);
// const [ task, setTasks ] = useState(initialTasks);
```

useReducer Hook은 useState와 유사하게 초기 state 값을 전달해야 하며, 그 결과로  state값과 state 설정자 함수를 반환한다. (useReducer의 경우 dispatch 함수 반환)

차이점은 useReducer 함수는 reducer 함수, 초기 state 값 2개의 인자를 받고 state 값과 dispatch 함수(사용자의 action을 reducer 에 전달해주는 함수)를 반환한다.

## **Comparing `useState` and `useReducer`**

> **`useState`와 `useReducer` 비교하기**
> 
- 코드 크기
    - useState : 미리 작성해야 하는 코드가 줄어듦
    - useReducer : reducer 함수와 action 을 전달하는 부분을 모두 작성해야 한다. 다만, 비슷한 방식으로 state를 업데이트하는 이벤트 핸들러가 많이 존재하는 경우에는 useReducer를 사용하면 오히려 코드를 줄일 수도 잇다.
- 가독성
    - useState : 간단한 state를 업데이트 하는 경우에는 가독성이 좋다.
    - useReducer : state 구조가 복잡할 수록, 컴포넌트 코드가 방대할 수록 업데이트 로직이 어떻게 동작하는지와 이벤트 핸드럴르 통해 무엇이 일어났는지를 깔끔하게 분리하여 가독성을 높일 수 있다.
- 디버깅
    - useState : state 가 어디서 잘못 되었는지, 왜 잘못 되었는지 확인이 어려울 수 있다.
    - useReducer : reducer에 콘솔 로그를 추가하여 state 업데이트와 왜(어떤 action으로 인해) 버그가 발생했는지 확인할 수 있다.
- 테스팅
    - useState : -
    - useReducer : reducer는 컴포넌트에 의존하지 않는 순수함수이다. 따라서 별로도 분리해서 내보내거나 테스트가 가능하다.

→ 일부 컴포넌트에서 state 업데이트로 인해 버그가 자주 발행하고 코드에 방대하고 복잡할 수록 reducer를 사용하는게 좋고, 취향에 따라 useState와 useReducer를 적절히 섞어써도 된다.

## **Writing reducers well**

> **reducer 잘 작성하기**
> 
- reducer 함수는 반드시 순수해야 한다.
    
    reducer는 렌더링 중에 실행된다.(action은 다음 렌더링까지 대기)  입력 값이 같다면 결과 값도 항상 같아야 한다. 요청을 보내거나 timeout을 스케쥴링하거나 사이드 이펙트(컴포넌트 외부에 영향을 미치는 작업)을 수행해서는 안된다. reducer는 객체 및 배열을 변이 없이 업데이트해야 한다.
    
- 각 action은 데이터가 변경되더라도, 하나의 사용자 상호작용을 설명해야 한다.
    
    모든 action을 reducer에 기록하면 어떤 상호작용이나 응답이 어떤 순서로 일어났는지 재구성할 수 있을 만큼 로그가 명확해야
    

## **Writing concise reducers with Immer**

> **Immer를 사용하여 간결한 reducer 작성하기**
> 

reducer 역시 Immer 를 사용하면 더 간결하게 만들 수 있다. 

useImmerReducer 를 사용하면 push 또는 arr[i] = 할당으로 state를 변이할 수 있다.

```jsx
function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false,
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex((t) => t.id === action.task.id);
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter((t) => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}
```

reducer 는 순수해야 하므로 state를 변이하지 않아야 한다. useImmerReducer 또한 안전하게 변이할 수 있는 draft 객체를 제공한다. 이 역시 내부적으로 draft로 state의 복사본을 생성하고 이 복사본을 업데이트 한다. 

## Recap

- reducer를 사용하면 코드를 조금 더 작성해야 하지만 디버깅과 테스트에 도움이 됩니다.
- reducer는 반드시 순수해야 합니다.
- 각 action은 단일 사용자 상호작용을 설명해야 합니다.
- 변이 스타일로 reducer를 작성하려면 Immer를 사용하세요.

---

# **Passing Data Deeply with Context**

> **context로 데이터 깊숙이 전달하기**
> 

Context를 활용하면 부모 컴포넌트가 props를 통해 명시적으로 전달하지 않고도 모든 컴포넌트에서 props에 접근할 수 있다.

<aside>
💡 **학습내용**

- “Prop drilling”이란 무엇인가
- 반복적인 prop 전달을 Context로 대체하는 방법
- Context의 일반적인 사용 사례
- Context에 대한 일반적인 대안
</aside>

## **The problem with passing props**

> **props 전달의 문제**
> 

prosp 전달은 컴포넌트간에 필요한 데이터를 전달할 수 있는 좋은 방법이지만, 컴포넌트가 반복적으로 중첩되어 있거나 특정 데이터를 여러 컴포넌트에서 동시에 필요로 하는 경우 코드 복잡성을 높일 수 있다. 또한 필요한 데이터가 아주 멀리 떨어진 경우 state를 원하는 컴포넌트까지 반복해서 전달해야하는 props drilling이 발생할 수 있다. 

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8883249-24a5-43d5-a7b0-f77dda4453cc/c5c216be-8837-4b6d-bcac-32d326e2541c/Untitled.png)

React의 Context를 사용하면 어느 컴포넌트에서든 필요한 데이터에 직접 접근할 수 있다. 

## **Context: an alternative to passing props**

> **Context: props 전달의 대안**
> 

Context를 사용하면 상위 컴포넌트가 그 아래 전체 트리에 데이터를 제공할 수 있다. 

```tsx
<Section>
  <Heading level={3}>About</Heading>
  <Heading level={3}>Photos</Heading>
  <Heading level={3}>Videos</Heading>
</Section>
```

```tsx
<Section level={3}>
  <Heading>About</Heading>
  <Heading>Photos</Heading>
  <Heading>Videos</Heading>
</Section>
```

2번 예시코드가 동작하려면 Heading 컴포넌트가 Section 컴포넌트의 level 을 알 수 있어야 한다.

자식 컴포넌트가 트리 위 어딘가에 데이터를 ‘요청’할 수 있어야 한다. → Context를 사용

1. context를 **생성**합니다. (제목 level을 위한 것이므로 `LevelContext`라고 부를 수 있다)
2. 데이터가 필요한 컴포넌트에서 해당 context를 **사용**한다. (`Heading`은 `LevelContext`를 사용한다.)
3. 데이터를 지정하는 컴포넌트에서 해당 context를 **제공**한다. (`Section`은 `LevelContext`를 제공한다).

context는 멀리 떨어져 있는 상위 트리라도 그 안에 있는 전체 트리에 일부 데이터를 제공할 수 있게 해준다. 

![Screen Shot 2023-09-19 at 11.39.09 PM.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8883249-24a5-43d5-a7b0-f77dda4453cc/102a3047-52dd-4df2-9c1b-94fcbaada5b9/Screen_Shot_2023-09-19_at_11.39.09_PM.png)

### **Step 1: Create the context**

> **Step1: Context 만들기**
> 

먼저 context를 만들어야 합니다. 컴포넌트에서 사용할 수 있도록 파일 내보내기를 해야한다.

```jsx
// LevelContext.js
import { createContext } from 'react';

export const LevelContext = createContext(1);
```

createContext 의 유일한 인수는 기본값이다. 기본값으로는 객체를 포함한 모든 종류의 값을 전달할 수 있다.

### **Step 2: Use the context**

> **Step 2: context 사용하기**
> 

React와 context에서 `useContext` Hook을 가져온다.

```jsx
// 기존에 props로 leve을 전달받던 Heading 컴포넌트
export default function Heading({ level, children }) {
  // ...
}

// context인 LevelContext에서 값을 읽어오는 Heading 컴포넌트
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

useContext 역시 Hook으로 useState, useReducer와 마찬가지로 React 컴포넌트 최상단에서만 호출할 수 있다. useContext를 사용하여 Heading 컴포넌트가 LevelContext 데이터에 접근하길 원한다고 알려준다. 

여기까지 코드를 작성하면 context를 사용하고 있지만 아직 context를 제공하지 않았기 대문에 아직 level 값에 접근할 수 없다. 

### **Step 3: Provide the context**

> **Step 3: context 제공하기**
> 

`Section` 컴포넌트는 현재 children을 렌더링한다.

```jsx
export default function Section({ children }) {
  return (
    <section className="section">
      {children}
    </section>
  );
}
```

그 다음 **context provider로 감싸** `LevelContext`를 제공한다.

```jsx
**import { LevelContext } from './LevelContext.js';**

export default function Section({ level, children }) {
  return (
    <section className="section">
      **<LevelContext.Provider value={level}>**
        {children}
      **</LevelContext.Provider>**
    </section>
  );
}
```

이제 React는 `<section>` 안에 있는 컴포넌트가 `LevelContext`를 요청하면 이 `level`을 제공한다.

```jsx
// App.js
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section level={1}>
      <Heading>Title</Heading>
      <Section level={2}>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section level={3}>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section level={4}>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

- 상기 코드 작동 방식
    1. `level` prop을 `<Section>`에 전달합니다.
    2. `Section`은 section의 children을 `<LevelContext.Provider value={level}>`로 감쌉니다.
    3. `Heading`은 `useContext(LevelContext)`를 사용하여 위의 `LevelContext`값에 가장 가까운 값을 요청합니다.

## **Using and providing context from the same component**

> **동일한 컴포넌트에서 context 사용 및 제공**
> 

<본문 내용 거의 복붙>

현재는 여전히 각 section의 `level`을 수동으로 지정해야 한다.

```jsx
export default function Page() {
  return (
    <Section level={1}>
      ...
      <Section level={2}>
        ...
        <Section level={3}>
          ...
```

context를 사용하면 위의 컴포넌트에서 정보를 읽을 수 있으므로 각 `Section`은 위의 `Section`에서 `level`을 읽고 `level + 1`을 자동으로 아래로 전달할 수 있습니다. 방법은 다음과 같습니다.

```jsx
import { useContext } from 'react';
import { LevelContext } from './LevelContext.js';

export default function Section({ children }) {
  **const level = useContext(LevelContext);**
  return (
    <section className="section">
      **<LevelContext.Provider value={level + 1}>**
        {children}
      **</LevelContext.Provider>**
    </section>
  );
}
```

이렇게 변경하면 `level` prop을 `<Section>`이나 `<Heading>`*모두*에 전달할 필요가 없다.

```jsx
import Heading from './Heading.js';
import Section from './Section.js';

export default function Page() {
  return (
    <Section>
      <Heading>Title</Heading>
      <Section>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Heading>Heading</Heading>
        <Section>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Heading>Sub-heading</Heading>
          <Section>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
            <Heading>Sub-sub-heading</Heading>
          </Section>
        </Section>
      </Section>
    </Section>
  );
}
```

이제 `Heading`과 `Section`은 모두 `LevelContext`를 읽어 얼마나 “깊은” 수준인지 파악합니다. 그리고 `Section`은 그 children을 `LevelContext`로 감싸 그 안에 있는 모든 것이 “더 깊은” level에 있음을 지정합니다.

<aside>
🌟 Note

이 예제에서는 중첩된 컴포넌트가 context를 재정의하는 방법을 시각적으로 보여주기 위해 제목 level을 사용합니다. 하지만 context는 다른 많은 사용 사례에도 유용합니다. context를 사용하여 현재 색상 테마, 현재 로그인한 사용자 등 전체 **하위 트리에 필요한 모든 정보를 전달할 수 있습니다**.

</aside>

## **Context passes through intermediate components**

> **Context는 중간 컴포넌트들을 통과합니다**
> 

context를 제공하는 컴포넌트과 context를 사용하는 컴포넌트 사이에 원하는 만큼의 컴포넌트를 삽입할 수 있다. 

<div>와 같은 기본 제공 컴포넌트는 물론, 커스텀 컴포넌트도  포함된다.

Context를 사용하면 ‘주변 환경에 적응’하고 렌더링 되는 **위치(context)에 따라 다르게 표시되는 컴포넌트**를 작성할 수 있다.

React에서 **위에서부터 내려오는 context를 재정의하는 방법은 children을 다른 값으로 context provider로 감싸는 것**이다.

서로 다른 React context는 서로 재정의 하지 않으며, createContext()로 만드는 각 context는 다른 context와 완전히 분리되어 있다.

하나의 컴포넌트는 다양한 context를 사용할 수도, 제공할 수도 있다. 

https://codesandbox.io/embed/beautiful-frost-75zrtz?fontsize=14&hidenavigation=1&theme=dark

<aside>
🌟 **Why?**

독립된 AnotherParent 컴포넌트는 ColorContext를 제공하지 않는데, Context.Provider 하위에 존재하지 않더라도 context를 사용할 수 있는 것인가?

</aside>

## **Before you use context**

> **context를 사용하기 전에**
> 

context는 사용하기 간편하기 때문에 쉽게 남용될 수 있다. TypeScript 기반으로 코드를 작성한다면 props를 전달할 때마다 type에 대해 신경을 써줘야 하기 때문에 더욱 쉽게 남용될 여지가 있다. props를 몇 단계 깊이 전달한다고 해서 해당 정보를 context에 넣는 것은 좋지 않다.

**context를 사용하기 전 고려해야할 대안**

1. props 전달하기
    
    props를 전달하는 것은 번거로운 작업이 될 수도 있지만 어떤 컴포넌트가 어떤 데이터를 사용하는지 매우 명확하게 드러난다. 따라서 코드를 유지보수하는데 이어 props를 사용하면 데이터 흐름을 보다 명확하게 보여줄 수 있다. 
    
2. 컴포넌트를 추출하고 JSX를 children으로 전달하기
    
    일부 데이터를 해당 데이터를 사용하지 않는 여러 레이어를 거쳐 전달한다면 이는 추출할 수 있는 일부 컴포넌트들에 대해 추출하는 작업을 잊었다고 볼 수 있다. 예를 들어 `Layout posts={posts} />` 와 같은 방법 대신, Layout이 children을 prop으로 사용하도록 만들고 `<Layout><Posts posts={posts} /></Layout>`를 렌더링합니다. 이렇게 하면 데이터를 지정하는 컴포넌트와 데이터를 필요로 하는 컴포넌트 간의 props 전달 단계를 줄일 수 있다. 
    

→ 이 2가지 방법이 적절하지 않을 때 context 사용을 고려할 것

## **Use cases for context**

> **context 사용 사례**
> 
- 테마 : 어플리케이션 전역에 걸쳐서 사용해야하는 스타일 테마의 경우 앱 최상단에 context provider를 배치하여 사용할 수 있다.
- 현재 계정 : 많은 컴포넌트에서 현재 로그인한 사용자를 알아야 할 수 있다.
- 라우팅 : 대부분의 라우팅 솔루션은 내부적으로 context를 사용하여 현재 경로를 유지한다. 프로젝트를 진행할때 최상단에서 Routing 설정을 하는 이유라고 보면 된다.
- state 관리 : 앱이 성장함에 따라 복잡한 state를 관리하고 컴포넌트간 props를 전달하는 과정은 간결하게 처리할 수 있다.

Context 는 이처럼 앱 전반에 걸쳐 언제 어디서 쓰일 지 모르는 데이터들을 관리하는데 효과적이다. 또한 정적 값에 국한 되지 않고 다음 렌더링에서 다른 값을 전달하면 React는 하위에서 이 값을 사용하는 모든 컴포넌트를 업데이트 한다. 

# **Scaling Up with Reducer and Context**

> **Reducer와 Context로 확장하기**
> 

Reducer를 사용하여 컴포넌트의 state 로직을 통합하고, Context를 사용하여 다른 컴포넌트들에 정보를 쉽게 전달한다.→ 2가지의 조합을 통해 복잡한 화면의 state를 관리한다.

<aside>
🌟 학습내용

- reducer와 context를 결합하는 방법
- state와 dispatch 함수를 prop으로 전달하지 않는 방법
- context와 state 로직을 별도의 파일에서 관리하는 방법
</aside>

## **Combining a reducer with context**

> **reducer와 context를 결합하기**
> 

다른 컴포넌트들에서 state를 읽고 변경하려면 props를 통해 state와 state를 변경할 수 있는 이벤트 핸들러를 명시적으로 전달해야 한다.

state와  reducer의 dispatch를 context에 넣어서 사용하면 반복적인 props drilling 없이 모든 컴포넌트에서 state를 읽고 dispatch 함수를 실행할 수 있다. 

**Reducer 와 context를 결합하는 방법**

1. Context를 **생성한다**.
2. State과 dispatch 함수를 context에 **넣는다**.
3. 트리 안에서 context를 **사용한다**.

### **Step 1: Create the context**

> **Context 생성하기**
> 

useReducer 훅으로 현재 state를 담고 있는 tasks와 업데이트할 수 있는 dispatch 함수를 반환한다.

```jsx
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

...

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

const initialTasks = [
  { id: 0, text: 'Philosopher’s Path', done: true },
  { id: 1, text: 'Visit the temple', done: false },
  { id: 2, text: 'Drink matcha', done: false }
];

// TasksContext.js
import { createContext } from 'react';

**export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);**
```

두 개의 별개의 context를 생성

### **Step 2: Put state and dispatch into context**

> **State와 dispatch 함수를 context에 넣기**
> 

```jsx
export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
...
```

### **Step 3: Use context anywhere in the tree**

> **트리 안에서 context 사용하기**
> 

props를 제거하고  context 사용하기 

```jsx
export default function TaskList() {
  **const tasks = useContext(TasksContext);**
  // ...

export default function AddTask() {
  const [text, setText] = useState('');
  **const dispatch = useContext(TasksDispatchContext);**
  // ...
  return (
    // ...
    <button onClick={() => {
      setText('');
      **dispatch({
        type: 'added',
        id: nextId++,
        text: text,
      });**
    }}>Add</button>
    // ...
```

## **Moving all wiring into a single file**

> **하나의 파일로 합치기**
> 
1. Reducer로 state를 관리합니다.
2. 두 context를 모두 자식 컴포넌트에 제공합니다.
3. `[children`을 prop으로 받기 때문에](https://react-ko.dev/learn/passing-props-to-a-component#passing-jsx-as-children) JSX를 전달할 수 있습니다.

Reducer 와 context 파일을 TasksProvider 컴포넌트 하나로 묶기

```jsx
export function TasksProvider({ children }) {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        {children}
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

# **Referencing Values with Refs**

> **ref로 값 참조하기**
> 

값을 저장하고 싶지만 state처럼 값의 변이가 렌더링을 촉발하지 않도록 할 때 ref를 사용할 수 있다.

<aside>
🌟 **학습내용**

- 컴포넌트에 ref를 추가하는 방법
- ref 값을 업데이트하는 방법
- state와 ref의 차이점
- ref를 안전하게 사용하는 방법
</aside>

## **Adding a ref to your component**

> **컴포넌트에 ref 추가하기**
> 

```jsx
import { useRef } from 'react';

const ref = useRef(0)
```

컴포넌트 내부에서 `useRef` 훅을 호출하고 참조할 초기값을 인자로 전달한다. 

여기서 `ref` 의 초기값은 0이고, useRef는 아래와 같은 객체를 반환한다.

```jsx
{ 
  current: 0 // The value you passed to useRef
}
```

`ref.current` 속성을 통해 ref의 현재 값에 액세스 할 수 있으며 ref 값은 변이가 가능하다. 

여기서는 숫자를 가리키고 있지만 state와 마찬가지로 ref 또한 문자열, 객체, 함수 등을 가리킬 수 있다. 

state와 ref의 차이점

- ref는 current 속성을 읽고 수정할 수 있는 일반 JavaScript 객체이다.
- ref를 변이해도 리렌더링이 촉발되지 않는다.

## **Example: building a stopwatch**

> **예제: 스톱워치 만들기**
> 

중략

“Stop” 버튼을 누르면 `now` state 변수의 업데이트를 중지하도록 기존 interval을 취소해야 합니다. 이 작업은 `[clearInterval](https://developer.mozilla.org/en-US/docs/Web/API/clearInterval)`을 호출하여 수행할 수 있지만, 사용자가 시작을 눌렀을 때 이전에 `setInterval` 호출에서 반환한 interval ID를 제공해야 합니다. interval ID를 어딘가에 보관해야 합니다. **interval ID는 렌더링에 사용되지 않으므로 ref에 보관할 수 있습니다.**

```jsx
export default function Stopwatch() {
  const [startTime, setStartTime] = useState(null);
  const [now, setNow] = useState(null);
  const intervalRef = useRef(null);

  function handleStart() {
    setStartTime(Date.now());
    setNow(Date.now());

    clearInterval(intervalRef.current);
    intervalRef.current = setInterval(() => {
      setNow(Date.now());
    }, 10);
  }

  function handleStop() {
    clearInterval(intervalRef.current);
  }
```

## **Differences between refs and state**

> **ref와 state의 차이점**
> 

| refs | state |
| --- | --- |
| useRef(initialValue)는 { current: initialValue }을 반환 | useState(initialValue)는 state 변수의 현재값과 state 설정자함수([value, setValue])를 반환 |
| 변경 시 리렌더링을 촉발하지 않음 | 변경 시 리렌더링을 촉발함 |
| Mutable— 렌더링 프로세스 외부에서 current 값을 수정하고 업데이트할 수 있음 | “Immutable”— state setting 함수를 사용하여 state 변수를 수정해 리렌더링을 대기열에 추가해야함 |
| 렌더링 중에는 current 값을 읽거나 쓰지 않아야 함 | 언제든지 state를 읽을 수 있음. 각 렌더링에는 변경되지 않는 자체 state https://react-ko.dev/learn/state-as-a-snapshot이 있음 |

# **Manipulating the DOM with Refs**

> **ref로 DOM 조작하기**
> 

React가 DOM 요소를 직접 조작하고자 할때 ref 를 통해 DOM 노드에 접근할 수 있다.

<aside>
🌟 **학습내용**

- `ref` 어트리뷰트로 React가 관리하는 DOM 노드에 접근하는 방법
- `ref` JSX 속성이 `useRef` 훅과 관련되는 방법
- 다른 컴포넌트의 DOM 노드에 접근하는 방법
- 어떤 경우에 React가 관리하는 DOM을 수정해도 안전한지
</aside>

## **Getting a ref to the node**

> **노드에 대한 ref 가져오기**
> 

```jsx
import { useRef } from 'react';

const myRef = useRef(null);

<div ref={myRef}>
```

`ref` 를 선언한 후 DOM 노드를 가져올 JSX 태그에 ref 속성으로 참조를 전달한다.

여기서 useRef 또한 current 프로퍼티가 포함된 객체를 반환하는데, 처음에 myRef.current 객체는 null이 될 것이다.

추후에 React가 <div>에 대한 DOM 노드를 생성하고 나서야 React는 이 노드에 대한 참조를 ref.current 객체에 넣는다. 그런 다음 이벤트 핸들러에서 이 DOM 노드에 액세스 하고 여기 정의된 빌트인 브라우저 API를 사용할 수 있다. 

## **Example: Focusing a text input**

> **예: 텍스트 input에 초점 맞추기**
> 
1. `useRef` 훅으로 `inputRef`를 선언합니다.
2. 이를 `<input ref={inputRef}>`에 전달합니다. 이렇게 하면 React가 이 `<input>`의 DOM 노드를 `inputRef.current`에 넣을 것입니다.
3. `handleClick` 함수에서 `inputRef.current`로부터 input DOM 노드를 읽어와서 `inputRef.current.focus()`로 `[focus()](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus)`를 호출합니다.
4. `<button>`의 `onClick`에 `handleClick` 이벤트 핸들러를 전달합니다.

## **Example: Scrolling to an element**

> **예: element로 스크롤하기**
> 

중략

<aside>
🌟 **DEEP DIVE: 콜백을 사용하여 refs 목록을 관리하는 방법**

훅은 컴포넌트 최상위 레벨에서만 호출해야 하므로 반복문 또는 map 메서드 내부에서는 useRef를 호출할 수 없다. 

ref 콜백은 ref 속성에 함수를 전달하는 것인데, React는 ref를 설정할 때가 되면 DOM 노드로, 지울 때가 되면 `null`로 ref 콜백을 호출할 것이다.

```jsx
<li
  key={cat.id}
  ref={node => {
    const map = getMap();
    if (node) {
      // Add to the Map
      map.set(cat.id, node);
    } else {
      // Remove from the Map
      map.delete(cat.id);
    }
  }}
>
```

`itemsRef`는 단일 DOM 노드를 보유하지 않습니다. 대신 항목 ID에서 DOM 노드로의 [Map](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Map)을 보유합니다. ([Ref는 모든 값을 보유할 수 있습니다!](https://react-ko.dev/learn/referencing-values-with-refs)) 모든 목록 항목의 `[ref` 콜백](https://react-ko.dev/reference/react-dom/components/common#ref-callback)은 맵을 업데이트합니다.

</aside>

## **Accessing another component’s DOM nodes**

> **다른 컴포넌트의 DOM 노드에 접근하기**
> 

**React는 컴포넌트가 다른 컴포넌트의 DOM노드에 접근하는 것을 허용하지 않는다.** 심지어 자식 컴포넌트라도 마찬가지이다.

다른 컴포넌트의 DOM 노드를 수동으로 조작하면 코드가 훨씬 취약해지기 때문에 DOM 노드를 노출하길 원하는 컴포넌트에 해당 동작을 설정해야 한다. **컴포넌트는 자신의 ref 를 자식 중 하나에 ‘전달’하도록 지정할 수 있다.** 

```jsx
import { forwardRef, useRef } from 'react';

const MyInput = forwardRef((props, ref) => {
  return <input {...props} ref={ref} />;
});

export default function Form() {
  const inputRef = useRef(null);

  function handleClick() {
    inputRef.current.focus();
  }

  return (
    <>
      <MyInput ref={inputRef} />
      <button onClick={handleClick}>
        Focus the input
      </button>
    </>
  );
}
```

1. `<MyInput ref={inputRef} />`는 React에게 해당 DOM 노드를 `inputRef.current`에 넣으라고 지시합니다. 그러나 이를 선택할지는 `MyInput` 컴포넌트에 달려 있으며, 기본적으로 그렇지 않습니다.
2. `MyInput` 컴포넌트를 `forwardRef`를 사용하여 선언하면, `props` 다음의 **두 번째 `ref` 인수**에 위의 `inputRef`를 받도록 설정됩니다.
3. `MyInput`은 수신한 `ref`를 내부의 `<input>`으로 전달합니다.

## **When React attaches the refs**

> **React가 ref를 첨부할 때**
> 

React에서 모든 업데이트는 아래 두 단계로 나뉜다.

- **렌더링** 하는 동안 React는 컴포넌트를 호출하여 화면에 무엇이 표시되어야 하는지 파악합니다.
- **커밋(commit)** 하는 동안 React는 DOM에 변경 사항을 적용합니다.

## **Best practices for DOM manipulation with refs**

> **ref를 이용한 DOM 조작 모범 사례**
> 

Ref는 탈출구로 ‘React 가 외부로 나가야’ 할때만 사용해야 한다. React가 노출하지 않는 브라우저 API를 호출하기도 한다.

포커스나 스크롤 같은 비파괴적 동작을 할 때는 문제가 없지만 DOM을 수동으로 수정하려고 한다면 React 가 수행하는 변경 사항과 충돌할 수 있다. → React가 관리하는 DOM 노드를 변경하지 말 것

단, **React가 업데이트할 *이유가 없는* DOM의 일부는 안전하게 수정할 수 있다.**

## **Recap**

> **요약**
> 
- Ref는 일반적인 개념이긴 하지만, 대부분 DOM 엘리먼트를 보관할 때 사용합니다.
- `<div ref={myRef}>`를 전달해 DOM 노드를 `myRef.current`에 넣으라고 React에 지시합니다.
- 보통은 포커스, 스크롤, DOM 엘리먼트 측정과 같은 비파괴적인 동작에 ref를 사용합니다.
- 컴포넌트는 기본적으로 DOM 노드를 노출하지 않습니다. `forwardRef`를 사용하고 두 번째 `ref` 인수를 특정 노드에 전달하여 DOM 노드를 노출하도록 설정할 수 있습니다.
- React가 관리하는 DOM 노드를 변경하지 마세요.
- React가 관리하는 DOM 노드를 수정해야 한다면 React가 업데이트할 이유가 없는 부분을 수정하세요.