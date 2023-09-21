# 6 - State 관리하기 (2)

# State 로직을 reducer로 작성하기

https://ko.react.dev/learn/extracting-state-logic-into-a-reducer

한 컴포넌트에서 state 업데이트가 여러 이벤트 핸들러로 분산되는 경우가 있다. 이런 경우 컴포넌트를 관리하기 어려워진다. 따라서 이 문제를 해결하기 위해 state를 업데이트하는 모든 로직을 reducer를 사용해 컴포넌트 외부의 단일 함수로 통합해 관리할 수 있다.

## reducer를 사용하여 state로직 통합하기

컴포넌트의 state가 업데이트되는 다양한 경우가 생기다보면 복잡해져 한눈에 알아보기가 어렵다. 아래의 예제에는 TaskApp 컴포넌트에서 tasks 라는 state 가 있고, 세가지의 이벤트 핸들러를 사용하여 tasks를 추가, 제거, 수정한다.

```jsx
import { useState } from "react";
import AddTask from "./AddTask.js";
import TaskList from "./TaskList.js";

export default function TaskApp() {
  const [tasks, setTasks] = useState(initialTasks);

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

  function handleChangeTask(task) {
    setTasks(
      tasks.map((t) => {
        if (t.id === task.id) {
          return task;
        } else {
          return t;
        }
      })
    );
  }

  function handleDeleteTask(taskId) {
    setTasks(tasks.filter((t) => t.id !== taskId));
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask onAddTask={handleAddTask} />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: "Visit Kafka Museum", done: true },
  { id: 1, text: "Watch a puppet show", done: false },
  { id: 2, text: "Lennon Wall pic", done: false },
];
```

각 이벤트핸들러는 state를 업데이트하기 위해 setTasks를 호출한다. 컴포넌트가 커질수록 그 안에서 state를 다루는 로직의 양도 늘어나게된다. 복잡성을 줄이고 접근성을 높이기 위해 컴포넌트 내부에있는 state 로직을 컴포넌트 외부의 reducer라고 하는 단일 함수로 옮길 수 있다.

- useState에서 useReducer로 바꾸는 단계
  1. state를 설정하는 것에서 action을 dispatch함수로 전달하는걸로 변경
  2. reducer함수 작성
  3. 컴포넌트에서 reducer 사용하기

## 1단계: state를 설정하는 것에서 action을 dispatch 함수로 전달하는걸로 변경하기

현재 이벤트 핸들러는 state를 설정함으로써 무엇을 할 것인지 명시한다.

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

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

위 코드에서 state 설정 관련 로직을 전부 지워보자. 그렇게하면 다음과 같이 세가지 이벤트 핸들러가 남게 된다.

- 사용자가 “Add” 를 눌렀을 때 호출되는 `handleAddTask(text)`
- 사용자가 task를 토글하거나 “저장”을 누르면 호출되는 `handleChangeTask(task)`
- 사용자가 “Delete”를 누르면 호출되는 `handleDeleteTask(taskId)`

reducer를 사용한 state 관리는 state를 직접 설정하는 것과 약간 다르다. state를 설정하여 React에게 ”무엇을 할지“를 지시하는 대신 이벤트 핸들러에서 ”action”을 전달하여 “사용자가 방금 한 일”을 지정한다. 즉, 이벤트핸들러를 통해 “task를 설정” 하는 대신 “task를 추가,변경,삭제“하는 action을 전달하는 것이다. 이러한 방식이 사용자의 의도를 더 명확하게 설명한다

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

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

`dispatch` 함수에 넣어준 객체를 “action”이라고 한다.

이 객체는 일반적인 자바스크립트 객체이다. 이 안에 어떤것이든 자유롭게 넣을 수 있지만, 일반적으로 어떤 상황이 발생하는지에 대한 최소한의 정보를 담고 있어야한다.

<aside>
💡 action 객체는 어떤 형태든 될 수 있다.
하지만 발생한 일을 설명하는 문자열 `type`을 넘겨주고 이외의 다른 정보는 다른 필드에 담아서 전달하도록 작성하는게 일반적이다. `type`은 컴포넌트에 따라 값이 다르다. 무슨 일이 일어나는지를 설명할 수 있는 이름을 넣어주면 된다.

</aside>

## 2단계: reducer 함수 작성하기

reducer 함수는 state에 대한 로직을 넣는 곳이다. 이 함수는 현재의 state 값과 action 객체 이렇게 두개의 인자를 받고 다음 state 값을 반환한다.

```jsx
function yourReducer(state, action) {
  // React가 설정하게될 다음 state 값을 반환한다.
}
```

위 예시에서 이벤트 핸들러에 구현되어있는 state 설정과 관련 로직을 reducer 함수로 옮기기 위해 다음과같이 진행해보자.

1. 첫 번째 인자에 현재 state(tasks) 선언하기
2. 두 번째 인자에 action 객체 선언하기
3. reducer에서 다음 state반환하기

다음은 state 설정과 관련된 모든 로직을 reducer 함수로 마이그레이션한 코드이다.

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

function handleChangeTask(task) {
  setTasks(
    tasks.map((t) => {
      if (t.id === task.id) {
        return task;
      } else {
        return t;
      }
    })
  );
}

function handleDeleteTask(taskId) {
  setTasks(tasks.filter((t) => t.id !== taskId));
}
```

reducer 함수는 state(tasks)를 인자로 받고 있기 때문에 이를 컴포넌트 위부에서 선언할 수 있다. 이렇게하면 들여쓰기 수준이 줄어들고 코드를 더 쉽게 읽을 수 있다.

<aside>
💡 위 예제에서는 if/else 문을 사용하고 있지만 일반적으로는 switch 문을 사용하는 것이 규칙이다. 물론 결과는 같지만 switch 문이 한눈에 읽기 더 쉽다.
각자 다른 case 속에서 선언된 변수들이 서로 충돌하지 않도록 case 블록을 중괄호로 감싸는 것을 추천한다. 또 case는 일반적인 경우라면 return으로 끝나야한다. return 하는 것을 잊으면 코드가 다음 case로 떨어져 실수할 수 있다.

</aside>

<aside>
💡 왜 reducer라고 부르게 되었을까?
배열의 메서드 reduce에서 따오게 되었다. reduce는 배열의 여러 값을 단일 값으로 “누적”하는 연산을 수행한다.
reduce로 전달하는 함수는 “reducer”로 알려져있다. 이 함수는 지금까지의 결과(result)와 현재 아이템(number)을 인자로 받고 다음 결과를 반환한다. 비슷한 아이디어의 예로 React의 reducer는 지금까지의 state와 action을 인자로 받고 다음 state를 반환한다. 이 과정에서 여러 action 을 누적하여 state를 반환한다.
initialState와 reducer 함수를 넘겨 받아 최종적인 state 값으로 계산하기위한 action 배열을 인자로 받은 reduce() 메서드를 사용할 수도 있다.

</aside>

```jsx
import tasksReducer from "./tasksReducer.js";

let initialState = [];
let actions = [
  { type: "added", id: 1, text: "Visit Kafka Museum" },
  { type: "added", id: 2, text: "Watch a puppet show" },
  { type: "deleted", id: 1 },
  { type: "added", id: 3, text: "Lennon Wall pic" },
];

let finalState = actions.reduce(tasksReducer, initialState);

const output = document.getElementById("output");
output.textContent = JSON.stringify(finalState, null, 2);
```

## 3단계: 컴포넌트에서 reducer 사용하기

마지막으로 컴포넌트에 tasksReducer를 연결할 차례이다. React에서 `useReducer` hook을 불러오자.

```jsx
import { useReducer } from "react";
```

그런 다음 useState를 useReducer로 바꿔보자

```jsx
const [tasks, setTasks] = useState(initialState);

const [tasks, dispatch] = useReducer(taskReducer, initialState);
```

useReducer 훅은 초기 state 값을 입력받아 유상태(stateful)값을 반환한다는 점과 state를 설정하는 함수(dispatch)의 원리를 보면 useState와 비슷하지만 조금 다른 점이 있다.

- useReducer 훅은 두 개의 인자를 넘겨받는다.
  (reducer 함수, 초기 state값)
- 그리고 아래와 같이 반환한다.
  (state를 담을 수 있는 값, ”사용자의 action 을 reducer 함수에게 전달하게 될“dispatch 함수)

reducer를 다른 파일로 분리하는 것도 가능하다.

관심사를 분리하면 컴포넌트의 로직은 읽기 더 쉬워질 것이다. 이렇게 하면 이벤트 핸들러는 action을 전달해줘서 무슨 일이 일어났는지에 관련한 것만 명시하면 되고 reducer 함수는 이에 대한 응답으로 state가 어떤 값으로 업데이트 될지를 결정하기만 하면 된다.

## useState와 useReducer 비교하기

useReducer가 좋은점만 있는것은 아니다. useState와 비교해보자

- 코드 크기
  (useState < useReducer)
- 가독성
  useState가 가독성은 좋으나 복잡해지는 경우 useReducer가 더 명확하게 구분할 수 있다.
- 디버깅
  useState를 사용하여 디버깅할 경우 어디서 잘못설정된건지 찾기 어려울 수 있다. useReducer의 경우 reducer에 콘솔로그를 추가하여 state가 업데이트 되는 모든 부분과 왜 버그가 발생했는지를 확인할 수 있다.
- 테스팅
  reducer는 컴포넌트에 의존하지 않는 순수함수이다. 이는 reducer를 독립적으로 분리해서 내보내거나 테스트할수 있음을 의미한다.
- 개인적인 취향
  reducer를 좋아하는 사람도 있지만 그렇지 않은 사람도 있다.

만약 일부 컴포넌트에서 잘못된 방식으로 state를 업데이트하는 것으로 인한 버그가 자주 발생하거나 해당 코드에 더 많은 구조를 도입하고 싶다면 reducer 사용을 권장한다. 이때 모두 reducer를 쓸 필요는 없다. useState와 적절히 매치하자.

## reducer 잘 작성하기

- reducer는 반드시 순수해야한다.
  - reducer는 렌더링 중에 실행된다.
    - 이는 reducer는 반드시 순수해야함을 의미한다.
  - 각 action은 데이터 안에서 여러 변경들이 있더라도 하나의 사용자와 상호작용을 설명해야한다.
    - 예를들어 사용자가 reducer에서 관리하는 5개의 필드가 있는 양식에서 ‘재설정’을 누른 경우 5개의 개별 action 보다는 하나의 action을 전송하는 것이 더 합리적이다.

---

# Context를 사용해 데이터를 깊게 전달하기

https://ko.react.dev/learn/passing-data-deeply-with-context

보통의 경우 부모 컴포넌트에서 자식 컴포넌트로 props를 통해 정보를 전달한다. 하지만 중간에 많은 컴포넌트를 거쳐야하거나 애플리케이션의 많은 컴포넌트에서 동일한 정보가 필요한 경우는 props를 전달하는 것이 번거롭고 불편할 수 있다. Context를 사용하면 명시적으로 props를 전달해주지 않아도 부모 컴포넌트가 트리에 있는 어떤 자식 컴포넌트에서나 정보를 사용할 수 있다.

## Props 전달하기의 문제점

prop을 트리를 통해 깊게 전해줘야하거나 많은 컴포넌트에서 같은 prop이 필요한 경우 장황하고 불편할 수 있다. 또한 데이터가 필요한 여러 컴포넌트의 가장 가까운 공통 조상이 트리상 높이 위치할 수 있어 그렇게 높게까지 끌어올리는 것은 “Prop drilling”이라는 상황을 초래할 수 있다.

데이터를 사용할 트리안의 컴포넌트에 props를 전달하는 대신 “순간이동” 시킬 방법은 React의 context를 사용하면 된다.

## Context: Props 전달하기의 대안

context는 부모 컴포넌트가 그 아래의 트리 전체에 데이터를 전달할 수 있도록 해준다.

- context 사용 방법
  1. Context를 생성
  2. 데이터가 필요한 컴포넌트에서 context를 사용
  3. 데이터를 지정하는 컴포넌트에서 context를 제공

### 1단계: Context 생성하기

```jsx
import { createContext } from "react";

export const LevelContext = createContext(1);
```

createContext 의 유일한 인자는 기본값이다. 여기서 1은 큰 제목 레벨을 나타내지만 모든 종류의 값(객체 등)을 전달할 수 있다.

### 2단계: Context 사용하기

React에서 useContext 훅과 생성한 context를 가져온다.

```jsx
import { useContext } from "react";
import { LevelContext } from "./LevelContext.js";
```

prop 대신에 가져온 LevelContext에서 값을 읽도록 하자.

```jsx
export default function Heading({ children }) {
  const level = useContext(LevelContext);
  // ...
}
```

useContext는 Hook이다. 훅은 리액트 컴포넌트의 바로 안에서만 호출할 수 있다.(반복문이나 조건문 내에선 안됨) useContext는 React에게 Heading 컴포넌트가 LevelContext를 읽으려한다고 알려준다.

이제 prop을 전달할 필요가 없다.

### 3단계: Context 제공하기

```jsx
import { LevelContext } from "./LevelContext.js";

export default function Section({ level, children }) {
  return (
    <section className="section">
      <LevelContext.Provider value={level}>{children}</LevelContext.Provider>
    </section>
  );
}
```

```jsx
import Heading from "./Heading.js";
import Section from "./Section.js";

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

이것은 리액트에게 Section 내의 어떤 컴포넌트가 LevelContext를 요구하면 level을 주라고 알려준다. 컴포넌트는 그 위에 있는 UI트리에서 가장 가까운 `<LevelContext.Provider>`의 값을 사용한다.

## 같은 컴포넌트에서 context를 사용하며 제공하기

지금까지는 각각의 섹션에서 level을 수동으로 지정해야한다.

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

Context를 통해 위의 컴포넌트에서 정보를 읽을 수 있으므로 각 Section은 위의 Section에서 level을 읽고 자동으로 level + 1 을 아래로 전달할 수 있다.

```jsx
import { useContext } from "react";
import { LevelContext } from "./LevelContext.js";

export default function Section({ children }) {
  const level = useContext(LevelContext);
  return (
    <section className="section">
      <LevelContext.Provider value={level + 1}>
        {children}
      </LevelContext.Provider>
    </section>
  );
}
```

이렇게 바꾸면 Section 과 Heading 모두에 level을 전달할 필요가 없다.

```jsx
import Heading from "./Heading.js";
import Section from "./Section.js";

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

<aside>
💡 이 예제는 하위 컴포넌트가 context를 오버라이드 할 수 있는 방법을 시각적으로 보여주기 때문에 제목 레벨을 사용한다. 하지만 context는 다른 많은 상황에서도 유용하다. 현재 색상테마, 로그인된 사용자 등 전체 하위 트리에 필요한 정보를 전달할 수 있다.

</aside>

## Context로 중간 컴포넌트 지나치기

context를 사용하면 “주변에 적응”하고 렌더링되는 위치(또는 어떤 context)에 따라 자신을 다르게 표시하는 컴포넌트를 작성할 수 있다.

Context의 작동 방식은 CSS 속성 상속을 연상시킨다. css에서 컬러를 지정할수 있고 중간에 있는 다른 노드가 다른 컬러로 재정의하지 않는 이상 그 안의 모든 dom노드가 그 색상을 상속한다. 마찬가지로 React에서 위에서 가져온 어떤 context를 재정의하는 유일한 방법은 다른 자식들을 다른 값을 가진 context provider로 래핑하는 것이다.

CSS에서 컬러나 백그라운드 컬러 처럼 다른 속성들은 서로 영향을 주지 않는다. 이처럼 서로 다른 React context에는 영향을 주지 않는다.

createContext()로 만든 각각의 context는 완벽히 분리되어 있고 특정 context를 사용 및 제공하는 컴포넌트끼리 묶여있다. 하나의 컴포넌트는 문제없이 많은 다른 context를 사용하거나 제공할 수 있다.

## Context를 사용하기 전에 고려할 것

context는 사용하기에 좋아보이지만 이는 또한 남용하기 쉽다는 의미기도 한다. 어떤 props를 여러 레벨 깊이 전달해야 한다고 해서 해당 정보를 context에 넣어야하는 것은 아니다.

다음은 context를 사용하기 전에 고려해볼 몇 가지 대안들이다.

1. Props 전달하기로 시작하기

   사소한 컴포넌트들이 아니라면 여러개의 props가 여러 컴포넌트를 거쳐가는 것은 이상한 일이 아니다.

2. 컴포넌트를 추출하고 JSX를 children으로 전달하기

   데이터를 사용하지 않는 많은 중간 컴포넌트 층을 통해 데이터를 전달하는 경우에는 컴포넌트 추출을 잊은 경우가 많다.

   ```jsx
   <Layout post={post} />

   // 대신
   <Layout>
   	<Posts post={post} />
   </Layout>
   ```

만약 이 접근 방법들이 잘 맞지 않다면 context를 고려해보자.

## Context 사용 예시

- 테마 지정하기 : 다크모드 등..
- 현재 계정 : 로그인한 사용자를 알아야하는 컴포넌트가 많을 수 있음.
- 라우팅 : 대부분의 라우팅 솔루션은 현재 경로를 유지하기 위해 내부적으로 context를 사용한다. 이것이 모든 링크의 활성화 여부를 “알수 있는” 방법이다.
- 상태 관리: 애플리케이션이 커지면서 결국 앱 상단에 수많은 state가 생긴다. 아래 멀리 떨어진 많은 컴포넌트가 그 값을 변경하고 싶어할 수 있다. 흔히 reducer를 context와 함께 사용하는 것은 복잡한 state를 관리하고 번거로운 작업 없이 멀리 있는 컴포넌트까지 값을 전달하는 방법이다.

Context는 정적인 값으로 제한되지 않는다. 다음 렌더링 시 다른 값을 준다면 React는 아래의 모든 컴포넌트에서 그 값을 갱신한다. 이것은 context와 state가 자주 조합되는 이유이다.

---

# Reducer와 Context로 앱 확장하기

https://ko.react.dev/learn/scaling-up-with-reducer-and-context

reducer와 context를 함께 사용하면 복잡한 화면의 state를 관리할 수 있다.

## Reducer와 context를 결합하기

1. Context 생성
2. State와 dispatch 함수를 context에 넣기
3. 트리 안에서 context를 사용

### 1단계: Context 생성

useReducer Hook은 현재 tasks와 업데이트할 수 있는 dispatch 함수를 반환한다

```jsx
const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
```

트리를 통해 전달하려면 두개의 별도의 context를 생성해야한다.

- TasksContext: 현재 tasks 리스트를 제공
- TaskDispatchContext: 컴포넌트에서 action을 dispatch 하는 함수를 제공

두 context는 나중에 다른 파일에서 가져올 수 있도록 별도의 파일에서 내보낸다.

```jsx
import { createContext } from "react";

export const TasksContext = createContext(null);
export const TasksDispatchContext = createContext(null);
```

### 2단계: State와 dispatch 함수를 context에 넣기

```jsx
import { TasksContext, TasksDispatchContext } from "./TasksContext.js";

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
  // ...
  return (
    <TasksContext.Provider value={tasks}>
      <TasksDispatchContext.Provider value={dispatch}>
        ...
      </TasksDispatchContext.Provider>
    </TasksContext.Provider>
  );
}
```

### 3단계: 트리 안에서 context 사용하기

이제 tasks 리스트나 이벤트 핸들러를 트리 아래로 전달할 필요가 없다

```jsx
<TasksContext.Provider value={tasks}>
  <TasksDispatchContext.Provider value={dispatch}>
    <h1>Day off in Kyoto</h1>
    <AddTask />
    <TaskList />
  </TasksDispatchContext.Provider>
</TasksContext.Provider>
```

대신 필요한 컴포넌트에서는 TaskContext에서 task 리스트를 읽을 수 있다.

```jsx
export default function TaskList() {
  const tasks = useContext(TasksContext);
  // ...
```

dispatch도 마찬가지

```jsx
export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  const dispatch = useContext(TasksDispatchContext);
  // ...
  return (
    // ...
    <button onClick={() => {
      setText('');
      dispatch({
        type: 'added',
        id: nextId++,
        text: text,
      });
    }}>Add</button>
    // ...
```

## 하나의 파일로 합치기

반드시 이런 방식으로 작성하지 않아도 되지만 reducer와 context를 모두 하나의 파일에 작성하면 컴포넌트들을 조금 더 정리할 수 있다.

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

```jsx
import AddTask from "./AddTask.js";
import TaskList from "./TaskList.js";
import { TasksProvider } from "./TasksContext.js";

export default function TaskApp() {
  return (
    <TasksProvider>
      <h1>Day off in Kyoto</h1>
      <AddTask />
      <TaskList />
    </TasksProvider>
  );
}
```

또한 context를 사용하기 위한 use 함수들도 내보낼 수 있다.

```jsx
export function useTasks() {
  return useContext(TasksContext);
}

export function useTasksDispatch() {
  return useContext(TasksDispatchContext);
}
```

```jsx
const tasks = useTasks();
const dispatch = useTasksDispatch();
```

이렇게 하면 동작이 바뀌는건 아니지만 다음에 context를 더 분리하거나 함수에 로직을 추가하기 쉬워진다.
