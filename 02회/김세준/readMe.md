### 링크 : https://woozy-stool-6eb.notion.site/2-UI-Props-55f28f60260946e3a8121678c286e481?pvs=4
# [2회차] UI 구성하기(첫 번째 컴포넌트~컴포넌트 Props 전달하기)

# Your First component, 첫 번째 컴포넌트

<aside>
💡 학습내용

- 컴포넌트가 무엇인지
- React 어플리케이션에서 컴포넌트의 역할
- 첫번째 React 컴포넌트를 작성하는 방법
</aside>

## Components: UI building blocks, 컴포넌트 : UI 구성요소

웹에서 HTML 태그를 활용하여 마크업을 만들고 CSS를 추가하여 스타일을 입히고 JavaScript를 결합하여 상호작용이 가능한 화면 구성요소인 컴포넌트를 생성한다. 

## Defining a component, 컴포넌트 정의하기

React 컴포넌트는 마크업으로 뿌릴 수 있는 JavaScript 함수

## Step1: Exporting the component, 컴포넌트 내보내기

표준 JavaScript 구문인 export 접두사를 사용한다. 이때 export 방식에는 default 방식과 named 방식이 존재한다.

## Step2: Define the function, 함수 정의하기

React 컴포넌트는 일반적으로 JavaScript 함수이지만 JSX를 return 하는 경우에는 반드시 이름이 대문자로 시작해야 한다.

만약 JSX를 return하지 않으면 소문자를 사용해도 무관하다. 

## Step3: Add markup, 마크업 추가하기

```jsx
return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />;

return **(
  <>**
    <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  **</>
);**
```

## Using a component, 컴포넌트 사용하기

컴포넌트는 중첩해서 사용이 가능하다.

```jsx
import InnerContent from './InnerContent.js'

export default function Wrapper() {
	return (
		<section>
			<InnerContent />
			<InnerContent />
			<InnerContent />
		</seciton>
)
```

## What the browser sees, 브라우저에 표시되는 내용

React는 소문자는 HTML 태그, 대문자로 시작하면 컴포넌트로 이해한다.

## Nesting and organizing components, 컴포넌트 중첩 및 구성

정의를 중첩해서는 안된다. 컴포넌트 안에서 컴포넌트를 정의할 수 없다는 이야기다. 컴포넌트는 개별적으로 정의하고 사용할 때만 중첩해서 사용할 수 있도록 하자. 

# Importing and Exporting Components, 컴포넌트 import 및 export

컴포넌트의 가장 큰 장점은 재사용성

## The root component file, 루트 컴포넌트 파일

CRA 로 프로젝트를 생성하면 기본적으로 App.js 가 루트 컴포넌트가 된다. 앱 전체가 App.js 에서 실행되며 모든 컴포넌트들은 root 컴포넌트 파일 하위에 위치한다. 

설정에 따라 다른 파일에 위치하기도 하며, Next.js 처럼 파일 기반으로 라우팅하는 프레임워크의 경우 페이지별로 root 컴포넌트가 다를 수 있다.

## Exporting and importing a component, 컴포넌트 import 및 export 하기

1. 컴포넌트를 넣을 js 파일을 생성
2. 생성한 파일에 함수 컴포넌트를 export 한다.
3. 컴포넌트를 사용할 파일에서 import 한다.

```jsx
export default DefaultComponent;   // default export
export NormalComponent.  // named export

import DefaultComponent from '[파일경로]'
import { NormalComponent } from '[파일경로]' 
```

> ⭐️ named export 와 default export의 차이
> 

![Screen Shot 2023-09-05 at 11.07.02 PM.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/a8883249-24a5-43d5-a7b0-f77dda4453cc/73825a9a-cfc9-4e51-863e-abf394eb7d1c/Screen_Shot_2023-09-05_at_11.07.02_PM.png)

<aside>
💡 가끔 .js 같은 파일 확장자가 없을때도 있다. 
(다만 확장자가 있는 경우가 ES Modules 사용 방법에 더 가깝다.)

</aside>

- What is ES Modules?
    
    JS 모듈을 내보내고 가져오는데 2가지 방법이 존재한다.
    
    - CommoinJS (CJS)
        
        ```jsx
        module.export = { ... } // 모듈 내보낼 때
        	
        const utils = require('utils');
        ```
        
        자바스크립트를 모듈화하기 위한 방법 중 하나로 Node 초기 버전에서는 유일한 모듈 시스템이었다. 
        
    - ES Modules
        
        ES6부터 적용된 모듈 시스템, CJS 경우 모듈 로더를 동기 환경에서 실행하는 반면 ESM은 비동기에서 환경에서 실행한다. 
        
        ESM은 모듈 내 모든 자식 스크립트를 병렬로 다운로드한 후에 순차적으로 실행한다.
        
        최근 ESM이 점차 주류로 자리잡고는 있지만 Node.js에서도 아직 기본 모듈 시스템으로 채택하고 있고, <script> 태그를 사용하는 브라우저 환경, Babel 같은 ES6 코드를 변환할 수 있는 도구를 사용할 수 없는 경우가 있을 수 있기 때문에 알아두는 것이 좋다.
        
        [CommonJS와 ESM에 모두 대응하는 라이브러리 개발하기: exports field](https://toss.tech/article/commonjs-esm-exports-field)
        
    

## Exporting and importing multiple components from the same file, 동일한 파일에서 여러 컴포넌트 import 및 export 하기

한 파일에 default export 는 반드시 한 개만 존재하고, named export는 한 파일에 여러개 존재할 수 있다. 또한 하나의 default export 와 여러개의 named export가 함께 존재하는 경우도 가능하다. 

## Recap

- root 컴포넌트란 무엇인지
- 컴포넌트를 import 하거나 export 하는 방법
- 언제, 어떻게 default 및 named import, default 및 named export를 사용하는지
- 한 파일에서 여러 컴포넌트를 export 하는 방법

# Writing Markup with JSX, JSX로 마크업 작성하기

JSX는 JavaScript 확장 문법으로 JavaScript 파일 안에 HTML과 유사한 마크업을 작성할 수 있도록 해준다. 

<aside>
💡 학습내용

- React가 마크업과 렌더링 로직을 같이 사용하는 이유
- JSX와 HTML의 차이점
- JSX로 정보를 보여주는 방법
</aside>

## JSX: Putting markup into JavaScript, JavaScript에 마크업 넣기

기존에는 HTML, CSS, JavaScript 를 기반으로 각각 컨텐츠, 디자인, 로직을 별도로 파일로 관리했다.

하지만 웹이 더욱 인터랙티브 해지면서 로직이 컨텐츠를 결정하는 경우가 많아졌고, JavaScript가 마크업 역시 담당하게 되면서 JSX를 사용하기 시작했다. 

→ React에서 랜더링 로직과 마크업이 같은 위치의 컴포넌트에 함께 있는 이유

> ⭐️ MVC, MVP, MVVM 과 같이 디자인 패턴을 사용하여 코드를 역할에 따라 분리하는 것과 차이점
> 

버튼의 랜더링 로직과 마크업이 함께 존재한다면 항상 서로 동기화 상태를 유지할 수 있다. 반대로 버튼의 마크업과 사이드바의 마크업처럼 서로 관련이 없는 항목들은 서로 분리되어 있으므로 각각 개별적으로 변경하는 것이 안전하다.

JSX는 HTML과 비슷해보이지만 조금 더 엄격하며 동적으로 정보를 표시할 수 있다.

> 💡 React ≠ JSX
JSX 는 구문 확장이며 React는 JavaScript 라이브러리
> 

## Converting HTML to JSX, HTML을 JSX로 변환하기

JSX는 HTML보다 더 엄격하며 몇 가지 규칙이 더 있으므로 HTML을 JSX에 붙여넣기만 해서는 에러가 발생한다.

## The Rules of JSX, JSX 규칙

### 1. Return a single root element, 단일 루트 엘리먼트를 반환하세요.

### 2. Close all the tags, 모든 태그를 닫으세요.

### 3. camelCase all most of the things!, 거의 대부분이 카멜 케이스 입니다!

## Pro-tip: Use a JSX Converter, 전문가 팁: JSX 변환기 사용

JSX로 코드를 작성하지만, 웹 브라우저는 JSX를 인식하지 않으므로 브라우저에서 랜더링 하기 위해서는 다시 JavaScript 코드로 변환해야 한다. 일반적으로 Babel 과 같은 컴파일러를 사용하여 작업을 수행하며, 변환 중에 JSX 요소는 함수 호출로 변환된다. React는 가상 DOM 요소를 생성하기 위해 이러한 함수(ex. React.createElement)를 제공한다. 

# Javascript in JSX with Curly Braces, JSX에서 JavaScript 사용하기

JSX를 사용하면 JavaScript 파일 내에 HTML 과 유사한 마크업을 작성하여 랜더링 로직과 컨텐츠를 같은 위치에서 유지할 수 있다. 뿐만 아니라 중괄호를 사용하여 JavaSCript 로직을 추가하거나 동적 프퍼티를 참조할 수도 있다. 

<aside>
💡 학습내용

- 따옴표로 문자열을 전달하는 방법
- 중괄호를 사용하여 JSX 내에서 JavaScript 변수를 참조하는 방법
- 중괄호를 사용하여 JSX 내에서 JavaScript 함수를 호출하는 방법
- 중괄호를 사용하여 JSX 내에서 JavaScript 객체를 사용하는 방법
</aside>

## Passing strings with quotes, 따옴표로 문자열 전달하기

JSX에 문자열 속성을 전달하려면 작은 따옴표 혹은 큰따옴표로 묶는다. 

## Using curly braces: A window into the JavaScript world, 중괄호 사용하기: JavaScript 세계를 들여다 보는 창

값을 동적으로 지정한다면 동적으로 변하는 변수를 중괄호를 사용하여 props로 전달할 수 있다.

또한 중괄호 사이에는 함수 호출을 포함하여 모든 JavaScript 표현식이 동작한다. 물론 함수를 호출할 때는 return 값이 존재해야 중괄호 사이에 해당 return value가 랜더링 된다. 

### Where to use curly braces, 중괄호 사용 위치

1. JSX 태그 안에서 직접 사용
2. `=` 기호 바로 뒤에 오는 속성

## Using “double **curlies**”: CSS and other objects in JSX, “이중 중괄호” 사용: JSX 내에서의 CSS 및 다른 객체

문자열, 숫자 및 기타 JavaScript 표현식 외에도 객체를 JSX 에 전달할 수 있는데, 객체를 전달할 때는 중괄호로 표시한다. 따라서 전달되는 JS 중괄호를 두 쌍 가지게 된다. 

예를 들어 인라인 스타일이 필요한 경우 style attribute 에 객체를 전달한다.

```jsx
export default Example() {
	return (
		<ul style={{ backgroundColor:'blakck', color: 'white' }}>
			<li>1</li>
			<li>2</li>
			<li>3</li>
			<li>4</li>
		</ul>
		)
}
```

## ****More fun with JavaScript objects and curly braces, JavaScript 객체와 중괄호로 더 재미있게 즐기기****

특정 객체를 선언하고 중괄호를 사용하여 해당 객체의 key, value 를 사용하여 값을 사용할 수 있다. 

# Passing Props to a Component, 컴포넌트에 props 전달하기

React 컴포넌트들은 props를 이용하여 서로 통신한다. 부모 컴포넌트는 props 전달을 통해 자식 컴포넌트에게 필요한 정보를 전달하며 props 안에는 모든 JavaScript 값을 전달할 수 있다. (객체, 배열, 함수 등)

<aside>
💡 학습내용

- 컴포넌트에 props를 전달하는 방법
- 컴포넌트에서 props를 읽는 방법
- props의 기본값을 지정하는 방법
- 컴포넌트에 JSX를 전달하는 방법
- 시간에 따라 props가 변하는 방식
</aside>

## Familiar props, 친숙한 props

props 는 JSX 태그에 전달하는 정보이다. ReactDOM은 HTML 표준을 준수하므로 JSX 태그의 내부에 있는 HTML 태그들 역시 HTML 표준에 따라 전달할 수 있는 props들이 정해져 있다. React 컴포넌트는 HTML 태그와는 별도로 모든 JavaScript 값을 props로 전달할 수 있다. 

## Passing props to a component, 컴포넌트에 props 전달하기

아래 2단계를 거쳐 컴포넌트에 props를 전달할 수 있다.

### Step1. Pass props to the child component, 자식 컴포넌트에 props 전달하기

### Step2. Read props inside the child component, 자식 컴포넌트 내부에서 props 읽기

## Specifying a default value for a prop, props의 기본값 지정하기

값이 지정되지 않았을 때 props의 기본값을 지정할 수 있다.

```jsx
function Avatar({ person, size = 100 }) {
  // ...
}
```

이 때 기본 값은 props 자체가 없거나 값이 undefined로 전달될 때 사용된다. 그러나 만약 null 혹은 0 이 전달되면 기본값은 사용되지 않는다. 

## Forwarding props with the JSX spread syntax, JSX 전개 구문으로 props 전달하기

때때로 전달되는 props들이 반복적인 경우가 있다. 이럴 때는 전개구문을 사용하면 보다 간결하게 전달할 수 있다. 

다만 전개구문 역시 제한적으로 사용하여야 한다. 컴포넌트 재사용성을 위해 컴포넌트를 분리한다면 전개구문을 사용할 수 없다. \

```jsx
/* 일반적인 props 사용 */
function Profile({ person, size, isSepia, thickBorder }) {
  return (
    <div className="card">
      <Avatar
        person={person}
        size={size}
        isSepia={isSepia}
        thickBorder={thickBorder}
      />
    </div>
  );
}

/* 전개구문을 사용할 경우 */
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
```

## Passing JSX as children, 자식을 JSX로 전달하기

여러 자식 컴포넌트들을 묶어서 부모 컴포넌트에 전달되는 경우가 있는데 이 경우 children props를 사용하여 전달할 수 있다.

## How props change over time, 시간에 따라 props가 변하는 방식

컴포넌트가 전달받는 props의 값은 이벤트에 의해 변화할 수 있다. props 는 컴포넌트의 데이터를 처음에만 반영하지 않고 모든 시점에 반영한다. 

컴포넌트가 props를 변경해야 하는 경우 부모 컴포넌트에 다른 props, 즉 새로운 객체를 전달하도록 요청해야 한다. 그러면 이전의 props는 버려지고(참조를 끊는다.) JavaScript 엔진은 기존의 props가 차지하고 있던 메모리를 회수(gc, 가비지 컬렉팅)하게 된다. 

props는 컴포넌트에서 직접 변경할 수 없다. 사용자의 동작, 이벤트에 의해 값을 변경해야하는 경우에는 props가 아니라 state를 사용해야 한다.