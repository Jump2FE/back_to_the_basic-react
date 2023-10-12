# [2회] UI표현하기( 첫 번째 컴포넌트 ~ 컴포넌트 Props 전달하기)

### 링크 : https://tidal-swoop-2d1.notion.site/UI-Props-e79870b9eb944c128d47a362441f7323?pvs=4

## **첫번째 컴포넌트**

- 컴포넌트가 무엇인지
- React 어플리케이션에서 컴포넌트의 역할
- 첫번째 React 컴포넌트를 작성하는 방법

```tsx
<article>
  <h1>My First Component</h1>
  <ol>
    <li>Components: UI Building Blocks</li>
    <li>Defining a Component</li>
    <li>Using a Component</li>
  </ol>
</article>
```

이 마크업은 기본적인 HTML의 형태를 띄며, Javascript와 결합되어 웹에서 사용하는 모든 UI의 기반이 됩니다.

React를 사용하면 마크업 Css, Javascript를 앱의 재사용 가능한 UI 요소인 사용자 정의 ‘컴포넌트’로 결합할 수 있다.

- TOC
  ```tsx
  const TableOfContents = () => (
    <article>
      <h1>My First Component</h1>
      <ol>
        <li>Components: UI Building Blocks</li>
        <li>Defining a Component</li>
        <li>Using a Component</li>
      </ol>
    </article>
  );
  ```

마크업 + js + css인 컴포넌트를 만들어서 어디서든 재사용하며 쓸 수 있다.

이러한 컴포넌트를 순서, 중첩을 통해 전체 페이지를 디자인 할 수 있다.

- PageLayout
  ```tsx
  <PageLayout>
    <NavigationHeader>
      <SearchBar />
      <Link to="/docs">Docs</Link>
    </NavigationHeader>
    <Sidebar />
    <PageContent>
      <TableOfContents />
      <DocumentationText />
    </PageContent>
  </PageLayout>
  ```

이러한 추상적인 페이지를 만들어서 컴포넌트의 조합만 보고 어떤 형태인지 알 수 있다.

이 코드를 만약에 컴포넌트가 아닌 단순 HTML로만 구성하면 어떻게 될까? 조금 끔찍할 수 있다.

- PageLayout To HTML
  ```tsx
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      <title>Your Page Title</title>
      <link rel="stylesheet" href="styles.css" />
    </head>
    <body>
      <header>
        <nav>
          <div class="navigation-header">
            <div class="search-bar">//검색</div>
            <a href="/docs">Docs</a>
          </div>
        </nav>
      </header>
      <aside class="sidebar">//사이드바</aside>
      <main>
        <section class="page-content">
          <div class="table-of-contents">
            <article>
              <h1>My First Component</h1>
              <ol>
                <li>Components: UI Building Blocks</li>
                <li>Defining a Component</li>
                <li>Using a Component</li>
              </ol>
            </article>
          </div>
          <div class="documentation-text">//본문</div>
        </section>
      </main>
    </body>
  </html>
  ```

이걸 보고 개발을 한다고 생각해보면, 한눈에 알기 굉장히 힘들다. 컴포넌트 구조로 추상적으로 해당 내용을 한번에 볼 수 있는 것 또한 장점이다. 또한, 코드의 js 내용까지 같이 있다고 생각하면, 더욱 커져 확인하기 힘들걸로 예상됩니다.

프로젝트가 성장하면, 이미 작성한 컴포넌트를 재사용하여 많은 디자인을 구성하며 개발 속도가 빨라진다.

TOC를 한번 만들어두면, 어떤 화면에서든 재사용하며 추가할 수 있게된다.

[sandpack-project - CodeSandbox](https://codesandbox.io/s/sb12gi?file=/App.tsx&from-sandpack=true)

[9+ Free React Templates - Material UI](https://mui.com/material-ui/getting-started/templates/)

만들어진 UI들을 사용하여 개발을 하면 개발 속도를 올릴 수 있다.

\*이러한 라이브러리를 사용할때 자체적으로 자유도가 높은경우는 괜찮으나, 자유도가 떨어지는 경우엔 너무 힘들다.

- 자유도가 떨어져서 생긴 이슈
  회사에서 elemental-ui를 사용하고 있는데, 최근에 발생한 이슈중에 컴포넌트 생성 시에 생성된 컴포넌트가 해당 위치에 생기는게 아닌, 외부의 modal처럼 생기는 것이었다. 강제 DOM을 변동시켜야 하는데 동작하지 않아서 문제된 적이 있다.
  https://github.com/element-plus/element-plus/issues/13888

# 컴포넌트 정의하기

**React 컴포넌트는 마크업으로 뿌릴 수 있는 JavaScript 함수입니다.**

- 컴포넌트 기본형태
  ```tsx
  export default function Profile() {
    return (
      <img src="https://i.imgur.com/MK3eW3Am.jpg" alt="Katherine Johnson" />
    );
  }
  ```

## Step1: 컴포넌트 내보내기

export default는 표준 JavaScript 구문입니다. 이 접두사를 사용하면 나중에 다른 파일에서 가져올 수 있도록 파일에 주요 기능을 표시할 수 있습니다.

- export
  JavaScript 모듈에서 함수, 객체 원시 값을 내보낼 때 사용합니다. 내보낸 값은 다른 프로그램에서 import문으로 가져가 사용할 수 있습니다.
  내보내는 모듈은 무조건 “엄격 모드”입니다.

## Step2: 함수 정의하기

컴포넌트 기본형태의 모습으로 만들면 된다.

주의점으로 Component의 시작은 대문자로 시작해야한다.

호출을 할때는 괜찮다

- 다양한 호출법
  ```tsx
  import DefaultProfile from "./ExportedDefaultProfile";
  import {
    NamedExportedProfileOne,
    namedExportProfileTwo as NamedTwo,
    namedExportProfileThree,
  } from "./Profiles";

  const user = {
    imageUrl: "https://i.imgur.com/yXOvdOSs.jpg",
    imageSize: 90,
  };

  const NamedThree = namedExportProfileThree;

  export default function App() {
    const NamedFour = namedExportProfileThree;
    return (
      <>
        <DefaultProfile user={{ ...user, name: "DefaultProfile" }} />
        <NamedExportedProfileOne user={{ ...user, name: "NamedExported" }} />
        <NamedTwo user={{ ...user, name: "namedExported2" }} />
        <NamedThree user={{ ...user, name: "NamedThree" }} />
        <NamedFour user={{ ...user, name: "NamedFour" }} />
      </>
    );
  }
  ```
  import한 컴포넌트를 대문자로 시작할 수 있게 변환시켜주면 된다.
  소문자로 시작하는 컴포넌트는 왜 안될까?
  가장 기본적인 이유는 기본 HTML 태그와 구별하기 쉽지 않기 떄문입니다.
  하지만, 기존 html 태그들을 예약어로 지정해두고, 안되는 단어로 지정해두면 되지 않을까? 어떻게 설정을 해뒀길래 오류로그가 발생하는 것일까?
  [Why component identifiers must be capitalized in React - React inDepth](https://indepth.dev/posts/1499/why-component-identifiers-must-be-capitalized-in-react)
  Babel이 JSX를 브라우저가 이해 가능한 코드로 변환하는 역할을 수행하고
  ‘transform-react-jsx’ 플러그인으로 이를 가능하게합니다.
  JSX → AST로 변환하고
  JSX 요소의 태그 이름이 소문자로 시작하는지 여부를 기준으로 합니다. 태그 이름이 소문자로 시작하면 해당 요소는 DOM 요소로 판단되고, 대문자로 시작하면 컴포넌트로 판단됩니다.
  이렇게 결정된 요소는 createElement 함수에 전달될 때 다른 방식으로 처리되며, 이에 따라 React가 다른 동작을 수행하게 됩니다.
  - createElement명세
    ```jsx

    // DOM Elements
    // TODO: generalize this to everything in `keyof ReactHTML`, not just "input"
    function createElement(
            type: "input",
            props?: InputHTMLAttributes<HTMLInputElement> & ClassAttributes<HTMLInputElement> | null,
            ...children: ReactNode[]): DetailedReactHTMLElement<InputHTMLAttributes<HTMLInputElement>, HTMLInputElement>;

    function createElement<P extends HTMLAttributes<T>, T extends HTMLElement>(
            type: keyof ReactHTML,
            props?: ClassAttributes<T> & P | null,
            ...children: ReactNode[]): DetailedReactHTMLElement<P, T>;

    function createElement<P extends SVGAttributes<T>, T extends SVGElement>(
            type: keyof ReactSVG,
            props?: ClassAttributes<T> & P | null,
            ...children: ReactNode[]): ReactSVGElement;

    function createElement<P extends DOMAttributes<T>, T extends Element>(
            type: string,
            props?: ClassAttributes<T> & P | null,
            ...children: ReactNode[]): DOMElement<P, T>;

    // Custom components

    function createElement<P extends {}>(
            type: FunctionComponent<P>,
            props?: Attributes & P | null,
            ...children: ReactNode[]): FunctionComponentElement<P>;
    function createElement<P extends {}>(
            type: ClassType<P, ClassicComponent<P, ComponentState>, ClassicComponentClass<P>>,
            props?: ClassAttributes<ClassicComponent<P, ComponentState>> & P | null,
            ...children: ReactNode[]): CElement<P, ClassicComponent<P, ComponentState>>;
    function createElement<P extends {}, T extends Component<P, ComponentState>, C extends ComponentClass<P>>(
            type: ClassType<P, T, C>,
            props?: ClassAttributes<T> & P | null,
            ...children: ReactNode[]): CElement<P, T>;
    function createElement<P extends {}>(
            type: FunctionComponent<P> | ComponentClass<P> | string,
            props?: Attributes & P | null,
            ...children: ReactNode[]): ReactElement<P>;

    createElement 명세
    소문자는 string으로 변환되어 들어와서 오버로드중 type이 string인 것으로 실행되고,
    DOM으로 처리되어 DOMElement가 생성된다.
    ```

‘

## Step 3: 마크업 추가하기

```jsx
return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />;
```

이러한 img태그들이 HTML처럼 작성되었지만, 실제로 Javascript입니다.

트랜스파일링을 거치면

```jsx
return React.createElement("img", {
  src: "https://i.imgur.com/MK3eW3As.jpg",
  alt: "Katherine Johnson",
});
```

이런식으로 쓰이게 될 것입니다.

```jsx
ex1)
return
<div>
    <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
</div>
->
return;
<div>
    <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
</div>
//별다른 설정을 하지 않았다면, prettier도 잡아주는 것 없이 error

ex2)
return <div>
    <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
</div>
->
return <div>;
    <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
</div>
---
prettier 사용시
return (
	<div>
	    <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
	</div>
	)

괄호가 없다면 retun 뒷라인에 있는 모든 코드가 무시됩니다!
```

## 컴포넌트 사용하기

```jsx
function Profile() {
  return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />;
}
```

이 Profile컴포넌트를 생성했으면, 어느 순서로 쓰거나 중첩해서 쓸 수 있습니다.

```jsx
Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
			<imNotHTML>imNotHTML</imNotHTML>
    </section>
  );
}
```

이렇게 만들어서 쓴다면 브라우저가 어떻게 받아드릴까요?

```jsx
<Profile /> 은 대문자 P로 시작하므로 Profile이라는 커스텀 컴포넌트를 사용한다고 이해합니다.
<section />은 소문자로 시작하므로 HTML 태그라고 이해합니다.
그 과정을 풀어서 나타내면
<imNotHTML>은 소문자로 시작하므로 HTML 태그라고 이해하고 내보냅니다.

<section>
  <h1>Amazing scientists</h1>
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
  <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
	<imNotHTML>imNotHTML</imNotHTML>
</section>

이러한 형태가 됩니다.
```

## 컴포넌트 중첩 및 구성

컴포넌트는 javascript함수이므로 같은 파일에 여러 컴포넌트를 포함할 수 있습니다. 물론 너무 커지면 작게 분해하여, 별도의 파일로 옮길 수 있습니다.

Profile 컴포넌트를 한번 정의하고 어디서든 사용할 수 있다는 것이 컴포넌트로 구성된 리액트의 힘

**주의** 컴포넌트를 다른 컴포넌트에서 렌더링 할 수 있지만, 정의를 중첩해서는 안됩니다!

```jsx
export default function Gallery() {
  // 🔴 다른컴포넌트 안에서 함수컴포넌트를 정의하면 안됩니다!
  function Profile() {
    // ...
  }
  // ...
}
```

위의 스니펫은 매우 느리고 버그를 촉발합니다. (느리다는 잘 모르겠다??)

자식 컴포넌트에 부모 컴포넌트의 일부 데이터가 필요한 경우 props로 전달하는걸 권장

## 컴포넌트의 모든 것

리액트 어플리케이션은 ‘root’컴포넌트에서 시작됩니다.

createRoot 를 통해 루트컴포넌트를 index.html에서 찾아서 실행하는 형태를 띄고 있습니다.

대부분의 리액트 앱은 모든 부분에서 컴포넌트를 사용합니다.

재사용이 많이 될 것 같은 부분뿐 아니라 모든 부분을 컴포넌트로 만들어서 사용합니다.

컴포넌트는 한 번만 사용되더라도 UI 코드와 마크업을 정리하는 편리한 방법입니다!

리액트 기반 프레임워크(Next.js, Remix, Gatsby, Expo)등은 빈 HTML을 사용하고 React가 js로 페이지 관리를 ‘대행’(take over) 하는 대신, 리액트 컴포넌트에서 HTML을 자동으로 생성하기도 합니다. 이를 통해 js 코드가 로드되기 전에 앱에서 일부 콘텐츠를 표시할 수 있습니다.

- ex) Next.js
  Next.js 애플리케이션을 빌드할 때 각 페이지에 대한 HTML을 빌드 프로세스 중에 생성할 수 있습니다.
  작동방식
  - 페이지에 대한 React 컴포넌트를 생성
  - 빌드 프로세스 중 Next.js는 이러한 컴포넌트를 기반으로 각 페이지에 대한 HTML 파일을 생성합니다. 이 HTML에는 페이지의 초기 콘텐츠가 포함되어 있습니다.
  - 사용자가 사이트를 방문하면 요청한 페이지에 대한 사전 렌더링된 HTML을 받게 되어 즉시 콘텐츠를 볼 수 있습니다.
  - JavaScript 번들이 로드되면 Next.js가 페이지를 가져와 완전히 상호 작용 가능하게 만듭니다.

그렇지만, 많은 웹사이트들은 아직도 약간의 상호작용을 추가하는 용도로만 사용한다.

→ 네이버 카페는 vue로 만들어져있다.

## 도전과제

- 컴포넌트 내보내기
  ```jsx
  function Profile() {
    return (
      <img
        src="https://i.imgur.com/lICfvbD.jpg"
        alt="Aklilu Lemma"
      />
    );
  }
  import하는 부분이 나와있지 않지만, 그냥 export하면 에러가 나서 기본 내보내기를 실행합니다.

  export default function Profile() {
    return (
      <img
        src="https://i.imgur.com/lICfvbD.jpg"
        alt="Aklilu Lemma"
      />
    );
  }

  ```
- return문을 고치세요
  ```jsx
  export default function Profile() {
    return
      <img src="https://i.imgur.com/jA8hHMpm.jpg" alt="Katsuko Saruhashi" />;
  }
  괄호가 없으면 무시합니다.
  export default function Profile() {
    return (
      <img src="https://i.imgur.com/jA8hHMpm.jpg" alt="Katsuko Saruhashi" />
  	)
  }

  그러나 한줄이라면

  export default function Profile() {
    return <img src="https://i.imgur.com/jA8hHMpm.jpg" alt="Katsuko Saruhashi" />
  }
  도 괜찮습니다. ;을 표기한거보니, 한줄로 표기하길 기대했을 것 같습니다.
  ```
- 실수를 찾아내세요
  ```jsx
  function profile() {
    return (
      <img
        src="https://i.imgur.com/QIrZWGIs.jpg"
        alt="Alan L. Hart"
      />
    );
  }

  export default function Gallery() {
    return (
      <section>
        <h1>Amazing scientists</h1>
        <profile />
        <profile />
        <profile />
      </section>
    );
  }
  소문자로 시작된 컴포넌트는 string으로 읽히며 HTML태그로 인식됩니다.
  <Profile />로 고치도록 합시다.

  export default function Gallery() {
    return (
      <section>
        <h1>Amazing scientists</h1>
        <Profile />
  			<Profile />
  			<Profile />
      </section>
    );
  }
  ```
- 컴포넌트를 새로 작성해보세요.
  ```jsx
  Congratulations 컴포넌트를 만들어봅시다.

  export default function  Congratulations (){
  	return <h1>Good job!</h1>
    }
  ```

# 컴포넌트 import 및 export

컴포넌트의 가장 큰 장점인 재사용성을 사용해봅시다.

## 루트 컴포넌트 파일

```tsx
App.js;
export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile />
      <Profile />
      <Profile />
    </section>
  );
}
```

App.js라는 **root 컴포넌트 파일**에 존재하는 예제들을 봤습니다.

```tsx
ReactDOM.createRoot(document.getElementById("root") as HTMLElement).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

index.js , main.js 등 root에서 실행된 createRoot 함수를 이용해서 만들게 됩니다. 설정에 따라 root 컴포넌트를 다른 위치에 둘 수 있습니다.

이 또한 js 함수이기 때문에 2개 3개 만들면 만들어집니다.

- 3개를 만든 예시
  ```tsx
  //main.js
  import React from "react";
  import ReactDOM from "react-dom/client";
  import App from "./App";
  import "./index.css";

  ReactDOM.createRoot(document.getElementById("root") as HTMLElement).render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
  ReactDOM.createRoot(document.getElementById("root2") as HTMLElement).render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );
  ReactDOM.createRoot(document.getElementById("root3") as HTMLElement).render(
    <React.StrictMode>
      <App />
    </React.StrictMode>
  );

  //index.html
  <!DOCTYPE html>
  <html lang="en">
    <head>
      <meta charset="UTF-8" />
      <link rel="icon" type="image/svg+xml" href="/vite.svg" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      <title>Vite + React + TS</title>
    </head>
    <body>
      <div id="root"></div>
      <div id="root2"></div>
      <div id="root3"></div>
      <script type="module" src="/src/main.tsx"></script>
    </body>
  </html>
  ```
  [xenodochial-phoebe-yymhlt](https://codesandbox.io/p/sandbox/xenodochial-phoebe-yymhlt?file=/node_modules/@types/react-dom/client.d.ts:1,1)

‘나중에 랜딩 화면을 변경하여, 과학 도서 목록을 놓으려면 어떻게 해야 하나요? ‘, ‘모든 프로필을 다른 곳에 배치파고자 한다면? ‘ 등의 컴포넌트 이동 및 화면 구성요소의 변경을 요구할 수 있습니다.

그럴때 요구사항에 쉽게 대응하기 위해 어떻게 할까요?

우리가 만든 Gallery와 Profile예시에선 Gallery 와 Profile을 root 컴포넌트 파일 밖으로 옮기는게 좋을 것 같습니다. 이렇게하면, 모듈성이 강화되고 다른 파일에서 재사용할 수 있게 됩니다.

일반적인 방법론으로

1. 컴포넌트를 넣을 JS 파일을 생성
2. 새로 만든 파일에서 함수 컴포넌트를 export 합니다. 이때, default 혹은 named export 방식을 사용하도록 합니다.
3. 컴포넌트를 사용할 파일에서 import 합니다.

- Galllery , Profile 폴더별 구분
  ```tsx
  Gallery.js
  function Profile() {
    return (
      <img
        src="https://i.imgur.com/QIrZWGIs.jpg"
        alt="Alan L. Hart"
      />
    );
  }

  export default function Gallery() {
    return (
      <section>
        <h1>Amazing scientists</h1>
        <Profile />
        <Profile />
        <Profile />
      </section>
    );
  }
  해당 예제에서 이렇게 쓰고 있지만,
  컴포넌트 하나당 하나의 파일을 갖는 것을 선호합니다.

  Gallery.js
  export default function Gallery() {
    return (
      <section>
        <h1>Amazing scientists</h1>
        <Profile />
        <Profile />
        <Profile />
      </section>
    );
  }

  Profile.js
  function Profile() {
    return (
      <img
        src="https://i.imgur.com/QIrZWGIs.jpg"
        alt="Alan L. Hart"
      />
    );
  }

  ```
  Gallary.js
  동일한 파일 내에서만 사용되며, export하지 않는 Profile 컴포넌트를 정의합니다.
  갤러리 컴포넌트를 default export 방식으로 export합니다.
  App.js
  Gallery컴포넌트를 Gallery.js 로 부터 default import 방식으로 import 합니다.
  root App 컴포넌트를 default export 방식으로 export합니다.
- esmodule node.js환경에서?
  [FECONF 2022 [B4] 내 import 문이 그렇게 이상했나요?](https://www.youtube.com/watch?v=mee1QbvaO10&t=95s)
  사실 ES Modules로 import 하기 위해선 .js를 붙이는게 반드시 필요하다고합니다. 근데 vite는 모듈기반인데 왜 생략을 해도 괜찮을까요?

가끔 .js 와 같은 파일 확장자가 없는 때도 있습니다. 그러나 ESM 사용 방법에선 .js를 붙이는게 맞습니다.

## Default vs names exports

보통 js에선 default eport와 neamed export라는 두가지 방법이 존재합니다.

default는 하나의 파일에서 하나만 갖고 있을 수 있지만, named는 여러개 존재할 수 있습니다.

default로 export하는 경우 ‘ import 파일명 from 경로 ‘ 형태를 띄는데, 이때 파일명을 임의로 설정 할 수 있습니다.

names로 export하고 import하는 경우엔 이름이 같아야 합니다. 그래서 named라는 명칭을 사용합니다. 별칭을 통해 이름을 바꿀 수 있습니다.

- import 와 export
  ```tsx
  export default function Test() {
    return <div>Test</div>;
  }

  const Test2 = () => {
    return <div>Test2</div>;
  };

  export const Test3 = () => <div>Test3</div>

  const Test4 = {
    Test : true
  }

  export { Test2, Test4 };

  import Test, { Test2  as TTT, Test4} from "./Test";
  import * as Test from "./Test";
  해당 import 시 Test.Test2 등으로 쓸 수 있음

  등 다양하게 import 해서 쓸 수 있다.
  export {Test4} from "./Test";바로내보내기
  export Test from "./Test.js; **오류발생 ?**

  ```

## 도전과제

는 위에서 해결했으니 안전합니다.

# JSX로 마크업 작성하기

JSX는 JavaScript를 확장한 문법으로 js 파일안에 HTML과 유사한 마크업을 작성할 수 있게 해줍니다. 컴포넌트를 작성하는 간결한 방법입니다.

웹은 HTML, CSS , JS 로 기반으로 만들어져왔습니다.

그리고 보통 각각 따로 만들어 보관했습니다.

하지만 웹이 더욱 인터랙티브 해지면서, **로직이 컨텐츠를 결정하는 경우가 많아졌습니다**

그래서 JS가 HTML을 담당하게 되었습니다.

React에서 렌더링 로직과 마크업이 같은 위치의 컴포넌트에 함께 있는 이유입니다!

- 로직이 컨텐츠를 결정했다고 하는 이유
  **사용자 상호 작용**: 사용자가 웹 애플리케이션과 상호 작용할 때, 즉 클릭, 드래그, 입력 등을 통해 웹페이지와 애플리케이션과 소통할 때, 그 상호 작용에 따라 콘텐츠가 변경.
  이를 위해서는 JavaScript와 같은 프로그래밍 언어를 사용하여 사용자의 행동에 따른 로직을 작성해야함
  - HTML
    ```tsx
    <!DOCTYPE html>
    <html>
    <head>
        <title>인터랙티브 예제</title>
    </head>
    <body>
        <button id="myButton">클릭하세요</button>
        <div id="output"></div>

        <script>
            const button = document.getElementById('myButton');
            const output = document.getElementById('output');

            button.addEventListener('click', () => {
                output.innerHTML = '버튼이 클릭되었습니다.';
            });
        </script>
    </body>
    </html>
    ```
  - React
    ```tsx
    import React, { useState } from "react";

    function 사용자상호작용() {
      const [message, setMessage] = useState("");

      const handleClick = () => {
        setMessage("버튼이 클릭되었습니다.");
      };

      return (
        <div>
          <button onClick={handleClick}>클릭하세요</button>
          <div>{message}</div>
        </div>
      );
    }
    ```
  **실시간 업데이트**: 웹사이트나 웹 애플리케이션에서는 실시간 업데이트가 중요한 경우가 많습니다. 예를 들어, 실시간으로 새로운 게시물을 받아오거나 경우에는 서버와의 통신과 클라이언트 사이의 로직이 내용을 채움
  - HTML
    ```tsx
    <!DOCTYPE html>
    <html>
    <head>
        <title>실시간 업데이트 예제</title>
    </head>
    <body>
        <div id="content"></div>

        <script>
            function updateContent() {
                fetch('/api/data') // 서버에서 데이터 가져오기
                    .then(response => response.text())
                    .then(data => {
                        document.getElementById('content').innerHTML = data;
                    });
            }

            // 주기적으로 업데이트
            setInterval(updateContent, 5000); // 5초마다 업데이트
            updateContent(); // 초기 업데이트
        </script>
    </body>
    </html>
    ```
  - React
    ```tsx
    function 실시간업데이트() {
      const [content, setContent] = useState("");

      const updateContent = () => {
        // 서버에서 데이터 가져오는 대신 예제에서는 간단한 메시지를 업데이트합니다.
        setContent("새로운 내용이 업데이트되었습니다.");
      };

      useEffect(() => {
        updateContent();
        const interval = setInterval(updateContent, 5000);
        return () => clearInterval(interval);
      }, []);

      return (
        <div>
          <div>{content}</div>
        </div>
      );
    }
    ```
  **조건부 렌더링**: 특정 조건에 따라 다른 내용을 보여주어야 하는 경우
  예를들어, 사용자가 로그인하지 않은 경우에는 로그인 폼을 보여주고, 로그인한 경우에는 사용자의 프로필 정보를 보여주는 등의 조건부 렌더링이 필요하다.
  - HTML
    ```tsx
    <!DOCTYPE html>
    <html>
    <head>
        <title>조건부 렌더링 예제</title>
    </head>
    <body>
        <div id="container"></div>

        <script>
            const isLoggedIn = true; // 예시: 사용자가 로그인한 경우

            if (isLoggedIn) {
                document.getElementById('container').innerHTML = '<p>로그인된 사용자입니다.</p>';
            } else {
                document.getElementById('container').innerHTML = '<p>로그인이 필요합니다.</p>';
            }
        </script>
    </body>
    </html>
    ```
  - React
    ```tsx
    function 조건부렌더링() {
      const isLoggedIn = true; // 예시: 사용자가 로그인한 경우

      return (
        <div>
          {isLoggedIn ? (
            <p>로그인된 사용자입니다.</p>
          ) : (
            <p>로그인이 필요합니다.</p>
          )}
        </div>
      );
    }
    ```
  **사용자 맞춤형 경험**: 사용자의 선호도, 행동 기록 등을 기반으로 사용자에게 맞춤형 콘텐츠를 제공하는 경우도 있습니다. 이를 위해서는 사용자 데이터를 수집하고 분석하여 로직을 작성해야 합니다.
  이렇게 HTML 과 React로 나눠서 보면 큰 차이가 없지만, 이러한 기능들이 점점 모이면 HTML페이지로 보기 너무 힘들어지고 해당 function하나씩 찾는게 너무 힘들다.

렌더링 로직과 마크업이 함께 존재한다면, 모든 편집에서 서로 동기화 상태르르 유지할 수 있습니다.

관련 없는 항목들이 서로 분리되어 있어야 개별적 변경에서 안전합니다!

각 React 컴포넌트는 리액트가 브라우저에 마크업 렌더링 할 수 있는 js 함수입니다. JSX라는 구문 확장자를 사용하여 마크업을 표현합니다. JSX는 HTML과 비슷해 보이지만 좀 더 엄격하며 동적으로 정보를 표시할 수 있습니다.

JSX ≠ React 입니다. JSX는 구문 확장, React는 js 라이브러리입니다.

- HTML보다 깐깐한 React
  ```tsx
  HTML
  <h1>Hedy Lamarr's Todos</h1>
  <img
    src="https://i.imgur.com/yXOvdOSs.jpg"
    alt="Hedy Lamarr"
    class="photo"
  >
  <ul>
      <li>Invent new traffic lights
      <li>Rehearse a movie scene
      <li>Improve the spectrum technology
  </ul>
  사실 이렇게 쓸 수 있는걸 처음 알았습니다.
  li태그든 닫아야되는줄 알고있었는데..

  export default function TodoList() {
    return (
      // This doesn't quite work!
      <h1>Hedy Lamarr's Todos</h1>
      <img
        src="https://i.imgur.com/yXOvdOSs.jpg"
        alt="Hedy Lamarr"
        class="photo"
      >
      <ul>
        <li>Invent new traffic lights
        <li>Rehearse a movie scene
        <li>Improve the spectrum technology
      </ul>
    );
  }
  아무튼 이렇게 쓰면 오류가 납니다.

  하나로 묶어주고 나면, class를 className으로 바꿔줘야합니다.
  export default function TodoList() {
    return (
      <>
  	    <h1>Hedy Lamarr's Todos</h1>
  	    <img
  	      src="https://i.imgur.com/yXOvdOSs.jpg"
  	      alt="Hedy Lamarr"
  	      class="photo"
  	      />
  	    <ul>
  	      <li>Invent new traffic lights
  	      <li>Rehearse a movie scene
  	      <li>Improve the spectrum technology
  	    </ul>
       </>
    );
  }
  img 태그도 닫고, li 태그도 닫아줘야합니다.
  export default function TodoList() {
    return (
      <>
  	    <h1>Hedy Lamarr's Todos</h1>
  	    <img
  	      src="https://i.imgur.com/yXOvdOSs.jpg"
  	      alt="Hedy Lamarr"
  	      class="photo"
  	      />
  	    <ul>
  	      <li>Invent new traffic lights</li>
  	      <li>Ryehearse a movie scene</li>
  	      <li>Improve the spectrum technology</li>
  	    </ul>
       </>
    );
  }

  className으로 바꿔줍시다. (대부분 카멜 케이스 입니다.)
  가끔 컴포넌트를 복사해와서 쓸 때 stroke-width같은 오류가 잔뜩 생길 수 있습니다.
  export default function TodoList() {
    return (
      <>
  	    <h1>Hedy Lamarr's Todos</h1>
  	    <img
  	      src="https://i.imgur.com/yXOvdOSs.jpg"
  	      alt="Hedy Lamarr"
  	      className="photo"
  	      />
  	    <ul>
  	      <li>Invent new traffic lights</li>
  	      <li>Ryehearse a movie scene</li>
  	      <li>Improve the spectrum technology</li>
  	    </ul>
       </>
    );
  }
  ```

왜 여러 JSX태그를 하나로 감싸줘야 할까요?

JSX는 HTML처럼 보이지만, 내부적으론 객체로 반환됩니다. 하나의 배열로 감싸지 않은 하나의 함수에서는 두 개의 객체를 반환할 수 없기 때문입니다.

- 감싸있지 않은 코드
  ```tsx
  export default function TodoList() {
    return (
      // This doesn't quite work!
      <h1>Hedy Lamarr's Todos</h1>
      <img
        src="https://i.imgur.com/yXOvdOSs.jpg"
        alt="Hedy Lamarr"
        class="photo"
      >
      <ul>
        <li>Invent new traffic lights
        <li>Rehearse a movie scene
        <li>Improve the spectrum technology
      </ul>
    );
  }
  가 변환하는 과정을 생각해보면
  createElement가 3개인 형태가 될 것입니다.
  createElement의 반환타입이 객체라고 생각한다면,
  export default function TodoList() {
    return (
      // JSX를 React.createElement로 변환
      React.createElement('h1', null, "Hedy Lamarr's Todos"),
      React.createElement('img', {
        src: "https://i.imgur.com/yXOvdOSs.jpg",
        alt: "Hedy Lamarr",
        className: "photo" // "class" 대신 "className" 사용
      }),
      React.createElement('ul', null,
        React.createElement('li', null, 'Invent new traffic lights'),
        React.createElement('li', null, 'Rehearse a movie scene'),
        React.createElement('li', null, 'Improve the spectrum technology')
      )
    );
  }
  이러한 형태가 될 것이고, 이것은
  ```

aria-* , data-*은 HTML에서와 동일하게 대시를 사용하여 작성합니다.

## 도전과제

- HTML to JSX
  ```tsx
  export default function Bio() {
    return (
      <div class="intro">
        <h1>Welcome to my website!</h1>
      </div>
      <p class="summary">
        You can find my thoughts here.
        <br><br>
        <b>And <i>pictures</b></i> of scientists!
      </p>
    );
  }
  1. 감싸줍시다.
  2. 모든 태그를 닫아줍시다.
  3. 대부분 카멜케이스입니다.
  export default function Bio() {
    return (
      <>
  	    <div className="intro">
  	      <h1>Welcome to my website!</h1>
  	    </div>
  	    <p className="summary">
  	      You can find my thoughts here.
  	      <br/>
  				<br/>
  	      And <i><b>pictures</b></i> of scientists!
  	    </p>
  	  </>
    );
  }

  ```

# JSX에서 JS사용하기

JSX에서 문자열 속성을 전달할땐, 작은 따옴표 또는 큰따옴표로 묶습니다.

```tsx
export default function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/7vQD0fPs.jpg"
      alt="Gregorio Y. Zara"
    />
  );
}
```

하지만 src 또는 alt 를 동적으로 지정하려면 어떻게 할까요?

{ } 중괄호로 대체하여 js 값을 사용할 수 있습니다.

```tsx
export default function Avatar() {
  const avatar = "https://i.imgur.com/7vQD0fPs.jpg";
  const description = "Gregorio Y. Zara";
  return <img className="avatar" src={avatar} alt={description} />;
}
```

중괄호를 사용하면 마크업에서 바로 js 로 작업할 수 있습니다.

className의 “avatar”와 구분해야합니다.

## 중괄호 사용하기 : JS 세계를 들여다 보는 창

JSX는 js를 작성하는 특별한 방법입니다.

중괄호 { } 안에서 js 를 사용할 수 있다는 의미입니다.

```tsx
export default function TodoList() {
  const name = "Gregorio Y. Zara";
  return <h1>{name}'s To Do List</h1>;
}
```

name값을 바뀌면! 바뀝니다~

```tsx
const today = new Date();

function formatDate(date) {
  return new Intl.DateTimeFormat(
    'kr',
    { weekday: 'long' }
  ).format(date);
}

export default function TodoList() {
  return (
    <h1>To Do List for {formatDate(today)}</h1>
  );
}
짜잔~ 날짜가 나오네요~
```

## 중괄호 사용 위치

JSX 내부에서는 두 가지 방법으로만 상요할 수 있습니다.

1. JSX 태그 안에 직접 텍스트 사용 (태그명 불가능)
2. = 기호 바로 뒤에 오는 속성

```tsx
1.
<h1>{name}'s To Do List</h1>
<{tag}>Gregorio Y. Zara's To Do List</{tag}> 는 불가

2.
<img src={avatar} />
<img src="{avatar}" /> 는 src에 "{avatar}"라는 문자열을 전달

```

## 이중 중괄호 사용: jsx 내에서의 css 및 다른 객체

문자열, 숫자 및 기타 js 표현식 외에도 jsx로 객체를 전달할 수 있습니다. 객체는 중괄호로 표시할 수 있습니다.

예를들어 { name: "Hedy Lamarr", inventions: 5 }를 전달하려면,

(person={{ name: "Hedy Lamarr", inventions: 5 }}) 로 쌍으로 감싸야 합니다.

하지만 자세히 살펴보면 { } 는 js를 보는 창이라고 생각하면, { } js를 쓸 수 있게 하고 { name: "Hedy Lamarr", inventions: 5 } 라는 객체를 전달하는 것과 같음을 알 수 있습니다.

인라인 css 를 전달할 경우,

```tsx
export default function TodoList() {
  return (
    <ul style={{
      backgroundColor: 'black',
      color: 'pink'
    }}>
      <li>Improve the videophone</li>
      <li>Prepare aeronautics lectures</li>
      <li>Work on the alcohol-fuelled engine</li>
    </ul>
  );
}
주의할점은 css의 키 또한 카멜케이스입니다.
```

## JS 객체와 중괄호로 더 재밌게? 즐기기

```tsx
const person = {
  name: 'Gregorio Y. Zara',
  theme: {
    backgroundColor: 'black',
    color: 'pink'
  }
};

export default function TodoList() {
  return (

    <div style={person.theme}>
      <h1>{person.name}'s Todos</h1>
      <img
        className="avatar"
        src="https://i.imgur.com/7vQD0fPs.jpg"
        alt="Gregorio Y. Zara"
      />
      <ul>
        <li>Improve the videophone</li>
        <li>Prepare aeronautics lectures</li>
        <li>Work on the alcohol-fuelled engine</li>
      </ul>
    </div>
  );
}
인라인 스타일인데 왜 한번만 감쌀까요?
첫 { } 가 js 를 가능하게 해준다고 생각하면, 내부가 객체
{
    backgroundColor: 'black',
    color: 'pink'
  }
형태임을 알 수있습니다.
{person.name}의 경우는 string을 받기 떄문에 가능합니다.
```

## 도전과제

- 1. 실수를 고쳐보세요
  ```tsx
  const person = {
    name: 'Gregorio Y. Zara',
    theme: {
      backgroundColor: 'black',
      color: 'pink'
    }
  };

  export default function TodoList() {
    return (
      <div style={person.theme}>
        <h1>{person}'s Todos</h1>
        <img
          className="avatar"
          src="https://i.imgur.com/7vQD0fPs.jpg"
          alt="Gregorio Y. Zara"
        />
        <ul>
          <li>Improve the videophone</li>
          <li>Prepare aeronautics lectures</li>
          <li>Work on the alcohol-fuelled engine</li>
        </ul>
      </div>
    );
  }
  여기에 어던 실수가 있을까요?
  1. 일단 감싸야됩니다.
  2. person이 객체인데 <h1> </h1>안에 Object가 들어가면 오류가 발생합니다.
  string이 되면 되므로 person.name도 가능하고 "person"도 가능하고 JSON.stringify(person)도 가능합니다.
  아무튼 됩니다.

  export default function TodoList() {
    return (
  		<>
  	    <div style={person.theme}>
  	      <h1>{person.name}'s Todos</h1>
  	      <img
  	        className="avatar"
  	        src="https://i.imgur.com/7vQD0fPs.jpg"
  	        alt="Gregorio Y. Zara"
  	      />
  	      <ul>
  	        <li>Improve the videophone</li>
  	        <li>Prepare aeronautics lectures</li>
  	        <li>Work on the alcohol-fuelled engine</li>
  	      </ul>
  	    </div>
  	  </>
    );
  }
  로 써주면 됩니다.
  ```
- 2.  정보를 객체로 추출하세요
  ```tsx
  const person = {
    name: 'Gregorio Y. Zara',
    theme: {
      backgroundColor: 'black',
      color: 'pink'
    }
  };

  export default function TodoList() {
    return (
      <div style={person.theme}>
        <h1>{person.name}'s Todos</h1>
        <img
          className="avatar"
          src="https://i.imgur.com/7vQD0fPs.jpg"
          alt="Gregorio Y. Zara"
        />
        <ul>
          <li>Improve the videophone</li>
          <li>Prepare aeronautics lectures</li>
          <li>Work on the alcohol-fuelled engine</li>
        </ul>
      </div>
    );
  }
  잘 나오고 있지만, 현실은 만만치 않습니다. src를 박아놓을 일은 없습니다.
  동적으로 받기 위해서 src를 뚫어둡시다.

  그럼
  <img
          className="avatar"
          src={}
          alt="Gregorio Y. Zara"
        />
  잘 동작 합니다.
  그리고, 여기를 동적으로 받을 수있게
  person 객체속에 넣어줍니다.
  const person = {
    name: 'Gregorio Y. Zara',
    theme: {
      backgroundColor: 'black',
      color: 'pink'
    },
  	src : "https://i.imgur.com/7vQD0fPs.jpg"
  };
  그리고
  <img
          className="avatar"
          src={person.src}
          alt="Gregorio Y. Zara"
        />
  를 해주면 성공!
  ```
- 3.  JSX 중괄호 안에 표현식을 작성하세요
  ```tsx
  const baseUrl = 'https://i.imgur.com/';
  const person = {
    name: 'Gregorio Y. Zara',
    imageId: '7vQD0fP',
    imageSize: 's',
    theme: {
      backgroundColor: 'black',
      color: 'pink'
    }
  };

  export default function TodoList() {
    return (
      <div style={person.theme}>
        <h1>{person.name}'s Todos</h1>
        <img
          className="avatar"
          src="{baseUrl}{person.imageId}{person.imageSize}.jpg"
          alt={person.name}
        />
        <ul>
          <li>Improve the videophone</li>
          <li>Prepare aeronautics lectures</li>
          <li>Work on the alcohol-fuelled engine</li>
        </ul>
      </div>
    );
  }
  " " 안에 들어가면 string이 됩니다.
  지금 src는 "{baseUrl}{person.imageId}{person.imageSize}.jpg"인 스트링입니다.
  일단 src를 뚫어줍시다.

        <img
          className="avatar"
          src={}
          alt={person.name}
        />
  이제 js 가 들어갈 수 있습니다.
  baseUrl 이 들어가면 string값이 들어가게 되는 것입니다!
  그런식으로 값을 하나 하나 씩 더하면 하나의 string값이겠죠?
  근데 여기서 실수하지말아야할 부분은 { } 안에 서 .jpg는 string이 아닙니다.
  참고로 변수도 될 수 없습니다.
  변수이름을 시작할 수 있는 특수문자는 $ 와 _ 밖에 없습니다.

  export default function TodoList() {
    return (
      <div style={person.theme}>
        <h1>{person.name}'s Todos</h1>
        <img
          className="avatar"
          src={baseUrl +  person.imageId + person.imageSize+".jpg"}
          alt={person.name}
        />
        <ul>
          <li>Improve the videophone</li>
          <li>Prepare aeronautics lectures</li>
          <li>Work on the alcohol-fuelled engine</li>
        </ul>
      </div>
    );
  }
  ```

# 컴포넌트에 props 전달하기

props는 JSX 태그에 전달하는 정보입니다.

```tsx
function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.imgur.com/1bX5QH6.jpg"
      alt="Lin Lanying"
      width={100}
      height={100}
    />
  );
}

export default function Profile() {
  return <Avatar />;
}
```

img 태그에 className, src, alt, width, height 를 props로 전달합니다.

(이러한 ReactDOM 은 HTML 표준을 준수합니다.)

그렇다면, <Avatar>같은 커스텀 컴포넌트엔 어떻게 props를 전달할 수 있을까요?

## 컴포넌트에 props 전달하기

```tsx
export default function Profile() {
  return (
    <Avatar />
  );
}
전달하는 props 가 없는 상태
```

## Step1 : 자식 컴포넌트에 props 전달하기

```tsx
export default function Profile() {
  return (
    <Avatar
      person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
      size={100}
    />
  );
}
이렇게 전달하면, person 객체와 size를 전달해 보겠습니다.
```

## Step2 : 자식 컴포넌트에서 props 읽기

```tsx
function Avatar({ person, size }) {
  // person and size are available here
}
이러한 props들은 Avatar의 인자로써 {person, size}를 받아서
바로 변수처럼 쓸 수 있습니다.

구조분해할당 보고가기
import { getImageUrl } from './utils.js';

//기본적인 구조분해
//바로 인자로 받은 객체를 person ,size 변수로 나눠서 쓴다
function Avatar({ person, size }) {
  return (
    <img
      className="avatar"
      src={getImageUrl(person)}
      alt={person.name}
      width={size}
      height={size}
    />
  );
}
export default function Profile() {
  return (
    <div>
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
      <Avatar
        size={80}
        person={{
          name: 'Aklilu Lemma',
          imageId: 'OKS67lh'
        }}
      />
      <Avatar
        size={50}
        person={{
          name: 'Lin Lanying',
          imageId: '1bX5QH6'
        }}
      />
    </div>
  );
}
//구조분해 할당 뒤 p 로 변수명을 변경하여 사용한다.
function Avatar({ person:p, size }) {}
//person을 또 분해하여, name이라는 변수로 사용하고, person 객체도 사용한다.
//구조분해할당을 하고나면 person을 쓸 수 없어서 추가해주어야한다.
function Avatar({ person, person:{name}, size }) {}

//person에 접근해서 person으로 쓸래,
person 으로 접근하고 name 으로 접근해서 name으로 쓸래라고 생각한다.
function Avatar({ person, person:{name}, size }) {}

```

props를 사용하면 부모 컴포넌트와 자식을 독립적으로 생각할 수 있습니다.

Avatar 가 props를 어떻게 쓰는지 생각할 필요 없이 Profile의 person 또는 size props를 수정할 수 있습니다.

마찬가지로 Profile을 보지않고 Avatar가 props를 사용하는 방식을 바꿀 수 있습니다.

## prop의 기본값 지정하기

```tsx
function Avatar({ person, size = 100 }) {
  // ...
}

단 기본값을 지정해주었을땐, 값을 null로 넣으면 안된다.

			<Avatar
        size={null}
        person={{
          name: 'Aklilu Lemma',
          imageId: 'OKS67lh'
        }}
      />
이렇게 null을 하게되면, 값이 사라진다. falsy한 값을 넣으면
기본값은 사용되지 않습니다.
없거나, undefined만 실행됩니다.
하지만 여기서 size ={0} 은 실행되는게 맞지 않나요??

```

## JSX 전개 구문으로 props 전달하기

```tsx
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
객체 전체를 넘기는 경우
function Profile(props) {
  return (
    <div className="card">
      <Avatar {...props} />
    </div>
  );
}
객체 전체를 { ...spread} 해서 넘기는 경우 객체에 그 자체로 넘어간다.

const Profile  = ({props:{name}})=> (
   <Avatar {...name} />
)

"use strict";

const Profile = ({
  props: {
    name
  }
}) => /*#__PURE__*/React.createElement(Avatar, name);
로 변한됨을 보면, es6의 spread가 아니지 않나 생각해보게 된다.

```

전개 구문은 제한적으로 사용하세요

→ 그대로 객체 전부를 넘긴다는 것은 구조가 이상한지 확인해봐야한다.

```tsx
function Q(props) {
	return <div>{props.x}</div>
}

function P(props) {
	return <Q {...props} />
}

function A(props) {
	return <P {...props} />
}

export default function App () {
	return <A x={3} y={2} />
}
하나의 props를 변동시키면, 모두 확인해봐야하는 문제가 생길 수 있다.
3개의 컴포넌트 모두 계산을 해야하고 성능저하로 유발될 수 있음

최소한

function A({children}) {
	return <div>{children}</div>
}
export default function App () {
	return <A><P x={1} y={2}></P> </A>
}
이런식으로 P에 다이랙트로 가던지 방법을 찾아야한다.
```

## 자식을 JSX로 전달하기

브라우저 빌트인 태그는 중첩하는 것이 일반적입니다.

커스텀도 중첩하고 싶을 수 있습니다:)

```tsx
<div>
  <img />
</div>

<Card>
  <Avatar />
</Card>
```

JSX 태그 내에 콘텐츠를 중첩하면, 부모 컴포넌트는 해당 컨텐츠를 children이라는 prop으로 받을 것이다.

```tsx
import Avatar from './Avatar.js';

function Card({ children }) {
  return (
    <div className="card">
      {children}
    </div>
  );
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{
          name: 'Katsuko Saruhashi',
          imageId: 'YfeOqp2'
        }}
      />
    </Card>
  );
}

Card컴포넌트의 children 으로 <Avatar> 컴포넌트가 들어온다.
```

이런식으로 children 내부에서 무엇이 렌더링 되는지 알 필요가 없습니다.

유연하게 JSX로 채울 수 있는 구멍이 있다고 생각할 수 있습니다.

## 시간에 따라 props가 변하는 방식

```tsx
App.js
import { useState, useEffect } from 'react';
import Clock from './Clock.js';

function useTime() {
  const [time, setTime] = useState(() => new Date());
  useEffect(() => {
    const id = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(id);
  }, []);
  return time;
}

export default function App() {
  const time = useTime();
  const [color, setColor] = useState('lightcoral');
  return (
    <div>
      <p>
        Pick a color:{' '}
        <select value={color} onChange={e => setColor(e.target.value)}>
          <option value="lightcoral">lightcoral</option>
          <option value="midnightblue">midnightblue</option>
          <option value="rebeccapurple">rebeccapurple</option>
        </select>
      </p>
      <Clock color={color} time={time.toLocaleTimeString()} />
    </div>
  );
}

Clock.js
export default function Clock({ color, time }) {
  return (
    <h1 style={{ color: color }}>
      {time}
    </h1>
  );
}

부모 컴포넌트에서 state값이 달라질때마다 props가 변동되어 넘어간다.

```

이 예시는 컴포넌트가 **시간에 따라 다른 props를 받을 수 있음을 보여줍니다.**

Props는 항상 고정되어있지 않고 매초마다 time prop은 매초마다 변경되고, color는 선택시 변경됩니다.

props는 컴폰너트의 데이터를 처음뿐 아니라 모든 시점에 반영합니다. (변경되는)

그러나 props는 불변으로, 변경할 수 없습니다. 컴포넌트가 props를 변경해야 하는 경우 (사용자 상호작용, 새로운 데이터 응답), 부모 컴포넌트에 다른 props 새로운 객체 전달을 요청 해야합니다.

그러면, 이전의 props는 버려지고 결국 js엔진은 props의 메모리를 회수합니다.

- time을 연결시키지 않고, props로 color만 연결
  ```tsx
  App.js
  import { useState, useEffect } from "react";
  import Clock from "./Clock.js";

  function useTime() {
    const [time, setTime] = useState(() => new Date());
    useEffect(() => {
      const id = setInterval(() => {
        setTime(new Date());
      }, 1000);
      return () => clearInterval(id);
    }, []);
    return time;
  }

  export default function App() {
    const [color, setColor] = useState("lightcoral");
    return (
      <div>
        <p>
          Pick a color:{" "}
          <select value={color} onChange={(e) => setColor(e.target.value)}>
            <option value="lightcoral">lightcoral</option>
            <option value="midnightblue">midnightblue</option>
            <option value="rebeccapurple">rebeccapurple</option>
          </select>
        </p>
        <Clock color={color} />
      </div>
    );
  }

  Clock.js
  export default function Clock({ color, time }) {
    return <h1 style={{ color: color }}>time</h1>;
  }
  이 경우엔 props를 변경해도 유지된다.

  ```
  https://8htp3k.csb.app/

## 도전과제

- 1 컴포넌트 추출하기
  ```tsx
  import { getImageUrl } from './utils.js';

  export default function Gallery() {
    return (
      <div>
        <h1>Notable Scientists</h1>
        <section className="profile">
          <h2>Maria Skłodowska-Curie</h2>
          <img
            className="avatar"
            src={getImageUrl('szV5sdG')}
            alt="Maria Skłodowska-Curie"
            width={70}
            height={70}
          />
          <ul>
            <li>
              <b>Profession: </b>
              physicist and chemist
            </li>
            <li>
              <b>Awards: 4 </b>
              (Nobel Prize in Physics, Nobel Prize in Chemistry, Davy Medal, Matteucci Medal)
            </li>
            <li>
              <b>Discovered: </b>
              polonium (element)
            </li>
          </ul>
        </section>
        <section className="profile">
          <h2>Katsuko Saruhashi</h2>
          <img
            className="avatar"
            src={getImageUrl('YfeOqp2')}
            alt="Katsuko Saruhashi"
            width={70}
            height={70}
          />
          <ul>
            <li>
              <b>Profession: </b>
              geochemist
            </li>
            <li>
              <b>Awards: 2 </b>
              (Miyake Prize for geochemistry, Tanaka Prize)
            </li>
            <li>
              <b>Discovered: </b>
              a method for measuring carbon dioxide in seawater
            </li>
          </ul>
        </section>
      </div>
    );
  }
  두가지 프로필에 대한 몇가지 비슷한 마크업이 있다니까
  찾아보면 <section> </section>이 className:profile로 구조가 일치한다.

  import { getImageUrl } from './utils.js';

  function Profile({ person, imageSize = 70 }) {

    return (
      <section className="profile">
        <h2>{person.name}</h2>
        <img
          className="avatar"
          src={getImageUrl(person)}
          alt={person.name}
          width={imageSize}
          height={imageSize}
        />
        <ul>
          <li>
            <b>Profession:</b> {person.profession}
          </li>
          <li>
            <b>Awards: {person.awards.length} </b>
            ({person.awards.join(', ')})
          </li>
          <li>
            <b>Discovered: </b>
            {person.discovery}
          </li>
        </ul>
      </section>
    )
  }

  을 해주면
  export default function Gallery() {
    return (
      <div>
        <h1>Notable Scientists</h1>
        <Profile person={{
          imageId: 'szV5sdG',
          name: 'Maria Skłodowska-Curie',
          profession: 'physicist and chemist',
          discovery: 'polonium (chemical element)',
          awards: [
            'Nobel Prize in Physics',
            'Nobel Prize in Chemistry',
            'Davy Medal',
            'Matteucci Medal'
          ],
        }} />
        <Profile person={{
          imageId: 'YfeOqp2',
          name: 'Katsuko Saruhashi',
          profession: 'geochemist',
          discovery: 'a method for measuring carbon dioxide in seawater',
          awards: [
            'Miyake Prize for geochemistry',
            'Tanaka Prize'
          ],
        }} />
      </div>
    );
  }
  로 호출해서 처리할 수 있다.
  ```
- 2 prop에 따라 이미지 크기 조정하기
  ```tsx
  import { getImageUrl } from './utils.js';

  function Avatar({ person, size }) {
  	let changedSize;
  	if (size > 90) {
  		changedSize = 'b'
  	} else {
  		changedSize = 's'
  	}
    return (
      <img
        className="avatar"
        src={getImageUrl(person, 'b')}
        alt={person.name}
        width={size}
        height={size}
      />
    );
  }

  export default function Profile() {
    return (
      <Avatar
        size={40}
        person={{
          name: 'Gregorio Y. Zara',
          imageId: '7vQD0fP'
        }}
      />
    );
  }

  아바타의 사이즈를 분기로 처리해야하므로 변수하나 만들고,
  분기별로 변동되는 값을 img에 넣어주도록하자
  function Avatar({ person, size }) {
  	let changedSize = 's'
  	if (size > 90) {
  		changedSize = 'b'
  	}
    return (
      <img
        className="avatar"
        src={getImageUrl(person, changedSize)}
        alt={person.name}
        width={size}
        height={size}
      />
    );
  }

  const ratio = window.devicePixelRatio;를 추가해서 DPI화면에서
  더 선명한 이미지를 표시할 수 있다고한다.
  ```
- 3 children prop에 JSX 전달하기
  ```tsx
  export default function Profile() {
    return (
      <div>
        <div className="card">
          <div className="card-content">
            <h1>Photo</h1>
            <img
              className="avatar"
              src="https://i.imgur.com/OKS67lhm.jpg"
              alt="Aklilu Lemma"
              width={70}
              height={70}
            />
          </div>
        </div>
        <div className="card">
          <div className="card-content">
            <h1>About</h1>
            <p>Aklilu Lemma was a distinguished Ethiopian scientist who discovered a natural treatment to schistosomiasis.</p>
          </div>
        </div>
      </div>
    );
  }

  Card를 추출하고, children prop을 사용하자
  className보고 유추해서 만들어보자

  Card를
        <div className="card">
          <div className="card-content">
            <h1>Photo</h1>
            <img
              className="avatar"
              src="https://i.imgur.com/OKS67lhm.jpg"
              alt="Aklilu Lemma"
              width={70}
              height={70}
            />
          </div>
        </div>
  전체라고 생각하고
          <div className="card-content">
            <h1>Photo</h1>
            <img
              className="avatar"
              src="https://i.imgur.com/OKS67lhm.jpg"
              alt="Aklilu Lemma"
              width={70}
              height={70}
            />
          </div>
  를 카드 컨텐츠라고 생각한다면

  function Card({children}) {
  	return (
  	<div className="card">
  		{children}
  	</div>
  	)
  }
  function CardContentImg({type, imgProps}) {
  	return (
  			<div className="card-content">
            <h1>{type}</h1>
            <img
  						{...imgProps}
            />
          </div>
  	)
  }

  function CardContentAbout({type, text}) {
  	return (
  			<div className="card-content">
            <h1>{type}</h1>
            <p>{text}</p>
          </div>
  	)
  }
  export default function Profile() {
    return (
      <div>
        <Card>
          <CardContentImg
            type="Photo"
            imgProps={{
              src: "https://i.imgur.com/OKS67lhm.jpg",
              alt: "Aklilu Lemma",
              width: 100,
              height: 100,
              className:'avatar'
            }}
          />
        </Card>
        <Card>
          <CardContentAbout
            type="About"
            text="Aklilu Lemma was a distinguished Ethiopian scientist who discovered a natural treatment to schistosomiasis."
          />
        </Card>
      </div>
    );
  }
  처럼 컴포넌트 구분을 할 수 있습니다.
  ```
