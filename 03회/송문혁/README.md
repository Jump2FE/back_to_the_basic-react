> [3회] UI 표현하기 - 2, 상호작용성 더하기 - 1

> 링크: https://bronzed-icicle-65c.notion.site/3-UI-2-1-d24b7fbf98454b09a550267b67c64a56?pvs=4

## 조건부 렌더링

- 컴포넌트는 조건에 따라 다른 것을 보여줘야 하는 경우가 많음.
- React에서는 JavaScript 문법을 사용해 조건부로 JSX를 렌더링할 수 있습니다.( `if`, `&&`, `? :` )

### 조건부로 JSX 반환하기

- `isPacked` 라는 boolean형 props로 Item 컴포넌트에서 반환하는 JSX를 구분
- 예시

  ```tsx
  function Item({ name, isPacked }) {
    if (isPacked) {
      return <li className="item">{name} ✔</li>;
    }
    return <li className="item">{name}</li>;
  }

  export default function PackingList() {
    return (
      <section>
        <h1>Sally Ride's Packing List</h1>
        <ul>
          <Item isPacked={true} name="Space suit" />
          <Item isPacked={true} name="Helmet with a golden leaf" />
          <Item isPacked={false} name="Photo of Tam" />
        </ul>
      </section>
    );
  }
  ```

### 조건부로 null 사용해서 아무것도 반환하지 않기

- 특정 상황에 따라 화면에 어떠한 것도 렌더링하고 싶지 않다 ⇒ `null`을 반환
- 예시 (packing한 item이면 화면에 보여주지 않기)

  ```tsx
  function Item({ name, isPacked }) {
    if (isPacked) {
      return null;
    }
    return <li className="item">{name}</li>;
  }

  export default function PackingList() {
    return (
      <section>
        <h1>Sally Ride's Packing List</h1>
        <ul>
          <Item isPacked={true} name="Space suit" />
          <Item isPacked={true} name="Helmet with a golden leaf" />
          <Item isPacked={false} name="Photo of Tam" />
        </ul>
      </section>
    );
  }
  ```

- 실제로 컴포넌트에서 `null`을 직접적으로 반환하는 것은 지양(💡)
  - 개발자가 렌더링하려고 할 때 놀랄 수 있기 때문
- 부모 컴포넌트 JSX에 컴포넌트를 조건부로 포함하거나 제외할 수 있습니다

### 조건부로 JSX 포함시키기

```tsx
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

- `<li className="item">...</li>;`
  - 중복코드 부분이 나쁘진 않으나 **유지보수에선 안좋음**
  - ex) className을 변경하려 할 때, 여러 곳을 찾아다녀야 함
- **DRY(**[https://ko.wikipedia.org/wiki/중복배제](https://ko.wikipedia.org/wiki/%EC%A4%91%EB%B3%B5%EB%B0%B0%EC%A0%9C))하게 만들 수 있다.

### 삼항 조건 연산자

- Javscript의 `?` 를 컴포넌트에서 사용하기
- 예시 코드

  ```tsx
  // before
  if (isPacked) {
    return <li className="item">{name} ✔</li>;
  }
  return <li className="item">{name}</li>;

  // after (DRY 코드 작성 성공)
  return <li className="item">{isPacked ? name + " ✔" : name}</li>;
  ```

- 두 예제는 완전 동일한가? (💡위 예시에서 before, after를 의미)
  - 객체 지향 프로그래밍에 익숙하면**, <li>는 서로 다른 인스턴스(instance)를 생성**하기에 다를 수 있다.
  - But, JSX 요소는 **내부 state를 보유하지 않고 실제 DOM node가 아니기**에 인스턴스가 아님
      <aside>
      ❓ 인스턴스의 정의: **직접 관리하는 stete가 있어야 하는 조건**만 맞추면 되는가?
      
      </aside>
      
      - 자세한 동작 방식은 이후 다루는 **[state 보존 & 재설정](https://ko.react.dev/learn/preserving-and-resetting-state)** 참고
- 조건 반환에 더 많은 JSX를 중첩할 수 있다.

  - 예시
    ```tsx
    function Item({ name, isPacked }) {
      return (
        <li className="item">{isPacked ? <del>{name + " ✔"}</del> : name}</li>
      );
    }
    ```
  - But, **렌더링 파트에 조건문 사용은 적당한 게 좋다 ⇒ 컴포넌트 반환부가 너무 길어짐**
  - 가독성이 떨어지기 시작한다면 **컴포넌트 추출(리팩토링)을 진행**하자.

    - 리팩토링 예시

      ```tsx
      function Item({ name, isPacked }) {
        return (
          <li className="item">
            <ItemInternal name={name} isPacked={isPacked} />
          </li>
        );
      }

      function ItemInternal({ name, isPacked }) {
        return <>{isPacked ? <del>{name + " ✔"}</del> : name}</>;
      }
      ```

### 논리 연산자 AND(&&)

- Javascript의 `&&`를 활용해 간단한 조건 렌더링이 가능

  - 보통 조건이 false일 때 아무것도 렌더링 하지 않을 때 많이 사용

  ```tsx
  return (
    <li className="item">
      {name} {isPacked && "✔"} // true일 때, 체크 표시가 화면에 뜸
    </li>
  );
  ```

- `&&` 왼쪽에 숫자를 두지 마세요
  - Javscript 조건식은 자동으로 조건 연산자 좌항을 boolean으로 변환하려 한다.
  - But, 좌항이 `0` 이면 전체 식이 `0`으로 판단해 화면에 `0`을 렌더링해버린다
    - `messageCount && <p>New messages</p>` (**[codesandbox](https://codesandbox.io/s/serene-rgb-x96fgs?file=/App.js:0-909)**)
    - 메세지 수가 0이면 화면에 아무것도 안 그려져야 하지만 `0`이 그려져있는 걸 볼 수 있다.

### 변수에 조건부로 JSX를 할당하기

- JSX반환에서 조건부 처리를 하는 게 아닌 **컴포넌트 내 로직을 작성할 수 있는 부분에서 처리하기**

  - `let`을 활용해 JSX 표현식을 재할당 하기
  - 예시

    ```tsx
    function Item({ name, isPacked }) {
      let itemContent = name; // let 활용
      if (isPacked) {
        itemContent = name + " ✔"; // text
      }
      return (
        <li className="item">
          {itemContent}
        </li>
      );
    }

    function Item2({ name, isPacked }) {
      let itemContent = name;
      if (isPacked) {
        itemContent = (
          <del>
            {name + " ✔"} {/* JSX 부분도 할당 가능 */}
          </del>
        );
      }

    export default function PackingList() {
      return (
        <section>
          <h1>Sally Ride's Packing List</h1>
          <ul>
            <Item
              isPacked={true}
              name="Space suit"
            />
            <Item2
              isPacked={true}
              name="Helmet with a golden leaf"
            />
            <Item
              isPacked={false}
              name="Photo of Tam"
            />
          </ul>
        </section>
      );
    }
    ```

    <aside>
    ❓ Item2 같이 let에 JSX를 넣어주는 패턴을 많이들 사용하는지 궁금합니다

    </aside>

### 요약

- React에서 JavaScript로 분기 로젝 제어 가능
- `if` 문과 함께 JSX 식을 반환 가능
  ```tsx
  function Test({ name }) {
    if (name === "송문혁") return <>안녕하세요, 송문혁입니다.</>;
    return <>안녕하세요, {name}입니다.</>;
  }
  ```
- 조건부로 일부 JSX를 변수에 저장한 다음 중괄호를 사용하여 다른 JSX에 포함할 수 있습니다.
  ```tsx
  function Test2({ name }) {
    let greeting;
    if (name === "송문혁") greeting = "안녕하세요, 송문혁입니다";
    else greeting = `안녕하세요, ${name}`;
    return <>{greeting}</>;
  }
  ```
- JSX에서 삼항연사자를 사용한 `{cond ? <A /> : <B />}`는?
  - ‘*`cond`이 **true**면  `<A />`를 렌더링하고, **false**면 `<B />`를 렌더링합니다.’* 를 의미.
- JSX에서 논리 연산자 AND를 사용한 `{cond && <A />}`는?

  - *“`cond`이면, `<A />`를 렌더링하되, 그렇지 않으면 아무것도 렌더링하지 않습니다.”* 를 의미
  - **주의) 논리 연산자를 사용할 때, 왼쪽 영역에는 number가 오는 것에 주의!** (`0`이 출력된다)

    ```tsx
    // solution1: cond가 숫자인 경우 범위 조건 추가
    {
      cond > 0 && <A />;
    }

    // solution2: !!(double exclamation mark) 적용해서 boolean으로 형변환
    {
      !!cond && <A />;
    }
    ```

- 위 예시는 흔한 방법이지만, `if`를 선호한다면 사용하지 않아도 됩니다.

---

## 리스트 렌더링

- 데이터 모음에서 유사한 컴포넌트를 여러 개(💡**연속해서)** 표시해야할 때가 있음.
- [JavaScript 배열 메서드](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array#)를 사용하여 데이터 배열 조작 가능
- `[**filter()**](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)`와 `[**map()**](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array/map)`을 사용해 **데이터 배열을 필터링, 컴포넌트 배열로 변환**

### 배열을 데이터로 렌더링하기(`map` 활용)

- 내용인 데이터는 각각 다르지만 이를 표현하는 형태(UI)가 동일할 때 ⇒ `map` / `filter` 활용
- 예시 데이터

  ```tsx
  <ul>
    <li>Creola Katherine Johnson: mathematician</li>
    <li>Mario José Molina-Pasquel Henríquez: chemist</li>
    <li>Mohammad Abdus Salam: physicist</li>
    <li>Percy Lavon Julian: chemist</li>
    <li>Subrahmanyan Chandrasekhar: astrophysicist</li>
  </ul>
  ```

- 1. 데이터를 배열로 이동
  ```tsx
  const people = [
    "Creola Katherine Johnson: mathematician",
    "Mario José Molina-Pasquel Henríquez: chemist",
    "Mohammad Abdus Salam: physicist",
    "Percy Lavon Julian: chemist",
    "Subrahmanyan Chandrasekhar: astrophysicist",
  ];
  ```
- 2. `peopel` 멤버를 JSX노드 배열인 listItems에 Mapping
  ```tsx
  const listItems = people.map((person) => <li>{person}</li>);
  ```
- 3. <ul>로 감싼 listItems을 반환
  ```tsx
  return <ul>{listItems}</ul>;
  ```
- 결과 코드

  ```tsx
  const people = [
    'Creola Katherine Johnson: mathematician',
    'Mario José Molina-Pasquel Henríquez: chemist',
    'Mohammad Abdus Salam: physicist',
    'Percy Lavon Julian: chemist',
    'Subrahmanyan Chandrasekhar: astrophysicist'
  ];

  export default function List() {
    const listItems = people.map(person =>
      <li>{person}</li> **// 여기서 key prop관련된 에러가 발생! (💡)**
    );
    return <ul>{listItems}</ul>;
  }
  ```

### 배열의 항목들을 필터링하기 (`filter` 활용)

- 예시 데이터

  ```tsx
  const people = [
    {
      id: 0,
      name: "Creola Katherine Johnson",
      profession: "mathematician",
    },
    {
      id: 1,
      name: "Mario José Molina-Pasquel Henríquez",
      profession: "chemist",
    },
    {
      id: 2,
      name: "Mohammad Abdus Salam",
      profession: "physicist",
    },
    {
      name: "Percy Lavon Julian",
      profession: "chemist",
    },
    {
      name: "Subrahmanyan Chandrasekhar",
      profession: "astrophysicist",
    },
  ];
  ```

- 1. `filter`를 사용해 원하는 `person.profession` 를 분리해서 **새로운 배열을 생성**
  ```tsx
  const chemists = people.filter((person) => person.profession === "chemist");
  ```
- 2. 이를 `map` 을 사용해 Mapping
  ```tsx
  const listItems = **chemists**.map(person =>
    <li>
       <img
         src={getImageUrl(person)}
         alt={person.name}
       />
       <p>
         <b>{person.name}:</b>
         {' ' + person.profession + ' '}
         known for {person.accomplishment}
       </p>
    </li>
  );
  ```
- 3. <ul>로 감싼 listItem을 반환
  ```tsx
  return <ul>{listItems}</ul>;
  ```
- 결과 코드 ([공식 사이트 참고](https://codesandbox.io/s/8tkjdq?file=%2FApp.js&utm_medium=sandpack))

### 주의

- 화살표 함수 뒤에 표현식을 암시적으로 반환(`return` 필요없음)
- But, `{}` 를 사용할 경우 `return`을 명시 작성
- 예시

  ```tsx
  // return 필요 없는 경우
  const listItems = chemists.map(
    (person) => <li>...</li> // Implicit return!
  );

  // return 필요한 경우
  const listItems = chemists.map((person) => {
    // Curly brace
    return <li>...</li>;
  });
  ```

### key를 사용해서 리스트 항목 순서 유지하기

- `map` 같은 **순회 메소드**에서 배열의 항목들을 표현할 땐, 고유 key가 필요
  - `<li key={person.id}>...</li>`
- `key`는 **각 컴포넌트가 어떤 배열 항목에 해당**하는지 React에 알려주어 나중에 매칭할 수 있도록 합니다.
  - 이는 **배열 항목이 (정렬 등으로 인해) 이동 / 삽입 / 삭제**될 수 있는 경우 중요해진다.
- 잘 만들어진 `key`는 **React가 정확히 무슨 일이 일어났는지 추론**하고 **DOM 트리를 제대로 업데이트하는 데 도움**이 된다.

- 각 리스트 항목에 대해 여러 DOM Node 표시하기

  - 여러개의 DOM Node를 렌더링할 때는 **<React.Fragment>** 나 **div**를 Wrapping용으로 활용
  - Fragment인 경우 DOM에서 사라져서 여러 이점이 있다. ([**김희현님 자료 참고**](https://www.notion.so/e9cd0420a0cd4648aa10e96076a37c44?pvs=21))

  ```tsx
  import { Fragment } from "react";

  // ...

  const listItems = people.map((person) => (
    <Fragment key={person.id}>
      <h1>{person.name}</h1>
      <p>{person.bio}</p>
    </Fragment>
  ));

  // 혹은
  const listItems = people.map((person) => (
    <div key={person.id}>
      <h1>{person.name}</h1>
      <p>{person.bio}</p>
    </div>
  ));
  ```

### Key를 얻을 수 있는 곳

- **데이터베이스의 데이터**
  - 고유한 데이터베이스 key/ID를 사용
- **로컬에서 생성된 데이터** (예: 메모 작성 앱의 메모)
  - 항목을 만들 때 증분 카운터, `[crypto.randomUUID()](https://developer.mozilla.org/en-US/docs/Web/API/Crypto/randomUUID)` 또는 `[uuid](https://www.npmjs.com/package/uuid)`와 같은 패키지를 사용

<aside>
💡 최근에는 **서버에서 생성해서 보내주는 id**나 **uuid 패키지**를 많이 사용하는 것 같다.

</aside>

### key 규칙

- **key는 형제간에 고유해야 합니다.**
  - *다른* 배열의 JSX 노드에는 동일한 key를 사용 가능!
- **key가 변경되지 않아야 합니다.**
  - **렌더링 중에는 생성하면 안됨 (**❓)

### React에서 Key가 필요한 이유! (⭐중요⭐)

- [참고](https://tecoble.techcourse.co.kr/post/2021-04-25-react-key/)
- 데스크톱의 파일에 이름이 없다고 상상
  - 파일 이름 대신 **첫 번째 파일, 두 번째 파일** 등의 순서로 파일을 참조
  - 첫 번째 파일을 삭제하면 **두 번째 파일이 첫 번째 파일**, **세 번째 파일이 두 번째 파일**이 되는 혼란 발생
- **폴더의 파일 이름**과 **배열의 JSX key**는 비슷한 역할을 한다.
  - key 사용시, 형제 항목 사이에서 특정 항목을 고유 식별 가능
  - **잘 선택한 key**는 **배열 내 위치보다 더 많은 정보(그 항목 자체 식별)를 제공**
    - 만약 재 정렬로 인해 어떤 항목의 *위치*가 변경되더라도, React는 `key`를 통해 그 항목을 식별할 수 있다.

### 주의

- 배열에서 항목의 인덱스(`map`의 두번째 인자)를 key로 사용하고 싶을 때가 많다.
  - 기본적으로 `key`를 지정하지 않으면, React는 인덱스를 key로 사용.
  - But, 여러분이 렌더링한 항목의 순서는 새 항목이 삽입되거나, 삭제되거나, 배열의 순서가 바뀌는 등에 따라 변경 가능함.
  - 인덱스를 key로 사용하면 **종종 미묘하고 혼란스러운 버그**가 발생합니다.
    - 💡참고
      1. **성능 문제**: 만약 리스트의 항목이 추가, 제거, 또는 재배열되면 index를 key로 사용한 경우 React가 컴포넌트를 업데이트하는 데 어려움이 생길 수 있습니다. 새 항목이 추가되면 모든 key가 변경되고, React는 모든 항목을 다시 렌더링해야 할 수 있습니다.
      2. **고유성 문제**: index는 항목의 고유성을 나타내지 않습니다. 두 항목이 동일한 값으로 렌더링되고 나중에 하나의 항목이 제거된다면 React는 어떤 항목을 제거해야 할지 판단하기 어려울 수 있습니다.
- 마찬가지로 `key={Math.random()}`과 같이 즉석에서 key를 생성하지 말 것.
  - 렌더링될 때마다 key가 일치하지 않아 **매번 모든 컴포넌트와 DOM이 다시 생성**
  - **속도가 느려**질 뿐만 아니라 **목록 항목 내부의 사용자 입력도 손실**
    - 대신 데이터에 기반한 안정적인 ID를 사용(uuid 패키지나 서버에서 고유 key제공)
- **컴포넌트는 `key`를 prop으로 받지 않음**
  - React 자체에서 힌트로만 사용
  - 컴포넌트에 ID가 필요한 경우 별도의 프로퍼티로 전달해
    - `<Profile key={id} **userId={id}** />`

### 요약

- 컴포넌트에서 배열이나 객체와 같은 **데이터 구조로 데이터를 이동하기**
- JavaScript의 `map()`을 사용하여 **유사한 컴포넌트 집합을 생성하기**
- JavaScript의 `filter()`를 사용하여 **필터링된 항목의 배열을 생성하기**
- 컬렉션에서 각 컴포넌트에 `key`를 설정하기
  - 위치나 데이터가 변경되더라도 React가 각 컴포넌트를 추적할 수 있음

---

## 컴포넌트를 순수하게 유지

- 일부 JavaScript 함수는 _순수_
  - 순수 함수는 **계산만 수행**하고 **그 이상은 수행하지 않는 함수**.
- 컴포넌트를 엄격하게 순수 함수로만 작성하기
  - 코드베이스가 커짐에 따라 당황스러운 버그와 예측할 수 없는 동작을 미연에 방지
  - But, 몇 가지 규칙을 준수해야 한다.

### 순수성: 공식으로서의 컴포넌트

- 순수 함수의 특징
  - **자신의 일에만 신경쓴다.**
    - 호출되기 전에 존재했던 객체나 변수를 변경하지 않음
  - **동일 입력? 동일 출력!**
    - 동일한 입력이 주어지면 항상 동일한 결과를 반환해야 합니다.
  - 예시
    ```tsx
    function double(number) {
      return 2 * number;
    }
    ```

> React는 순수 함수의 특징을 중심으로 설계되었다.
> 고로, **React의 모든 컴포넌트가 순수 함수라고 가정해야 함.
> ⇒** 작성한 React 컴포넌트는 동일한 입력이 주어졌을 때, 항상 동일한 JSX를 반환해야 한다.

!https://react-ko.dev/images/docs/illustrations/i_puritea-recipe.png

> 컴포넌트 === 레시피라고 생각.
> 레시피를 따르고 요리 과정에서 새로운 재료를 넣지 않으면 매번 같은 요리를 얻을 수 있음
> ⇒ 그 “**요리**”는 컴포넌트가 [렌더링](https://react-ko.dev/learn/render-and-commit)에 반응하기 위해 제공하는 **JSX**.

### 사이드 이펙트: 의도치 않은 결과

- React의 렌더링 프로세스는 항상 순수해야 한다.
- **컴포넌트는 오직 JSX만을 반환**, 렌더링 전에 존재했던 **객체나 변수를 *변경*해서는 안 된다**.
    <aside>
    💡 그래서 const를 많이 사용하는 거라 생각. (불변성도 유지)
    
    </aside>

- 예시

  ```tsx
  // 나쁜 예시
  let guest = 0;

  function Cup() {
    // 나쁨: 기존 변수(let guest)를 변경합니다!
    guest = guest + 1;
    return <h2>Tea cup for guest #{guest}</h2>;
  }

  export function TeaSet() {
    return (
      <>
        <Cup />
        <Cup />
        <Cup />
      </>
    );
  }

  // 개선
  function Cup2({ guest }) {
    return <h2>Tea cup for guest #{guest}</h2>;
  }

  export function TeaSet2() {
    return (
      <>
        <Cup guest={1} />
        <Cup guest={2} />
        <Cup guest={3} />
      </>
    );
  }
  ```

  - 이 컴포넌트는 외부에서 선언된 `guest` 변수를 읽고 쓰고 있습니다. 즉,**이 컴포넌트는 호출할 때마다 다른 JSX가 생성된다는 뜻입니다!** 게다가 다른 컴포넌트가 `guest`를 읽으면 렌더링된 시점에 따라 JSX도 다르게 생성됩니다! 예측할 수 없는 일입니다.
  - `[guest`를 prop으로 전달](https://ko.react.dev/learn/passing-props-to-a-component)함으로써 이 컴포넌트를 고칠 수 있습니다:

- 일반적으로 **컴포넌트가 특정 순서로 렌더링될 것이라고 기대해서는 안 됨**

  - 컴포넌트 각각이 독립적으로 렌더링된다고 생각!

- 엄격 모드(**React.StricMode**)로 순수하지 않은 연산을 감지

  - 아직 다 활용하지 않았을 수도 있지만 React에는 렌더링하면서 읽을 수 있는 세 가지 종류의 입력 요소가 있습니다. [props](https://ko.react.dev/learn/passing-props-to-a-component), [state](https://ko.react.dev/learn/state-a-components-memory), 그리고 [context](https://ko.react.dev/learn/passing-data-deeply-with-context). 이러한 입력 요소는 항상 읽기전용으로 취급해야 합니다.
  - 사용자의 입력에 따라 무언가를 _변경_ 하려는 경우, 변수를 직접 수정하는 대신 [set state](https://ko.react.dev/learn/state-a-components-memory)를 활용해야 합니다. 컴포넌트가 렌더링되는 동안엔 기존 변수나 개체를 변경하면 안됩니다.
  - React는 개발 중에 각 컴포넌트의 함수를 두 번 호출하는 “엄격 모드”를 제공합니다. **컴포넌트 함수를 두 번 호출함으로써, 엄격 모드는 이러한 규칙을 위반하는 컴포넌트를 찾는데 도움을 줍니다.**
  - 원래 예시에서 “Guest #1”, “Guest #2”, 그리고 “Guest #3” 대신 “Guest #2”, “Guest #4”, 그리고 “Guest #6”이 어떻게 표시되었는지 확인해보세요. 원래 함수는 순수하지 않았기에 두 번 호출하는 것이 이 부분을 망가트렸습니다. 그러나 고정된 순수 버전은 함수가 매번 두 번 호출되더라도 동작합니다. **순수 함수는 연산만 하므로 두 번 호출해도 아무 것도 변경되지 않습니다.**—`double(2)`을 두번 호출하는게 반환된 것을 변경하지 않고 y = 2x을 두 번 푸는게 y의 답을 바꾸지 않는 것 처럼, 항상 같은 입력이면 같은 출력입니다.
  - 엄격 모드는 프로덕션에 영향을 주지 않기 때문에 사용자의 앱 속도가 느려지지 않습니다. 엄격 모드를 사용하기 위해서, 최상단 컴포넌트를 `<React.StrictMode>`로 감쌀 수 있습니다. 몇몇 프레임워크는 기본적으로 이 문법을 사용합니다.
    <aside>
    💡 번외) `npx create-next-app@latest` 로 설치시
    Next.js에서 strictMode에 대한 configuration이 빠지면서 문제가 발생. ([참고](https://nextjs.org/docs/pages/api-reference/next-config-js/reactStrictMode), [참고2](https://github.com/vercel/next.js/blob/a4094e28a19b14153fab087aa8602e97d32d0aed/packages/next/src/telemetry/events/version.ts#L118C4-L118C52))

    ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/693bc352-6af5-47d2-866f-3108bb9e50f3/f51c599e-a54f-4c5b-b16d-dfe9f77ba7fd/Untitled.png)

    </aside>

### 지역 변형: 컴포넌트의 작은 비밀

- 위의 예시에선 컴포넌트가 렌더링하는 동안 **_기존_ 변수를 변경하는 것이 문제**
  - 이를 좀 더 무섭게 들리게 하기 위해 **“변이”(mutation)**라고 부르기도 함.\
  - **순수 함수는 함수의 범위를 벗어난 변수나 호출 전에 생성된 객체를 변이하지 않는다.**
- 하지만 **렌더링하는 동안 ‘방금’ 생성한 변수와 객체를 변경하는 것은 Fine!**

  - 예시

    ```tsx
    function Cup({ guest }) {
      return <h2>Tea cup for guest #{guest}</h2>;
    }

    export default function TeaGathering() {
      let cups = [];
      for (let i = 1; i <= 12; i++) {
        cups.push(<Cup key={i} guest={i} />);
      }
      return cups;
    }
    ```

  - 만약 `cups` 변수나 `[]` 배열이 `TeaGathering`의 바깥에서 생성되었다면 큰 문제
    - 항목을 해당 배열에 푸시(push)하여 _기존_ 객체를 변경할 수 있다.
  - But, `TeaGathering` 내부에서 동일한 렌더링 중에 생성했기 때문에 괜찮다.

    - `TeaGathering` 외부의 어떤 코드도 이런 일이 일어났다는 것을 알 수 없다.
    - 이를 **“지역 변이”(local mutation)**라고 하며, 컴포넌트의 작은 비밀과 같습니다.
      <aside>
      💡 변이가 완료되고 나서 반환을 해주기 때문에 변이가 일어났어도 문제없다고 이해함.

      </aside>

### 사이드 이펙트를 일으킬 수 있는 곳

- **함수형 프로그래밍**은 **순수성에 크게 의존**하지만, **언젠가는, 어딘가에서, _무언가가_ 바뀌어야 함**.

  - 이러한 변화들을 **사이드 이펙트**라고 한다.

    - 렌더링 중에 발생하는 것이 아니라 _“사이드에서,”_ 발생하는 현상입니다.
        <aside>
        💡 렌더링 중에 변이를 일으키는 케이스는 어떤 상황?
        
        </aside>

    - ex. 화면 업데이트, 애니메이션 시작, 데이터 변경

- 리액트에선 **사이드 이펙트는 보통 [이벤트 핸들러](https://ko.react.dev/learn/responding-to-events)에 포함**
  - 이벤트 핸들러는 리액트가 일부 작업을 수행할 때 반응하는 기능(ex. 버튼을 클릭)
  - **이벤트 핸들러**가 컴포넌트 _내부에_ 정의되었다 하더라도 **렌더링 _중에는_ 실행 안 한다**!
    ⇒ **그래서 이벤트 핸들러는 순수할 필요가 없다.**
- 다른 옵션을 모두 사용했지만 사이드 이펙트에 적합한 이벤트 핸들러를 찾을 수 없는 경우에도, 컴포넌트에서 `[useEffect](https://ko.react.dev/reference/react/useEffect)` 호출을 사용하여 반환된 JSX에 해당 이벤트 핸들러를 연결할 수 있습니다.
  - 이것은 리액트에게 사이드 이펙트가 허용될 때 렌더링 후 나중에 실행하도록 지시합니다.
  - **그러나 이 접근 방식이 마지막 수단이 되어야 합니다. (**❓**)**
- 가능하면 **렌더링만으로 로직을 표현**하자.

### **리액트는 왜 순수함을 중요시할까? (중요)**

순수 함수를 작성하려면 약간의 습관과 훈련이 필요. But, 그것은 놀라운 기회를 열어준다다:

- **컴포넌트를 다른 환경(예: 서버)에서 실행할 수 있다**!
  동일한 입력에 대해 동일한 결과를 반환하기 때문에 하나의 컴포넌트가 많은 사용자 요청을 처리할 수 있다.
- **입력이 변경되지 않은 컴포넌트는 [렌더링 건너뛰기](https://ko.react.dev/reference/react/memo)를 통해 성능을 향상시킬 수 있다.**
  순수 함수는 항상 동일한 결과를 반환하므로 **캐싱**해도 안전합니다.
- **깊은 컴포넌트 트리를 렌더링하는 도중**에 일부 데이터가 변경되면 React는 오래된 렌더링을 완료하기 위해 시간을 낭비하지 않고 렌더링을 다시 시작할 수 있습니다.
  **순수성 덕분에 언제든지 계산을 중단해도 안전**하다.

우리가 구축하는 모든 새로운 React 기능은 순수성의 이점을 활용한다.

데이터 불러오기부터 애니메이션, 성능에 이르기까지, 컴포넌트를 순수하게 유지하면 React 패러다임의 힘을 발휘할 수 있습니다. (리액트를 리액트스럽게 쓰게 됨)

### 요약

- 컴포넌트는 **순수**해야 한다.
  - 컴포넌트는 **자신의 일만에 집중!**
    - 렌더링 전에 존재했던 객체나 변수를 변경하지 않아야 합니다.
  - **같은 입력 ⇒ 같은 결과물이 나와야한다.**
    - **입력이 같을 경우**, 컴포넌트는 **항상 같은 JSX를 반환**해야 합니다.
- 렌더링은 언제든지 발생 가능! 컴포넌트는 **서로의 렌더링 순서에 의존하지 않아야 한다**
- 컴포넌트가 렌더링을 위해 사용되는 입력을 변형해서는 안 됩니다. 여기에는 프로퍼티즈(Properties), 상태(state), 그리고 컨텍스트(context)가 포함됩니다. 화면을 업데이트하려면 기존 객체를 변환하는 대신 [상태를 “설정”](https://ko.react.dev/learn/state-a-components-memory)하십시오.
- 반환하는 JSX에서 컴포넌트의 로직을 표현하기 위해 노력하라.
  - “무언가를 변경”해야 할 경우 일반적으로 이벤트 핸들러에서 변경하고 싶을 것입니다.
  - **마지막 수단으로 `useEffect`를 사용**할 수 있습니다.
- 순수 함수를 작성하는 것은 약간의 연습이 필요하지만, React 패러다임의 힘을 풀어줍니다.

---

# 상호작용성 더하기

- 다룰 내용
  ```markdown
  - 사용자 이벤트를 처리하는 방법
  - 컴포넌트가 state를 이용하여 정보를 “기억”하는 방법
  - React가 UI를 업데이트하는 두 가지 단계
  - state가 변경된 후 바로 업데이트되지 않는 이유
  - 여러 개의 state 업데이트를 대기열에 추가하는 방법
  - state에서 객체를 업데이트하는 방법
  - state에서 배열을 업데이트하는 방법
  ```

## 이벤트에 응답하기

- React를 사용하면 JSX에 이벤트 핸들러를 추가할 수 있음.
  - 이벤트 핸들러는 상호작용에 반응하여 발생하는 자체 함수(ex. click, hover, input의 focus)

### 이벤트 핸들러 추가하기

- 이벤트 핸들러 추가를 위해서는 1. 먼저 함수를 정의하고 이를 2. 적절한 JSX 태그에 [prop 형태로 전달](https://ko.react.dev/learn/passing-props-to-a-component)해야 합니다.

- 예시

  1. `Button` 컴포넌트 *내부에* `handleClick` 함수를 선언합니다.
  2. 해당 함수 내부 로직을 구현합니다. 이번에는 메시지를 표시하기 위해 `alert`를 사용
  3. `<button>` JSX에 `onClick={handleClick}`을 추가

  ```tsx
  export default function Button() {
    function handleClick() {
      // 1번
      alert("You clicked me!"); // 2번
    }

    return (
      <button onClick={handleClick}>
        {" "}
        {/* 3번*/}
        Click me
      </button>
    );
  }
  ```

  - 이벤트 핸들러 함수
    - 일반적으로 컴포넌트 안에 정의
    - `**handle**`로 시작하는 이름 뒤에 이벤트 이름이 오도록 합니다.
    - 관례상 이벤트 핸들러의 이름은 `handle` 뒤에 이벤트 이름을 붙이는 것이 일반적
      - `onClick={handleClick}`, `onMouseEnter={handleMouseEnter}`등
  - JSX에서 인라인으로 정의할 수도, 더 간결하게 화살표 함수를 사용할 수도 있음

    - 예시

      ```tsx
      <button onClick={function handleClick() {
        alert('You clicked me!');
      }}>

      <button onClick={() => {
        alert('You clicked me!');
      }}>
      ```

        <aside>
        💡 단, 이 경우 렌더링 할때마다 매번 함수를 새로 만들게 되므로 순수성이 짙은 함수라면 memoization하는 게 성능을 향상시킬 수 있다.
        https://rules.sonarsource.com/typescript/RSPEC-6480/
        
        </aside>

### 주의

```tsx
// 함수를 전달
<button onClick={handleClick}>
// 함수를 호출
<button onClick={handleClick()}>
```

- 이벤트 핸들러로 전달한 함수들은 호출이 아닌 **전달해야 함**
- 예시에서 `handleClick()`끝에 있는 `()`은 클릭 없이 [렌더링](https://react-ko.dev/learn/render-and-commit) 중에 _즉시_ 함수(**[IIFE](https://developer.mozilla.org/ko/docs/Glossary/IIFE)**)를 실행.
  - 이는 [JSX의 `{` 및 `}`](https://react-ko.dev/learn/javascript-in-jsx-with-curly-braces) 내부의 JavaScript가 바로 실행되기 때문

```tsx
// 함수 전달
<button onClick={() => alert('...')}>
// 함수 호출
<button onClick={alert('...')}>
```

### 이벤트 핸들러 내에서 props 읽기

- 이벤트 핸들러는 컴포넌트 내부에서 선언되기 때문에 컴포넌트의 props에 접근 가능

```tsx
function AlertButton({ message, children }) {
  return <button onClick={() => alert(message)}>{children}</button>;
}

export default function Toolbar() {
  return (
    <div>
      <AlertButton message="Playing!">Play Movie</AlertButton>
      <AlertButton message="Uploading!">Upload Image</AlertButton>
    </div>
  );
}
```

### 이벤트 핸들러를 prop으로 전달하기

- 가끔 부모 컴포넌트가 자식의 이벤트 핸들러를 지정하고 싶을 때가 있다.
  - `Button` 컴포넌트를 사용하는 위치에 따라 버튼은 동영상을 재생하고 다른 버튼은 이미지를 업로드하는 등 서로 다른 기능을 실행하고 싶을 수 있습니다.
- 이런 경우, 자식 컴포넌트가 부모로부터 받는 prop을 이벤트 핸들러로써 받을 수 있다.(❓render props 패턴)

- 예시

  ```tsx
  function Button({ onClick, children }) {
    return <button onClick={onClick}>{children}</button>;
  }

  function PlayButton({ movieName }) {
    function handlePlayClick() {
      alert(`Playing ${movieName}!`);
    }

    return <Button onClick={handlePlayClick}>Play "{movieName}"</Button>;
  }

  function UploadButton() {
    return <Button onClick={() => alert("Uploading!")}>Upload Image</Button>;
  }

  export default function Toolbar() {
    return (
      <div>
        <PlayButton movieName="Kiki's Delivery Service" />
        <UploadButton />
      </div>
    );
  }
  ```

  - `Toolbar` 컴포넌트가 `PlayButton`과 `UploadButton`을 렌더링.
    - `PlayButton`은 `handlePlayClick`을 `Button` 내 `onClick` prop으로 전달.
    - `UploadButton`은 `() => alert('Uploading!')`을 `Button` 내 `onClick` prop으로 전달합니다.
  - 최종적으로, `Button` 컴포넌트는 `onClick` prop을 받음
    - 이후 받은 prop을 브라우저 빌트인 `<button>`의 `onClick={onClick}`으로 직접 전달
    - 이를 통해 React가 전달받은 함수를 클릭 시점에 호출함을 알 수 있다.

- 만약 [디자인 시스템](https://uxdesign.cc/everything-you-need-to-know-about-design-systems-54b109851969)([번역](https://github.com/yamoo9/Figma/blob/master/DesignSystem/everything-you-need-to-know-about-design-systems.md))을 적용한다면
  - 버튼과 같은 컴포넌트는 동작을 지정하지 않고 스타일만 지정하는 것이 일반적.
  - 대신, `PlayButton`과 `UploadButton` 같은 컴포넌트가 이벤트 핸들러를 전달하도록 한다.

### 이벤트 핸들러 props 이름 정하기

- `<button>`과 `<div>` 같은 빌트인 컴포넌트는 `onClick`과 같은 [브라우저 이벤트 이름](https://ko.react.dev/reference/react-dom/components/common#common-props) 만을 지원
  - But, 사용자 정의 컴포넌트에서는 이벤트 핸들러 prop의 이름을 원하는 대로 작성 가능
- 관습적으로 이벤트 핸들러 prop의 이름은 `**on`으로 시작하여 대문자 영문\*\*으로 이어진다.
  - ex) `Button` 컴포넌트의 `onClick` prop은 `onSmash`라는 이름으로 호출 가능
- 예시

  ```tsx
  export default function App() {
    return (
      <Toolbar
        onPlayMovie={() => alert("Playing!")}
        onUploadImage={() => alert("Uploading!")}
      />
    );
  }

  function Toolbar({ onPlayMovie, onUploadImage }) {
    return (
      <div>
        <Button onClick={onPlayMovie}>Play Movie</Button>
        <Button onClick={onUploadImage}>Upload Image</Button>
      </div>
    );
  }

  function Button({ onClick, children }) {
    return <button onClick={onClick}>{children}</button>;
  }
  ```

  - `<button onClick={onSmash}>`은 브라우저의 `<button>`(소문자)이 여전히 `onClick` prop을 필요로 하고 있음을 보여준다
    - But, 여러분이 직접 정의한 `Button` 컴포넌트가 받게 될 prop의 이름은 여러분이 원하는 대로 작성 가능.
  - 컴포넌트가 여러 상호작용을 지원한다면
    - 이벤트 핸들러 prop을 애플리케이션에 특화시켜 작성 가능
    - 예시에서는 `Toolbar` 컴포넌트가 `onPlayMovie`와 `onUploadImage` 이벤트 핸들러를 받음

- `App` 컴포넌트는 `**Toolbar`가 `onPlayMovie` 또는 `onUploadImage`를 가지고 *무엇*을 할 것인지 알 필요가 없음\*\*. (이 `Toolbar` 구현의 특별한 부분)
  - 지금 `Toolbar`는 위 요소들을 `Button`의 `onClick` 핸들러 요소로 내려보내지만, 추후에는 키보드 바로가기 키 입력을 통해 이들을 활성화할 수도 있다.
  - `onPlayMovie`와 같이 prop 이름을 애플리케이션별 상호작용에 기반하여 작성한다면 나중에 어떻게 이를 이용하게 될지에 대한 유연성을 제공할 것이다.

> 이벤트 핸들러에 적절한 HTML 태그를 사용하고 있는지 확인

예를 들어 클릭을 핸들링하기 위해서는 `<div onClick={handleClick}>` 대신 `[<button onClick={handleClick}>](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/button)`을 사용.
실제 브라우저에서 `<button>`은 키보드 내비게이션과 같은 빌트인 브라우저 동작을 활성화한다.
만일 버튼의 기본 브라우저 스타일링이 싫어서 링크나 다른 UI 요소처럼 보이도록 하고 싶다?
⇒ CSS를 통해 그 목적을 이룰 수 있다. [접근성을 위한 마크업 작성법에 대해 더 알아보세요.](https://developer.mozilla.org/en-US/docs/Learn/Accessibility/HTML)

>

### 이벤트 전파 (⭐ 중요⭐)

- 이벤트 핸들러는 해당 컴포넌트가 가진 어떤 자식 컴포넌트의 이벤트를 수신할 수도 있다.
  - 이를 **이벤트가 트리를 따라 “bubble” 되거나 “전파된다**”고 표현
  - 이때 이벤트는 발생한 지점에서 시작하여 **트리를 따라 위로 전달**됩니다.
- 예시
  - 아래 `<div>`는 두 개의 버튼을 포함하고 있고 `<div>` _그리고_ 각각의 버튼은 각자의 `onClick` 핸들러를 가지고 있다.
  - 둘 중의 어느 버튼을 클릭하더라도 해당 버튼의 `onClick`이 먼저 실행될 것이며 이후 부모인 `<div>`의 `onClick`이 뒤이어 실행될 것이다.
    - **두 개의 메시지가 표시**될 거다.
    - 만약 **툴바 자체를 클릭**한다면 **오직 부모인 `<div>`의 `onClick` 만이 실행될 것**이다.
  ```tsx
  export default function Toolbar() {
    return (
      <div
        className="Toolbar"
        onClick={() => {
          alert("You clicked on the toolbar!");
        }}
      >
        <button onClick={() => alert("Playing!")}>Play Movie</button>
        <button onClick={() => alert("Uploading!")}>Upload Image</button>
      </div>
    );
  }
  ```

### 주의

첨부한 JSX 태그에서만 작동하는 `onScroll`을 제외한 모든 이벤트는 React에서 전파된다.

### 이벤트 전파 멈추기

- 이벤트 핸들러는 **이벤트 오브젝트**를 유일한 매개변수로 받는다.
  - 관습을 따르자면 “event”를 의미하는 `e`로 호출되는 것이 일반적.
  - 이 오브젝트를 이벤트의 정보를 읽어들이는데 사용할 수 있다.
- 이벤트 오브젝트는 전파를 멈출 수 있게 해준다.

  - `e.stopPropagation()`를 호출한다.
  - 예시

    ```tsx
    function Button({ onClick, children }) {
      return (
        <button
          onClick={(e) => {
            e.stopPropagation(); // 전파가 위로 가는걸 막음
            onClick();
          }}
        >
          {children}
        </button>
      );
    }

    export default function Toolbar() {
      return (
        <div
          className="Toolbar"
          onClick={() => {
            alert("You clicked on the toolbar!");
          }}
        >
          <Button onClick={() => alert("Playing!")}>Play Movie</Button>
          <Button onClick={() => alert("Uploading!")}>Upload Image</Button>
        </div>
      );
    }
    ```

버튼을 클릭하면 다음과 같은 절차가 진행됩니다.

1. React가 `<button>`에 전달된 `onClick` 핸들러를 호출.
2. `Button`에 정의된 해당 핸들러는 다음을 수행.
   - `e.stopPropagation()`을 호출하여 이벤트가 더 이상 bubbling 되지 않도록 방지.
   - `Toolbar` 컴포넌트가 전달해 준 `onClick` 함수를 호출.
3. `Toolbar` 컴포넌트에서 정의된 위 함수가 버튼의 alert를 표시.
4. 전파가 중단되었으므로 부모인 `<div>`의 `onClick`은 *실행되지 않는다*.

- `e.stopPropagation()`의 결과, 버튼을 클릭하는 것은 이제 `<button>`과 그 부모인 툴바의 `<div>`가 보내는 두 개의 alert를 표시하지 않고 단 하나의 `<button>` alert 만을 표시한다.
- 버튼을 클릭하는 것은 그 주변의 툴바 부분을 클릭하는 것과 같지 않기에 이 UI 상에서는 전파를 멈추는 것이 합리적일 것이다.

- 단계별 이벤트 캡처
  - 드물게 **\*전파가 중단된 상황**일 지라도\* **자식 컴포넌트의 모든 이벤트를 캡처해 확인해야 할 수도 있음**
    - 분석을 위해 전파 로직에 상관 없이 모든 클릭 이벤트를 기록하고 싶을 수 있다.
    - 이를 위해서는 이벤트명 마지막에 `Capture`를 추가하면 된다.
  ```tsx
  <div
    onClickCapture={() => {
      /* this runs first | 먼저 실행됨 */
    }}
  >
    <button onClick={(e) => e.stopPropagation()} />
    <button onClick={(e) => e.stopPropagation()} />
  </div>
  ```
  각각의 이벤트는 세 단계를 거쳐 전파됩니다.
  1. **아래로 전달되면서 만나는 모든 `onClickCapture` 핸들러를 호출**.
  2. **클릭된 요소의 `onClick` 핸들러를 실행**.
  3. **위로 전달되면서 만나는 모든 `onClick` 핸들러를 호출**.
  - 이벤트 캡처는 **라우터나 분석을 위한 코드에 유용**할 수 있다
    - But, 일반 애플리케이션 코드에서는 사용하지 않을 가능성이 높다.

### 전파 대신 핸들러 전달하기

```tsx
function Button({ onClick, children }) {
  return (
    <button
      onClick={(e) => {
        e.stopPropagation();
        onClick();
      }}
    >
      {children}
    </button>
  );
}
```

- 핸들러 내에서 부모에게서 받은 `onClick` 이벤트 핸들러를 호출하는 부분 앞에 코드 추가 가능.
  - 이런 패턴은 전파의 대안을 제공한다.
  - 부모 컴포넌트가 일부 추가적인 동작에 특화되도록 하면서 자식 컴포넌트가 이벤트를 핸들링할 수 있도록 한다.
  - 전파와는 다르게 자동으로 동작하지 않음.
    - 이 패턴은 일부 이벤트의 결과로 실행되는 전체 코드 체인을 명확히 추적할 수 있게 해줌
- 전파를 활용하고 있지만 **어떤 핸들러가 왜 실행되는 지 추적하는데 어려움을 겪고 있다?**
  - 이러한 접근법을 시도해 보는걸 추천

### 기본 동작 방지

- 일부 브라우저 이벤트는 그와 관련된 기본 브라우저 동작을 가진다.
  - 일례로 `<form>`의 제출 이벤트 (페이지 전체를 리로드하는 것이 기본 동작)
- 이벤트 객체에서 `e.preventDefault()`를 호출하여 이런 일이 발생하지 않도록 해준다

```tsx
export default function Signup() {
  return (
    <form
      onSubmit={(e) => {
        e.preventDefault(); // 리로드 방지함
        alert("Submitting!");
      }}
    >
      <input />
      <button>Send</button>
    </form>
  );
}
```

`**e.stopPropagation()`와 `e.preventDefault()`를 혼동하지 말 것\*\*.

- 둘 다 유용하지만, 서로 전혀 관련 없는 기능.
- `[e.stopPropagation()](https://developer.mozilla.org/docs/Web/API/Event/stopPropagation)`은 **이벤트 핸들러가 상위 태그에서 실행되지 않도록 멈춘다**.
- `[e.preventDefault()`](https://developer.mozilla.org/docs/Web/API/Event/preventDefault) 는 **기본 브라우저 동작을 가진 일부 이벤트가 해당 기본 동작을 실행하지 않도록 방지**.

### 이벤트 핸들러에 부작용이 생길 수 있나요?

- 가능! **이벤트 핸들러는 사이드 이펙트를 위한 최고의 위치**
- 함수를 렌더링하는 것과 다르게 **이벤트 핸들러는 [순수할](https://ko.react.dev/learn/keeping-components-pure) 필요가 없기에 무언가를 *변경*하는데 최적의 위치**입니다.
  - ex) 타이핑에 반응해 입력 값 수정, 버튼 입력에 따라 리스트를 변경.
- But, 일부 정보를 수정하기 위해서는 먼저 그 정보를 저장하기 위한 수단이 필요
  - React에서는 [컴포넌트의 기억 역할을 하는 state](https://ko.react.dev/learn/state-a-components-memory)를 이용할 수 있다.

### 요약

- `<button>`과 같은 요소에 함수를 prop으로 전달하여 이벤트를 핸들링할 수 있다.

  ```tsx

  // case 1
  <button onClick={function handleClick() {
  	console.log('clicked!');
  }}>Click</button>

  // case 2
  <button onClick={() => {
  	console.log('clicked!');
  }}>Click</button>

  // case 2-2
  const handleClick = useCallback(() => {
  	console.log('clicked!');
  }, []);
  ...
  <button onClick={handleClick}>Click</button>
  ```

- 이벤트 핸들러는 **호출이 아니라** 전달만 가능합니다! `onClick={handleClick()}`이 아니라 `onClick={handleClick}`
  - 단, 함수 호출이더라도 javascript 패턴 중 currying을 사용해 함수를 반환하면 문제가 없다.
  ```tsx
  const handleClick = () => {
    return () => {
      console.log("Currying!");
    };
  };
  <button onClick={handleClick()}>Click</button>;
  ```
- 이벤트 핸들러 함수는 **별개의 함수** 혹은 **인라인 형태로 정의** 가능
  - 리액트 성능을 위해 별개의 함수로 빼놓는걸 권장 (메모이제이션은 상황에 따라 적용하기)
- 이벤트 핸들러는 **컴포넌트 내부에서 정의**되기에 **다른 prop에 접근 가능**
- 이벤트 핸들러는 **부모에서 선언하여 자식에게 prop으로 전달 가능**
- 사용자 정의 이벤트 핸들러의 이름을 애플리케이션에 특화된 이름으로 작성할 수 있다.
- **이벤트는 위쪽으로 전파**
  - 첫 번째 매개변수로 `e.stopPropagation()`를 호출하여 방지 가능
- **이벤트는 의도치 않은 기본 브라우저 동작을 유발**할 수 있다.
  - `e.preventDefault()`를 호출하여 방지 가능
- 명시적으로 이벤트 핸들러 prop을 자식 핸들러에서 호출하는 건 전파에 대한 좋은 대안이 될 수 있다.

---

## State: 컴포넌트의 메모리

- 컴포넌트는 **상호 작용의 결과로 화면의 내용을 변경**해야 하는 경우가 많음
  - ex) 폼에 입력하면 입력 필드가 업데이트, 이미지 캐러셀에서 ‘다음’을 클릭하면 표시되는 이미지가 변경, ‘구매’를 클릭하면 제품이 장바구니에 담겨야 한다.
- 컴포넌트는 현재 입력값, 현재 이미지, 장바구니와 같은 **현재의 것들을 “기억”해야 한다.**
  - React에서는 이런 종류의 **컴포넌트별 메모리**를 **_state_**

### 일반 변수로 충분하지 않을 때

- 예시

  ```tsx
  import { sculptureList } from "./data.js";

  export default function Gallery() {
    let index = 0;

    function handleClick() {
      index = index + 1;
    }

    let sculpture = sculptureList[index];
    return (
      <>
        <button onClick={handleClick}>Next</button>
        <h2>
          <i>{sculpture.name} </i>
          by {sculpture.artist}
        </h2>
        <h3>
          ({index + 1} of {sculptureList.length})
        </h3>
        <img src={sculpture.url} alt={sculpture.alt} />
        <p>{sculpture.description}</p>
      </>
    );
  }
  ```

  - `handleClick` 이벤트 핸들러가 지역 변수 `index`를 업데이트하고 있다.
    - But, 두 가지 이유로 인해 변경 사항이 표시되지 않음
    1. **지역 변수는 렌더링 간에 유지되지 않는다.**
       - **React는 컴포넌트를 두 번째로 렌더링할 때 지역 변수에 대한 변경 사항을 고려하지 않고 처음부터 렌더링**.
    2. **지역 변수를 변경해도 렌더링을 발동시키지 않는다.**
       - React는 **새로운 데이터로 컴포넌트를 다시 렌더링해야 한다는 것을 인식하지 못함**.

컴포넌트를 새 데이터로 업데이트하려면 두 가지 작업이 필요

1. **렌더링 사이에 데이터를 유지.**
2. **새로운 데이터로 컴포넌트를 렌더링(리렌더링)하도록 React를 촉발한다**

여기서, `[useState](https://react-ko.dev/reference/react/useState)` 훅은 이 두 가지를 제공

1. 렌더링 사이에 **데이터를 유지**하기 위한 **state 변수**.
2. 변수를 업데이트, React가 컴포넌트를 다시 렌더링하도록 **촉발**하는 **state 설정자 함수(setState).**

### state 변수 추가

```tsx
import { useState } from "react";

// ... in the top of React Component.
const [index, setIndex] = useState(0);
// index는 state 변수이고 setIndex는 설정자 함수입니다.
```

> 여기서 `[` 와 `]` 구문을 [배열 구조분해](https://ko.javascript.info/destructuring-assignment)라고 하며, 배열에서 값을 읽을 수 있다.
> `useState`가 반환하는 배열에는 항상 정확히 두 개의 항목이 있다.

- 예시 리팩토링 결과

  ```tsx
  import { useState } from "react";
  import { sculptureList } from "./data.js";

  export default function Gallery() {
    const [index, setIndex] = useState(0); // 렌더링사이에 데이터 유지, 리렌더링 촉발

    function handleClick() {
      setIndex(index + 1);
    }

    let sculpture = sculptureList[index];
    return (
      <>
        <button onClick={handleClick}>Next</button>
        <h2>
          <i>{sculpture.name} </i>
          by {sculpture.artist}
        </h2>
        <h3>
          ({index + 1} of {sculptureList.length})
        </h3>
        <img src={sculpture.url} alt={sculpture.alt} />
        <p>{sculpture.description}</p>
      </>
    );
  }
  ```

### 첫 번째 Hook과의 만남

- React에서는 `useState`를 비롯해 ”`use`“로 시작하는 다른 함수를 훅(hook)이라고 부름.
  - *훅*은 React가 [렌더링](https://ko.react.dev/learn/render-and-commit#step-1-trigger-a-render) 중일 때만 사용할 수 있는 특별한 함수
  - 이를 통해 다양한 React 기능을 “**연결”**할 수 있다.
- state는 이러한 기능 중 하나, 다양한 훅이 있음 (Reference에서 자세히 다룰 예정)

### 주의

- **훅(`use`로 시작하는 함수)은 “컴포넌트의 최상위 레벨” (최상위 컴포넌트 아님) 또는 [커스텀 훅](https://react-ko.dev/learn/reusing-logic-with-custom-hooks)에서만 호출할 수 있다.**
- 조건문, 반복문 또는 기타 중첩된 함수 내부에서는 훅을 호출할 수 없다.
- 훅은 함수이지만 **컴포넌트의 필요에 대한 무조건적인 선언으로 생각**하면 도움이 된다.
- 파일 상단에서 모듈을 “import”하는 것과 유사하게 컴포넌트 상단에서 React 기능을 “사용”한다.

### useState 해부하기

```tsx
const [index, setIndex] = useState(0);
```

- `[useState](https://react-ko.dev/reference/react/useState)` 호출 === **React에게 이 컴포넌트가 무언가를 기억하기를 원함**
  - 이 경우 React가 `index`를 기억하기를 원합니다.

> 이름은 `const [something, setSomething]`과 같이 지정하는 것이 일반적.(React Convention)
> 원하는 대로 이름을 지을 수 있지만, 규칙을 정하면 프로젝트 전반에서 이해하기 쉬워진다.

- `useState`의 유일한 인수는 state 변수의 **초기값**
  - 예제에서는 `useState(0)`에 의해 `index`의 초기값이 `0`으로 설정
- 컴포넌트가 렌더링될 때마다 `useState`는 두 개의 값을 포함하는 배열을 제공

  1. 저장한 값을 가진 **state 변수**(`index`).
  2. state 변수를 업데이트, React가 컴포넌트를 다시 렌더링하도록 촉발할 수 있는 **state 설정자 함수** (`setIndex`).

- 동작 과정 (`const [index, setIndex] = useState(0);`)
  1. **컴포넌트가 처음 렌더링**
     - `index`의 초기값으로 `0`을 `useState`에 전달했으므로 `[0, setIndex]`가 반환됩니다.
     - React는 `0`을 최신 state 값으로 기억합니다.
  2. **state를 업데이트**
     - 사용자가 버튼을 클릭하면 `setIndex(index + 1)`를 호출
     - `index`는 `0`이므로 `setIndex(1)`.
     - 이렇게 하면 React는 이제 `index`가 `1`임을 기억하고 다음 렌더링을 촉발한다.
  3. **컴포넌트가 두 번째로 렌더링**
     - React는 여전히 `useState(0)`을 보지만, `index`를 `1`로 설정한 것을 기억하고 있기 때문에, 이번에는 `[1, setIndex]`를 반환
  4. **1 ~ 3의 반복!**

### 컴포넌트에 여러 state 변수 지정

- 하나의 컴포넌트에 원하는 만큼 많은 유형의 state 변수를 가질 수 있다.

- 예시

  ```tsx
  import { useState } from "react";
  import { sculptureList } from "./data.js";

  export default function Gallery() {
    const [index, setIndex] = useState(0);
    const [showMore, setShowMore] = useState(false);

    function handleNextClick() {
      setIndex(index + 1);
    }

    function handleMoreClick() {
      setShowMore(!showMore);
    }

    let sculpture = sculptureList[index];
    return (
      <>
        <button onClick={handleNextClick}>Next</button>
        <h2>
          <i>{sculpture.name} </i>
          by {sculpture.artist}
        </h2>
        <h3>
          ({index + 1} of {sculptureList.length})
        </h3>
        <button onClick={handleMoreClick}>
          {showMore ? "Hide" : "Show"} details
        </button>
        {showMore && <p>{sculpture.description}</p>}
        <img src={sculpture.url} alt={sculpture.alt} />
      </>
    );
  }
  ```

  - 예제에서 `index`와 `showMore`처럼 서로 연관이 없는 경우 , 여러 개의 state 변수를 갖는 것이 좋습니다.
  - But, 두 개의 state 변수를 자주 함께 변경하는 경우에는 두 변수를 하나로 합치는 것이 더 좋을 수 있다.
    - ex) 필드가 많은 폼의 경우 **필드별로 state 변수를 사용하는 것**보다 **객체를 값으로 하는 하나의 state 변수를 사용하는 것**이 더 편리
    - 자세한 팁은 [state 구조 선택](https://ko.react.dev/learn/choosing-the-state-structure)에서 다룰 예정.

### React는 어떤 state를 반환할지 알 수 있는가?

- `useState` 호출이 다른 `state 변수`를 참조하는 지에 대한 정보를 받지 못한다
  - `useState`에 전달되는 “식별자”가 없는데 어떤 state 변수를 반환할지 어떻게 알 수 있을까요? 함수를 파싱하는 것과 같은 마법에 의존할까?
  - **NO!**
- 간결한 구문을 구현하기 위해
  - 훅은 **동일한 컴포넌트의 모든 렌더링에서 안정적인 호출 순서에 의존합.**
  - 위의 규칙(“**최상위 수준에서만 훅 호출**”)을 따르면, **훅은 항상 같은 순서로 호출**되기 때문에 실제로 잘 작동합니다.
    - 또한 [린터 플러그인](https://www.npmjs.com/package/eslint-plugin-react-hooks)은 대부분의 실수를 잡아줍니다.
- 내부적으로 **React는 모든 컴포넌트에 대해 한 쌍의 state 배열을 가짐**.
- **또한 렌더링 전에 `0`으로 설정된 현재 쌍 인덱스를 유지**
  - `useState`를 호출할 때마다 React는 다음 state 쌍을 제공하고 인덱스를 증가시킨다.
  - 이 메커니즘은 [React Hook: 마법이 아니라 배열일 뿐입니다](https://medium.com/@ryardley/react-hooks-not-magic-just-arrays-cd4f1857236e)에서 확인 가능
    - 이 예제에서는 **React를 사용하지 않지만,** 내부적으로 `useState`가 어떻게 작동하는지에 대한 아이디어를 제공한다.
- 예시

  ```tsx
  let componentHooks = [];
  let currentHookIndex = 0;

  // How useState works inside React (simplified).
  function useState(initialState) {
    let pair = componentHooks[currentHookIndex];
    if (pair) {
      // This is not the first render,
      // so the state pair already exists.
      // Return it and prepare for next Hook call.
      currentHookIndex++;
      return pair;
    }

    // This is the first time we're rendering,
    // so create a state pair and store it.
    pair = [initialState, setState];

    function setState(nextState) {
      // When the user requests a state change,
      // put the new value into the pair.
      pair[0] = nextState;
      updateDOM();
    }

    // Store the pair for future renders
    // and prepare for the next Hook call.
    componentHooks[currentHookIndex] = pair;
    currentHookIndex++;
    return pair;
  }

  function Gallery() {
    // Each useState() call will get the next pair.
    const [index, setIndex] = useState(0);
    const [showMore, setShowMore] = useState(false);

    function handleNextClick() {
      setIndex(index + 1);
    }

    function handleMoreClick() {
      setShowMore(!showMore);
    }

    let sculpture = sculptureList[index];
    // This example doesn't use React, so
    // return an output object instead of JSX.
    return {
      onNextClick: handleNextClick,
      onMoreClick: handleMoreClick,
      header: `${sculpture.name} by ${sculpture.artist}`,
      counter: `${index + 1} of ${sculptureList.length}`,
      more: `${showMore ? "Hide" : "Show"} details`,
      description: showMore ? sculpture.description : null,
      imageSrc: sculpture.url,
      imageAlt: sculpture.alt,
    };
  }

  function updateDOM() {
    // Reset the current Hook index
    // before rendering the component.
    currentHookIndex = 0;
    let output = Gallery();

    // Update the DOM to match the output.
    // This is the part React does for you.
    nextButton.onclick = output.onNextClick;
    header.textContent = output.header;
    moreButton.onclick = output.onMoreClick;
    moreButton.textContent = output.more;
    image.src = output.imageSrc;
    image.alt = output.imageAlt;
    if (output.description !== null) {
      description.textContent = output.description;
      description.style.display = "";
    } else {
      description.style.display = "none";
    }
  }

  let nextButton = document.getElementById("nextButton");
  let header = document.getElementById("header");
  let moreButton = document.getElementById("moreButton");
  let description = document.getElementById("description");
  let image = document.getElementById("image");
  let sculptureList = [
    {
      name: "Homenaje a la Neurocirugía",
      artist: "Marta Colvin Andrade",
      description:
        "Although Colvin is predominantly known for abstract themes that allude to pre-Hispanic symbols, this gigantic sculpture, an homage to neurosurgery, is one of her most recognizable public art pieces.",
      url: "https://i.imgur.com/Mx7dA2Y.jpg",
      alt: "A bronze statue of two crossed hands delicately holding a human brain in their fingertips.",
    },
    {
      name: "Floralis Genérica",
      artist: "Eduardo Catalano",
      description:
        "This enormous (75 ft. or 23m) silver flower is located in Buenos Aires. It is designed to move, closing its petals in the evening or when strong winds blow and opening them in the morning.",
      url: "https://i.imgur.com/ZF6s192m.jpg",
      alt: "A gigantic metallic flower sculpture with reflective mirror-like petals and strong stamens.",
    },
    {
      name: "Eternal Presence",
      artist: "John Woodrow Wilson",
      description:
        'Wilson was known for his preoccupation with equality, social justice, as well as the essential and spiritual qualities of humankind. This massive (7ft. or 2,13m) bronze represents what he described as "a symbolic Black presence infused with a sense of universal humanity."',
      url: "https://i.imgur.com/aTtVpES.jpg",
      alt: "The sculpture depicting a human head seems ever-present and solemn. It radiates calm and serenity.",
    },
    {
      name: "Moai",
      artist: "Unknown Artist",
      description:
        "Located on the Easter Island, there are 1,000 moai, or extant monumental statues, created by the early Rapa Nui people, which some believe represented deified ancestors.",
      url: "https://i.imgur.com/RCwLEoQm.jpg",
      alt: "Three monumental stone busts with the heads that are disproportionately large with somber faces.",
    },
    {
      name: "Blue Nana",
      artist: "Niki de Saint Phalle",
      description:
        "The Nanas are triumphant creatures, symbols of femininity and maternity. Initially, Saint Phalle used fabric and found objects for the Nanas, and later on introduced polyester to achieve a more vibrant effect.",
      url: "https://i.imgur.com/Sd1AgUOm.jpg",
      alt: "A large mosaic sculpture of a whimsical dancing female figure in a colorful costume emanating joy.",
    },
    {
      name: "Ultimate Form",
      artist: "Barbara Hepworth",
      description:
        "This abstract bronze sculpture is a part of The Family of Man series located at Yorkshire Sculpture Park. Hepworth chose not to create literal representations of the world but developed abstract forms inspired by people and landscapes.",
      url: "https://i.imgur.com/2heNQDcm.jpg",
      alt: "A tall sculpture made of three elements stacked on each other reminding of a human figure.",
    },
    {
      name: "Cavaliere",
      artist: "Lamidi Olonade Fakeye",
      description:
        "Descended from four generations of woodcarvers, Fakeye's work blended traditional and contemporary Yoruba themes.",
      url: "https://i.imgur.com/wIdGuZwm.png",
      alt: "An intricate wood sculpture of a warrior with a focused face on a horse adorned with patterns.",
    },
    {
      name: "Big Bellies",
      artist: "Alina Szapocznikow",
      description:
        "Szapocznikow is known for her sculptures of the fragmented body as a metaphor for the fragility and impermanence of youth and beauty. This sculpture depicts two very realistic large bellies stacked on top of each other, each around five feet (1,5m) tall.",
      url: "https://i.imgur.com/AlHTAdDm.jpg",
      alt: "The sculpture reminds a cascade of folds, quite different from bellies in classical sculptures.",
    },
    {
      name: "Terracotta Army",
      artist: "Unknown Artist",
      description:
        "The Terracotta Army is a collection of terracotta sculptures depicting the armies of Qin Shi Huang, the first Emperor of China. The army consisted of more than 8,000 soldiers, 130 chariots with 520 horses, and 150 cavalry horses.",
      url: "https://i.imgur.com/HMFmH6m.jpg",
      alt: "12 terracotta sculptures of solemn warriors, each with a unique facial expression and armor.",
    },
    {
      name: "Lunar Landscape",
      artist: "Louise Nevelson",
      description:
        "Nevelson was known for scavenging objects from New York City debris, which she would later assemble into monumental constructions. In this one, she used disparate parts like a bedpost, juggling pin, and seat fragment, nailing and gluing them into boxes that reflect the influence of Cubism’s geometric abstraction of space and form.",
      url: "https://i.imgur.com/rN7hY6om.jpg",
      alt: "A black matte sculpture where the individual elements are initially indistinguishable.",
    },
    {
      name: "Aureole",
      artist: "Ranjani Shettar",
      description:
        'Shettar merges the traditional and the modern, the natural and the industrial. Her art focuses on the relationship between man and nature. Her work was described as compelling both abstractly and figuratively, gravity defying, and a "fine synthesis of unlikely materials."',
      url: "https://i.imgur.com/okTpbHhm.jpg",
      alt: "A pale wire-like sculpture mounted on concrete wall and descending on the floor. It appears light.",
    },
    {
      name: "Hippos",
      artist: "Taipei Zoo",
      description:
        "The Taipei Zoo commissioned a Hippo Square featuring submerged hippos at play.",
      url: "https://i.imgur.com/6o5Vuyu.jpg",
      alt: "A group of bronze hippo sculptures emerging from the sett sidewalk as if they were swimming.",
    },
  ];

  // Make UI match the initial state.
  updateDOM();
  ```

### State는 격리된, 비공개 상태다.

- state는 화면의 컴포넌트 인스턴스에 지역적
  - 즉, **동일한 컴포넌트를 두 군데에서 렌더링하면 각 사본은 완전히 격리된 state를 갖게 된다!**
  - 이 중 하나를 변경해도 다른 컴포넌트에는 영향을 미치지 않는다.
- 예시 ([codesandbox](https://codesandbox.io/s/rodp3h?file=%2FApp.js&utm_medium=sandpack))
  - 각각의 state가 독립적인 것을 확인
- 이것이 바로 모듈 상단에 선언하는 일반 변수와 state의 차이점입니다. state는 특정 함수 호출에 묶이지 않고, 코드의 특정 위치에 묶이지도 않지만, 화면상의 특정 위치에 “지역적”입니다. 두 개의 `<Gallery />` 컴포넌트를 렌더링했으므로 해당 state는 별도로 저장됩니다.
- `Page` 컴포넌트는 `Gallery`의 state뿐 아니라 심지어 state가 있는지 여부조차 전혀 “알지 못한다”는 점도 주목하세요. props와 달리 **state는 이를 선언하는 컴포넌트 외에는 완전히 비공개이며, 부모 컴포넌트는 이를 변경할 수 없습니다.** 따라서 다른 컴포넌트에 영향을 주지 않고 state를 추가하거나 제거할 수 있습니다.
- 두 Gallery 컴포넌트의 state를 동기화하려면?
  - 자식 컴포넌트에서 state를 *제거*하고 가장 가까운 공유 부모 컴포넌트에 추가하는 것
    - 이 주제는 [컴포넌트 간의 state 공유](https://ko.react.dev/learn/sharing-state-between-components)에서 다시 다룰 예정

### 요약

- 컴포넌트가 렌더링 사이에 일부 정보를 “**기억**”해야 할 때 , **state 변수를 사용**
- state 변수는 `useState` 훅을 호출하여 선언한다
- **훅은 `use`로 시작하는 특수 함수**
  - state와 같은 React 기능을 “연결”할 수 있게 해준다.
- 훅은 모듈에서 import할 때와 마찬가지로, 컴포넌트 안에서 조건과 무관하게 항상 호출
  - `useState`를 포함한 훅을 호출하는 것은 컴포넌트나 다른 훅의 최상위 레벨에서만 유효
- `useState` 훅은 **현재 state(첫번째 인자)**와 이를 **업데이트할 함수(두번째 인자)**로 이루어진 한 쌍을 반환한다.
- state 변수는 둘 이상 가질 수 있습니다.
  - 내부적으로 React는 이를 순서대로 일치시킨다.
- state는 컴포넌트 외부에 비공개이다.
  - 두 곳에서 렌더링하면 각 복사본은 고유한 state를 갖게 된다.
