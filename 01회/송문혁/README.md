> [1회] 설치하기, 빠르게 시작하기

> 링크 : https://bronzed-icicle-65c.notion.site/1-353f61089b524189ae8e83a594c88407?pvs=4

# 설치하기

---

## 새 프로젝트로 시작

- React기반의 프레임워크 중 하나를 사용하는게 최근에는 좋다.
  - [Next.js](https://nextjs.org/), [Remix](https://remix.run/)가 가장 각광받고 있다.
  - 주로, 라우팅(Routing), 데이터 통신(data fetching), HTML 생성 같은 기능을 제공

<aside>
💡 여기서 [Vue](https://kr.vuejs.org/), [Angular](https://angular.io/)는 프레임워크, [React](https://ko.react.dev/)는 라이브러리라는 점도 기억하자.

</aside>

- 로컬에서 개발을 위해 [Node.js](https://nodejs.org/ko)를 설치해야 한다.

<aside>
💡 브라우저에서만 사용할 수 있었단 Javascript를 브라우저 밖에서도 코드를 실행할 수 있도록 해준 Javascript 런타임임을 기억하자

</aside>

### Next.js

- [vercel](https://vercel.com/)에서 관리하고 있으며 정적 블로그부터 복잡한 동적 애플리케이션까지 다양한 React 애플리케이션을 개발할 수 있도록 해줌

### Remix

- [Shopify](https://www.shopify.com/)가 관리하는 프레임워크로 중첩 라우팅이 가능하고 각 파트별로 데이터를 읽어서 사용자의 행동에 반응해서 다시 그려질 수 있다.

### Gatsby

- Netlify에서 관리하며 빠른 CMS 지원 웹사이트를 제작하는데 좋은 프레임워크

### Expo

- Expo라는 기업에서 관리하는 것으로 네이티브 UI를 갖춘 범용 안드로이드, iOS , 웹앱을 만들 수 있는 프레임워크

### 프레임워크없이 React 사용이 가능하나 프레임워크 사용을 권장

- 라우팅, 데이터 통신을 위한 라이브러리 추가는 필수적
- But, 라이브러리 추가할 때마다 번들 사이즈 증가 → **code spliting 필요**
- 데이터 통신이 복잡해지면 서버 - 클라이언트 네트워크에 워터폴이 발생 → **SSR 필요**
  ⇒ 번들러 + 라우터 + 데이터 통신 라이브러리 통합 필요
- 여기서, 프레임워크는 개발자의 추가적인 노력없이 위 문제들을 해결해주고 있음
- 게다가 프레임워크라 컨텍스트와 기술을 유지할 수 있다 → **유지보수에 용이(?)**

- 그렇다고 프레임워크 사용을 강제하는 것은 아님. (Vite, Parcel 사용하고, react, react-dom 등을 다운받아서 커스텀)

### 최첨단 React 프레임워크

- React를 지속적 개선하기 위해 React 프레임워크를 더욱 발전시키는 방향으로 결정
    <aside>
    💡 [Andrew Clark](https://twitter.com/acdlite)가 next.js로 이적한 것처럼 react core팀에서 몇 명의 개발자들이 next.js로 이적
    
    </aside>

- 예시로 Next.js에서 [React Server Component](https://react.dev/blog/2020/12/21/data-fetching-with-react-server-components)(RSC, [한글 자료](https://tech.kakaopay.com/post/react-server-components/))와 같은 최첨단 React 기능을 연구, 개발, 통합 및 테스트하는데 React core팀과 손을 잡음
- 1, 2년 안에 위에서 언급한 모든 React 프레임워크에서 이러한 기능을 지원하기를 희망하는 중
    <aside>
    💡 React 기반의 개발자들이라면 Next.js의 트렌드 팔로잉은 필수가 되지 않았나 싶다
    
    </aside>

- **Next.js의 App Router는 React 팀의 풀스택 아키텍처 비전 실현을 위해 재설계된 Next.js API**

### React 팀의 풀스택 아키텍처 비전

- Next.js의 App Router 번들러가 RSC 명세를 완벽하게 구현하고 있다.
  - 빌드 타임, 서버 전용, 인터랙티브 컴포넌트를 하나의 React 트리에서 사용 가능

```tsx
// ! Next.js 13.4 공식버전에서는 별도의 'use client'가 없으면 RSC로 실행

// 이 컴포넌트는 *서버(또는 빌드 중)에서만* 실행됩니다.
async function Talks({ confId }) {
  // 1. 서버에 있으므로 데이터 계층과 통신가능 => API 엔드포인트가 필요하지 않습니다.
  const talks = await db.Talks.findAll({ confId });

  // 2. 자유로운 렌더링 로직 추가. => JavaScript 번들 크기가 커지지 않습니다.
  const videos = talks.map((talk) => talk.video);

  // 3. 브라우저(클라이언트)에서 실행될 컴포넌트에 데이터를 전달합니다.
  return <SearchableVideoList videos={videos} />;
}
```

- 데이터 통신과 Suspense 적용에도 Next.js의 App Router는 용이
  - 여기서 Suspense와 RSC는 Next.js의 기능이 아닌 순수 React 기능

```tsx
<Suspense fallback={<TalksLoading />}>
  <Talks confId={conf.id} />
</Suspense>
```

## 기존 프로젝트에 추가하기

- 기존 프로젝트에 추가하고 싶다고 React로 프로젝트를 재작성할 필요가 없다

1. 기존 웹 사이트의 전체 하위경로에 적용하기 (legacy.com/**react-app/\*)**
   - React 프레임워크 중 하나를 사용
   - 해당 프레임워크에서 기본 경로를 설정 (**[Next.js](https://nextjs.org/docs/pages/api-reference/next-config-js/basePath)**)
   - 서버 또는 프록시를 구성해 해당 아래의 모든 요청을 React 앱에서 처리하도록 설정

https://codesandbox.io/s/c6h3h7?file=/index.js&utm_medium=sandpack

1. 기존 페이지 일부에 React 사용하기
   - 다른 기술로 빌드된 기존페이지가 있고 일부 페이지에서만 React를 사용하고픈 상황
     1. JSX를 사용할 수 있게 모듈 JavaScript 환경 설정하기
     2. 특정 페이지에서 React 컴포넌트 렌더링
        https://codesandbox.io/s/blissful-alex-mhs3m4?file=/public/index.html

## 편집기

- [VS Code](https://code.visualstudio.com/)는 현재 가장 많이 사용되는 편집기 중 하나

  - 확장 기능의 대규모 마켓플레이스 존재
  - GitHub와 같은 인기 서비스와의 쉬운 연동

- [웹스톰](https://www.jetbrains.com/webstorm/)은 JavaScript를 위해 특별히 설계된 통합 개발 환경(IDE)
- [Sublime Text](https://www.sublimetext.com/)는 JSX와 TypeScript를 지원하며, [구문 강조](https://stackoverflow.com/a/70960574/458193) 및 자동완성 기능 빌트인
- [Vim](https://www.vim.org/)은 모든 종류의 텍스트를 매우 효율적으로 생성하고 변경할 수 있도록 고도로 구성 가능한 텍스트 편집기. (대부분의 UNIX 시스템과 Apple OS X에 “vi”로 존재)

- 추가 편집기 기능

  - Linting → [Eslint](https://www.npmjs.com/package/eslint-config-react-app), [SonarLint](https://www.sonarsource.com/products/sonarlint/?gads_campaign=SL-Class02-Brand&gads_ad_group=SonarLint&gads_keyword=sonarlint&gclid=Cj0KCQjwusunBhCYARIsAFBsUP-P42imHR74C3tHagbWraq3yxA929w4eUAapzgaNvhCu1szyrWTrXUaAjroEALw_wcB)
  - Formatting → [Prettier](https://marketplace.visualstudio.com/items?itemName=esbenp.prettier-vscode)
    <aside>
    💡 저장할 때마다 안쓰는 import 제거하는 vscode 설정 사용 (https://webruden.tistory.com/1069)

    </aside>

## 개발자 도구

- [React Devtools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi?hl=en)
  - components와 profiler 패널을 활용해 React 애플리케이션을 더 React스럽게 개발할 수 있습니다.

# React식 사고하기

---

<aside>
💡 최근 프론트엔드 라이브러리나 프레임워크를 보면 `Thinking in ***`를 심심치 않게 볼 수 있다. ([예시1](https://relay.dev/docs/principles-and-architecture/thinking-in-relay/), [예시2](https://tkdodo.eu/blog/thinking-in-react-query)) 
`thinking 블라블라`가 있다면 해당 부분을 자주 읽으면 좋다. 읽다보면 제작자의 의도를 파악해 기술 숙련도가 올라갈 것이다.

</aside>

- React는 (개발자가 제품의) 디자인을 바라보는 방식, 앱을 빌드하는 방식을 바꿔줄 수 있다.
  - UI를 **컴포넌트**라는 작은 조각으로 나눈다.
  - 각 컴포넌트들의 시각적 상태를 정의한다.
  - 분해된 컴포넌트들을 다시 연결해 데이터가 흐르도록 한다.

## 1. UI를 컴포넌트 계층으로 나누기

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/693bc352-6af5-47d2-866f-3108bb9e50f3/1b2bd9af-86d1-46f6-8173-59e4725278f5/Untitled.png)

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/693bc352-6af5-47d2-866f-3108bb9e50f3/5969c26c-9b60-41f0-9dd8-55a2d2fa6e13/Untitled.png)

- 디자인 시안이 있다면 모든 컴포넌트에 영역 표시, 이름 태킹
  - 만약 디자이너가 정해둔게 있다면 그 이름을 적용
- [단일 책임 원칙](https://ko.wikipedia.org/wiki/%EB%8B%A8%EC%9D%BC_%EC%B1%85%EC%9E%84_%EC%9B%90%EC%B9%99)을 반영하고자 한다면 컴포넌트는 한 번에 한가지 일만 하도록 설정
  - 컴포넌트 코드가 생각보다 비대하다면 리팩토링을 통해 작은 단위의 컴포넌트로 분해
- 당장은 컴포넌트로 관리할 필요가 없지만 기획이 변경됨에 따라 컴포넌트가 필요해 질 수 있다
  - ex) 위 예시에서 Name, Price는 `ProductTableHeader`로 분리 가능

## 2. React로 정적 UI 만들기

[eloquent-shockley-s9yrdz - CodeSandbox](https://codesandbox.io/s/s9yrdz?file=/App.js&utm_medium=sandpack)

- 상호작용 기능을 제외
- 단순 데이터 모델(Mocking)만으로 UI 렌더링하기

- Props를 이용해 데이터를 넘겨주기
  - 해당 파트에서는 state를 사용하지 말 것(정적 버전에서는 불필요)
- (⭐중요⭐) 앱을 만드는 방식 2가지
  - 상향식(Bottom-Up)
    - 일반 사용자뿐만 아니라 **개발자도 사용**할 수 있는 것을 고려해야한다
    - **사용될 컴포넌트(with 스타일) + 스토리**
    - 컴포넌트 사용할 사람이 개발자라 생각하면서 개발
      - ex) 아토믹 디자인
    - 숙련도가 부족하면 **[YAGNI](https://ko.wikipedia.org/wiki/YAGNI)**에 빠지기 쉽다
  - 하향식(Top-Down)
    - 중복되는 컴포넌트를 나중에 분리하려고 한다
    - 전체적인 디자인과 구조를 먼저 구성하고 이를 바탕으로 세부 개발
      - 보통 next.js가 하향식으로 개발하도록 유도함 → 역량 증진에 방해
    - 개발자가 아닌 **사용자**를 위해 개발하면서 분리
    - **르블랑의 법칙**
      - 한번 작성한 쓰레기 코드를 수정할 일이 없다
- 컴포넌트 계층구조에서 최상단 컴포넌트(FilterableProductTable) → 제일 아래 컴포넌트
  ⇒ **단방향 데이터 흐름**

## 3. UI 상태의 최소한의 완전한 표현 찾기

- 사용자가 데이터 모델을 변경 → 상호작용하는 UI ⇒ State를 사용
  - State 구조화 시 [중복배제원칙](https://ko.wikipedia.org/wiki/%EC%A4%91%EB%B3%B5%EB%B0%B0%EC%A0%9C)을 위배하지 않도록 주의

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/693bc352-6af5-47d2-866f-3108bb9e50f3/5969c26c-9b60-41f0-9dd8-55a2d2fa6e13/Untitled.png)

- 예시 애플리케이션에서는 4가지 데이터를 다룸
  1. 물품 원본 목록
  2. 사용자가 입력한 검색어
  3. 체크박스 값
  4. 필터링된 제품 목록

**State가 아닌 케이스**

- 시간이 지나도 변하지 않는다 → State X
- 부모로부터 props로 전달된다? → State X
- 다른 state나 props로 계산이 가능? → State X

### Props vs State

- Props: 함수를 통해 전달되는 인자
    <aside>
    💡 제일 외각에서 바라보면 객체이므로 props로 전달시 `{}` 유의할 것
    
    </aside>

- State: 컴포넌트의 메모리 같은 성격을 띔
    <aside>
    💡 State를 자식 컴포넌트에 전달하면 자식 컴포넌트에겐 props가 됨
    
    </aside>

## 4. state가 어디에 있어야 할 지 정하기

<aside>
💡 부모 → 자식으로 데이터를 전달하는 단방향 데이터 흐름이 전제

</aside>

- 어떤 컴포넌트에서 특정 State를 관리할 지를 잘 정해야 함

  1. 해당 state를 기반으로 렌더링하는 모든 컴포넌트 확인
  2. 해당 컴포넌트를 아우르는 부모(상위) 컴포넌트 탐색
     1. 만약 못찾았다면, 해당 state를 소유하는 상위 컴포넌트를 새롭게 추가

- useState()를 이용해 state를 컴포넌트에 추가

```tsx
// https://codesandbox.io/s/9sj55s?file=%2FApp.js&utm_medium=sandpack
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState("");
  const [inStockOnly, setInStockOnly] = useState(false);
  // ! 하지만 SearchBar에서 value를 수정하는 onChange handler가 없어서 오류가 발생
  return (
    <div>
      <SearchBar filterText={filterText} inStockOnly={inStockOnly} />
      <ProductTable
        products={products}
        filterText={filterText}
        inStockOnly={inStockOnly}
      />
    </div>
  );
}
```

## 5. 역방향으로 데이터 흐름 추가하기

- 단방향 데이터 흐름이지만 state를 변경하기 위해선 반대 방향의 데이터 흐름을 만들어야 함
  - 아쉽게도 기존 양방향 데이터 바인딩보단 작업량이 있다.
  - state를 변경해주는 함수를 자식 컴포넌트에게 props로 전달

```tsx
function FilterableProductTable({ products }) {
  const [filterText, setFilterText] = useState('');
  const [inStockOnly, setInStockOnly] = useState(false);

  return (
    <div>
      <SearchBar
        filterText={filterText}
        inStockOnly={inStockOnly}
        onFilterTextChange={setFilterText}
        onInStockOnlyChange={setInStockOnly} />
...
```

[mystifying-glitter-hqrx35 - CodeSandbox](https://codesandbox.io/s/hqrx35?file=/App.js&utm_medium=sandpack)

# 자습서: Tic-Tac-Toc

---

<aside>
💡 앞서 학습한 Thinking in React를 기준으로 Tic-Tac-Toc을 개발해 보았습니다.
예제코드를 기반으로 논의하고 중간중간 중요한 개념만 정리했습니다.

</aside>

![image-1.jpg](https://prod-files-secure.s3.us-west-2.amazonaws.com/693bc352-6af5-47d2-866f-3108bb9e50f3/c1bf02b5-a3f8-4dce-b270-61e7c72dbc46/image-1.jpg)

https://stackblitz.com/edit/stackblitz-starters-xgptov?file=src/CustomApp.js

<aside>
💡 Open in new tab을 사용하면 local에서 개발하듯이 사이트를 띄울 수 있다. ⇒ React devtool 사용 가능

</aside>

### React.Fragment(<></> 혹은 <React.Fragment></React.Fragment>)

- React 컴포넌트는 두 개의 버튼처럼 인접한 여러 개의 JSX 엘리먼트가 아닌 **단일 JSX 엘리먼트**를 반환해야함.

  - _fragments_(`<>` 와 `</>`)로 감싸주기
    <aside>
    💡 <React.Fragment >{…}</React.Fragment>가 필요한 케이스도 있다(ex. key 부여)

    </aside>

### State, Props

- 여러 자식 컴포넌트에서 데이터를 수집 or 두 자식 컴포넌트가 서로 통신하길 희망
  → **부모 컴포넌트에서 공유하는 state**
- 부모 컴포넌트는 props drilling을 통해 해당 state를 자식 컴포넌트에 전달
  → **자식 컴포넌트 서로 동기화 + 부모 컴포넌트와도 동기화**

### Callback함수에 함수가 아닌 값을 넣는 실수

```tsx
<Square value={squares[0]} onSquareClick={handleClick(0)} />
```

- `onSquareClick={handleClick}` 전달할 땐 함수 *호출*이 아니라 `handleClick` 함수를 prop로 전달
- 지금은 `handleClick(0)` === 함수 호출
  → 해당 함수가 너무 일찍 실행(사용자가 클릭하기도 전에 `handleClick` 동작)
- 해결책

  1.  `handleClick(n)`을 호출하는 `handle{n번째}SquareClick` 함수 제작
  2.  `onSquareClick={handleFirstSquareClick}`와 같이 prop로 함수 전달

      1. 단, 9개의 서로 다른 함수를 정의하고 각각에 이름을 붙이는 것은 너무 장황합니다.

      - 참고
        ```tsx
        <Square value={squares[0]} onSquareClick={() => handleClick(0)} />
        ```

      <aside>
      💡 JSX안에 Arrow function이 들어가는건 지향해야 함. 개선필요 (https://stackoverflow.com/questions/36677733/why-shouldnt-jsx-props-use-arrow-functions-or-bind)

      </aside>

### 불변성

```tsx
function handlePlay(nextSquares) {
  // 불변성이 중요 ->
  const nextHistory = [...history.slice(0, currentMove + 1), nextSquares];
  setHistory(nextHistory);
  setCurrentMove(nextHistory.length - 1);
}
```

- `handleClick`에서 기존 배열을 수정하는 대신 `.slice()`를 호출하여 `squares` 사본을 생성

일반적으로 데이터를 변경하는 방법, 2가지

1. 데이터의 값을 직접 변경 → 데이터 _변형_

```tsx
const squares = [null, null, null, null, null, null, null, null, null];
squares[0] = "X"; // Now `squares` is ["X", null, null, null, null, null, null, null, null];
```

2. 원하는 변경 사항이 있는 새 복사본으로 데이터 대체

```tsx
const squares = [null, null, null, null, null, null, null, null, null];
const nextSquares = ["X", null, null, null, null, null, null, null, null];
// Now `squares` is unchanged, but `nextSquares` first element is 'X' rather than `null`
```

최종 결과는 같지만, 불변성 사용시

1. **복잡한 기능을 훨씬 쉽게 구현**(Tic-Tac-Toc에서 특정 단계로 이동)
2. **불필요한 리렌더링 방지** → 성능 저하 방지
   - `[memo` API 레퍼런스](https://ko.react.dev/reference/react/memo)에서 React가 컴포넌트를 다시 렌더링할 시점을 선택
     (기본적으로 부모 컴포넌트 state 변경 → 모든 자식 컴포넌트 리렌더링)

### React에서 배열 사용시 고유한 key 사용

- 배열 렌더링시, 의도하지 않는 결과 or 비효율적인 동작 방지.
  - 하지만, 변경이 일어나지 않는 경우 인덱스 사용가능.

# 추가 자료

---

- React Server Component
  - https://ui.toast.com/weekly-pick/ko_20210119
- Suspense
  - 데이터 통신 과정에서 로딩 상태를 관리하기 위해 탄생한 매커니즘
  - https://www.daleseo.com/react-suspense/
- Next.js
  - Code spliting: https://nextjs.org/learn/foundations/how-nextjs-works/code-splitting
  - Bundling: https://nextjs.org/learn/foundations/how-nextjs-works/bundling
  - Rendering: https://nextjs.org/learn/foundations/how-nextjs-works/rendering
- Closure
  - https://developer.mozilla.org/ko/docs/Web/JavaScript/Closures
- Typescript 관련
  - https://rules.sonarsource.com/typescript/RSPEC-6480/
- React의 key
  - https://tecoble.techcourse.co.kr/post/2021-04-25-react-key/
- Fiber, Reconciliation
  - [https://velog.io/@syoung125/eact-Reconciliation이란-virtual-DOM-리액트가-선언적](https://velog.io/@syoung125/eact-Reconciliation%EC%9D%B4%EB%9E%80-virtual-DOM-%EB%A6%AC%EC%95%A1%ED%8A%B8%EA%B0%80-%EC%84%A0%EC%96%B8%EC%A0%81)
  - [https://velog.io/@yeonbot/React에서-key의-역할-컴포넌트를-다시그리는-과정](https://velog.io/@yeonbot/React%EC%97%90%EC%84%9C-key%EC%9D%98-%EC%97%AD%ED%95%A0-%EC%BB%B4%ED%8F%AC%EB%84%8C%ED%8A%B8%EB%A5%BC-%EB%8B%A4%EC%8B%9C%EA%B7%B8%EB%A6%AC%EB%8A%94-%EA%B3%BC%EC%A0%95)
