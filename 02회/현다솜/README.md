2주차 - UI 표현하기 (1)

https://www.notion.so/dasom-hyeon/2-UI-1-a93165af97fb43cb844d35bc8ff26926?pvs=4

# 첫 번째 컴포넌트

https://ko.react.dev/learn/your-first-component

마크업, CSS, JavaScript를 앱의 재사용 가능한 UI 요소인 ‘컴포넌트’로 결합할 수 있다.

- 컴포넌트 정의하기
  - 마크업으로 뿌릴 수 있는 JavaScript 함수
- 컴포넌트 빌드하는 방법
  1. 컴포넌트 내보내기

     ```jsx
     export default //...
     ```

  2. 함수 정의하기

     ```jsx
     function Profile() {}
     ```

     - React 컴포넌트는 일반 JS 함수이지만 이름은 대문자로 시작해야한다.

  3. 마크업 추가하기

     ```jsx
     return <img src="..." alt="..." />;

     return (
       <div>
         <img src="..." alt="..." />
       </div>
     );
     ```

     - 괄호가 없으면 뒷라인에 있는 모든 코드가 무시 된다.

## 컴포넌트 사용하기

```jsx
function A() {
  return <span>TEXT</span>;
}

function App() {
  return (
    <div>
      <A />
    </div>
  );
}
```

## 컴포넌트 중첩 및 구성

컴포넌트는 일반 JavaScript 함수이므로 같은 파일에 여러 컴포넌트를 포함할 수 있다. 컴포넌트가 상대적으로 작거나 서로 밀접하게 관련되어 있을 때 편리하다. 이를 부모 컴포넌트, 자식 컴포넌트로 구분할 수 있다.

<aside>
💡 주의
컴포넌트는 다른 컴포넌트를 렌더링할 수 있지만 매우 느리고 버그를 유발하기 때문에 정의를 중첩해서는 안된다. 
(https://ko.react.dev/learn/preserving-and-resetting-state#different-components-at-the-same-position-reset-state)

</aside>

### key를 이용한 state 초기화

key를 명시하면 React는 부모 내에서의 순서 대신에 key 자체를 위치의 일부로 사용한다. 이것이 컴포넌트를 jsx에서 같은 자리에 렌더하지만 React 관점에서는 다른 카운터인 이유이다. 결과적으로 그들은 절대 state를 공유하지 않는다.

→ key가 전역적으로 유일하지 않다는 것을 기억하자. key는 오직 부모 안에서만 자리를 명시한다.

- DEEP DIVE : 제거된 컴포넌트의 state를 보존하기
  - CSS로 가리기
  - state를 상위로 올리기
  - localStorage 이용하기

---

# 컴포넌트 import 및 export 하기

https://ko.react.dev/learn/importing-and-exporting-components

컴포넌트의 가장 큰 장점은 재사용성으로 컴포넌트를 조합하여 또 다른 컴포넌트를 만들 수 있다는 것이다. 분리할수록 나중에 파일을 찾기 더 쉽고 재사용이 편리해진다.

## Root 컴포넌트란

기본적으로는 App.js

Next.js 같은 경우는 페이지별로 root 컴포넌트가 다를 수 있다.

## 컴포넌트를 import, export 하는 방법

1. 컴포넌트를 추가할 js 파일 생성
2. 새로만든 파일에서 함수 컴포넌트를 export

   (default / named export)

3. 컴포넌트를 사용할 파일에서 import

   (default / named export)

<aside>
💡 Default 와 Named Exports
default export 의 경우 한 파일에 하나만 존재,
named export 는 여러개가 존재존재

export 방식에 따라 import 하는 방식이 정해져있다.

</aside>

---

# JSX로 마크업 작성하기

https://ko.react.dev/learn/writing-markup-with-jsx

JSX는 JavaScript를 화강한 문법으로 js파일을 HTML과 비슷하게 마크업을 작성할 수 있도록 해준다.

## JSX: JavaScript에 마크업 넣기

기존에는 HTML, JavaScript, CSS 를 따로 관리하였지만 웹이 더욱 인터렉티브해지면서 로직이 내용을 결정하고 스타일을 변경하는 경우가 많아졌다. 그렇기 때문에 React에서 렌더링 로직과 마크업이 같은 위치에 함께 있게된 이유이다.

<aside>
💡 JSX는 확장된 문법, 독립적으로도 사용가능
React는 JavaScript 라이브러리
별개의 개념임을 인지하자!

</aside>

## JSX의 규칙

1. 하나의 루트 엘리먼트로 반환
   - JSX는 HTML처럼 보이지만 내부적으로는 일반 JavaScript 객체로 변환된다. 하나의 배열로 감싸지 않은 하나의 함수에서는 두개의 객체를 반환할 수 없다. 따라서 또 다른 태그나 Fragment로 감싸지 않으면 JSX 태그를 반환할 수 없다.
2. 모든 태그는 닫아주기
3. 거의 대부분 카멜 케이스로
   - class → className
   - aria-_, data-_ 는 대시 사용

---

# 중괄호가 있는 JSX 안에서 자바스크립트 사용하기

https://ko.react.dev/learn/javascript-in-jsx-with-curly-braces

## 따옴표로 문자열 전달하기

```jsx
<img className="test" src={url} alt="..." />
```

문자열을 따옴표, 큰따옴표

변수를 넣기 위해서는 중괄호를 사용한다

## 중괄호 사용하기: JavaScript 세계로 연결하는 창

JSX 내에서 JavaScript 를 사용하기위해 중괄호를 사용한다.

```jsx
const TestComponent = () => {
  const name = "dasom";

  const addZzang = (name) => {
    return name + "ZZANG";
  };
  return (
    <h1>
      {name} {addZzang(name)}
    </h1>
  );
};
```

## 중괄호를 사용하는 곳

- JSX 태그 안의 문자
- = 바로 뒤에 오는 어트리뷰트

## 이중 중괄호 사용하기: JSX의 CSS와 다른 객체

```jsx
<div style={{ color: "red" }} />
```

<aside>
💡 인라인 style 프로퍼티는 카멜 케이스로 작성된다.

</aside>

---

# 컴포넌트에 props 전달하기

https://ko.react.dev/learn/passing-props-to-a-component

react 컴포넌트는 props를 이용해 서로 통신한다. 모든 부모 컴포넌트는 props를 자식 컴포넌트에 정보를 전달해줄 수 있다.

props는 HTML 어트리뷰트를 생각나게 할 수도 있지만 객체나 배열, 함수 등 모든 JavaScript 값을 전달할 수 있다.

받아온 props는 컴포넌트에서

```jsx
const TestComponent = ({ name, age }) => {};
```

구조분해할당을 이용해서 명시할 수 있다.

### prop의 기본값 지정

```jsx
const Test = ({ name = "dasom", age = "30" }) => {};
```

### JSX spread 문법으로 props 전달

```jsx
<Test {...props} />
```

### 자식 컴포넌트를 JSX로 전달

```jsx
const Test = ({ children }) => {
  return <div>{children}</div>;
};
```

### 시간에 따라 props가 변하는 방식

시간이나 상태를 이용해서 내려주는 props를 바로바로 적용시킬 수 있다.

<aside>
💡 props는 컴퓨터 과학에서 “변경할 수 없다”라는 의미의 불변성을 가지고 있다.  자식 컴포넌트에서 부모 컴포넌트에서 받아온 props를 변경해야 하는 경우 부모 컴포넌트에 다른 props를 받도록 ‘요청’해야 한다.
props 변경을 시도하지 말자. → state를 사용

</aside>
