# 7 - useEffect

# Effect로 동기화하기

https://ko.react.dev/learn/synchronizing-with-effects

일부 컴포넌트에서는 외부 시스템과 동기화해야할 수 있다. 예를들어 리액트의 state를 기준으로 React와 상관없는 구성요소를 제어하거나, 서버 연결을 설정하거나 구성요소가 화면에 나타날 때 분석 목적의 로그를 전송할 수 있다. Effect를 사용하면 렌더링 후 특정코드를 실행하여 react 외부의 시스템과 컴포넌트를 동기화할 수 있다.

## Effect란 무엇이고 이벤트와 어떻게 다를까?

effect에 대해 자세히 알아보기 전에 컴포넌트 내부의 2가지 로직 유형에 대해 알아야한다.

- 렌더링 코드를 주관하는 로직은 컴포넌트의 최상단에 위치하며 props와 state를 적절히 변형해 결과적으로 JSX를 반환한다. 렌더링 코드 로직은 순수해야한다. 수학 공식처럼 결과만 계산해야하고 그외에는 아무것도 하지 않는다.
- 이벤트 핸들러는 단순한 계산 용도가 아닌 무언가를 하는 컴포넌트 내부의 중첩함수이다. 이벤트 핸들러는 입력 필드를 업데이트하거나 제품을 구입하기 위해 HTTP POST 요청을 보내거나 사용자를 다른 화면으로 이동시킬 수 있다. 이벤트 핸들러에는 특정 사용자 입력(클릭 혹은 입력)으로 인해 발생하는 부수효과를 포함한다.

가끔은 이것만으로는 충분하지 않을 때가 있다.

예를들어 화면에 보일 때 마다 채팅 서버에 접속해야하는 ChatRoom 컴포넌트를 생각해보자. 서버에 접속하는 것은 순수한 계산이 아니고 부수효과를 발생시키기 때문에 렌더링중에는 할 수 없다. 하지만 클릭한번으로 ChatRoom이 표시되는 특정 이벤트는 하나도 없다.

Effect는 렌더링 자체에 의해 발생하는 부수 효과를 특정하는 것으로 특정이벤트가 아닌 렌더링에 의해 직접 발생한다. 채팅에서 메세지를 보내는 것은 이벤트이다. 왜냐하면 이건 사용자가 특정 버튼을 클릭함에 따라 직접적으로 발생한다. Effect는 커밋이 끝난 후에 화면 업데이트가 이루어지고 나서 실행된다. 이 시점이 React 컴포넌트를 외부 시스템과 동기화하기 좋은 타이밍이다.

<aside>
💡 이 텍스트에서 “Effect”는 위에서 언급한 React에 특화된 정의를 나타내며, 곧 렌더링에 의한 부수효과를 의미한다. 보다 일반적인 프로그래밍 개념을 언급할 때에는 ”부수효과“라고 말하겠다.

</aside>

## Effect가 필요 없을지도 모른다.

컴포넌트에 Effect를 무작정 추가하지 말자.

Effect는 주로 리액트 코드를 벗어난 특정 외부 시스템과 동기화하기 위해 사용된다. 이는 브라우저 API, 서드파티 위젯, 네트워크 등을 포함한다. 만약 순수하게 다른 상태에 기반하여 일부 상태를 조정하는 경우에는 Effect가 필요하지 않을 수 있다.

## Effect를 작성하는 방법

1. Effect 선언
2. Effect 의존성 지정
3. 필요한 경우 클린업 함수 추가

### 1단계: Effect 선언

기본적으로 Effect는 모든 렌더링 후에 실행된다.

```jsx
import { useEffect } from "react";

function MyComponent() {
  useEffect(() => {
    // 이 곳의 코드는 모든 렌더링 후에 실행된다
  });
  return <div />;
}
```

컴포넌트가 렌더링될 때마다 리액트는 화면을 업데이트한 다음 useEffect 내부의 코드를 실행한다. 다시말해 `useEffect`는 화면에 렌더링이 반영될 때까지 코드 실행을 “지연”시킨다.

이제부터 외부 시스템과 동기화하기 위해 어떻게 Effect를 사용할 수 있는지 알아보자. `<videoPlayer>`라는 React 컴포넌트를 살펴보자. 이 컴포넌트를 `isPlaying` 이라는 props를 통해 재생중인지 일시정지상태인지 제어하는 것이 좋아보인다.

```jsx
<VideoPlayer isPlaying={isPlaying} />
```

커스텀 VideoPlayer 컴포넌트는 내장 브라우저 `<video>`태그를 렌더링한다.

```jsx
function VideoPlayer({ src, isPlaying }) {
  // TODO: isPlaying을 활용하여 무언가 수행하기
  return <video src={src} />;
}
```

하지만 `<video>` 태그에는 isPlaying 이라는 prop을 받지 않늗나. 이를 제어하는 유일한 방법은 DOM 요소에서 수동으로 play() 및 pause() 메서드를 호출하는 것이다. isPlaying prop의 값(현재 비디오가 재생중인지 아닌지 여부)을 play() 및 pause()와 같은 호출과 동기화해야한다.

먼저 `<video>` DOM 노드의 ref를 가져와야한다.

play() 또는 pause()를 렌더링 중에 호출하려고 시도할 수 있겠지만 이는 올바른 접근이 아니다.

```jsx
import { useState, useRef, useEffect } from "react";

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    // useEffect 없이 호출하면 오류가 생긴다.
    if (isPlaying) {
      ref.current.play();
    } else {
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  return (
    <>
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? "일시정지" : "재생"}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

DOM 업데이트를 Effect로 감싸면 react가 화면을 업데이트한 다음에 Effect가 실행된다.

VideoPlayer 컴포넌트가 렌더링될 때 몇가지 일이 발생한다. 먼저 리액트는 화면을 업데이트하여 `<video>` 태그가 올바른 속성과 함께 DOM에 있는지 확인한다. 그런 다음 리액트는 useEffect를 실행한다. 마지막으로 Effect에서는 isPlaying 값에 따라 play() 또는 pause()를 호출한다.

이 예제에서 React 상태와 동기화된 “외부 시스템”은 브라우저 미디어 API였다. 이와 비슷한 접근 방식으로 리액트가 아닌 레거시코드(jQuery)를 선언적인 리액터 컴포넌트로 감싸는 데에도 사용할 수 있다.

실제로 비디오 플레이어를 제어하는 것은 훨씬 복잡하다. play()를 호출하는 것이 실패할 수 있으며, 사용자는 컴포넌트의 UI가 아닌 브라우저 내장 컨트롤을 사용하여 동영상을 재생 또는 일시정지할 수 있다. 이 예제는 매우 단순화되었고 불완전한 것임을 유의하자.

<aside>
💡 기본적으로 Effect는 모든 렌더링 후에 실행된다. 이러한 이유로 다음과 같은 코드는 무한 루프를 만들어 낸다
useEffect(() ⇒ {
  setCount(count + 1);
})

Effect는 렌더링의 결과로 실행된다. state를 설정하면 렌더링이 트리거된다. Effect 안에서 즉시 상태를 설정하는 것은 기계의 전원 플러그를 기계 그 자체에 연결하는 것과 비슷하다.
Effect는 일반적으로 컴포넌트를 외부 시스템과 동기화하는데 사용된다. 외부 시스템이 없고 다른 상태에 기반하여 조정하려는 경우에는 Effect가 필요하지 않을 수 있다

</aside>

### 2단계: Effect 의존성 저장하기

기본적으로 Effect는 모든 렌더링 후에 실행되지만 이는 가끔 원하는 동작이 아닐 수 있다.

- 때때로 느릴 수 있다.
  외부 시스템과 동기화하는 작업은 항상 즉시 이루어지진 않는다. 때문에 필요하지 않을 경우 실행을 건너뛰고 싶을 수 있다.
- 때때로 잘못될 수 있다.
  예를들어 모든 키 입력마다 컴포넌트 fade-in 애니메이션을 트리거하길 원하지 않늗나. 애니메이션은 컴포넌트가 처음 나타날 때에만 한 번 실행되어야 한다.

```jsx
import { useState, useRef, useEffect } from "react";

function VideoPlayer({ src, isPlaying }) {
  const ref = useRef(null);

  useEffect(() => {
    if (isPlaying) {
      console.log("video.play() 호출");
      ref.current.play();
    } else {
      console.log("video.pause() 호출");
      ref.current.pause();
    }
  });

  return <video ref={ref} src={src} loop playsInline />;
}

export default function App() {
  const [isPlaying, setIsPlaying] = useState(false);
  const [text, setText] = useState("");
  return (
    <>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <button onClick={() => setIsPlaying(!isPlaying)}>
        {isPlaying ? "일시 정지" : "재생"}
      </button>
      <VideoPlayer
        isPlaying={isPlaying}
        src="https://interactive-examples.mdn.mozilla.net/media/cc0-videos/flower.mp4"
      />
    </>
  );
}
```

위 예제에서는 input에 택스트를 입력할때마다 useEffect가 실행되는 것을 확인할 수 있다.

리액트에게 Effect를 불필요하게 다시 실행하지 않도록 지시하려면 useEffect 호출의 두 번째 인자로 의존성 배열을 지정하자.

```jsx
useEffect(() => {
  // ...
}, []);
```

위 코드에 적용한다면 isPlaying 에 대한 의존성이 누락되었다는 오류가 표시될 것이다. 문제는 Effect 내부의 코드가 어떤 작업을 수행할지 결정하기 위해 isPlaying prop에 의존하지만 이 의존성이 명시적으로 선언되지 않았다는 것이다. 이 문제를 해결하기 위해서는 의존성 배열에 isPlaying을 추가하면 된다.

```jsx
useEffect(() => {
  // ...
}, [isPlaying]);
```

이제 모든 의존성이 의존성 배열 안에 선언되어 오류가 없을 것이다. 의존성 배열로 지정하면 리액트에게 이전 렌더링 중에 의존성 배열에 있는 값이 이전과 동일하다면 useEffect를 다시 실행하지 않도록 해야한다고 알려준다.

의존성 배열에는 여러 개의 종속성을 포함할 수 있다. 리액트는 지정한 모든 종속성이 이전 렌더링의 값과 동일한 값을 가진 경우에만 Effect를 다시 실행하지 않는다. React는 `[Object.is](http://Object.is)` 비교를 사용하여 종속성 값을 비교한다.

의존성을 “선택”할 수 없다는 점에 유의하자. 의존성 배열에 지정한 종속성이 Effect 내부의 코드 기반으로 React가 기대하는 것과 일치하지 않으면 린트 에러가 발생한다. 이를 통해 코드 내의 많은 버그를 잡을 수 있다. 코드가 다시 실행되길 원하지 않는 경우 Effect 내부를 수정하여 그 종속성이 “필요”하지 않도록 만들자.( https://ko.react.dev/learn/lifecycle-of-reactive-effects#what-to-do-when-you-dont-want-to-re-synchronize )

<aside>
💡 의존성 배열이 없는 경우와 빈 의존성배열이 있는 경우의 동작이 다르다.

</aside>

```jsx
useEffect(() => {
  // 모든 렌더링 후에 실행됩니다
});

useEffect(() => {
  // 마운트될 때만 실행됩니다 (컴포넌트가 나타날 때)
}, []);

useEffect(() => {
 // 마운트될 때 실행되며, *또한* 렌더링 이후에 a 또는 b 중 하나라도 변경된 경우에도 실행됩니다
```

<aside>
💡 왜 ref는 의존성 배열에서 생략해도 되는가?
위의 예제에서는 useEffect 내에서 ref도 사용하지만 의존성 배열 안에서는 선언되지 않는다. 이것은 ref 객체가 안정된 식별성(stable identity)을 가지기 때문이다. 리액트는 동일한 useRef 호출에서 항상 같은 객체를 얻을 수 있음을 보장한다. 이 객체는 절대 변경되지 안힉 때문에 자체적으로 Effect를 다시 실행시키지 않는다. 따라서 ref는 의존성 배열에 포함하든 포함하지 않든 상관없고, 포함해도 문제가 되지 않는다.
useState로 반환되는 set 함수들도 안정된 식별성을 가지기 때문에 의존성에서 생략된다. 린터가 의존성을 생략해도 오류를 표시하지 않는다면 그렇게해도 안전하다.
안정된 식별성을 가진 의존성을 생략하는 것은 린터가 해당 객체가 안정적임을 “알 수“ 있는 경우에만 작동한다. 예를들어 ref가 부모 컴포넌트에서 전달되었다면 의존성 배열에 명시해야한다. 이것은 좋은 접근방식이다. 왜냐면 부모컴포넌트가 항상 동일한 ref를 전달하는지 또는 여러 ref중 하나를 조건부로 전달하는지 알 수 없기 때문이다.

</aside>

### 3단계: 필요하다면 클린업을 추가하자

예를들어 채팅 서버를 연결하는 코드를 useEffect 내에 넣는다고 생각해보자. console.log와 함께 출력되도록 작성한다면 개발모드에서는 콘솔이 2번 찍히는 것을 확인 할 수 있다. 이는 리액트가 개발모드에서 초기 마운트 후에 모든 컴포넌트를 다시 한번 마운트 시키기 때문이다. 하지만 초기 마운트에서 실행되었던 채팅서버의 연결이 끊기고 있지 않은 상태이다. 이 문제를 해결하려면 Effect에서 클린업 함수를 반환하면 된다.

```jsx
useEffect(() => {
  // ...
  return () => {
    // cleanup
  };
}, []);
```

리액트는 Effect가 다시 실행되기 전마다 클린업 함수를 호출하고 컴포넌트가 언마운트(제거)될 때에도 마지막으로 호출한다.

## 개발 중에 Effect가 두번 실행되는 경우를 다루는 방법

React는 버그를 찾기 위해 개발 중에는 컴포넌트를 명시적으로 다시 마운트한다. “Effect를 한번만 실행하는 방법”이 아니라 ”어떻게 Effect가 다시 마운트 된 후에도 작동하도록 고칠것인가“라는 것이 옳은 질문이다.

일반적으로 정답은 클린업 함수를 구현하는 것이다. 클린업함수는 Effect가 수행하던 작업을 중단하거나 되돌리는 역할을 한다. 기본 원칙은 사용자가 Effect가 한번 실행되는 것과 설정 → 클린업 → 설정 의 순서간에 차이를 느끼지 못해야한다.

## React로 작성되지 않는 위젯 제어하기

```jsx
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

가끔 React로 작성되지 않은 UI 위젯을 추가해야 할 때가 있다. 예를들어 페이지에 지도 컴포넌트를 추가해본다고 해보자. 이 지도 컴포넌트에는 setZoomLevel() 메서드가 있으며 zoomLevel 상태 변수와 동기화하려고 할 것이다.

이 경우에는 클린업이 필요가 없다. 개발모드에선 두번 호출되지만 같은 값으로 메서드를 두번 호출 하는 것은 아무런 문제가 되지 않는다.

일부 API는 연속해서 두번 호출하는 것을 허용하지 않을 수 있는데 예를 들어 내장된 <dialog> 요소의 showModal 메서드는 두번 호출되면 예외를 던진다. 이런 경우는 클린업 함수를 구현하고 이 함수에서 대화상자를 닫도록 하자.

## 이벤트 구독하기

만약 Effect가 어떤것을 구독한다면 클린업 함수에서 구독을 해지해야한다.

```jsx
useEffect(() => {
  const map = mapRef.current;
  map.setZoomLevel(zoomLevel);
}, [zoomLevel]);
```

개발 중에는 Effect가 addEventListener()를 호출한 다음 즉시 removeEventListener()를 호출하고 그 다음 동일한 핸들러로 addEventListener()를 호출한다. 따라서 한번에 하나의 활성 구독만 있게 된다. 이것은 프로덕션 환경에서 한번 addEventListener()를 호출하는 것과 동일한 동작을 가진다.

## 애니메이션 트리거

Effect가 어떤 요소를 애니메이션으로 표시하는 경우, 클린업 함수에서 애니메이션을 초기값으로 재설정해야 한다.

```jsx
useEffect(() => {
  const node = ref.current;
  node.style.opacity = 1; // Trigger the animation
  return () => {
    node.style.opacity = 0; // Reset to the initial value
  };
}, []);
```

## 데이터 페칭

만약 Effect가 어떤 데이터를 가져온다면 클린업 함수에서는 fetch를 중단하거나 결과를 무시해야한다.

```jsx
useEffect(() => {
  let ignore = false;

  async function startFetching() {
    const json = await fetchTodos(userId);
    if (!ignore) {
      setTodos(json);
    }
  }

  startFetching();

  return () => {
    ignore = true;
  };
}, [userId]);
```

이미 발생한 네트워크 요청을 “실행취소”할 수는 없지만 클린업 함수는 더 이상 관련이 없는 페치가 애플리케이션에 계속 영향을 미치지 않도록 보장해야한다.

개발중에는 네트워크 탭에서 두 개의 페치가 표시된다. 이는 문제가 없고 프로덕션 환경에서는 하나의 요청만 있을 것이다. 개발 중에 두번재 요청이 문제라면 가장 좋은 방법은 중복 요청을 제거하고 컴포넌트간에 응답을 캐시하는 솔류션을 사용하는 것이다.

```jsx
function TodoList() {
  const todos = useSomeDataLibrary(`/api/user/${userId}/todos`);
  // ...
```

이렇게 하면 개발 환경을 개선하는데 도움이 될 뿐만 아니라 애플리케이션의 반응 속도도 향상된다. 예를 들어 사용자가 뒤로가기 버튼을 눌렀을 때 데이터를 다시 로드하는 것을 기다릴 필요가 없다. 데이터가 캐시되기 때문이다. 이러한 캐시를 직접 구축하거나 비슷한 효과를 누릴 수 있는 여러 대안 중 하나를 사용할 수 있다.

<aside>
💡 Effect에서 데이터를 가져오는 좋은 대안은?

Effect 안에서 fetch 호출을 작성하는 것은 데이터를 가져오는 인기있는 방법이다. 특히 완전히 클라이언트 측에 있는 앱에서. 하지만 이는 수동적인 접근 방식이면서 중요한 단점이 있다.

- Effect는 서버에서 실행되지 않는다.
  ⇒ 따라서 초기 서버 렌더링된 HTML은 데이터가 없는 로딩상태를 포함하고 클라이언트 컴퓨터는 모든 JavaScript를 다운로드하고 앱을 렌더링해야만 로드해야 한다는 것을 알게 될 것이다. 이는 효율적이지 않다.
- Effect 안에서 직접 가져오면 “네트워크 폭포”를 쉽게 만들 수 있다.
  ⇒ 부모 컴포넌트에서 렌더링하면 일부 데이터를 가져오고 그 자식 컴포넌트를 렌더링한 다음 그들이 데이터를 가져오기 시작한다. 네트워크가 빠르지 않으면 병렬로 가져오는 것 보다 훨씬 느리다.
- Effect 안에서 직접 가져오는 것은 일반적으로 데이터를 미리 로드하거나 캐시하지 않음을 의미한다.
- 그리 편리하지 않다.
  ⇒ fetch 호출을 작성할 때 경쟁 상태와 같은 버그에 영향을 받지 않는 방식으로 작성하는데 꽤 많은 보일러플레이트 코드가 필요하다.

이 단점 목록은 리액트에만 해당하는 것이 아니다. 어떤 라이브러리든 마운트 시에 데이터를 가져온다면 비슷한 단점이 존재한다. 따라서 다음 접근방식을 권장한다.

- 프레임워크를 사용하는 경우 해당 프레임워크의 내장 데이터 페칭 매커니즘을 사용하자.
  ⇒ 현대적인 리액트 프레임워크에는 위 단점을 겪지 않는 효율적이고 통합적인 데이터 페칭 매커니즘이 포함되어있다.
- 그렇지 않은 경우 클라이언트 측 캐시를 사용하거나 구축하는 것을 고려하자.
  ⇒ 대표적으로 React Query, useSWR 등이 있다.
  하지만 이런 접근방식 중 어느 것도 적합하지 않은 경우 Effect 내에서 데이터를 직접 가져오는 것을 계속 해도 된다.

</aside>

## 분석 보내기

방문 로그 코드를 생각해보자

```jsx
useEffect(() => {
  logVisit(url);
}, [url]);
```

개발환경에서 두번 실행되겠지만 우리는 이 코드를 그대로 유지하는 것을 권장한다. 실제로 개발환경에서는 아무 작업도 수행하지 않아야한다.

제품환경에서는 중복된 방문 로그가 없을 것이다.

더 정밀한 분석을 위해 Intersection Observer를 사용하면 어떤 컴포넌트가 뷰포트에 있는지와 얼마나 오래 보이는지 추적하는데 도움이 될수 있다.

## Effect가 아닌 경우: 애플리케이션 초기화

일부 로직은 애플리케이션 시작시에 한번만 실행되어야 한다. 이러한 로직은 컴포넌트 외부에 배치할 수 있다.

```jsx
if (typeof window !== "undefined") {
  // 브라우저에서 실행 중인지 확인합니다.
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

이러한 로직은 브라우저가 페이지를 로드한 후 한번만 실행됨이 보장된다.

## Effect가 아닌 경우: 제품 구입하기

가끔 클린업 함수를 작성하더라도 Effect가 두번실행되는 것에 대해 방지할 방법이 없을 수 있다.

```jsx
useEffect(() => {
  // 🔴 잘못된 방법: 이 Effect는 개발 환경에서 두 번 실행되며 코드에 문제가 드러납니다.
  fetch("/api/buy", { method: "POST" });
}, []);
```

사용자는 제품을 두 번 구매하고 싶지 않을 것이다. 이런 로직을 Effect에 넣지 않아야하는 이유이다. 다른페이지로 갔다가 돌아오는 경우에도 실행되어 버린다.

구매와 같은 로직은 렌더링에 의해 발생하는 것이 아니라 특정 상호 작용에 의해 발생한다. (버튼을 누른다던가) 따라서 이러한 로직은 Effect가 아닌 이벤트 핸들러로 이동하는 편이 좋다.

만약 컴포넌트를 다시 마운트 했을 때 애플리케이션의 논리가 깨진다면 기존에 존재했던 버그가 드러난 것이다. 사용자 관점에서 페이지를 방문하는 것과 페이지를 방문하고 링크를 클릭한 다음 뒤로 가기 버튼을 누르는 것에는 차이가 없어야 한다.

## 위에 설명한 모든 것들을 적용해보기

각 Effect는 해당 렌더의 text 값을 “캡쳐”한다. 이런 작동 방식에 대해서 궁금하다면 클로저에 대해 읽어보자.( https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures )

---

# Effect가 필요하지 않을 수 있다

https://ko.react.dev/learn/you-might-not-need-an-effect

Effect는 React 패러다임에서 벗어날 수 있는 탈출구이다. Effect를 사용하면 리액트를 벗어나 컴포넌트를 리액트가 아닌 위젯, 네트워크, 브라우저 DOM과 같은 외부 시스템과 동기화할 수 있다. 외부 시스템이 관여하지 않는 경우(예를 들어 일부 prop 또는 state가 변경될 때 컴포넌트의 state를 업데이트 하려는 경우), Effect가 필요하지 않다. 불필요한 Effect를 제거하면 코드를 더 쉽게 따라갈 수 있고, 실행속도가 빨라지며 에러 발생 가능성이 줄어든다.

## 불필요한 Effect를 제거하는 방법

Effect가 필요하지않은 두 가지 일반적인 경우가 있다.

- 렌더링을 위해 데이텉를 변환하는데 Effect가 필요하지 않다.
  예를들어 리스트를 표시하기 전에 필터링을 하고싶을 때 Effect를 사용하는 것은 비효율적이다. 렌더링 후에 Effect에서 또 state를 업데이트 해야한다면 전체 프로세스가 처음부터 다시 시작된다. 불필요한 렌더링 패스를 피하려면 컴포넌트의 최상위 레벨에서 모든 데이터를 변환하자.
- 사용자 이벤트를 핸들링하는데 Effect가 필요하지 않다.
  Effect는 실행될 때 까지 사용자가 무엇을 했는지 알 수 없기 때문에 일반적으로 해당되는 이벤트 핸들러에서 사용자 이벤트를 핸들링 하자.

## props 또는 state에 따라 state 업데이트하기

기존 props나 state에서 계산할 수 있는 것이 있으면 그것을 state에 넣지 말자. 대신 렌더링중에 계산하도록 하자

## 비용이 많이 드는 계산 캐싱하기

props로 받은 todos 를 filter prop에 따라 필터링하여 visibleTodos를 계산한다고 했을 때, 결과를 state에 저장하고 Effect에서 업데이트하고 싶을 수 있다. 하지만 이는 불필요하고 비효율적이다.

이는 state와 Effect를 제거하는 것 만으로 괜찮다. 하지만 필터링이 느리거나 todos가 많은 경우 관련없는 state가 변경된 경우 다시 계산되는 것은 비효율적이다. 이런 경우에는 useMemo로 래핑해서 값비싼 계산을 캐시할 수 있다.

```jsx
import { useMemo, useState } from "react";

function TodoList({ todos, filter }) {
  const [newTodos, setNewTotos] = useState("");
  //
  const visibleTodos = useMemo(
    () => getFilteredTodos(todos, filter),
    [todos, filter]
  );
  // ...
}
```

이렇게 하면 todos나 filter가 변경되지 않는 한 내부 함수가 다시 실행되지 않기를 원한다는 것을 react에게 알린다. 리액트는 초기 렌더링 중에 해당 함수의 반환값을 기억한다. 다음 렌더링 중에 todos나 filter가 다른지 확인하고 지난 번과 동일하다면 useMemo는 마지막으로 저장한 결과를 반환한다.

useMemo로 감싸는 함수는 렌더링 중에 실행되므로 순수한 계산에만 작동한다.

<aside>
💡 계산이 비싼지 알 수 있는 방법
- console.time() , console.timeEnd() 를 사용
일반적으로는 컴퓨터가 빠를 수 있기 때문에 개발자도구에서 CPU 스로틀링 옵션을 사용해보자.
또한 개발 중에 성능을 측정하는 것은 가장 정확한 결과를 제공하지 않는다는 점을 유의하자(strict mode)

</aside>

## prop 변경 시 모든 state 초기화

어떠한 컴포넌트에서 id를 props로 받아온다고 생각해보자. 어떠한 사용자가 댓글을 입력하려고 텍스트를 입력한 상태에서 사용자가 다른 프로필로 이동했을 경우 댓글이 재설정되지 않고 이전에 입력했던 값이 남아있다면 실수로 잘못된 사용자의 프로필에 댓글을 달기 쉽다.

이런 문제를 해결하기위해 userId가 변경될 때마다 comment 상태 변수를 비우기위해 Effect를 사용하려고 한다. 이는 값이 변경될때마다 다시 Effect를 통해 다시 렌더링되기 때문에 비효율적이다.

이러한 경우에는 명시적인 key를 전달하여 사용자의 프로필이 개념적으로 다른 프로필임을 React에 알릴 수 있다.

## prop이 변경될 때 일부 state 조정하기

List 컴포넌트는 items 목록을 props으로 받고 selection state 변수에 선택된 item을 유지한다. items prop이 다른 배열을 받을 때마다 selection을 null로 재설정하고 싶다고 가정해보자.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 🔴 피하세요: Effect에서 prop 변경 시 state 조정하기
  useEffect(() => {
    setSelection(null);
  }, [items]);
  // ...
}
```

이렇게 Effect를 이용해서 일부 조정할 수 있겠지만 이 역시 이상적이지 않다. Effect를 제거하고 렌더링 중에 직접 state를 조정해보도록 수정해보자.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selection, setSelection] = useState(null);

  // 더 좋습니다: 렌더링 중 state 조정
  const [prevItems, setPrevItems] = useState(items);
  if (items !== prevItems) {
    setPrevItems(items);
    setSelection(null);
  }
  // ...
}
```

이렇게 이전 렌더링의 정보를 저장하는 것은 이해하기 어려울 수 있지만 Effect에서 동일한 state를 업데이트하는 것보다 낫다. 렌더링 도중 컴포넌트를 업데이트하면 React는 반환된 JSX를 버리고 즉시 렌더링을 다시 시도한다. 매우 느린 연속적 재시도를 피하기 위해서는 리액트는 렌더링 중에 동일한 컴포넌트의 state만 업데이트할 수 있도록 한다. 렌더링 도중 다른 컴포넌트의 state를 업데이트하면 에러가 발생한다. 반복을 피하려면 `item !== prevItem` 과 같은 조건이 필요하다. 이런식으로 state를 조정할 수는 있지만 컴포넌트를 순수하게 유지하기 위해 DOM 변경이나 타임아웃 설정과 같은 다른 사이드 이펙트들은 이벤트 핸들러나 Effect에 남겨두어야 한다.

이 패턴이 Effect보다 더 효율적이지만 대부분의 컴포넌트에는 이 패턴이 필요하지 않다. 어떻게 하든 props나 다른 state에 따라 state를 조정하면 데이터 흐름을 이해하고 디버깅하기 어려워진다. 대신 key를 사용하여 모든 state를 초기화하거나 렌더링 중 모든 state를 계산할 수 있는지 항상 확인해보자. 예를들어 선택한 item을 저장(및 초기화)하는 대신 선택한 item ID를 저장할 수 있다.

```jsx
function List({ items }) {
  const [isReverse, setIsReverse] = useState(false);
  const [selectedId, setSelectedId] = useState(null);
  // ✅ 최고예요: 렌더링 중에 모든 것을 계산
  const selection = items.find((item) => item.id === selectedId) ?? null;
  // ...
}
```

이제 state를 “조정”할 필요가 전혀 없다. 선택한 ID를 가진 item이 목록에 있으면 선택된 state로 유지된다.

(?)

## 이벤트 핸들러 간 로직 공유

**어떤 코드가 Effect에 있어야 하는지 이벤트 핸들러에 있어야 하는지 확실하지 않은 경우 이 코드가 실행되어야 하는 *이유*를 자문해 보자. 컴포넌트가 사용자에게 표시되었기 _때문에_ 실행되어야 하는 코드에만 Effect를 사용하자.**

사용자 입력(클릭 혹은 텍스트입력)에 의해 동작해야하는 로직의 경우에는 Effect를 삭제하고 공유 로직을 두 이벤트 핸들러에서 호출되는 함수에 넣자.

```jsx
function ProductPage({ product, addToCart }) {
  // ✅ 좋습니다: 이벤트 핸들러에서 이벤트별 로직이 호출됩니다.
  function buyProduct() {
    addToCart(product);
    showNotification(`Added ${product.name} to the shopping cart!`);
  }

  function handleBuyClick() {
    buyProduct();
  }

  function handleCheckoutClick() {
    buyProduct();
    navigateTo("/checkout");
  }
  // ...
}
```

## POST 요청 보내기

```jsx
function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");

  // ✅ 좋습니다: 컴포넌트가 표시되었으므로 이 로직이 실행되어야 합니다.
  useEffect(() => {
    post("/analytics/event", { eventName: "visit_form" });
  }, []);

  // 🔴 피하세요: Effect 내부의 이벤트별 로직
  const [jsonToSubmit, setJsonToSubmit] = useState(null);
  useEffect(() => {
    if (jsonToSubmit !== null) {
      post("/api/register", jsonToSubmit);
    }
  }, [jsonToSubmit]);

  function handleSubmit(e) {
    e.preventDefault();
    setJsonToSubmit({ firstName, lastName });
  }
  // ...
}
```

첫 analytics POST 요청은 Effect에 남아있어야한다. 이는 이벤트를 전송하는 이유가 폼이 표시되었기 때문이다.

하지만 두번째 POST 요청의 경우는 표시되는 폼으로 인해 발생하는 것이 아니라 사용자가 버튼을 누를 때 라는 특정 시점에만 요청을 보내려고 한다. 이 요청은 특정 상호작용에서만 발생해야한다. 이런 경우 Effect를 삭제하고 POST 요청을 이벤트 핸들러로 이동한다.

```jsx
function Form() {
  const [firstName, setFirstName] = useState("");
  const [lastName, setLastName] = useState("");

  // ✅ 좋습니다: 컴포넌트가 표시되었으므로 이 로직이 실행됩니다.
  useEffect(() => {
    post("/analytics/event", { eventName: "visit_form" });
  }, []);

  function handleSubmit(e) {
    e.preventDefault();
    // ✅ 좋습니다: 이벤트별 로직은 이벤트 핸들러에 있습니다.
    post("/api/register", { firstName, lastName });
  }
  // ...
}
```

어떤 로직을 이벤트 핸들러에 넣을지 Effect 에 넣을지 선택할 때 사용자 관점에서 “어떤 종류의 로직인지”에 대한 답을 찾아야 한다. 이 로직이 특정 상호작용으로 발생하는 것인지(이벤트 핸들러), 사용자가 화면에서 컴포넌트를 보는것이 원인인지(Effect) 체크해보자.

## 연쇄 계산

때때로 다른 state에 따라 각각 state를 조정하는 Effect를 체이닝하고 싶을 때가 있을 수 있다.

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);
  const [isGameOver, setIsGameOver] = useState(false);

  // 🔴 피하세요: 서로를 트리거하기 위해서만 state를 조정하는 Effect 체인
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

이 코드에는 두가지 문제점이 있다.

한가지 문제는 매우 비효율적이라는 점이다. 컴포넌트는 체인의 각 set 호출사이 마다 다시 렌더링해야한다. 위 예시에서 최악의 경우 불필요한 리렌더링이 3번 발생한다.

또한 속도가 느리지 않더라도 코드가 발전하다보면 ”체인“이 새로운 요구 사항에 마짖 않는 경우가 발생할 수 있다. 따라서 이러한 코드는 융통성이 없고 취약한 경우가 많다.

이 경우 렌더링 중 가능한 것을 계산하고 이벤트 핸들러에서 state를 조정하는 것이 좋다.

```jsx
function Game() {
  const [card, setCard] = useState(null);
  const [goldCardCount, setGoldCardCount] = useState(0);
  const [round, setRound] = useState(1);

  // ✅ 렌더링 중에 가능한 것을 계산합니다.
  const isGameOver = round > 5;

  function handlePlaceCard(nextCard) {
    if (isGameOver) {
      throw Error('Game already ended.');
    }

    // ✅ 이벤트 핸들러에서 다음 state를 모두 계산합니다.
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

훨씬 더 효율적이다. 또한 게임 기록을 볼 수 있는 방법을 구현하면 이제 다른 모든 값을 조정하는 Effect 체인을 트리거하지 않고도 각 state 변수를 과거의 행동으로 설정할 수 있다. 여러 이벤트 핸들러 간에 로직을 재사용해야하는 경우 함수를 추출하여 해당 핸들러에서 호출할 수 있다.

이벤트 핸들러 내부에서 state는 스냅샷처럼 동작한다는 점을 기억하자.

이벤트 핸들러에서 직접 다음 state를 계산할 수 없는 경우도 있다. 이런 경우는 네트워크와 동기화가 필요한 경우에 Effect 체인이 적절하다고 볼 수 있다.

## 애플리케이션 초기화

일부 로직은 앱이 로드될때 한번만 실행되어야한다.

그것을 최상위 컴포넌트의 Effect에 배치하고 싶을 수도 있다.

하지만 이 함수가 개발중에 두번 실행된다는 사실을 금방 알 수 있다. 함수가 두번 호출되도록 설계되지 않았기 때문에 인증 토큰이 무효화되는 등의 문제가 발생할 수 있다. 프로덕션환경에서 실제로 다시 마운트되지 않을 수 있지만 모든 컴포넌트에서 동일한 제약 조건을 따르면 코드를 이동하고 재사용하기 더 쉬워진다. 일부 로직이 컴포넌트 마운트당 한번이 아니라 앱 로드당 한번 실행되어야하는 경우 최상위 변수를 추가하여 이미 실행되었는지를 추적해보자.

```jsx
let didInit = false;

function App() {
  useEffect(() => {
    if (!didInit) {
      didInit = true;
      // ✅ 앱 로드당 한 번만 실행
      loadDataFromLocalStorage();
      checkAuthToken();
    }
  }, []);
  // ...
}
```

혹은 모듈 초기화 중이나 앱이 렌더링되기 전에 실행할수도 있다.

```jsx
if (typeof window !== "undefined") {
  // 브라우저에서 실행 중인지 확인합니다.
  // ✅ 앱 로드당 한 번만 실행
  checkAuthToken();
  loadDataFromLocalStorage();
}

function App() {
  // ...
}
```

컴포넌트를 import할 때 최상위 레벨의 코드는 렌더링되지 않더라도 한번 실행된다. 임의의 컴포넌트를 import할 때 속도 저하나 예상치 못한 동작을 방지하려면 이 패턴을 과도하게 사용하지 말자.

## state 변경을 부모 컴포넌트에게 알리기

자식 컴포넌트에서 특정 상태 변수의 변화를 부모 컴포넌트에 알리려 할 때 useEffect에서 이를 처리할 수 있도록 작성할 수 있지만 이것도 마찬가지로 이상적인 방법이 아니다.

```jsx
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  function updateToggle(nextIsOn) {
    // ✅ 좋습니다: 업데이트를 유발한 이벤트가 발생한 동안 모든 업데이트를 수행합니다.
    setIsOn(nextIsOn);
    onChange(nextIsOn);
  }

  function handleClick() {
    updateToggle(!isOn);
  }

  function handleDragEnd(e) {
    if (isCloserToRightEdge(e)) {
      updateToggle(true);
    } else {
      updateToggle(false);
    }
  }

  // ...
}
```

```jsx
// ✅ 이것도 좋습니다: 컴포넌트는 부모에 의해 완전히 제어됩니다.
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

state 끌어올리기를 적용할 수도있다.

부모 컴포넌트에 더 많은 로직을 포함해야 전체적으로 걱정해야하는 state는 줄어든다. 두개의 서로 다른 state 변수를 동기화하려고 하는 경우 state 끌어올리기를 사용해보도록 하자.

## 부모에데 데이터 전달하기

자식 컴포넌트에서 데이터를 받아와 부모 컴포넌트에 전달하려고 하는 경우를 조심하도록하자.

```jsx
function Parent() {
  const [data, setData] = useState(null);
  // ...
  return <Child onFetched={setData} />;
}

function Child({ onFetched }) {
  const data = useSomeAPI();
  // 🔴 피하세요: Effect에서 부모에게 데이터 전달하기
  useEffect(() => {
    if (data) {
      onFetched(data);
    }
  }, [onFetched, data]);
  // ...
}
```

리액트에서 데이터는 부모 컴포넌트에서 자식 컴포넌트로 흐른다. 화면에 뭔가 잘못된 것이 보이면 컴포넌트 체인을 따라 올라가서 어떤 컴포넌트가 잘못된 prop을 전달하거나 잘못된 state를 가진지 찾아내면 정보 출처를 추적할 수 있다. 자식 컴포넌트가 Effect에서 부모 컴포넌트의 state를 업데이트하면 데이터 흐름을 추적하기가 매우 어려워진다. 자식과 부모 모두 동일한 데이터가 필요한 경우 부모 컴포넌트가 해당 데이터를 가져와서 자식에게 내려주도록 하자.

```jsx
function Parent() {
  const data = useSomeAPI();
  // ...
  // ✅ 좋습니다: 자식에서 데이터를 전달
  return <Child data={data} />;
}

function Child({ data }) {
  // ...
}
```

## 외부 저장소 구독하기

때로는 컴포넌트가 React state 외부의 일부 데이터를 구독해야할 수 있다. 이 데이터는 서드파티 라이브러리 또는 내장 브라우저 API에서 가져올 수 있다. 이 데이터는 React가 모르는 사이 변경될 수 있으므로 컴포넌트를 수동으로 구독해야한다. 이 작업은 종종 Effect를 통해 수행된다.

```jsx
function useOnlineStatus() {
  // 이상적이지 않습니다: Effect에서 저장소를 수동으로 구독
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function updateState() {
      setIsOnline(navigator.onLine);
    }

    updateState();

    window.addEventListener("online", updateState);
    window.addEventListener("offline", updateState);
    return () => {
      window.removeEventListener("online", updateState);
      window.removeEventListener("offline", updateState);
    };
  }, []);
  return isOnline;
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

이를 위해 Effect를 사용하는 것이 일반적이지만 React에는 외부 저장소를 구독하기 위해 특별히 제작된 Hook이 있다. Effect를 삭제하고 `[useSyncExternalStore](https://ko.react.dev/reference/react/useSyncExternalStore)`에 대한 호출로 대체한다.

```jsx
function subscribe(callback) {
  window.addEventListener("online", callback);
  window.addEventListener("offline", callback);
  return () => {
    window.removeEventListener("online", callback);
    window.removeEventListener("offline", callback);
  };
}

function useOnlineStatus() {
  // ✅ 좋습니다: 내장 Hook으로 외부 스토어 구독하기
  return useSyncExternalStore(
    subscribe, // 동일한 함수를 전달하는 한 React는 다시 구독하지 않습니다.
    () => navigator.onLine, // 클라이언트에서 값을 얻는 방법
    () => true // 서버에서 값을 얻는 방법
  );
}

function ChatIndicator() {
  const isOnline = useOnlineStatus();
  // ...
}
```

이러한 접근 방식은 변경가능한 데이터를 Effect를 사용해 React state에 수동으로 동기화하는 것 보다 에러가 덜 발생한다. 일반적으로 위의 useOnlineStatus()와 같은 사용자 정의 Hook을 작성하여 개별 컴포넌트에서 이 코드를 반복할 필요가 없도록 한다.

## 데이터 가져오기

많은 앱들이 데이터 가져오기를 시작하기위해 Effect를 사용한다. 이와 같은 데이터를 가져오는 Effect를 작성하는 것은 매우 일반적이다.

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 피하세요: 정리 로직 없이 가져오기
    fetchResults(query, page).then((json) => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

데이터 가져오기를 이벤트 핸들러로 옮길 필요는 없다.

앞서 했던 말과 모순되어 보일 수 있지만 데이터를 가져오기를 해야하는 주된 이유가 입력이벤트가 아니라는 점을 고려하자. 검색 입력은 URL에서 미리 채워지는 경우가 많으며 사용자는 입력을 건드리지 않고 뒤로 앞으로 탐색할 수 있다.

page와 query의 출처가 어디인지는 중요하지 않다. 이 컴포넌트가 표시되는 동안에는 현재 page 및 query 에 대한 네트워크와 데이터와 result를 동기화하고 싶을 것이다. 이것이 바로 Effect의 이유이다.

하지만 위의 코드에는 버그가 있다. 사용자가 입력을 빠르게 하는 경우 각각의 요청이 어떤 순서로 응답이 올거라는 보장이 없다는 것이다. (hello의 경우 hello 응답 후 hell의 응답이 올 수 있다) 따라서 setResults()를 마지막으로 호출하므로 잘못된 검색 결과가 표시될 수 있다. 이를 “경쟁 조건”이라고 하는데 서로 다른 두 요청이 “경쟁”하여 에상과 다른 순서로 도착하는 것을 말한다.

경쟁조건을 수정하려면 오래된 응답을 무시하는 정리함수를 추가해야한다.

```jsx
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);
  useEffect(() => {
    let ignore = false;
    fetchResults(query, page).then((json) => {
      if (!ignore) {
        setResults(json);
      }
    });
    return () => {
      ignore = true;
    };
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```

이렇게하면 Effect가 데이터를 가져올 때 마지막으로 요청된 응답을 제외한 모든 응답이 무시된다.

데이터를 가져오기를 구현할 때 경쟁조건 외에도 처리해야하는 것들이 많다. 응답 캐싱(사용자가 뒤로가기 버튼을 클릭하여 이전 화면을 즉시 볼 수 있도록), 서버에서 데이터를 가져오는 방법(초기 서버 렌더링 HTML에 스피너 대신 가져온 콘텐츠가 포함되도록), 네트워크 워터풀을 피하는 방법(자식이 모든 부모를 기다리지 않고 데이터를 가져올 수 있도록)]도 고려해야한다.

**이러한 문제는 React뿐만 아니라 모든 UI 라이브러리에 적용된다. 이러한 문제를 해결하는 것은 간단하지 않기 때문에 모던 [프레임워크](https://ko.react.dev/learn/start-a-new-react-project#production-grade-react-frameworks)는 Effect에서 데이터를 가져오는 것보다 더 효율적인 내장 데이터 가져오기 메커니즘을 제공한다.**

프레임워크를 사용하지 않고(그리고 직접 빌드하고 싶지 않고) Effect에서 데이터를 보다 인체공학적으로 가져오고 싶다면 이 예시처럼 가져오기 로직을 사용자 정의 Hook으로 추출하는 것을 고려하자.

또한 에러 핸들링과 콘텐츠 로딩 여부를 추적하기위한 로직을 추가하고 싶을 것이다. 이와 같은 Hook을 직접 빌드하거나 리액트 에코시스템에서 이미 사용한 많은 솔루션 중 하나를 사용할 수 있다. **이 방법만으로는 프레임워크에 내장된 데이터 가져오기 메커니즘을 사용하는 것만큼 효율적이지는 않지만, 데이터 가져오기 로직을 사용자 정의 Hook으로 옮기면 나중에 효율적인 데이터 가져오기 전략을 취하기가 더 쉬워진다.**

일반적으로 Effect를 작성해야 할 때마다 위의 `useData`와 같이 보다 선언적이고 목적에 맞게 구축된 API를 사용하여 일부 기능을 커스텀 Hook으로 추출할 수 있는 경우를 주시하자. 컴포넌트에서 원시 `useEffect` 호출이 적을수록 애플리케이션을 유지 관리하기가 더 쉬워지다.
