> [2회] UI 표현하기 - 1

> 링크 : https://bronzed-icicle-65c.notion.site/2-UI-1-e38a2fb0e31b4b45ad50ba7447250836?pvs=4

# UI 표현하기

---

- React는 UI를 렌더링하기 위한 Javascript 라이브러리
- UI는 버튼, 텍스트, 이미지같은 작은 요소로 구성

  - 여기서 작은 요소들은 재사용 가능 + 중첩가능? ⇒ 컴포넌트
    <aside>
    💡 여기서 컴포넌트는 **리액트 컴포넌트**를 칭한다고 생각한다.

    </aside>

## 첫 컴포넌트

---

- 컴포넌트 === 독립된 UI 조각, UI 구성 요소
- **리액트 컴포넌트** === 마크업을 얹을 수 있는 Javascript **함수**

<aside>
💡 리액트 컴포넌트는 클래스 컴포넌트, 함수 컴포넌트 2가지가 있다.
최근에는 함수 컴포넌트를 메인으로 밀고 있어서 위와 같이 표현한 것 같다.

</aside>

- 마크업뿐이지만 CSS와 JS를 결합하여 상호작용하는 UI 애플리케이션 개발이 가능하다

  ```html
  // example
  <article>
    <h1>My First Component</h1>
    <ol>
      <li>Components: UI Building Blocks</li>
      <li>Defining a Component</li>
      <li>Using a Component</li>
    </ol>
  </article>
  ```

  ```tsx
  // 1. React component의 이름은 사용되는 시점에 대문자 시작이어야 한다.
  // 2. 마크업에 style(CSS), onClick(javascript) 결합가능

  function MyFirstComponent() {
    return (
      <article>
        <h1 style={{ color: "blue" }}>My First Component</h1>
        <ol>
          <li onClick={() => console.log("first item")}>
            Components: UI Building Blocks
          </li>
          <li onClick={() => console.log("second item")}>
            Defining a Component
          </li>
          <li onClick={() => console.log("second item")}>Using a Component</li>
        </ol>
      </article>
    );
  }
  ```

- HTML 태그와 마찬가지로 컴포넌트를 사용할 수 있다.
    <aside>
    💡 아래 예시코드를 대충 그려보자면…
    
    </aside>
    
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/693bc352-6af5-47d2-866f-3108bb9e50f3/39a8feb5-ed11-42c1-bb86-1f21d5956191/Untitled.jpeg)
    
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

- [Chakra UI](https://chakra-ui.com/), [Material UI.](https://material-ui.com/) 같은 유명 Design System을 사용해서 별도의 디자인 작업없이도 프로젝트를 예쁘게 만들 수 있다.

<aside>
💡 최근에 디자인 뿐만 아니라 사용방식도 고려했을 때 [Radix UI](https://www.radix-ui.com/)  추천

</aside>

### 컴포넌트 정의하기

- 기존 : 컨텐츠를 마크업 작업한 후, JS를 입히기
- 최근: 왠만하면 상호작용은 디폴트
- 리액트 컴포넌트는 마크업으로 뿌릴 수 있는 Javascript 함수입니다(?)

### 컴포넌트 빌드 방식

1. 컴포넌트 내보내기(export, export default)
2. 함수 정의하기

   ### React 컴포넌트는 일반 Javascript 함수지만 대문자로 시작해야 한다

   - **꼭 그렇지는 않다! - 정재남, 이승효**

     - **JSX안에서 컴포넌트가 사용될 시점에 대문자로 시작하기만 하면 된다.**
     - 아래 참고 ([codesandbox](https://codesandbox.io/s/clfc0d?file=/App.js&utm_medium=sandpack))

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

3. 마크업 추가하기

   - 아래코드는 HTML처럼 보이지만 실제로는 Javascript다!
   - **JSX**라는 DX를 증진시키기 위해 리액트가 적용한 [설탕 문법](https://en.wikipedia.org/wiki/Syntactic_sugar)이다 ([예제 링크](https://stackblitz.com/edit/stackblitz-starters-irvb8w?file=src%2FApp.js))

     ```tsx
     return <div>Hello {props.toWhat}</div>;
     // return React.createElement('div', null, `Hello ${props.toWhat}`);

     return (
       <div>
         <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
       </div>
     ); // 괄호로 감싸져 있어야한다. 안그럼 return 뒷 라인의 코드들은 무시됨
     ```

### 컴포넌트 사용하기

```tsx
function Profile() {
  return <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />;
}

export default function Gallery() {
  return (
    <section>
      <h1>Amazing scientists</h1>
      <Profile /> {/* 다른 컴포넌트 안에서 중복 사용이 가능하다 */}
      <Profile />
      <Profile />
    </section>
  );
}
```

### 브라우저에 표시되는 내용

- 대소문자에 주의!
  - **소문자는 HTML 태그**로 인식
  - **대문자는 React Component**로 인식

### 컴포넌트 중첩 및 구성

- **컴포넌트는 함수**! 같은 파일에서 여러 컴포넌트를 포함할 수 있다.
    <aside>
    💡 그럼에도 파일 하나당 컴포넌트가 하나이면 좋은건 관련된 코드는 최대한 가까울 수록 유지보수와 가독성 면에서 이점이 있기 때문이다.
    
    </aside>

- 특정 컴포넌트 A 안에서 B라는 특정 컴포넌트가 렌더링(`함수 컴포넌트에선 return 영역`) 되고 있다면 A는 **부모 컴포넌트**, B는 **자식 컴포넌트**라 부를 수 있다.

  ```tsx
  function Profile() {
    return (
      <img src="https://i.imgur.com/MK3eW3As.jpg" alt="Katherine Johnson" />
    );
  }

  export default function Gallery() {
    // 부모
    return (
      <section>
        <h1>Amazing scientists</h1>
        <Profile /> // 자식
        <Profile /> // 자식
        <Profile /> // 자식
      </section>
    );
  }
  ```

- 컴포넌트 정의를 중첩해서는 안된다
  ```tsx
  export default function Gallery() {
    // 🔴 Never define a component inside another component!
    function Profile() {
      // ...
    }
    // ...
  }
  ```

### 컴포넌트의 모든 것

- React 애플리케이션은 `root`컴포넌트에서 시작
- 대부분의 React앱은 모든 부분에서 컴포넌트를 사용
- React 기반의 프레임워크들은 한층 더 발전
  - React 컴포넌트를 HTML에 자동으로 생성
  - Javascript가 로드되기 전에 일부 컨텐츠를 표시
    ⇒ Server Sider Rendering(SSR), Static Site Generation(SSG)
- But, 여전히 많은 웹사이트에서 약간의 상호작용 추가 목적으로만 사용중

### 요약

- 앱의 **재사용 가능한 UI 요소**인 **컴포넌트** 제작 가능
- 앱에서 모든 UI는 컴포넌트
- React 컴포넌트는 몇 가지를 제외하고는 JavaScript 함수다
  1. 컴포넌트의 이름은 **항상 대문자로 시작**합니다.
  2. **JSX 마크업을 반환**합니다.

### 챌린지 도전

- 1번
  - export default 추가 ([링크](https://codesandbox.io/s/amazing-pond-8v653n?file=/App.js))
- 2번
  - 괄호 추가
    ```tsx
    export default function Profile() {
      return (
        <img src="https://i.imgur.com/jA8hHMpm.jpg" alt="Katsuko Saruhashi" />
      );
    }
    ```
- 3번

  - 리액트 컴포넌트는 대문자로 시작

    ```tsx
    function Profile() {
      return <img src="https://i.imgur.com/QIrZWGIs.jpg" alt="Alan L. Hart" />;
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
    ```

- 4번

  ```tsx
  // Write your component below!

  export default function Congratulations() {
    return <h1>Good job!</h1>;
  }
  ```

---

## 컴포넌트 import, export

- 컴포넌트를 조합해서 새로운 컴포넌트 제작 가능
- 파일 내 코드가 길어져 컴포넌트를 분리
  → import, export로 다른 컴포넌트에서 쉽게 가져오고, 다른 컴포넌트로 보낼 수 있음

### Root 컴포넌트

- Pure React 프로젝트라면
  - 기본적으로 App.js에 있는 `export default` 컴포넌트 === root 컴포넌트
  - 설정에 따라 root 컴포넌트가 다른 파일에 위치할 수도 있음
  - Next.js는 페이지별로 root 컴포넌트가 다를 수 있음

### 컴포넌트 import 혹은 export 하는 방법

- 컴포넌트를 다른 파일로 이동시키는 세 가지 단계

  1. 컴포넌트를 추가할 JS 파일을 **생성**합니다.
  2. 새로 만든 파일에서 함수 컴포넌트를 **export** 합니다. ([default](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/export#using_the_default_export) 또는 [named](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/export#using_named_exports) export 방식을 사용합니다)
  3. 컴포넌트를 사용할 파일에서 **import** 합니다. (적절한 방식을 선택해서 [default](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/import#importing_defaults) 또는 [named](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/import#import_a_single_export_from_a_module)로 import 합니다)

  - 예제

    ```tsx
    // App.js
    import Gallery from "./Gallery.js";

    export default function App() {
      return <Gallery />;
    }

    // Gallery.js
    function Profile() {
      return <img src="https://i.imgur.com/QIrZWGIs.jpg" alt="Alan L. Hart" />;
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
    ```

> 가끔 `.js` 와 같은 파일 확장자가 없을 때도 있음 (`import Gallery from './Gallery';`)
> (.js를 붙이는 경우는 [native ES modules](https://developer.mozilla.org/ko/docs/Web/JavaScript/Guide/Modules) 사용법에 가깝)

### Default와 Named Exports

- 보통 자바스크립트에선 두가지 방식으로 export
  - default
  - named export
  - **단, default export는 한 파일에 하나, named export는 여러개 존재 가능**
    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/693bc352-6af5-47d2-866f-3108bb9e50f3/b3a76abc-9a1f-44f0-ae2e-241470296aec/Untitled.png)
- _Default_ import를 사용하는 경우 원한다면 `import` 단어 후에 다른 이름으로 값을 가져올 수 있습니다.

  - `import Banana from './button.js'` 라고 쓰더라도 같은 default export 값을 가져오게 됩니다.
  - 반대로 named import를 사용할 때는 양쪽 파일에서 사용하고자 하는 값의 이름이 같아야 하기 때문에 _named_ import라고 불립니다

- 보편적으로
  - 한 파일에 하나의 컴포넌트만 export할 땐, **default export** 사용
  - 여러 컴포넌트 export할 땐, **named export** 사용

<aside>
💡 생각보다 default export보단 named export를 많이 사용하고 있다.
(ex. 라이브러리 export할 때 생각보다 좋음, index.js 작업을 좀 하긴 해야함)
([링크](https://github.com/mui/material-ui/tree/master/packages/mui-base/src), [링크](https://stackblitz.com/edit/stackblitz-starters-irvb8w?file=src%2FApp.js))

</aside>

### 한 파일에서 여러 컴포넌트 import / export 하기

- 한 파일에서는 단 하나의 default export만 사용할 수 있지만
- named export는 여러 번 사용할 수 있습니다.

### 요약

- Root 컴포넌트
  ⇒ 💡 기본적으로 App.js. 프레임워크는 또 다름
- 컴포넌트를 import 하거나 export 하는 방법
  ⇒ 💡 default export, named export
- 언제 default 또는 named imports와 exports를 사용할지
  ⇒ 기본적으로 한 파일에 하나면 default, 여러 파일이면 named export
  ⇒ 💡 상황에 따라 import 라인을 간단하게 하기 위해 named export와 index.js를 활용
- 한 파일에서 여러 컴포넌트를 export 하는 방법
  ⇒ named export!

### 챌린지 (생략 - 너무 간단)

---

## JSX로 마크업 작성하기

- *JSX*는 JavaScript를 확장한 설탕 문법
- JavaScript 파일을 HTML과 비슷하게 마크업을 작성할 수 있도록 해줌
- 다른 방법도 있지만, DX(Developer Experience)를 고려해 JSX 사용

```tsx
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

- 위와 같은 마크업 구문을 JSX라고 한다. (return 영역에 마크업 형태의 구문들이 들어가 있는 형태)
- JSX는 HTML보다 엄격하게 문법을 맞춰줘야 한다.

  - 1. 태그는 닫아줘야 한다.
       `<br />` 처럼 닫아주기
  - **2. 컴포넌트에서 여러개의 JSX 태그들을 반환할 수 없다**

    ```tsx
    function AboutPage() {
      return (
        <>
          <h1>About</h1>
          <p>
            Hello there.
            <br />
            How do you do?
          </p>
        </>
      );
    }
    ```

    <aside>
    💡 특별한 이유가 없다면
    React.Fragment(<></> 혹은 <React.Fragment> </React.Fragment>)로 감싸주는게 좋다.

    </aside>

### JSX: Javascript에 마크업 넣은 형태

[ HTML](https://ko.react.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fwriting_jsx_html.png&w=750&q=75)

                                     HTML

[ Javascript](https://ko.react.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fwriting_jsx_js.png&w=750&q=75)

                                  Javascript

- 보통은 HTML, CSS, Javascript를 각각의 파일로 분리해서 관리
- 웹이 발전하면서 사용자와의 상호작용이 필요해지면서 **Javascript가 HTML을 담당**
  ⇒ React 컴포넌트의 탄생 배경

![            `Sidebar.js` React component](https://prod-files-secure.s3.us-west-2.amazonaws.com/693bc352-6af5-47d2-866f-3108bb9e50f3/88f538f3-b1a1-4f6d-88a5-5eefa6014ced/Untitled.png)

            `Sidebar.js` React component

[ `Form.js` React component](https://ko.react.dev/_next/image?url=%2Fimages%2Fdocs%2Fdiagrams%2Fwriting_jsx_form.png&w=750&q=75)

               `Form.js` React component

- 버튼의 렌더링 로직과 버튼의 마크업이 함께 있으면, 매번 변화가 생길 때마다 서로 동기화 상태를 유지할 수 있습니다. 반대로 버튼의 마크업과 사이드바의 마크업처럼 서로 관련이 없는 항목들은 서로 분리되어 있으므로, 각각 개별적으로 변경하는 것이 더 안전합니다
- 각 React 컴포넌트는 React가 브라우저에 마크업을 렌더링할 수 있는 JavaScript 함수입니다. React 컴포넌트는 JSX라는 확장된 문법을 사용하여 마크업을 나타냅니다. JSX는 HTML과 비슷해 보이지만, 조금 더 엄격하며 동적으로 정보를 표시할 수 있습니다. JSX를 이해하는 가장 좋은 방법은 HTML 마크업을 JSX 마크업으로 변환해 보는 것입니다.

JSX ≠ React

- JSX는 React 개발자들의 DX 향상을 위해 만든 설탕 문법
- React는 Javascript다

### HTML ⇒ JSX 변환하기

- 예시
  ```tsx
  export default function TodoList() {
    return (
      // 이것은 동작하지 않습니다!
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
  ```
- 💡 정답
  ```tsx
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
          <li>Rehearse a movie scene</li>
          <li>Improve the spectrum technology</li>
        </ul>
      </>
    );
  }
  ```

### JSX 사용 규칙

1. 하나의 Root 엘리먼트로 반환
   - **왜 여러 JSX 태그를 하나로 감싸줘야 하는가?**
     - JSX는 HTML처럼 보이지만 내부적으로는 일반 JavaScript 객체로 변환
     - 하나의 배열로 감싸지 않은 하나의 함수에서는 두 개의 객체를 반환할 수 없음
       ⇒ 따라서 또 다른 태그나 React.Fragment(<></> 혹은 <React.Fragment></React.Fragment>)로 감싸지 않으면 두 개의 JSX 태그를 반환할 수 없습니다.
2. 모든 태그는 명시적으로 닫아주기
3. 대부분 camelCase로 작성

   - JSX의 어트리뷰트는 Javascript 객체의 키가 된다
   - 컴포넌트에서는 종종 어트리뷰트를 변수로 읽으려 함 ⇒ 변수명에 제한이 있음
     ⇒ React에서 HTML, SVG의 어트리뷰트 대부분이 캐멀케이스인 이유 (충돌남! [여기서 찾자](https://ko.react.dev/reference/react-dom/components/common))

   > **역사적인 이유로 `aria-*` `data-*` 의 속성은 HTML과 동일하게 사용**

### 팁: JSX 변환기 사용하기

- 변환기를 사용해서 HTML, SVG를 JSX로 변환하는 것을 추천 ([링크](https://transform.tools/html-to-jsx))
    <aside>
    💡 SVG인 경우 라이브러리 중 [svgr](https://react-svgr.com/) 라는 라이브러리를 이용하면 별도의 변환기 없이도 SVG를 React 컴포넌트처럼 사용할 수 있다.
    
    ```tsx
    import Star from './star.svg'
    const Example = () => (
      <div>
        <Star />
      </div>
    )
    ```
    
    </aside>

### 요약

- React 컴포넌트는 관련 있는 마크업과 렌더링 로직을 함께 그룹화
- JSX는 HTML과 비슷하지만 몇 가지 차이점 존재.
  - 필요한 경우 [변환기](https://transform.tools/html-to-jsx) 사용
- 오류 메시지로 마크업 수정 힌트를 얻을 수 있음

### 챌린지 (생략 - 너무 간단)

---

## 중괄호가 있는 JSX안에서 Javascript 사용하기

### 따옴표로 문자열 전달

- `{}`를 사용해 Javascript의 값을 사용할 수 있다.
    <aside>
    💡 사실 값보단 무언가라고 표현하고 싶다. 
    왜냐면 이벤트 핸들링 함수도 `{}`를 통해서 전달하고 있기 때문
    ex) `<button onClick={() ⇒ console.log('hello')}>Greeting</button>`
    
    </aside>
    
    ```tsx
    export default function Avatar() {
      const avatar = 'https://i.imgur.com/7vQD0fPs.jpg';
      const description = 'Gregorio Y. Zara';
      return (
        <img
          className="avatar"
          src={avatar}
          alt={description}
        />
      );
    }
    ```

### 중괄호 사용하기

- 중괄호 사용처
  - JSX 태그안의 문자
    ```tsx
    <h1>{name}'s To Do List</h1>
    ```
  - = 바로뒤에 오는 속성
    ```tsx
    <img className="avatar" src={avatar} alt={description} />
    ```

### 이중 중괄호 사용하기

- JSX의 CSS와 다른 객체
  ```tsx
  export default function TodoList() {
    return (
      <ul
        style={{
          backgroundColor: "black",
          color: "pink",
        }}
      >
        <li>Improve the videophone</li>
        <li>Prepare aeronautics lectures</li>
        <li>Work on the alcohol-fuelled engine</li>
      </ul>
    );
  }
  ```
  - HTML태그 안 style 속성에 인라인으로 스타일을 적용할 경우, camelCase로 작성해야 한다.

### Javascript 객체, 중괄호 더 알아보기

```tsx
const person = {
  name: "Gregorio Y. Zara",
  theme: {
    backgroundColor: "black",
    color: "pink",
  },
};

export default function TodoList() {
  return (
    <div style={person.theme}>
      <h1>{person.name}'s Todos</h1> // person 객체의 name이라는 요소로 접근
      가능
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
```

### 요약

- 따옴표 안의 JSX 어트리뷰트는 문자열로 전달
- 중괄호를 사용 ⇒ 마크업에서 JavaScript 논리와 변수 사용 가능
- JSX 태그 내부 또는 어트리뷰트의 `=` 뒤에서 작동
- `{{` 및 `}}` 는 특별한 문법이 아닙니다. (JSX 중괄호 안에 들어 있는 JavaScript 객체)

### 챌린지 (간단해서 패스)

---

## 컴포넌트에 Props 전달하기

- React 컴포넌트는 props를 이용해 서로 통신
- 모든 부모 컴포넌트는 props 통해서 자식 컴포넌트에게 데이터를 전달 가능

### 친숙한 Props

- Props: JSX 태그에 전달하는 정보

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

### 컴포넌트에 Props 전달하기

1. 자식 컴포넌트에 Props 전달하기
   - person과 size를 전달하고 있다
     ```tsx
     export default function Profile() {
       return (
         <Avatar
           person={{ name: "Lin Lanying", imageId: "1bX5QH6" }}
           size={100}
         />
       );
     }
     ```
2. 자식 컴포넌트에서 Props 받아서 읽기

   - 자식 컴포넌트에서 props은 객체로 넘어온다.

     ```tsx
     function Avatar({ person, size }) {
       // person과 size는 이곳에서 사용가능합니다.
     }

     function Avatar(props) {
     	const {person, size} = props; // 구조분해를 활용할 수 있다.
     ...
     }
     // 구조분해(https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment#Unpacking_fields_from_objects_passed_as_a_function_parameter)
     ```

### prop 기본값 지정하기

- 아래 예시에서 size가 props로 전달안되도 100이라는 기본값으로 사용 가능

```tsx
function Avatar({ person, size = 100 }) {
  // ...
}
```

### spread 문법으로 props 전달하기

- 반복적인 코드는 가독성 높이긴 하나 간결함이 좋을 때도 있다

  ```tsx
  // before
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

  // after
  function Profile(props) {
    return (
      <div className="card">
        <Avatar {...props} />
      </div>
    );
  }

  // after2
  function Profile(props) {
    const { person, ...rest } = props;
    return (
      <div className="card">
        <Avatar person={person} {...rest} />
      </div>
    );
  }
  ```

### 자식을 JSX로 전달하기

- `children` 를 사용해 부모 컴포넌트의 내부에 위치한 자식 컴포넌트를 prop으로 받는다.
  - `children` prop을 가지고 있는 컴포넌트는 마치 임의의 JSX로 채울 수 있는 구멍이 있는 것

```tsx
import Avatar from "./Avatar.js";

function Card({ children }) {
  return <div className="card">{children}</div>;
}

export default function Profile() {
  return (
    <Card>
      <Avatar
        size={100}
        person={{
          name: "Katsuko Saruhashi",
          imageId: "YfeOqp2",
        }}
      />
    </Card>
  );
}
```

### 시간에 따라 Props가 변하는 방식

- [링크](https://codesandbox.io/s/jrycgd?file=%2FClock.js&utm_medium=sandpack)
- 컴포넌트가 시간에 따라 다른 props를 받을 수 있음
  - **props는 고정되어 있지 않다! 동적이다!**
  - But, 불변성을 갖고 있음
    ⇒ 부모 컴포넌트에 다른 props(새로운 객체)를 전달하도록 **요청**해야함
    ⇒ **props 변경을 시도하지 마세요! (State를 활용하자)**
  - **💡 불변성 추가 설명 (By ChatGPT)**
    불변성(Immutability)은 소프트웨어 개발에서 중요한 개념 중 하나
    - 데이터나 객체의 상태가 생성된 후에 변경되지 않음을 의미
      ⇒ 한 번 생성된 데이터는 이후에 수정되지 않는 특성 가짐
    1. **데이터 신뢰성 (Data Integrity):** 불변한 데이터는 한 번 설정된 값을 변경할 수 없으므로 데이터의 신뢰성이 보장됩니다. 다른 부분에서 데이터를 변경하거나 수정하는 부작용이 없으므로 코드의 예측 가능성이 향상됩니다.
    2. **다중 스레드 환경에서의 안정성 (Thread Safety):** 불변한 데이터는 여러 스레드에서 안전하게 공유될 수 있습니다. 데이터가 변경되지 않기 때문에 동시에 여러 스레드가 접근해도 데이터의 무결성을 위협하지 않습니다.
    3. **디버깅과 추적 용이성 (Debugging and Traceability):** 불변한 데이터는 시간에 따른 데이터의 상태 변경을 추적하기 쉽게 만듭니다. 버그를 디버깅하거나 코드의 이해를 돕는 데 유용합니다.
    4. **성능 최적화 (Performance Optimization):** 불변한 데이터는 어떤 상황에서는 변경 가능한 데이터보다 효율적일 수 있습니다. 예를 들어, 변경 가능한 데이터 구조를 복사하는 대신 불변한 데이터 구조의 변경 버전을 쉽게 생성할 수 있습니다.
       불변성은 함수형 프로그래밍과도 관련이 깊으며, 함수형 언어에서는 불변성이 매우 중요한 원칙 중 하나입니다. 함수형 프로그래밍에서는 함수가 입력 값을 변경하지 않고 새로운 값을 반환하는 것이 일반적입니다.
       예를 들어, 불변성을 가진 언어 또는 라이브러리에서는 문자열이나 배열과 같은 데이터 구조를 변경하는 대신, 새로운 문자열이나 배열을 생성하고 필요한 수정을 적용합니다. 이렇게 하면 원본 데이터는 보존되며 부작용을 줄이고 코드의 예측 가능성과 안정성을 향상시킬 수 있습니다.
       불변성은 현대적인 소프트웨어 개발에서 중요한 개념 중 하나이며, 코드의 품질과 안정성을 향상시키는 데 기여합니다.

### 요약

- Props를 전달? JSX에 props를 추가 (Like HTML 어트리뷰트)
- Props를 읽으려면? `function Avatar({ person, size })` 구조 분해 할당!
- Props에 기본값을 지정 가능(`size = 100`)
  - 이는 누락되거나 `undefined` 인 props에 사용
- 모든 props를 `<Avatar {...props} />`로 전달할 수 있습니다
  - JSX spread 문법을 사용할 수 있지만 과용 금지!
- `<Card><Avatar /></Card>`와 같이 중첩된 JSX는 `Card`컴포넌트의 자식 컴포넌트로 표시.
- Props는 **읽기 전용 스냅샷**으로, 렌더링할 때마다 새로운 버전의 props를 받습니다.
- Props는 변경할 수 없습니다**(불변성)**.
  - 상호작용이 필요한 경우 **state**를 설정해야 함

### 챌린지

- 1번

  ```tsx
  import { getImageUrl } from "./utils.js";

  export default function Gallery() {
    return (
      <div>
        <h1>Notable Scientists</h1>
        <Profile
          name={"Maria Skłodowska-Curie"}
          src={getImageUrl("szV5sdG")}
          profession={"physicist and chemist"}
          award={"(Miyake Prize for geochemistry, Tanaka Prize)"}
          discoverd={`polonium (element)`}
        />
        <Profile
          name={"Katsuko Saruhashi"}
          src={getImageUrl("YfeOqp2")}
          profession={"geochemist"}
          award={
            "(Nobel Prize in Physics, Nobel Prize in Chemistry, Davy Medal, Matteucci Medal)"
          }
          discoverd={"a method for measuring carbon dioxide in seawater"}
        />
      </div>
    );
  }

  function Profile({ name, src, profession, award, discovered }) {
    return (
      <section className="profile">
        <h2>{name}</h2>
        <img className="avatar" src={src} alt={name} width={70} height={70} />
        <ul>
          <li>
            <b>Profession: </b>
            {profession}
          </li>
          <li>
            <b>Awards: 4 </b>
            {award}
          </li>
          <li>
            <b>Discovered: </b>
            {discovered}
          </li>
        </ul>
      </section>
    );
  }
  ```

- 2번

  ```tsx
  import { getImageUrl } from "./utils.js";

  const ratio = window.devicePixelRatio; // 높은 DPI 화면에서 더 선명하게!

  function Avatar({ person, size }) {
    let thumbnailSize = "s";
    if (size * ratio > 90) {
      thumbnailSize = "b";
    }
    return (
      <img
        className="avatar"
        src={getImageUrl(person, thumbnailSize)}
        alt={person.name}
        width={size}
        height={size}
      />
    );
  }

  export default function Profile() {
    return (
      <>
        <Avatar
          size={40}
          person={{
            name: "Gregorio Y. Zara",
            imageId: "7vQD0fP",
          }}
        />
        <Avatar
          size={70}
          person={{
            name: "Gregorio Y. Zara",
            imageId: "7vQD0fP",
          }}
        />
        <Avatar
          size={120}
          person={{
            name: "Gregorio Y. Zara",
            imageId: "7vQD0fP",
          }}
        />
      </>
    );
  }
  ```

- 3번
