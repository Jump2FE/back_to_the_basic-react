> [6회] State 관리하기 - 2, 탈출구 - 1

> 링크: https://bronzed-icicle-65c.notion.site/6-State-2-1-fb11ef0747fd4e17a97ab7ea6a073653?pvs=4

(이어서)

## State 로직을 reducer로 추출하기

- 여러 state 업데이트가 여러 이벤트 핸들러에 분산되 있으면 컴포넌트는 과부하에 걸릴 수 있다.
  - 이를 `reducer`라는 단일 함수를 통해 컴포넌트 외부의 모든 state 업데이트 로직을 통합 가능하다

**useState에서 useReducer로 마이그레이션하는 절차**

1. state를 설정하는 것에서 action들을 전달하는 것으로 변경
   - `state`를 설정해서 무엇을 할 지를 지시하는 게 아닌 **어떤 `action`을 전달**
   - 예시
     ```tsx
     // 1. 이벤트 핸들러만 남기고 다 없다고 생각하자
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

     // 2. dispatch 함수로 액션 형태를 통일해준다
     function handleAddTask(text) {
       dispatch({
         // dispatch안에 들어가는 **객체** === action
         type: "added",
         id: nextId++,
         text: text,
       });
     }

     function handleChangeTask(task) {
       dispatch({
         type: "changed",
         task: task,
       });
     }

     function handleDeleteTask(taskId) {
       dispatch({
         type: "deleted",
         id: taskId,
       });
     }
     ```
2. **reducer** 함수 작성
   - 현재 state와 action객체를 순서대로 받는 reducer 함수 / 반환은 다음 state를 반환한다
   - 이벤트 핸들러 ⇒ reducer 함수로 옮기기
     1. 현재의 state(`tasks`)를 첫 번째 매개변수로 선언
     2. `action` 객체를 두 번째 매개변수로 선언
     3. *다음* state를 reducer 함수에서 반환하도록 구조 작성
   - 이럴 경우 **state를 매개변수로 전달 받기** 때문에 **reducer 함수를 컴포넌트 밖에 선언할 수 있다.**
   - 예시
     ```tsx
     function tasksReducer(tasks, action) {
       if (action.type === "added") {
         return [
           ...tasks,
           {
             id: action.id,
             text: action.text,
             done: false,
           },
         ];
       } else if (action.type === "changed") {
         return tasks.map((t) => {
           if (t.id === action.task.id) {
             return action.task;
           } else {
             return t;
           }
         });
       } else if (action.type === "deleted") {
         return tasks.filter((t) => t.id !== action.id);
       } else {
         throw Error("Unknown action: " + action.type);
       }
     }
     ```
3. 컴포넌트에서 **reducer 사용**
   - 예시
     ```tsx
     import { useReducer } from "react";
     // const [tasks, setTasks] = useState(initialTasks);

     // tasksReducer = reducer 함수
     // initialTasks = 초기 state
     const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
     // tasks = state 값
     // dispatch = dispatch 함수 (사용자의 action객체를 reducer에게 전달)
     ```
   - 예시2
     ```tsx
     import { useReducer } from 'react';
     import AddTask from './AddTask.js';
     import TaskList from './TaskList.js';

     export default function TaskApp() {
       const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);

       function handleAddTask(text) {
         **dispatch**({
           type: 'added',
           id: nextId++,
           text: text,
         });
       }

       function handleChangeTask(task) {
         dispatch({
           type: 'changed',
           task: task,
         });
       }

       function handleDeleteTask(taskId) {
         dispatch({
           type: 'deleted',
           id: taskId,
         });
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

     let nextId = 3;
     const initialTasks = [
       {id: 0, text: 'Visit Kafka Museum', done: true},
       {id: 1, text: 'Watch a puppet show', done: false},
       {id: 2, text: 'Lennon Wall pic', done: false},
     ];
     ```

- **Note**
  - Action 객체는 어떤 형태든 될 수 있다
    - But, Action별로 구분해주는 문자열 타입의 `type`과 추가적인 정보는 다른 필드를 통해 전달하도록 작성하는 게 일반적
    - `type`은 컴포넌트를 따라가는게 일반적
      (무슨 일이 일어났는지를 알려주는게 중요!)
    ```tsx
    dispatch({
      // specific to component
      type: "what_happened",
      // other fields go here
    });
    ```
- **Note2**
  - if / else를 써도 무방하지만 switch 구문을 쓰면 더 직관적으로 reducer 구조를 파악할 수 있다
  - 추가로 case마다 `{}` 로 감싸서 스코프를 지정해주는게 좋다.
  - 예시
    ```tsx
    function tasksReducer(tasks, action) {
      switch (action.type) {
        case "added": {
          return [
            ...tasks,
            {
              id: action.id,
              text: action.text,
              done: false,
            },
          ];
        }
        case "changed": {
          return tasks.map((t) => {
            if (t.id === action.task.id) {
              return action.task;
            } else {
              return t;
            }
          });
        }
        case "deleted": {
          return tasks.filter((t) => t.id !== action.id);
        }
        default: {
          throw Error("Unknown action: " + action.type);
        }
      }
    }
    ```
- **왜 reducer라 부르는가**
  - reducer들이 비록 컴포넌트 안에 있는 코드의 양을 **“줄여주긴”** 하지만, 이건 사실 배열에서 사용하는 `[reduce()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)` 연산을 따서 지은 것
  - `reduce()` 연산은 배열을 가지고 많은 값들을 하나의 값으로 **“누적”**할 수 있습니다.
  - 예제
    ```tsx
    const arr = [1, 2, 3, 4, 5];
    const sum = arr.reduce((result, number) => result + number); // 1 + 2 + 3 + 4 + 5
    // 여기서 reduce에 넘기는 함수가 **reducer** 라 생각하면 된다.
    ```

> **reducer를 사용해 관심사(업데이트 관련)를 분리** ⇒ 컴포넌트 로직을 더 쉽게 읽을 수 있음

### useState vs useReducer

- **코드 크기**
  일반적으로 `useState`를 사용하면 미리 작성해야 하는 코드가 줄어듭니다. `useReducer`를 사용하면 reducer 함수 *와* action을 전달하는 부분 모두 작성해야 합니다. 하지만 많은 이벤트 핸들러가 비슷한 방식으로 state를 업데이트하는 경우 `useReducer`를 사용하면 코드를 줄이는 데 도움이 될 수 있습니다.
- **가독성**
  `useState`로 간단한 state를 업데이트 하는 경우 가독성이 좋습니다. 그렇지만 state의 구조가 더욱 복잡해지면, 컴포넌트의 코드의 양이 부풀어 오르고 한눈에 읽기 어려워질 수 있습니다. 이 경우 `useReducer`를 사용하면 업데이트 로직이 *어떻게 동작* 하는지와 이벤트 핸들러를 통해 *무엇이 일어났는지* 를 깔끔하게 분리할 수 있습니다.
- **디버깅**
  `useState`에 버그가 있는 경우, state가 *어디서* 잘못 설정되었는지, 그리고 *왜 그런지* 알기 어려울 수 있습니다. `useReducer`를 사용하면, reducer에 콘솔 로그를 추가하여 모든 state 업데이트와 *왜* (어떤 action으로 인해) 버그가 발생했는지 확인할 수 있습니다. 각 `action`이 정확하다면, 버그가 reducer 로직 자체에 있다는 것을 알 수 있습니다. 하지만 `useState`를 사용할 때보다 더 많은 코드를 살펴봐야 합니다.
- **테스팅**
  reducer는 컴포넌트에 의존하지 않는 **순수 함수**입니다. 즉, 별도로 분리해서 내보내거나 테스트할 수 있습니다. 일반적으로 보다 현실적인 환경에서 컴포넌트를 테스트하는 것이 가장 좋지만, 복잡한 state 업데이트 로직의 경우 reducer가 특정 초기 state와 action에 대해 특정 state를 반환한다고 단언하는 것이 유용할 수 있습니다.
- **개인 취향**
  어떤 사람은 reducer를 좋아하고 어떤 사람은 싫어합니다. 괜찮습니다. 취향의 문제니까요. `useState` 와 `useReducer`는 언제든지 앞뒤로 변환할 수 있으며, 서로 동등합니다!

일부 컴포넌트에서 잘못된 state 업데이트로 인해 버그가 자주 발생하고 코드에 더 많은 구조를 도입하려는 경우 reducer를 사용하는 것이 좋습니다. 모든 컴포넌트에 reducer를 사용할 필요는 없으니 자유롭게 섞어서 사용하세요! 심지어 같은 컴포넌트에서 `useState`와 `useReducer`를 함께 사용할 수도 있습니다.

### reducer 잘 작성하기

- **reducer는 반드시 순수해야 한다.**
  - [state 설정 함수](https://react-ko.dev/learn/queueing-a-series-of-state-updates)와 비슷하게, reducer는 렌더링 중에 실행됩니다! (action은 다음 렌더링까지 대기합니다.)
  - 즉, 입력 값이 같다면 결과 값도 항상 같아야 합니다.
  - 요청을 보내거나 timeout을 스케쥴링하거나 사이드 이펙트(컴포넌트 외부에 영향을 미치는 작업)을 수행해서는 안 됩니다.
  - reducer는 [객체](https://react-ko.dev/learn/updating-objects-in-state) 및 [배열](https://react-ko.dev/learn/updating-arrays-in-state)을 변이 없이 업데이트해야 합니다.
- **각 action은 여러 데이터가 변경되더라도, 하나의 사용자 상호작용을 설명해야 한다**
  예를 들어, 사용자가 reducer가 관리하는 5개의 필드가 있는 양식에서 ‘재설정’을 누른 경우, 5개의 개별 `set_field action`보다는 하나의 `reset_form action`을 전송하는 것이 더 합리적입니다. 모든 action을 reducer에 기록하면 어떤 상호작용이나 응답이 어떤 순서로 일어났는지 재구성할 수 있을 만큼 로그가 명확해야 합니다. 이는 디버깅에 도움이 됩니다!

### Immer를 사용하여 간결한 reducer 작성

(생략)

### 요약

- `useState`에서 `useReducer`로 변환하려면:
  1. 이벤트 핸들러에서 action(객체)를 전달한다.
  2. 주어진 state와 action에 대해 다음 state를 반환하는 reducer 함수를 작성한다
  3. `useState`를 `useReducer`로 바꾼다.
- reducer를 사용하면
  - 단점: 코드를 조금 더 작성해야 하지만
  - 장점: 디버깅과 테스트에 도움이 된다
- reducer는 **반드시 순수해야 한다**.
- 각 action은 **단일 사용자 상호작용**을 설명해야 한다.
- 변이(mutation) 스타일로 reducer를 작성하려면 Immer를 사용하세요.

---

## Context로 데이터 깊숙이 전달하기

- 일반적으로 부모 컴포넌트에서 자식 컴포넌트로 props를 통해 정보를 전달한다
  - But, 중간에 여러 컴포넌트를 거쳐야 하거나 여러 컴포넌트가 동일한 정보를 필요로 하는 경우
    ⇒ props를 전달하면 장황하고 불편해질 수 있다.
- **_Context_**를 사용하면
  - 부모 컴포넌트가 props를 통해 명시적으로 전달하지 않아도 됨
  - 깊이 여부와 무관하게 context 영역 아래의 모든 컴포넌트에서 일부 정보를 사용할 수 있다.

### Props 전달 문제

- [props 전달](https://react-ko.dev/learn/passing-props-to-a-component)은 UI 트리를 통해 데이터를 사용하는 컴포넌트로 명시적으로 연결할 수 있는 좋은 방법
  - But, 트리 깊숙이 prop을 전달해야 하거나 많은 컴포넌트에 동일한 prop이 필요한 경우 prop 전달이 장황하고 불편해질 수 있다 (**Props Drilling**)
  ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/693bc352-6af5-47d2-866f-3108bb9e50f3/44f8ccba-f43a-4c27-926c-8f7760843e05/Untitled.png)
- Props 전달없이 필요한 컴포넌트에서만 데이터를 사용하는 방법 ⇒ **Context**

**Context 적용하는 과정**

1. context를 **생성한**다.
   - `export const LevelContext = createContext(1);`
   - 예시에서 제목 level을 위한 것이므로 `LevelContext`라고 부를 수 있습니다
2. 데이터가 필요한 컴포넌트에서 해당 context를 **사용한다 ⇒ useContext**
   - `const level = useContext(LevelContext);`
   - `Heading`은 `LevelContext`를 사용한다.
3. 데이터를 지정하는 컴포넌트에서 해당 context를 **제공한다**. ⇒ **Context.Provider** 적용

   ```tsx
   <section className="section">
     **
     <LevelContext.Provider value={level}>
       **
       {children}
       **
     </LevelContext.Provider>
     **
   </section>
   ```

   - `Section`은 `LevelContext`를 제공한다

- **Note**
  - 이 예제에서는 중첩된 컴포넌트가 context를 재정의하는 방법을 시각적으로 보여주기 위해 제목 level을 사용한다.
  - 하지만 context는 다른 많은 사용 사례에도 유용
    - **현재 색상 테마**, **현재 로그인한 사용자** 등 전체 하위 트리에 필요한 모든 정보를 전달할 수 있다.

### \***\*Context는 중간 컴포넌트들을 통과한다\*\***

- context를 제공하는 컴포넌트와 context를 사용하는 컴포넌트 사이에 원하는 만큼의 컴포넌트를 삽입할 수 있다.
  - `<div>`와 같은 기본 제공 컴포넌트와 사용자가 직접 빌드할 수 있는 컴포넌트가 모두 포함
- Context를 사용하면 **“주변 환경에 적응”**하고 렌더링되는 _위치_ (_context_)에 따라 다르게 표시되는 **컴포넌트를 작성할 수 있다**
- context가 작동하는 방식은 [CSS 속성 상속](https://developer.mozilla.org/en-US/docs/Web/CSS/inheritance)을 떠올리게 할 수 있다.
  - CSS에서는 `<div>`에 `color: blue`을 지정할 수 있으며, 중간에 다른 DOM 노드가 `color: green`으로 재정의하지 않는 한 그 안에 있는 모든 DOM 노드는 아무리 깊어도 그 색을 상속받는다.
  - 마찬가지로 React에서 위에서 오는 context를 재정의하는 유일한 방법은 **children을 다른 값으로 context provider로 감싸는 것**.
- CSS에서는 `color` 및 `background-color`와 같은 서로 다른 속성이 서로 재정의되지 않는다. `background-color`에 영향을 주지 않고 모든 `<div>`의 `color`를 빨간색으로 설정할 수 있습니다.
  - 마찬가지로 서로 다른 React context도 서로 재정의하지 않는다
  - `createContext()`로 만드는 각 context는 다른 context와 완전히 분리되어 있으며, _특정_ context를 사용하거나 제공하는 컴포넌트를 함께 묶습니다. 하나의 컴포넌트가 문제없이 다양한 context를 사용하거나 제공할 수 있다

<aside>
💡

</aside>

### context 사용하기 전

1. **Props 전달로 시작하자**

   컴포넌트가 사소하지 않다면, 수십 개의 props를 수십 개의 컴포넌트에 전달해야 하는 경우가 드물지 않습니다. 지루하게 느껴질 수도 있지만, 어떤 컴포넌트가 어떤 데이터를 사용하는지 매우 명확해집니다! 코드를 유지 관리하는 사람은 props를 사용하여 데이터 흐름을 명확하게 만든 것에 만족할 것입니다.

2. **컴포넌트를 추출하고 JSX를 children으로 전달하자**

   일부 데이터를 해당 데이터를 사용하지 않는 중간 컴포넌트의 여러 레이어를 거쳐 전달한다면(그리고 더 아래로만 전달한다면), 이는 종종 그 과정에서 일부 컴포넌트를 추출하는 것을 잊었다는 것을 의미합니다. 예를 들어, `posts`과 같은 데이터 props를 직접 사용하지 않는 시각적 컴포넌트에 `<Layout posts={posts} />` 와 같은 방법 대신, Layout이 children을 prop으로 사용하도록 만들고 `<Layout><Posts posts={posts} /></Layout>`를 렌더링합니다. 이렇게 하면 데이터를 지정하는 컴포넌트와 데이터를 필요로 하는 컴포넌트 사이의 레이어 수가 줄어듭니다.

### Context 적용 사례

- theme 적용
  - 다크모드가 대표적인 적용 사례
- 로그인한 계정 확인
  - 앱 어디서든 로그인한 유저가 누구인지 확인 가능
- 라우팅 (💡)
  - 대부분의 라우팅 솔루션은 내부적으로 context를 사용하여 현재 경로를 유지
- 전역 state 관리

  - 많이 떨어져있는 컴포넌트에서 state를 변경하고 싶을 때 사용
  - context + reducer를 사용해서 복잡한 state 관리가 가능

- Context는 정적 값에만 국한되지 않는다.
  - 다음 렌더링에서 다른 값을 전달하면 React는 아래에서 이를 읽는 모든 컴포넌트를 업데이트하고 이것이 context가 state와 함께 자주 사용되는 이유
  - 일반적으로 트리의 다른 부분에 있는 멀리 떨어진 컴포넌트에서 일부 정보가 필요한 경우, context가 도움이 될 수 있다

### 요약

- Context는 컴포넌트가 그 아래 전체 트리에 일부 정보를 제공할 수 있도록 한다
- context를 전달하려면:
  1. `export const MyContext = createContext(defaultValue)`를 사용하여 context를 생성하고 export.
  2. `useContext(MyContext)` 훅에 전달하여 깊이에 상관없이 모든 하위 컴포넌트에서 읽을 수 있도록 한다.
  3. 자식 컴포넌트를 `<MyContext.Provider value={...}>`로 감싸서 부모로부터 제공 받는다.
- Context는 중간에 있는 모든 컴포넌트를 통과한다.
- Context를 사용하면 “주변 환경에 적응”하는 컴포넌트를 작성할 수 있다.
- context를 사용하기 전에 props를 전달하거나 JSX를 `children`으로 전달

---

## Reducer와 context로 확장하기

- Reducer를 사용하면 **컴포넌트의 state 업데이트 로직을 통합**할 수 있다.
- Context를 사용하면 **다른 컴포넌트들에 정보를 전달**할 수 있다.

  ⇒ Reducer와 context를 함께 사용하여 복잡한 화면의 state를 관리할 수 있다.

- Reducer를 사용하면 이벤트 핸들러를 간결하고 명확하게 만들 수 있다.

  - But, 앱이 커질수록 다른 어려움에 부딪힐 수 있습니다.
  - **현재 `tasks` state와 `dispatch` 함수는 최상위 컴포넌트인 `TaskBoard`에서만 사용할 수 있습니다.**
  - 다른 컴포넌트들에서 tasks의 리스트를 읽고 변경하려면 prop을 통해 현재 state와 state를 변경할 수 있는 이벤트 핸들러를 명시적으로 [전달](https://react-ko.dev/learn/passing-props-to-a-component)해야 합니다.
  - 예시
    ```tsx
    <TaskList
      tasks={tasks}
      onChangeTask={handleChangeTask}
      onDeleteTask={handleDeleteTask}
    />
    ```

- 그래서 `tasks` state와 `dispatch` 함수를 props를 통해 전달하는 대신 [context에 넣어서 사용](https://react-ko.dev/learn/passing-data-deeply-with-context)
  - 반복적인 **“prop drilling” 없이 `TaskBoard` 아래의 모든 컴포넌트 트리에서 tasks를 읽고 dispatch 함수를 실행할 수 있다.**

**reducer와 context 결합하는 과정**

1. Context를 **생성한다**.

   - 별도의 파일에서 context 생성

   ```tsx
   export const TasksContext = createContext(null);
   export const TasksDispatchContext = createContext(null);
   ```

2. State과 dispatch 함수를 context에 **넣는다**.

   ```tsx
   // useReducer로 state와 dispatch를 받고
   // 이를 각 context의 value로 전달해준다
   const [tasks, dispatch] = useReducer(tasksReducer, initialTasks);
   // ...
   return (
     <TasksContext.Provider value={tasks}>
       <TasksDispatchContext.Provider value={dispatch}>
         {children}
       </TasksDispatchContext.Provider>
     </TasksContext.Provider>
   );
   ```

3. 트리 안에서 context를 **사용한다**.

   - useContext를 사용해 사용하고자 하는 값을 불러온다

   ```tsx
   // state를 가져다 사용하는 컴포넌트
   export default function TaskList() {
     const tasks = useContext(TasksContext);
     // ...

   //// dispatch를 가져와 사용하는 컴포넌트
   export default function AddTask() {
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
   ```

### 하나의 파일로 합치기

- Reducer를 같은 파일로 옮기고 `TasksProvider` 컴포넌트를 새로 선언.

  - `TasksProvider`는 모든 것을 하나로 묶는 역할을 하게 됩니다
    ```tsx
    import { createContext, useReducer } from "react";

    export const TasksContext = createContext(null);
    export const TasksDispatchContext = createContext(null);

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

    function tasksReducer(tasks, action) {
      switch (action.type) {
        case "added": {
          return [
            ...tasks,
            {
              id: action.id,
              text: action.text,
              done: false,
            },
          ];
        }
        case "changed": {
          return tasks.map((t) => {
            if (t.id === action.task.id) {
              return action.task;
            } else {
              return t;
            }
          });
        }
        case "deleted": {
          return tasks.filter((t) => t.id !== action.id);
        }
        default: {
          throw Error("Unknown action: " + action.type);
        }
      }
    }

    const initialTasks = [
      { id: 0, text: "Philosopher’s Path", done: true },
      { id: 1, text: "Visit the temple", done: false },
      { id: 2, text: "Drink matcha", done: false },
    ];
    ```
  - 별도의 use함수들로 명시해서 context를 내보낼 수 있음
    ```tsx
    export function useTasks() {
      return useContext(TasksContext);
    }

    export function useTasksDispatch() {
      return useContext(TasksDispatchContext);
    }
    ```

- **note**
  - `useTasks`와 `useTasksDispatch` 같은 함수들을 *[커스텀 훅](https://react-ko.dev/learn/reusing-logic-with-custom-hooks)* 이라고 한다.
  - 커스텀 훅 안에서도 `useContext` 등 다른 Hook을 사용할 수 있습니다.

### 요약

- Reducer와 context를 결합해서 컴포넌트가 상위 state를 읽고 수정할 수 있도록 할 수 있다.
- State와 dispatch 함수를 자식 컴포넌트들에 제공하는 방법
  1. 두 개의 context를 만든다. (각각 state와 dispatch 함수를 위한 것).
  2. 하위 컴포넌트들에서 필요한 context를 사용한다.
  3. 하위 컴포넌트들에서 필요한 context를 사용한다.
- 더 나아가 하나의 파일로 합쳐서 컴포넌트들을 정리할 수 있다.
  - Context를 제공하는 `TasksProvider` 같은 컴포넌트를 내보낼 수 있다.
  - 바로 사용할 수 있도록 `useTasks`와 `useTasksDispatch` 같은 사용자 Hook을 내보낼 수 있다.
- context-reducer 조합을 앱에 여러 개 만들 수 있다.

---

# 탈출구

## Ref로 값 참조하기

- 컴포넌트가 특정 정보를 **‘기억’**하도록 하고 싶지만 해당 정보가 [새 렌더링을 촉발](https://react-ko.dev/learn/render-and-commit)하지 않도록 하려는 경우 **_ref_**를 사용할 수 있다.

### 컴포넌트에 ref 추가하기

```tsx
// 1. useRef 가져오기
import { useRef } from "react";

// 2. useRef 초기화 in Component
const ref = useRef(0);
/*
	{
		current = 0;
	}
*/
// heap <= 여기에 저장되서 컴포넌트가 리렌더링 되어도 값이 유지된다
```

- `ref.current` 로 해당 **ref의 현재 값에 액세스**할 수 있다.

  - 이 값은 의도적으로 **변이(mutation)** 가능 (읽기와 쓰기가 모두 가능)
    - React가 추적하지 않는 컴포넌트의 비밀 주머니와 같습니다.
      (React의 단방향 데이터 흐름에서 “탈출구”가 되는 이유)

- 단순 카운팅 예제

  ```tsx
  import { useRef } from "react";

  export default function Counter() {
    let ref = useRef(0);

    function handleClick() {
      ref.current = ref.current + 1;
      alert("You clicked " + ref.current + " times!");
      // alert을 통해 ref.current가 클릭할 때마다 +1씩 증가하고 있다
    }

    return <button onClick={handleClick}>Click me!</button>;
  }
  ```

  - P.S.) 여기서 ref는 숫자를 가리키지만 `state` 처럼 문자열, 객체, 함수 등 뭐든 가리킬 수 있다.
    - state와 달리 `current` 을 갖고있는 **일반 JS 객체(mutable)**다
  - P.S. 2) **ref가 증가할 때마다 컴포넌트가 리렌더링되지 않는다**는 점에 유의하자!
    - 컴포넌트가 리렌더링되진 않지만 값은 **React에 의해 유지(**💡)됨

- 스탑워치 예제
  ```tsx
  import { useState, useRef } from "react";

  export default function Stopwatch() {
    const [startTime, setStartTime] = useState(null);
    const [now, setNow] = useState(null);
    const intervalRef = useRef(null);

    function handleStart() {
      setStartTime(Date.now());
      setNow(Date.now());

      clearInterval(intervalRef.current);
      // setInterval의 반환인 interval ID는 렌더링에 사용되지 않음
      // 고로, ref에 보관 가능!
      // let을 안쓴 이유는 값을 유지하기 위해!
      intervalRef.current = setInterval(() => {
        setNow(Date.now());
      }, 10);
    }

    function handleStop() {
      clearInterval(intervalRef.current);
    }

    let secondsPassed = 0;
    if (startTime != null && now != null) {
      secondsPassed = (now - startTime) / 1000;
    }

    return (
      <>
        <h1>Time passed: {secondsPassed.toFixed(3)}</h1>
        <button onClick={handleStart}>Start</button>
        <button onClick={handleStop}>Stop</button>
      </>
    );
  }
  ```
  - 렌더링에 사용되는 정보라면
    ⇒`state`로 유지하는게 좋다
  - 이벤트 핸들러만 정보를 필요로 하고 변경해도 리렌더링 필요없다면
    ⇒ ref를 사용하는 것이 더 효율적일 수 있다.

### ref와 state의 차이점

| ref                                                            | state                                                                                     |
| -------------------------------------------------------------- | ----------------------------------------------------------------------------------------- |
| useRef(initialValue)는 { current: initialValue }을 반환 (객체) | useState(initialValue)는 state 변수의 현재값과 state 설정자함수([value, setValue])를 반환 |
| 변경 시 리렌더링을 촉발하지 않음                               | 변경 시 리렌더링을 촉발함                                                                 |

| Mutable
: 렌더링 프로세스 외부에서 current 값을 수정하고 업데이트할 수 있음 | Immutable
: state setting 함수를 사용하여 state 변수를 수정해 리렌더링을 대기열에 추가해야함 |
| 렌더링 중에는 current 값을 읽거나 쓰지 않아야 함 | 언제든지 state를 읽을 수 있음.
각 렌더링에는 변경되지 않는 자체 state https://react-ko.dev/learn/state-as-a-snapshot이 있음 |

- 예시) ref로 카운팅하려면 안됨

  ```tsx
  import { useRef } from "react";

  export default function Counter() {
    let countRef = useRef(0);

    function handleClick() {
      // This doesn't re-render the component!
      countRef.current = countRef.current + 1;
    }

    return (
      <button onClick={handleClick}>
        You clicked {countRef.current} times
      </button>
    );
  }
  ```

  - `count` 값이 표시되므로 state 값을 사용하는 것이 맞다
    - 카운터 값이 `setCount()`로 설정되면 React는 컴포넌트를 다시 렌더링하고 화면이 새로운 카운트를 반영하도록 업데이트한다
  - 만약 이것을 ref로 구현하려고 한다면,
    - React는 **컴포넌트를 다시 렌더링하지 않으**므로 카운트가 변경되는 것을 볼 수 없다!
    - 이것이 **렌더링 중에 ref.current를 읽으면 코드가 불안정해지는 이유**

- **ref는 내부에서 어떻게 작동하는가?**
  - `useState`와 `useRef`는 모두 React에서 제공
    - But, 원칙적으로 `useRef`는 `useState` 위에 구현될 수 있다.
    - React 내부에서 `useRef`는 다음과 같이 구현된다고 볼 수 있다
    ```tsx
    function useRef(initialValue) {
      const [ref, unused] = useState({ current: initialValue });
      return ref; // 첫번째 인자만 반환
    }

    // 혹은
    const [ref, _] = useState({ current: initialValue });
    ```
  - 첫 번째 렌더링 중에 `useRef` 는 `{ current: initialValue }`를 반환한다.
    - 이 객체는 React에 의해 저장되므로 다음 렌더링 중에 동일한 객체가 반환된다.
    - state setter가 어떻게 사용되지 않는지 기억하자.
      - `useRef`는 항상 동일한 객체를 반환해야 하기 때문에 setter는 불필요!
  - React는 충분히 일반적인 상황이라 판단하고 빌트인 버전의 `useRef`를 제공합니다. ref를 설정자(setter)가 없는 일반 state 변수라고 생각하면 된다.
  - 객체지향 프로그래밍에 익숙하다면 인스턴스 필드를 떠올릴 수 있는데, `this.something` 대신 `somethingRef.current`를 사용하면 된다.

### ref를 사용해야 하는 경우

- 일반적으로 ref는 **컴포넌트의 형상에 영향을 주지 않는 브라우저 API 등과 통신**해야 할 때 주로 사용
  1. [timeout ID](https://developer.mozilla.org/docs/Web/API/setTimeout) 저장
  2. [다음 페이지](https://react-ko.dev/learn/manipulating-the-dom-with-refs)에서 다룰 [DOM elements](https://developer.mozilla.org/docs/Web/API/Element) 저장 및 조작
  3. JSX를 계산하는 데 필요하지 않은 다른 객체 저장

### ref 사례

- **ref를 탈출구로 취급하자**.
  - ref는 외부 시스템이나 브라우저 API로 작업할 때 유용하다.
  - 애플리케이션 로직과 데이터 흐름의 대부분이 ref에 의존하는 경우 접근 방식을 재고해봐야 할 수도 있다.
- **렌더링 중에는 `ref.current`를 읽거나 쓰지 마세요. (**💡)
  - 렌더링 중에 일부 정보가 필요한 경우, 대신 [state](https://react-ko.dev/learn/state-a-components-memory)를 사용하세요.
  - React는 `ref.current`가 언제 변경되는지 알지 못하기 때문에, 렌더링 중에 읽어도 컴포넌트의 동작을 예측하기 어렵다.
    (유일한 예외는 첫 번째 렌더링 중에 ref를 한 번만 설정하는 `if (!ref.current) ref.current = new Thing()`과 같은 코드).

React state의 제한은 ref에는 적용되지 않는다.

- 예를 들어, state는 [모든 렌더링에 대해 스냅샷](https://react-ko.dev/learn/state-as-a-snapshot)처럼 작동하며 [동기적으로 업데이트](https://react-ko.dev/learn/queueing-a-series-of-state-updates)되지는 않는다.
- 하지만 ref의 현재 값을 변이하면 즉시 변경이 된다

```tsx
ref.current = 5;
console.log(ref.current); // 5
```

- 이는 **ref 자체가 일반 JavaScript 객체이므로** JavaScript 객체처럼 동작하기 때문
- 또한 ref로 작업할 때 [변이를 피하고자](https://react-ko.dev/learn/updating-objects-in-state) 조심할 필요가 없다.
  - 변이하는 객체가 렌더링에 사용되지 않는 한, React는 ref나 그 콘텐츠로 무엇을 하든 상관하지 않다 (React의 불변성 원칙에 위배되지 않음!)

### \***\*Ref와 DOM\*\***

- ref는 모든 값을 가리킬 수 있습니다.
  - But, ref의 가장 일반적인 사용 사례는 DOM 요소에 액세스하는 것이다.
  - 프로그래밍 방식으로 input에 focus를 맞추고자 할 때 유용
  - `<div ref={myRef}>`와 같이 JSX의 `ref` 어트리뷰트에 ref를 전달하면 React는 해당 DOM 엘리먼트를 `myRef.current`에 넣습니다.

### 요약

- ref는 렌더링에 사용되지 않는 값을 유지하기 위한 탈출구이다.
  - 자주 필요하지 않다.
- ref는 `current`라는 단일 프로퍼티를 가진 일반 JavaScript 객체로, 읽거나 설정할 수 있다.
- `useRef` 훅을 호출하여 React에 ref를 제공하도록 요청할 수 있다.
- state와 마찬가지로 ref를 사용하면
  - **컴포넌트의 리렌더링 사이에 정보를 유지할 수 있다**.
- state와 달리 ref의 `current` 값을 설정해도 리렌더링이 촉발되지 않는다.
- 렌더링 중에는 `ref.current`를 읽거나 쓰지 마라.
  - 컴포넌트를 예측하기 어렵다.

---

- **이전 세션 요약**
  - ref는 **리렌더링은 발생시키지 않으면서 값은 유지**시켜주는 특징이 있다.
  - ref는 current를 속성으로 가지고 있는 변이가 가능한 일반 JS 객체다.
  - useRef를 사용해 ref를 제공받을 수 있다.
  - 렌더링 도중에 ref.current를 읽거나 쓰지 말자
    - React의 렌더링 주기에서 current값이 충돌날 수 있기 때문이다.

## Ref로 DOM 조작하기

- React는 렌더링 출력과 일치하도록 [DOM](https://developer.mozilla.org/docs/Web/API/Document_Object_Model/Introduction)을 자동으로 업데이트한다
  ⇒ 고로, 컴포넌트가 자주 조작할 필요가 없다.
- But, React가 관리하는 DOM 요소에 접근해야할 때도 있다.
  - 노드에 초점을 맞추거나 (ex. 입력 버튼 클릭시 input focusing)
  - 스크롤하거나 크기와 위치를 측정 (ex. 무한 스크롤)

⇒ React에는 이러한 작업을 수행할 수 있는 빌트인된 방법이 없기에 DOM 노드에 대한 *ref*가 필요하다.

<aside>
💡 Ref의 두번째 기능인 DOM 노드에 직접 접근이다.
(개인적으로 두번째 기능으로만 ref를 사용하고 있다)

</aside>

### node에 대한 ref 가져오기

```tsx
import { useRef } from 'react';

const myRef = useRef(null);

/// DOM 노드를 가져올 JSX의 ref에 참조로 전달하면 된다
<div ref={myRef}>
```

- ref로 DOM 노드를 가져오는 과정

  1. 처음에는 `myRef.current` === `null` 이다.
  2. 이후, React가 참조하려는 `<div>`에 대한 DOM 노드를 생성하면,
  3. React는 이 노드에 대한 참조를 `myRef.current`에 넣는다.
  4. 이후, [이벤트 핸들러](https://react-ko.dev/learn/responding-to-events)에서 이 DOM 노드에 액세스하면 DOM 노드에 정의된 빌트인 [브라우저 API](https://developer.mozilla.org/docs/Web/API/Element)를 사용할 수 있다.
     - 예시) `myRef.current.scrollIntoView();`

- **예) 텍스트 Input 초점 맞추기**
  ```tsx
  import { useRef } from "react";

  export default function Form() {
    const inputRef = useRef(null);

    function handleClick() {
      inputRef.current.focus();
    }

    return (
      <>
        <input ref={inputRef} />
        <button onClick={handleClick}>Focus the input</button>
      </>
    );
  }
  ```
  1. `useRef` 훅으로 `inputRef`를 선언.
  2. `<input ref={inputRef}>`에 inputRef 전달
     - 이 과정에서 React가 이 `<input>`의 DOM 노드를 `inputRef.current`에 넣는다
  3. `handleClick` 함수에서 `inputRef.current`로부터  `[focus()](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/focus)`를 호출
     - 즉, input DOM 노드의 빌트인 브라우저 API 사용가능
  4. `<button>`의 `onClick`에 `handleClick` 이벤트 핸들러를 전달

**ref의 가장 일반적인 사용**

1. **DOM 조작**
2. But, `useRef` 는 timer ID와 같은 **다른 것들을 React 외부에 저장하는 데 사용될 수 있다. (ref로 값 참조하기 복습)**
   - **state와 유사하게 ref는 렌더링 사이에 유지**된다.
   - ref는 state 변수와 비슷하지만 설정할 때 리렌더링을 촉발하지 않는다.

- **예) element로 스크롤 하기**
  ```tsx
  import { useRef } from "react";

  export default function CatFriends() {
    // 1. ref를 여러개 작성할 수 있다
    const firstCatRef = useRef(null);
    const secondCatRef = useRef(null);
    const thirdCatRef = useRef(null);

    function handleScrollToFirstCat() {
      // 2. DOM 노드에 빌트인된 scrollIntoView 메소드로 이미지를 중앙 배치
      firstCatRef.current.scrollIntoView({
        behavior: "smooth",
        block: "nearest",
        inline: "center",
      });
    }

    function handleScrollToSecondCat() {
      secondCatRef.current.scrollIntoView({
        behavior: "smooth",
        block: "nearest",
        inline: "center",
      });
    }

    function handleScrollToThirdCat() {
      thirdCatRef.current.scrollIntoView({
        behavior: "smooth",
        block: "nearest",
        inline: "center",
      });
    }

    return (
      <>
        <nav>
          <button onClick={handleScrollToFirstCat}>Tom</button>
          <button onClick={handleScrollToSecondCat}>Maru</button>
          <button onClick={handleScrollToThirdCat}>Jellylorum</button>
        </nav>
        <div>
          <ul>
            <li>
              <img
                src="https://placekitten.com/g/200/200"
                alt="Tom"
                ref={firstCatRef}
              />
            </li>
            <li>
              <img
                src="https://placekitten.com/g/300/200"
                alt="Maru"
                ref={secondCatRef}
              />
            </li>
            <li>
              <img
                src="https://placekitten.com/g/250/200"
                alt="Jellylorum"
                ref={thirdCatRef}
              />
            </li>
          </ul>
        </div>
      </>
    );
  }
  ```
- **ref callback을 사용하여 refs 목록을 관리하는 방법**
  - 위의 사진 중앙 정렬 예시에서 ref를 항목 수만큼 미리 정의해 두었다.
    - but, list의 각 item에서 ref를 사용할 때 문제가 될 수 있다.
      - 아래 경우는 제대로 **작동하지 않을 것이다**
    ```tsx
    <ul>
      {items.map((item) => {
        // Doesn't work! 작동하지 않습니다!
        const ref = useRef(null);
        return <li ref={ref} />;
      })}
    </ul>
    ```
    ⇒ **훅은 컴포넌트의 최상위 레벨에서만 호출**이라는 규칙 위반
  - 해결 방법은

    - 예시
      ```tsx
      function ParentComponent() {
        // 부모 엘리먼트의 ref를 생성
        const parentRef = useRef(null);

        useEffect(() => {
          // 컴포넌트가 마운트된 후에 실행
          if (parentRef.current) {
            // 부모 엘리먼트에서 하위 노드를 선택
            const childNodes =
              parentRef.current.querySelectorAll(".child-element");

            // 선택한 하위 노드를 순회하면서 작업을 수행.
            childNodes.forEach((childNode, index) => {
              console.log(`Child ${index + 1}:`, childNode.textContent);
              // 이곳에서 선택한 노드에 대한 작업을 수행
            });
          }
        }, []);

        return (
          <div ref={parentRef}>
            <div className="child-element">Child 1</div>
            <div className="child-element">Child 2</div>
            <div className="child-element">Child 3</div>
          </div>
        );
      }
      ```

    1. 부모 엘리먼트에 대한 단일 ref를 가져온 다음
    2. `[querySelectorAll](https://developer.mozilla.org/ko/docs/Web/API/Document/querySelectorAll)`과 같은 DOM 조작 메서드를 사용하여 개별 하위 노드를 “**찾는**”것
       - But, DOM 구조가 변경되면 깨질 수 있다.

  - 또 다른 해결 방법은 **`ref` 속성에 함수를 전달**하는 것
    - `[ref` 콜백](https://react-ko.dev/reference/react-dom/components/common#ref-callback)이라고 한다.
    - React는 ref를 설정할 때가 되면 DOM 노드로, 지울 때가 되면 `null`로 ref 콜백을 호출할 것이다.
      ⇒ 자신만의 배열이나 [Map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map)을 유지 관리하고, 인덱스나 일종의 ID로 모든 ref에 접근할 수 있다.
    - 예시
      ```tsx
      import { useRef } from "react";

      export default function CatFriends() {
        const itemsRef = useRef(null);

        function scrollToId(itemId) {
          const map = getMap();
          const node = map.get(itemId);
          node.scrollIntoView({
            behavior: "smooth",
            block: "nearest",
            inline: "center",
          });
        }

        function getMap() {
          if (!itemsRef.current) {
            // 처음 사용할 때 Map으로 초기화
            itemsRef.current = new Map();
          }
          return itemsRef.current;
        }

        return (
          <>
            <nav>
              <button onClick={() => scrollToId(0)}>Tom</button>
              <button onClick={() => scrollToId(5)}>Maru</button>
              <button onClick={() => scrollToId(9)}>Jellylorum</button>
            </nav>
            <div>
              <ul>
                {catList.map((cat) => (
                  <li
                    key={cat.id}
                    // ref에 함수 전달 (ref callback)
                    ref={(node) => {
                      const map = getMap();
                      if (node) {
                        map.set(cat.id, node);
                      } else {
                        map.delete(cat.id);
                      }
                    }}
                  >
                    <img src={cat.imageUrl} alt={"Cat #" + cat.id} />
                  </li>
                ))}
              </ul>
            </div>
          </>
        );
      }

      const catList = [];
      for (let i = 0; i < 10; i++) {
        catList.push({
          id: i,
          imageUrl: "https://placekitten.com/250/200?image=" + i,
        });
      }
      ```
    - 이 예제에서 `itemsRef`는 단일 DOM 노드를 보유하지 않는다.
      - 대신, 항목 ID에서 DOM 노드로의 [Map](https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Global_Objects/Map)을 보유한다.
      - 모든 목록 항목의 `[ref` 콜백](https://react-ko.dev/reference/react-dom/components/common#ref-callback)은 맵을 업데이트한다
      ```tsx
      <li
        key={cat.id}
        ref={node => {
          const map = getMap();
          if (node) {
            map.set(cat.id, node); // Add to the Map
          } else {
            map.delete(cat.id); // Remove from the Map
          }
        }}
      >
      ```
      - 이렇게 하면 나중에 Map에서 개별 DOM 노드를 읽을 수 있다.\*\*\*\*
      - **💡추가 ([ref에 callback이 들어간다고?](https://react-ko.dev/reference/react-dom/components/common#ref-callback))**
        - DOM 노드가 화면에 추가되면, React는 DOM node를 인수로 사용하는 callback을 호출한다
          ⇒ **💡**추가되는게 callback을 실행시키는 trigger인 듯
        - 그리고 해당 DOM node가 화면에서 제거되면 React는 null로 ref callback을 호출한다

### \***\*다른 컴포넌트의 DOM 노드에 접근하기\*\***

- `<input />`같은 브라우저 엘리먼트를 출력하는 빌트인 컴포넌트에 ref를 넣으면,
  - React는 해당 ref의 `current` 프로퍼티를 해당 DOM 노드로 설정한다.
    (예: 브라우저의 실제 `<input />`)
- But, `<MyInput />`과 같은 직접 만든 **컴포넌트에 ref를 넣으려고** 하면 기본적으로 `null` 이 반환된다. ⇒ 💡forwardRef를 적용하면 됨

  - 예제
    ```tsx
    import { useRef } from "react";

    function MyInput(props) {
      return <input {...props} />;
    }

    export default function MyForm() {
      const inputRef = useRef(null);

      function handleClick() {
        inputRef.current.focus();
      }

      return (
        <>
          <MyInput ref={inputRef} />
          <button onClick={handleClick}>Focus the input</button>
        </>
      );
    }

    // 경고: 함수 컴포넌트에는 refs를 지정할 수 없습니다.
    // 이 ref에 접근하려는 시도는 실패합니다.
    // React.forwardRef()를 사용하려고 하셨나요?
    ```

- 기본적으로 **React에서 컴포넌트가 다른 컴포넌트의 DOM 노드에 접근하는 것을 허용하지 않기** 때문
  - 컴포넌트의 자식에게도 허용하지 않는다 (의도적)
  - _다른_ 컴포넌트의 DOM 노드를 수동으로 조작하면 코드가 훨씬 취약해진다.
  ⇒ 고로, ref는 탈출구이기 때문에 아껴서 사용해야 한다.
- 그렇다해도 조작하려면 DOM 노드를 노출하길 _원하는_ 컴포넌트에 해당 동작을 따로 **설정**해야 한다 ⇒ `**forwardRef` 사용!\*\*

  - 컴포넌트는 자신의 ref를 자식 컴포넌트 중 하나에 “전달”하도록 지정할 수 있다.

  ```tsx
  // 예제에선 자식 컴포넌트가 input 하나인 경우다
  const MyInput = forwardRef((props, ref) => {
    return <input {...props} ref={ref} />;
  });
  ```

  1. `**<MyInput ref={inputRef} />**`는 React에게 해당 DOM 노드를 `inputRef.current`에 넣으라고 지시하지만 선택권은 `MyInput` 컴포넌트에 달려 있다. (기본적으론 그렇게 행동 안 함)
  2. `MyInput` 컴포넌트를 `forwardRef`를 사용하여 선언하면, `props` 다음의 **두 번째 `ref` 인수**에서 `inputRef`를 받도록 설정
  3. `MyInput`은 수신한 `ref`를 내부의 `<input>`으로 전달한다

- 디자인 시스템에서

  - 버튼, 입력 등과 같은 **저수준 컴포넌트**는 ref를 DOM 노드로 전달하는 것이 일반적이다.
  - 반면, 양식, 목록 또는 페이지 섹션과 같은 **고수준 컴포넌트**는 일반적으로 DOM 구조에 대한 우발적 의존성을 피하기 위해 해당 DOM 노드를 노출하지 않는다.

    <aside>
    💡 아토믹 디자인 기준으로 atom, moculer까지가 저수준 컴포넌트로 보인다
    
    </aside>


- \***\*명령형 핸들로 API의 하위 집합 노출하기 (\*\***위 예시 참고)
  ```tsx
  const MyInput = forwardRef((props, ref) => {
    return <input {...props} ref={ref} />;
  });
  ```
  - `MyInput`은 원본 DOM input 엘리먼트를 노출하고 있다.
    - 여기서 부모 컴포넌트가 input에 `focus()`를 호출할 수 있다.
    - But, 이렇게 하면 부모 컴포넌트가 다른 작업(예: CSS 스타일 변경)을 할 수도 있다.
      - 이렇게 제작자의 의도와 다르게 사용하는 걸 막기 위해, 노출되는 기능을 제한할 수 있다 ⇒ `**useImperativeHandle**`을 사용하면 된다:
  - 예시
    ```tsx
    import { forwardRef, useRef, useImperativeHandle } from "react";

    const MyInput = forwardRef((props, ref) => {
      const realInputRef = useRef(null);
      useImperativeHandle(ref, () => ({
        // ref는 오직 focus에 대한 명령만 받아서 정해진 액션만 취한다
        focus() {
          realInputRef.current.focus();
        },
      }));
      return <input {...props} ref={realInputRef} />;
    });

    export default function Form() {
      const inputRef = useRef(null);

      function handleClick() {
        inputRef.current.focus();
      }

      return (
        <>
          <MyInput ref={inputRef} />
          <button onClick={handleClick}>Focus the input</button>
        </>
      );
    }
    ```
  - 여기서 `MyInput` 내부의 `realInputRef`는 실제 input DOM 노드를 보유한다.
  - but `useImperativeHandle`은 부모 컴포넌트에 대한 ref 값으로 고유한 특수 객체를 제공하도록 React에 지시한다.
    ⇒ 따라서 `Form` 컴포넌트 내부의 `inputRef.current`에는 `focus` 메서드만 있게 됩니다.
    - 이 경우 ref handle은 ~~DOM 노드~~가 아니라 `useImperativeHandle()` 내부에서 생성한 사용자 정의 객체이다.

### \***\*React가 ref를 첨부할 때\*\***

React에서 업데이트는 두 단계로 진행된다

1. **렌더링** 하는 동안
   - React는 **컴포넌트를 호출하여 화면에 무엇이 표시되어야 하는지 파악**
2. **커밋(commit)** 하는 동안
   - React는 **DOM에 변경 사항을 적용**

일반적으로 렌더링 중에는 ref에 액세스하는 것을 [원하지 않을 것이다](https://react-ko.dev/learn/referencing-values-with-refs#best-practices-for-refs). (좋지 않다)

- 첫 번째 렌더링 중에는 DOM 노드가 아직 생성되지 않았으므로 `ref.current`는 `null`로 되어있다
- 그리고 업데이트를 렌더링하는 동안에는 DOM 노드가 아직 업데이트되지 않았기에 읽기에는 너무 이르다.

React는 커밋하는 동안에 `ref.current`를 설정한다.

- React는 DOM이 업데이트 되기 전에는 `ref.current`의 값을 `null`로 설정하였다가, DOM이 업데이트된 직후 해당 DOM 노드로 다시 설정한다.

**일반적으로 이벤트 핸들러에서 ref에 접근가능하다.**

- ref로 무언가를 하고 싶지만 그 작업을 수행할 특정 이벤트가 없다면, Effect가 필요할 수 있다.

- \***\*Flushing state는 flushSync와 동기식으로 업데이트됩니다. (\*\***❓)

### \***\*ref를 이용한 DOM 조작 모범 사례\*\***

- Ref는 **React 영역 밖**으로 나갈 수 있는 탈출구이다.
- 초점을 맞추거나, 스크롤 위치를 관리하거나, React가 노출하지 않는 브라우저 API를 호출하는 것이 있습니다.

- 포커스나 스크롤 같은 비파괴적 동작에서만 이용한다면 문제가 발생하지 않을 거다.
- But, DOM을 수동으로 **수정**하려고 하면

  - React가 수행하는 변경 사항과 충돌할 위험이 있다.

- **예) 조건부 렌더링 + state 와 remove로 강제 제거**
  ```tsx
  //“Toggle with setState”를 몇 번 눌러보라 => 메세지가 보였다 말았다
  // 그런 다음 “Remove from the DOM”을 누르면 => 강제 제거
  // 마지막으로 “Toggle with setState”를 누르면 => 에러발생
  import { useState, useRef } from "react";

  export default function Counter() {
    const [show, setShow] = useState(true);
    const ref = useRef(null);

    return (
      <div>
        <button
          onClick={() => {
            setShow(!show);
          }}
        >
          Toggle with setState
        </button>
        <button
          onClick={() => {
            ref.current.remove();
          }}
        >
          Remove from the DOM
        </button>
        {show && <p ref={ref}>Hello world</p>}
      </div>
    );
  }
  ```
  ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/693bc352-6af5-47d2-866f-3108bb9e50f3/9d8356d9-f111-4a10-aa2a-8bb28a0d2b36/Untitled.png)
  - DOM 엘리먼트를 수동으로 제거한 후 `setState`를 사용하여 다시 표시하려고 하면 충돌 발생.
    - 사용자가 DOM을 변경했고
    - 변경된 DOM으로 React가 올바르게 관리하는 방법을 모르기 때문이다.

**React가 관리하는 DOM 노드를 변경하지 말자.**

- React가 관리하는 요소를 수정 or 자식 추가 or 제거하면,
  ⇒ 일관성 없는 시각적 결과나 충돌이 발생할 수 있다. (위 예시)
- 무조건 하지말라는 건 아니고 주의해서 쓰라는 의미다.

**React가 업데이트할 _이유가 없는_ DOM의 일부는 안전하게 수정할 수 있다.**

- ex) JSX에서 일부 `<div>`가 항상 비어 있는 경우, React는 그 자식 목록을 건드릴 이유가 없다. (위의 예시처럼 조건부 렌더링으로 건드리면 얘기가 다름)
  ⇒ 수동으로 요소를 추가하거나 제거하더라도 안전하다.

### 요약

- Ref는 일반적인 개념이긴 하지만, 대부분 DOM 엘리먼트를 보관할 때 사용.
- `<div ref={myRef}>`를 전달해 DOM 노드를 `myRef.current`에 넣으라고 React에 지시
- 보통은 포커스, 스크롤, DOM 엘리먼트 측정과 같은 비파괴적인 동작에 ref를 사용
- 컴포넌트는 기본적으로 DOM 노드를 노출하지 않습니다. `forwardRef`를 사용하고 두 번째 `ref` 인수를 특정 노드에 전달하여 DOM 노드를 노출하도록 설정 가능
- React가 관리하는 DOM 노드를 변경하지 마라.
- React가 관리하는 DOM 노드를 수정해야 한다면 React가 업데이트할 이유가 없는 부분을 수정하라.
