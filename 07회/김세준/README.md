# [7회차] - EscapeHatches(2)(Effect로 동기화하기 ~ Effect가 필요하지 않을 수도 있습니다.)
>링크 : https://woozy-stool-6eb.notion.site/7-EscapeHatches-2-Effect-Effect-19fb53b4d4004f36a4bb5dd5a3c27c7c?pvs=4

# **Synchronizing with Effects**

> **Effect와 동기화하기**
> 

React state를 기반으로 서버 연결을 설정하거나 컴포넌트가 화면에 나타날 때 분석 로그를 보내는 등의 외부 시스템과 동기화 해야하는 컴포넌트들이 존재한다. Effect를 사용하면 렌더링 이후 일부 코드 실행함으로써 컴포넌트를 외부 시스템과 동기화할 수 있다. 

<aside>
🌟 학습목표

- Effect가 무엇인지
- Effect와 이벤트의 차이점
- 컴포넌트에서 Effect를 선언하는 방법
- 불필요하게 Effect를 재실행하는 것을 건너뛰는 방법
- 개발시 Effect가 두번 실행되는 이유와 해결 방법
</aside>

## **What are Effects and how are they different from events?**

> **Effect란 무엇이며 이벤트와는 어떤게 다른가요?**
> 

React 컴포넌트 내부의 2가지 유형의 논리

- **랜더링 코드** : 컴포넌트 최상위 레벨에 위치하며 props, state를 가져와 변환하고 화면에 표시할 JSX를 반환한다. 렌더링 코드는 순수해야 한다.
- **이벤트 핸들러** : 컴포넌트 내부에 있는 중첩된 함수로 여러가지 명령을 수행한다. 특정 사용자 작업(e.g 버튼클릭)으로 인해 발생하는 ‘사이드 이펙트’(e.g state변경)가 포함되어 있다.

때로는 위 2가지로 구현할 수 없는 경우도 존재한다. 화면에 표시할 때마다 채팅 서버에 연결해야 하는 ChatRoom 이라는 컴포넌트가 존재한다고 가정해보자. 서버에 연결하는 것은 순수한 계산이 아니므로 (사이드 이펙트) 렌더링 중에 발생할 수 없다. 또한 ChatRoom 컴포넌트를 촉발할 이벤트도 존재하지 않는다.

이런 경우 **Effect 를 사용하여 특정 이벤트가 아닌 렌더링 자체로 촉발되는 사이드 이펙트를 명시할 수 있다.**

Effect는 화면 업데이트 후 커밋이 끝날 때 실행된다. → React 컴포넌트와 외부 시스템과 동기화 하기 좋은 시점

<aside>
🌟 **Note**

Effect 는 React에 한정된 정의, 즉 렌더링으로 인해 발생하는 사이드 이펙트를 나타낸다.

</aside>

## **You might not need an Effect**

> **Effect가 필요하지 않을 수도 있습니다**
> 

Effect는 React 코드에서 벗어나 외부시스템과 동기화하는데 사용된다.다른 state를 기반으로 일부 state 만을 조정하는 경우 Effect가 필요하지 않을 수 있다. 

## **How to write an Effect**

> **Effect 작성 방법**
> 
1. Effect 선언
2. Effect 의존성 명시
3. 필요한 경우 클린업 추가
    
    일부 Effect는 수행 중이던 작업을 중지, 취소 하는 방법을 명시해야 한다. 
    

### **Step 1: Declare an Effect**

> **Effect를 선언하세요**
> 

컴포넌트가 렌더링 될때마다 React는 화면을 업데이트하고 useEffect 내부의 코드를 실행한다. 즉, **`useEffect`는 해당 렌더링이 화면에 반영이 될 때까지 코드 조각의 실행을 “지연”한다.**

props를 전달 받아서 props 값과 함수호출을 동기화 하려고 한다면 DOM 요소에서 메서드를 수동으로 호출해야 한다.

**e.g. 비디오가 현재 재생되어야 하는지 여부를 알려주는 `isPlaying` prop값을 `play()` 및 `pause()`와 같은 함수를 호출과 동기화 하기 위한 코드**

```tsx
import { useState, useRef, useEffect } from 'react';

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  if (isPlaying) {
    ref.current.play();  // Calling these while rendering isn't allowed.
  } else {
    ref.current.pause(); // Also, this crashes.
  }

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? 'Pause' : 'Play'}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

먼저 <video> DOM 노드에 대한 ref를 가져온 뒤 렌더링 중에 isPlaying props 값을 기준으로 play() 혹은 pause() 메서드를 호출하고 싶지만 이는 올바르지 않다.

- 렌더링 중에 DOM 노드로 무언가 동작을 시도하므로 JSX가 순수한 계산이어야 하며 DOM 수정과 같은 사이드 이펙트를 포함해서는 안된다는 원칙에 어긋남
- 해당 컴포넌트가 처음 호출될 때 DOM이 아직 존재하지 않음, 메서드를 호출할 DOM 노드가 없는 상태임(React는 JSX를 반환하기 전까지 어떤 DOM을 생성할 지 모름)

🌟 **해결책!! 사이드 이펙트를 useEffect로 감싸 렌더링 계산 밖으로 옮김**

```tsx
function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}
```

⇒ DOM 업데이트를 useEffect로 감싸면, React가 화면을 먼저 업데이트한 후 Effect 내부 코드를 실행한다.

### **Step 2: Specify the Effect dependencies**

> **Effect 의존성을 지정하세요**
> 

기본적으로 Effect는 매번 렌더링 후에 실행된다. 

useEffect 호출의 두 번째 인자로 의존성 배열을 지정하면 불필요하게 Effect를 다시 실행하지 않도록 할 수 있다. 

의존성 배열은 여러 개의 의존성을 포함할 수 있다. React는 지정한 모든 의존성의 값이 이전 렌더링 때와 정확히 동일한 경우에만 Effect 의 재실행을 건너뛴다. 의존성 값을 비교할 때는 [Object.is](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 비교를 사용한다.

덧붙여 의존성은 ‘선택’할 수 없다. 의존성을 일부만 넣고 싶다면 빼고 싶은 의존성은 필요하지 않도록 Effect 코드 자체를 편집해야 한다.

<aside>
🌟 **DEEP DIVE | 심층탐구**

### **Why was the ref omitted from the dependency array?**

> **의존성 배열에서 ref가 생략된 이유는 무엇인가요?**
> 

`ref` 객체가 *안정적인 정체성*을 가지고 있기 때문이다: React는 렌더링할 때마다 동일한 useRef 호출에서 [항상 동일한 객체를 얻을 수 있도록](https://react-ko.dev/reference/react/useRef#returns) 보장한다. 

다만 이 논리는 linter가 객체가 안정적이라는 것을 ‘확인’할 수 있을 때만 동작한다. 부모 컴포넌트에서 ref가 전달된 경우 부모 컴포넌트가 항상 동일한 ref를 전달하는지 아니면 여러 개의 ref 중 하나를 조건부로 전달하는지 알 수 없으므로 의존성 배열에 이를 지정해야 한다. 

</aside>

### **Step 3: Add cleanup if needed**

> **필요한 경우 클린업을 추가하세요**
> 

**e.g. 채팅 서버가 나타날 때 채팅 서버에 연결해야 하는 `ChatRoom` 컴포넌트를 작성하고 있다고 가정해보자.** `connect()` 와 `disconnect()` 메서드가 있는 객체를 반환하는 `createConnection()` API가 주어진다고 할 때 컴포넌트가 사용자에게 표시되는 동안 어떻게 연결상태를 유지할 수 있을까?

```jsx
useEffect(() => {
  const connection = createConnection();
  connection.connect();
});

/ =>
useEffect(() => {
  const connection = createConnection();
  connection.connect();
}, []);
```

의존성 배열을 비어둘 경우 컴포넌트가 ‘마운트’될 때 즉 화면에 처음 나타낼 때만 이 코드를 실행하도록 한다.

**But!! 아래 코드가 화면에 정상적으로 출력된다고 할 때  ‘✅ Connecting...’ 몇번이나 콘솔에 찍히게 될까?**

- 해답
    
    strict 모드로 인해 2번이 출력될 것으로 예상했지만, 그것이 아니라 
    
    1. ChatRoom 컴포넌트가 마운트되고 connection.connect() 를 호출한다. 
    2. 그런 다음 사용자가 다른 화면으로 이동하면 CharRoom 컴포넌트 가마운트해제 된다.
    3. 마지막 사용자가 뒤로가기(Back)를 클릭하면 ChatRoom이 다시 마운트 된다. 

이러면 2번째 연결이 설정되지만, 여전히 첫번째 연결이 남아있므로 콘솔창에 2번 출력이 된다.

```jsx
// App.js
import { useEffect } from 'react';
import { createConnection } from './chat.js';

export default function ChatRoom() {
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
  }, []);
  return <h1>Welcome to the chat!</h1>;
}

// Chart.js
export function createConnection() {
  // A real implementation would actually connect to the server
  return {
    connect() {
      console.log('✅ Connecting...');
    },
    disconnect() {
      console.log('❌ Disconnected.');
    }
  };
}
```

⇒ 이 문제를 해결하려면 Effect에서 클린업 함수를 반환하라.

```jsx
useEffect(() => {
    const connection = createConnection();
    connection.connect();
    return () => {
      connection.disconnect();
    };
  }, []);
```

**상용 환경에서는 `"✅ Connecting..."`이 한 번만 인쇄된다.** 컴포넌트를 다시 마운트하는 동작은 클린업이 필요한 Effect를 찾는 것을 돕기 위해 오직 개발 환경에서만 수행된다.

## **How to handle the Effect firing twice in development?**

> **개발 환경에서 두 번씩 실행되는 Effect를 처리하는 방법은 무엇인가요?**
> 

⇒ **어떻게 다시 마운트한 후에도 Effect가 잘 작동하도록 수정하는가? : 더 올바른 질문**

⇒ 클린업 함수(Effect 가 수행 중이던 작업을 주잊하거나 취소해야 한다.)를 구현

### **Controlling non-React widgets**

> **React가 아닌 위젯 제어하기**
> 

```jsx
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

가끔 React로 작성되지 않은 UI 위젯을 추가해야 할 때가 있다. 예를 들어 페이지에 지도 컴포넌트를 추가한다고 가정하자. 이 지도 컴포넌트에는 setZoomLevel() 메서드가 있으며 zoomLevel 상태 변수와 동기화하려고 한다.

이 경우에는 클린업이 필요가 없다. 개발모드에선 두번 호출되지만 같은 값으로 메서드를 두번 호출 하는 것은 아무런 문제가 되지 않는다.

연속해서 두번 호출하는 것을 허용하지 않는 일부 API는 메서드를 두 번 호출하면 에러를 던진다. 이런 경우에는 클린업 함수를 구현해야 한다.

```jsx
useEffect(() => {
  const dialog = dialogRef.current;
  dialog.showModal();
  return () => dialog.close();
}, []);
```

### **Subscribing to events**

> **이벤트 구독하기**
> 

Effect가 무언가를 구독하는 경우, 클린업 함수는 구독을 취소해야 한다.

```jsx
useEffect(() => {
  function handleScroll(e) {
    console.log(window.scrollX, window.scrollY);
  }
  window.addEventListener('scroll', handleScroll);
  return () => window.removeEventListener('scroll', handleScroll);
}, []);
```

개발 중에는 Effect가 addEventListener()를 호출한 다음 즉시 removeEventListener()를 호출하고 그 다음 동일한 핸들러로 addEventListener()를 호출한다. 따라서 한번에 하나의 활성 구독만 있게 된다. 이것은 프로덕션 환경에서 한번 addEventListener()를 호출하는 것과 동일한 동작을 가진다.

### **Triggering animations**

> **애니메이션 촉발하기**
> 

Effect가 무언가를 애니메이션 하는 경우 클린업 함수는 애니메이션을 초기값으로 재설정 해야 한다.

```jsx
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
                          // 애니메이션 촉발
  return () => {
    node.style.opacity = 0; // Reset to the initial value
                            // 초기값으로 재설정
  };
}, []);
```

### **Fetching data**

> **데이터 페칭하기**
> 

 Effect가 어떤 데이터를 가져온다면 클린업 함수에서는 fetch를 중단하거나 결과를 무시해야한다.

```jsx
useEffect(() => {
  **let ignore = false;**

  async function startFetching() {
    const json = await fetchTodos(userId);
    **if (!ignore) {**
      setTodos(json);
    }
  }

  startFetching();

  **return () => {
    ignore = true;
  };**
}, [userId]);
```

이미 발생한 네트워크 요청을 ‘실행 취소’할 수는 없으므로 클린업 함수를 사용하여 관련 없는 fetch가 어플리케이션에 영향을 미치지 않도록 해야 한다. 

개발중에는 네트워크 탭에서 두 개의 fetch가 표시된다. 프로덕션 환경에서는 하나의 요청만 있을 것이므로 신경쓰지 않아도 된다. 개발 중에 두번째 요청이 문제라면 가장 좋은 방법은 **중복 요청을 제거하고 컴포넌트간에 응답을 캐시하는 솔류션을 사용**하는 것이다.

```jsx
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
  // ...
```

이렇게 하면 개발 환경을 개선하는데 도움이 될 뿐만 아니라 애플리케이션의 반응 속도도 향상된다. 예를 들어 사용자가 뒤로가기 버튼을 눌렀을 때 데이터를 다시 로드하는 것을 기다릴 필요가 없다. 데이터가 캐시되기 때문이다. 

<aside>
🌟 ****DEEP DIVE | Effect에서 데이터를 fetching하는 것의 대안은 무엇입니까?****

Effect 안에서 fetch 호출을 작성하는 것은 데이터를 가져오는 인기있는 방법이다.(특히 완전히 클라이언트 측에만 작성된 앱에서) 하지만 이는 수동적인 접근 방식이며 중요한 단점이 있다.

- **Effect는 서버에서 실행되지 않는다.** ⇒ 따라서 초기 서버 렌더링된 HTML은 데이터가 없는 로딩상태를 포함하고 클라이언트 컴퓨터는 모든 JavaScript를 다운로드하고 앱을 렌더링해야만 로드해야 한다는 것을 알게 될 것이다. 이는 효율적이지 않다.
- **Effect 안에서 직접 가져오면 “네트워크 폭포”를 쉽게 만들 수 있다.** ⇒ 부모 컴포넌트에서 렌더링하면 일부 데이터를 가져오고 그 자식 컴포넌트를 렌더링한 다음 그들이 데이터를 가져오기 시작한다. 네트워크가 빠르지 않으면 병렬로 가져오는 것 보다 훨씬 느리다.
- **Effect 안에서 직접 fetch하는 것은 일반적으로 데이터를 미리 로드하거나 캐시하지 않음을 의미한다.**
- 그리 편리하지 않다. ⇒ fetch 호출을 작성할 때 경쟁 상태와 같은 버그에 영향을 받지 않는 방식으로 작성하는데 꽤 **많은 보일러플레이트 코드가 필요**하다.

이 단점 목록은 리액트에만 해당하는 것이 아니다. 어떤 라이브러리든 마운트 시에 데이터를 가져온다면 비슷한 단점이 존재한다. 따라서 다음 접근방식을 권장한다.

- 프레임워크를 사용하는 경우 해당 **프레임워크의 내장 데이터 fetching 매커니즘을 사용**하자. ⇒ 현대적인 리액트 프레임워크에는 위 단점을 겪지 않는 효율적이고 통합적인 데이터 페칭 매커니즘이 포함되어있다.
- 그렇지 않은 경우 **클라이언트 측 캐시를 사용하거나 직접 구축하는 것을 고려**하자. ⇒ 대표적으로 React Query, useSWR 등이 있다.
</aside>

### **Sending analytics**

> **분석 보내기**
> 

페이지 방문 시 분석 이벤트를 보내는 코드를 살펴보자.

```jsx
useEffect(() => {
  logVisit(url);
}, [url]);
```

개발환경에서 모든 URL에 대해 두 번 호출 되지만 이 코드는 수정하지 않는 것이 좋다.

제품 환경에서는 중복된 방문 로그가 없을 것이다. 보다 정확한 분석을 위해서는 [intersection observers](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)를 활용하여 뷰포트에 어떤 컴포넌트가 얼마나 오래 표시되었는지를 추적하는데 도움이 될 수 있다. 

### **Not an Effect: Initializing the application**

> **Effect가 아님: 애플리케이션 초기화하기**
> 

일부 로직은 애플리케이션이 시작될 때 한 번만 실행되어야 한다. 이런 로직은 컴포넌트 외부에 넣을 수 있다.

```jsx
if (typeof window !== 'undefined') { // Check if we're running in the browser.
                                     // 실행환경이 브라우저인지 여부 확인
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

### **Not an Effect: Buying a product**

> **Effect가 아님: 제품 구매하기**
> 

클린업 함수를 작성하더라도 Effect가 두 번 실행되면서 결과가 달라지는 것을 막을 수 없을 수도 있다.

```jsx
useEffect(() => {
  // 🔴 틀렸습니다: 이 Effect는 개발모드에서 두 번 실행되며, 문제를 일으킵니다.
  fetch('/api/buy', { method: 'POST' });
}, []);
```

제품 구매를 위한 POST 요청이 Effect 내부에 존재 한다면 사용자가 다른 페이지로 이동한 다음 Back(뒤로가기)를 클릭하면 다시 Effect가 실행될 것이다. 제품 구매와 같은 로직은 렌더링이 아니라 특정 상호 작용과 연관되어 발생한다. 따라서 이럴 때는 Effect를 삭제하고 이벤트 핸들러를 사용하는 것이 좋다. 

```jsx
function handleClick() {
    // ✅ 구매는 특정 상호작용으로 인해 발생하므로 이벤트입니다.
    fetch('/api/buy', { method: 'POST' });
  }
```

컴포넌트가 다시 마운트 되었을 때 애플리케이션 로직이 깨진다면 버그를 발견할 수 있다. React 개발 단계에서 컴포넌트를 다시 한 번 마운트하여 이 원칙이 준수되는지 확인해야 한다.

## **Putting it all together**

> **한데 모으기**
> 

```jsx
import { useState, useEffect } from 'react';

function Playground() {
  const [text, setText] = useState('a');

  useEffect(() => {
    function onTimeout() {
      console.log('⏰ ' + text);
    }

    console.log('🔵 Schedule "' + text + '" log');
    const timeoutId = setTimeout(onTimeout, 3000);

    return () => {
      console.log('🟡 Cancel "' + text + '" log');
      clearTimeout(timeoutId);
    };
  }, [text]);

  return (
    <>
      <label>
        What to log:{' '}
        <input
          value={text}
          onChange={e => setText(e.target.value)}
        />
      </label>
      <h1>{text}</h1>
    </>
  );
}

export default function App() {
  const [show, setShow] = useState(false);
  return (
    <>
      <button onClick={() => setShow(!show)}>
        {show ? 'Unmount' : 'Mount'} the component
      </button>
      {show && <hr />}
      {show && <Playground />}
    </>
  );
}
```

**React는 항상 다음 렌더링의 Effect 전에 이전 렌더링의 Effect를 정리하며 각 Effect는 해당 렌더링에서 `text` 값을 “캡처”한다.** 이러한 작동 방식이 궁금하다면 [클로저](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures)를 읽어보자.

<aside>
🌟 **[DEEP DIVE | 각 렌더링에는 고유한 Effect가 있습니다](https://react-ko.dev/learn/synchronizing-with-effects#each-render-has-its-own-effects)**

</aside>

# **You Might Not Need an Effect**

> **Effect가 필요하지 않을 수도 있습니다**
> 

Effect는 React 패러다임에서 벗어날 수 있는 탈출구이다. Effect를 사용하면 외부 시스템과 동기화할 수 있다. 외부 시스템이 관여하지 않는 경우(e.g. props나 state 변경에 따라 컴포넌트 state가 업데이트 되는 경우)에는 필요하지 않다. 불필요한 Effect를 제거하여 오류 발생 가능성을 줄이고 속도를 높은 더 간결한 코드를 작성할 수 있다. 

<aside>
🌟 학습 내용

- 컴포넌트에서 불필요한 Effect를 제거하는 이유와 방법
- Effect 없이 값비싼 계산을 캐시하는 방법
- Effect 없이 컴포넌트 state를 리셋하고 조정하는 방법
- 이벤트 핸들러 간에 로직을 공유하는 방법
- 이벤트 핸들러로 이동되어야 하는 로직
- 부모 컴포넌트에 변경 사항을 알리는 방법
</aside>

## **How to remove unnecessary Effects**

> **불필요한 Effect를 제거하는 방법**
> 
- **렌더링을 위해 데이터를 변환하는 경우 Effect는 필요하지 않다.**
    
    **불필요한 렌더링을 피하려면 모든 데이터 변환을 컴포넌트 최상위 레벨에서 해야한다.** 그래야 props 나 state가 변경될 때마다 해당 코드가 자동으로 다시 실행될 것이다.
    
    컴포넌트의 state를 업데이트할 때 React는 먼저 컴포넌트 함수를 호출하여 화면에 표시될 내용을 계산한다. 그리고 변경사항을 DOM에 commit 하여 화면을 업데이트하고 그 후에 Effect를 실행한다.
    
- **사용자 이벤트를 처리하는 데에 Effect는 필요하지 않다.**

## **Updating state based on props or state**

> **props 또는 state에 따라 state 업데이트하기**
> 

e.g. firstName, lastName 두 개의 state가 존재하며 이 두 state를 연결하여 fullName을 계산하려고 한다. 또한 둘 중 어느 변수라도 변경되면 fullName 역시 업데이트 되기를 원한다.

```jsx
// 잘못된 코드
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 이러지 마세요: 중복 state 및 불필요한 Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
}

// 개선된 코드
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // ✅ 좋습니다: 렌더링 과정 중에 계산
  const fullName = firstName + ' ' + lastName;
  // ...
}
```

기존 props나 state에서 계산할 수 있는 것이 있으면 state에 넣지 말고 렌더링 중에 계산하는게 좋다. 성능도 더 좋아지고(추가적인 계단식 업데이트를 피함), 더 간결해지며 오류 발생 가능성도 낮아진다.(서로 다른 state 변수가 동기화 되지 않기 때문이다.)

## **Caching expensive calculations**

> **고비용 계산 캐싱하기**
> 

e.g 데이터를 필터링 하고 싶을 때

```jsx
// 잘못된 코드
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // 🔴 이러지 마세요: 중복 state 및 불필요한 Effect
  const [visibleTodos, setVisibleTodos] = useState([]);
  useEffect(() => {
    setVisibleTodos(getFilteredTodos(todos, filter));
  }, [todos, filter]);

  // ...
}

// 개선된 코드
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

  // ✅ getFilteredTodos()가 느리지 않다면 괜찮습니다.
  const visibleTodos = getFilteredTodos(todos, filter);
  // ...
}
```

만약 이 때 getFilteredTodos 함수가 느리거나, todos 데이터가 많을 경우, 관련 없는 state 변수가 변경되었을 때 getFilteredTodos를 다시 계산하고 싶지 않는 등 제약조건이 존재한다면 해당 함수를 useMemo 훅으로 감싸서 캐시(memoize)할 수 있다. 

```jsx
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');

	// useMemo를 활용할 수 있는 2가지 방법

  // ✅ 1. todos나 filter가 변하지 않는 한 재실행되지 않음
  const visibleTodos = **useMemo**(() => {
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
	// ✅ 2. todos나 filter가 변하지 않는 한 getFilteredTodos()가 재실행되지 않음
  const visibleTodos = **useMemo**(() => getFilteredTodos(todos, filter), [todos, filter]);
  
	// ...
}
```

이렇게 작성하면 `todos`나 `filter`가 변경되지 않는 한 내부 함수가 다시 실행되지 않는다.

`[useMemo](https://react-ko.dev/reference/react/useMemo)`로 감싸는 함수는 렌더링 중에 실행되므로, [순수 계산](https://react-ko.dev/learn/keeping-components-pure)에만 작동한다.

<aside>
🌟 **DEEP DIVE | 계산이 비싼지는 어떻게 알 수 있나요?**

측정하려는 상호작용(예: input에 입력)을 수행하면 콘솔에 로그를 확인할 수 있다. 기록된 시간이 상당(e.g. 1ms 이상) 하다면 해당 계산은 메모화하는 것이 좋을 수 있다. 

```jsx
console.time('filter array');
const visibleTodos = useMemo(() => {
  return getFilteredTodos(todos, filter); // Skipped if todos and filter haven't changed
}, [todos, filter]);
console.timeEnd('filter array');
```

다만 useMemo는 업데이트 시 불필요한 작업을 건너뛰게만 해줄 뿐 첫 번째 렌더링에는 영향을 주지 않는다. 

개발 중 성능 측정은 정확한 결과를 제공하지 않는다. 상용 앱을 빌드하고 사용자와 동일한 환경에서 테스트하는 것이 정확하다.

</aside>

## **Resetting all state when a prop changes**

> **prop이 변경되면 모든 state 재설정하기**
> 

e.g. 특정 유저의 프로필을 보여주기 위한 ProfilePage 라는 컴포넌트가 존재한다.

ProfilePage 컴포넌트는 userId prop을 받아 해당 userId에 해당하는 프로필 정보를 보여준다. 이 페이지에는 코멘트 Input이 포함되어 있고 comment state 변수를 사용하여 그 값을 보관한다. 

이 때 , 특정 프로필에서 다른 프로필로 이동할 때 comment state가 재설정 되지 않는 문제를 발견했다. 그 결과 의도치 않게 잘못된 사용자의 프로필에 댓글을 게시할 수 있는 상황이다. 

⇒ **userId가 변경될 때마다 comment state 변수를 지워줘야 한다.**

만약 댓글 UI가 중첩되어 있는 경우 중첩된 하위 댓글 state들도 모두 지워야 한다.

```jsx
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 이러지 마세요: prop 변경시 Effect에서 state 재설정 수행
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
```

**key를 전달하여 각 사용자의 프로필이 다른 프로필이라는 것을 명시할 수 있다.** 

```jsx
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ key가 변하면 이 컴포넌트 및 모든 자식 컴포넌트의 state가 자동으로 재설정됨
  const [comment, setComment] = useState('');
  // ...
}
```

일반적으로 React는 같은 컴포넌트가 같은 위치에서 렌더링될 때 state를 유지한다. **`userId`를 `key`로 `Profile` 컴포넌트에 전달하는 것은 곧, `userId`가 다른 두 `Profile` 컴포넌트를 state를 공유하지 않는 별개의 컴포넌트들로 취급하도록 React에게 요청하는 것이다.**

React는 (`userId`로 설정한) key가 변경될 때마다 DOM을 다시 생성하고 state를 재설정하며, `Profile` 컴포넌트 및 모든 자식들의 [state를 재설정](https://react-ko.dev/learn/preserving-and-resetting-state#option-2-resetting-state-with-a-key)할 것이다. 그 결과 `comment` 필드는 프로필들을 탐색할 때마다 자동으로 지워진다.

## **Adjusting some state when a prop changes**

> **props가 변경될 때 일부 state 조정하기**
> 

items 목록을 props로 전달 받고, selection state 변수에 선택된 항목을 유지하는 List 컴포넌트를 구현하려고 한다. 

items prop이 다른 배열을 받을 때마다 selection 을 null 로 재설정 하고 싶다.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 이러지 마세요: prop 변경시 Effect에서 state 조정
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

이렇게 작성하면 items가 변경될 때마다 List 와 그 하위 컴포넌트는 처음에는 오래된 selection 값으로 렌더링 된다. 그 다음 DOM이 업데이트 되고 Effects가 실행된다. 마지막으로 setSelection(null) 호출은 List와 자식 컴포넌트를 다시 렌더링 하여 전체 과정을 재시작한다. 

⇒ Effect를 삭제하는 대신 렌더링 중에 직접 state를 조정한다.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 더 나음: 렌더링 중에 state 조정
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

이렇게 작성하면 렌더링 React는 return문과 함께 종료된 직후에 List를 다시 렌더링 한다. 이 시점에서 React는 아직 List의 자식들을 렌더링 하거나 DOM을 업데이트 하지 않았기 때문에 List의 자식들을 기존의 selection 값에 대한 렌더링을 건너뛰게 된다.

렌더링 도중 컴포넌트를 업데이트 하면 React는 그 즉시 렌더링을 다시 시작한다. 단, 이때 동일한 컴포넌트의 state 만 업데이트 해야하며, 다른 컴포넌트의 state를 업데이트하면 에러가 발생한다. 또한 **무한 리렌더링을 방지하기 위해 `items !== prevItems` 같은 조건이 반드시 필요하다.**

이 패턴은 Effect를 보다 효율적으로 사용할 수 있지만 props나 state를 바탕으로 state를 조정하면 데이터 흐름을 이해하고 디버깅하기 어려워 질 수 있다. 

⇒ 항상 **key로 모든 state를 재설정하거나 렌더링 중에 모두 계산할 수 있는지 확인하자.**

선택한 item을 저장(및 재설정)하는 대신 선택한 item의 id를 저장할 수 있다.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);

  // ✅ 가장 좋음: 렌더링 중에 모든 값을 계산
  const selection = items.find(item => item.id === selectedId) ?? null;
  // ...
}
```

이렇게 작성하면 state를 조정할 필요가 없게된다. 

이 방식을 사용하면 대부분의 items 변경과 무관하게 slecetion 항목은 그대로 유지되므로 대체로 더 나은 방법이다. 

## **Sharing logic between event handlers**

> **이벤트 핸들러 간 로직 공유**
> 

예를 들어 장바구니 담기 버튼과 구매 버튼, 2가지 버튼이 있는 제품 페이지가 있다고 가정하자. 사용자가 제품을 장바구니에 넣을 때마다 알림을 표시하고 싶다. 두 버튼의 클릭 핸들러에 알림을 표시하기 위한 showNotification() 호출을 추가하는 것은 반복적으로 느껴져 이를 Effect에 배치하고 싶을 수 있다.

```jsx
function ProductPage({ product, addToCart }) {

  // 🔴 이러지 마세요: Effect 내부에 특정 이벤트에 대한 로직 존재
  **useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);**

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
```

여기서 Effect는 불필요하다. 

이렇게 코드를 작성하면 버튼을 클릭했을 때 뿐만 아니라, 페이지가 새로고침 하는 경우에도 불필요하게 알림이 계속 등장한다.

**코드를 Effect 내부에 작성 할지, 이벤트 핸들러에 작성할 지 결정할 때는 코드의 실행 목적에 대해 고민해보자. 컴포넌트가 사용자에게 표시되었기 때문에 실행되어야하는 코드만 Effect를 사용하자.**

위 예시 코드는 사용자가 버튼을 눌렀을 때만 알림을 표시해야 하므로 이벤트 핸들러에서 호출하는 함수에 넣어야 한다.

```jsx
function ProductPage({ product, addToCart }) {

  // ✅ 좋습니다: 이벤트 핸들러 안에서 각 이벤트별 로직 호출
  **function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }**

  function handleBuyClick() {
    **buyProduct();**
  }

  function handleCheckoutClick() {
    **buyProduct();**
    navigateTo('/checkout');
  }
  // ...
}
```

## **Sending a POST request**

> **POST요청 보내기**
> 
- 마운트 될 때 페이지 분석 이벤트를 보내는 POST 요청
- 양식하고 제출 버튼을 클릭하면 submit 하기 위한 POST 요청

```jsx
function Form() {
  const [firstName, setFirstName] = useState('');
  const [lastName, setLastName] = useState('');

  // ✅ Good: This logic should run because the component was displayed
  // ✅ 좋습니다: '컴포넌트가 표시되었기 때문에 로직이 실행되어야 하는 경우'에 해당
  useEffect(() => {
    post('/analytics/event', { eventName: 'visit_form' });
  }, []);

  **// 🔴 이러지 마세요: Effect 내부에 특정 이벤트에 대한 로직 존재**
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post('/api/register', jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }

	**// 상기의 잘못된 코드 개선**
	**// Effect 내부의 특정 이벤트에 대한 로직 제거**
  **// ✅ 좋습니다: 이벤트 핸들러 안에서 특정 이벤트 로직 호출**
	**function handleSubmit(e) {
	    e.preventDefault();

    post('/api/register', { firstName, lastName });
  }**

  // ...
}
```

## **Chains of computations**

> **연쇄 계산**
> 

다른 state를 바탕으로 또 다른 state를 조정하는 Effect를 연쇄적으로 사용하고 싶을 때도 있다. 

Effect를 연쇄적으로 사용할 경우 불필요한 리렌더링이 발생하게 된다. 

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 이러지 마세요: 오직 서로를 촉발하기 위해서만 state를 조정하는 Effect 체인
  useEffect(() => {
    if (card !== null && card.gold) {
      setGoldCardCount(c => c + 1);
    }
  }, [card]);

  useEffect(() => {
    if (goldCardCount > 3) {
      setRound(r => r + 1)
      setGoldCardCount(0);
    }
  }, [goldCardCount]);

  useEffect(() => {
    if (round > 5) {
      setIsGameOver(true);
    }
  }, [round]);

  useEffect(() => {
    alert('Good game!');
  }, [isGameOver]);

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    } else {
      setCard(nextCard);
    }
  }

  // ...
```

렌더링 중에 가능한 것을 계산하고 이벤트 핸들러에서 state를 조정하는 것이 좋다. 

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ 가능한 것을 렌더링 중에 계산
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ Calculate all the next state in the event handler
    // ✅ 이벤트 핸들러에서 다음 state를 모두 계산
    setCard(nextCard);
    if (nextCard.gold) {
      if (goldCardCount <= 3) {
        setGoldCardCount(goldCardCount + 1);
      } else {
        setGoldCardCount(0);
        setRound(round + 1);
        if (round === 5) {
          alert('Good game!');
        }
      }
    }
  }

  // ...
```

이벤트 핸들러 내부에서 state는 스냅샷처럼 동작한다. 

따라서 `setRound(round + 1)` 을 호출한 후에도 round 변수는 +1이 증가되기 전의 초기 값을 갖는다. state를 계산한 후 해당 값을 사용하려고 한다면 `const nextRound = round + 1` 과 같이 수동으로 정의해야 한다. 

## **Initializing the application**

> **애플리케이션 초기화하기**
> 

최초에 앱이 로드될 때 한 번만 실행되어야 하는 로직도 존재한다. 

React 에서 개발 중에는 함수가 두 번 실행된다. 이는 최상위 App 컴포넌트도 마찬가지이다. 따라서 한 번만 호출되어야 하는 로직은 인증 토큰이 무효화 되는 등의 문제가 발생할 수 있다. 

상용 환경에서는 다시 마운트되지 않을 수 있지만 모든 컴포넌트에서 동일한 제약 조건을 따르면 코드를 이동하고 재사용하기 훨씬 용이하므로 이런 경우에는 최상위 변수를 추가하여 실행 여부를 추적함으로써 문제를 해결할 수 있다. 

```jsx
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ Only runs once per app load
      // ✅ 앱 로드당 한 번만 실행됨
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}

// 더 나아가, 모듈 초기화 중이나 앱 랜더링 이전에 실행할 수도 있다.
if (typeof window !== 'undefined') { 
	// 브라우저에서 실행중인지 확인

  // ✅ Only runs once per app load
  // ✅ 앱 로드당 한 번만 실행됨
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

컴포넌트를 import 할 때 최상위 레벨의 코드는 렌더링되지 않더라도 일단 한 번 실행된다. 임의의 컴포넌트를 import 할때 속도 저하, 예상치 못한 동작을 방지하려면 이 패턴을 남용하는 것은 지양하는게 좋다. 

앱 전체 초기화 로직은 App.js와 같은 루트 컴포넌트 모듈 혹은 애플리케이션의 엔트리 포인트에 유지하자.

## **Notifying parent components about state changes**

> **state변경을 부모 컴포넌트에 알리기**
> 

자식 컴포넌트에서 state 변경이 발생할 때마다 부모 컴포넌트에 알리고 싶을 때 state 업데이트 이벤트를 prop으로 받고 자식 컴포넌트의 Effect에서 이를 호출하는 방법도 좋은 방법은 아니다.

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 이러지 마세요: onChange 핸들러가 너무 늦게 실행됨
  useEffect(() => {
    onChange(isOn);
  }, [isOn, onChange])

  function handleClick() {
    setIsOn(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      setIsOn(true);
    } else {
      setIsOn(false);
    }
  }

  // ...
}
```

⇒ 모든 동작이 단일 명령 안에서 처리 되도록 하는게 좋다. ( Effect 를 삭제하고 동일한 이벤트 핸들러에서 투 컴포넌트의 state 업데이트)

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {

    // ✅ 좋습니다: 이벤트 발생시 모든 업데이트를 수행
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    **updateToggle(!isOn);**
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      **updateToggle(true);**
    } else {
      **updateToggle(false);**
    }
  }

  // ...
}
```

⇒ Toggle 컴포넌트 및 부모 컴포넌트 모두 이벤트가 발생하는 동안 state를 업데이트 하므로 React는 서로 다른 컴포넌트에서 일괄 업데이트를 함께 처리하므로 렌더링 과정은 한 번만 발생한다.

```jsx
// ✅ 좋습니다: 부모 컴포넌트에 의해 완전히 제어됨
function Toggle({ isOn, onChange }) {
  function handleClick() {
    onChange(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      onChange(true);
    } else {
      onChange(false);
    }
  }

  // ...
}
```

⇒ state를 완전히 제거하고 그 대신 부모 컴포넌트를 받을 수 있다. (서로 다른 두 state 변수를 동기화 하려고 할 때 “state 끌어올리기”를 사용해보자.

## **Passing data to the parent**

> **부모에게 데이터 전달하기**
> 

자식 컴포넌트에서 데이터를 fetch 한 다음 부모 컴포넌트에 전달하는 경우를 조심하자.

```jsx
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 Avoid: Passing data to the parent in an Effect
  // 🔴 이러지 마세요: Effect에서 부모에게 데이터 전달
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}
```

React에서 데이터는 기본적으로 부모 컴포넌트에서 자식 컴포넌트로 흐른다. 자식 컴포넌트가 Effect 에서 부모 컴포넌트의 state를 업데이트 하면 데이터 흐름을 추적하기 어려워지므로 부모 컴포넌트에서 데이터를 fetch 해서 자식에서 전달하자.

```jsx
function Parent() {
  const data = useSomeAPI();
  // ...

  // ✅ 좋습니다: 자식에게 데이터 전달
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```

## **Subscribing to an external store**

> **외부 스토어 구독하기**
> 

서드파티 라이브러리나 브라우저 빌트인 API에서 데이터를 가져와야 하는 경우, React가 데이터 변경을 알아채지 못할 수도 있다. 이런 경우에는 수동으로 컴포넌트가 해당 데이터를 구독하도록 Effect에서 로직을 구현해야 한다. 

```jsx
function useOnlineStatus() {

  // 이상적이지 않음: Effect에서 수동으로 store 구독
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener('online', updateState);
    window.addEventListener('offline', updateState);
    return () => {
      window.removeEventListener('online', updateState);
      window.removeEventListener('offline', updateState);
    };
  }, []);
  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

```jsx
function subscribe(callback) {
  window.addEventListener('online', callback);
  window.addEventListener('offline', callback);
  return () => {
    window.removeEventListener('online', callback);
    window.removeEventListener('offline', callback);
  };
}

function useOnlineStatus() {

  **// ✅ 좋습니다: 빌트인 훅에서 외부 store 구독**
  return **useSyncExternalStore**(
    subscribe, 
          // React는 동일한 함수를 전달하는 한 다시 구독하지 않음
    () => navigator.onLine, 
          // 클라이언트에서 값을 가져오는 방법
    () => true 
          // 서버에서 값을 가져오는 방법
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

[React 컴포넌트에서 외부 store를 구독하는 방법 참고하기](https://react-ko.dev/reference/react/useSyncExternalStore)

## **Fetching data**

> **데이터 페칭하기**
> 

데이터 페칭은 일반적으로 Effect를 사용한다. 

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  **useEffect(() => {

    // 🔴 이러지 마세요: 클린업 없이 fetch 수행
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);**

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

위 예시코드는 버그가 존재하는 코드 “hello”를 빠르게 입력하면 query가 “h”에서 “he”, “hel”, “hell”, ”hello” 로 변경된다. 이런 경우 각각 페칭이 수행되지만 어떤 순서로 응답이 도착할지 보장할 수 없다. 이에 따라 마지막에 호출된 setResults()로부터 잘못된 검새결과가 표시될 수 있다. ⇒ 경쟁조건

**경쟁조건을 수정하기 위해 오래된 응답을 무시하도록 클린업 함수를 추가해야 한다.**

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    **let ignore = false;**
    fetchResults(query, page).then(json => {
      if (!ignore) {
        setResults(json);
      }
    });
    **return () => {
      ignore = true;
    };**
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

**최신 [프레임워크](https://react-ko.dev/learn/start-a-new-react-project#building-with-a-full-featured-framework)들은 컴포넌트에서 직접 Effect를 작성하는 것보다 더 효율적인 빌트인 데이터 페칭 메커니즘을 제공한다.**

데이터 페칭 로직을 커스텀 훅으로 추출하는 방법도 있다.

```jsx
function SearchResults({ query }) {
  const [page, setPage] = useState(1);
  const params = new URLSearchParams({ query, page });
  **const results = useData(`/api/search?${params}`);**

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}

**function useData(url) {
  const [data, setData] = useState(null);
  useEffect(() => {
    let ignore = false;
    fetch(url)
      .then(response => response.json())
      .then(json => {
        if (!ignore) {
          setData(json);
        }
      });
    return () => {
      ignore = true;
    };
  }, [url]);
  return data;
}**
```

⇒ **데이터 페칭 로직을 커스텀 훅으로 옮기면 나중에 효율적인 데이터 페칭 전략을 채택하기가 더 쉬워진다.**