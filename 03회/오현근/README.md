# [3회]조건부 렌더링 ~ state: 컴포넌트의 메모리

### 링크 : https://tidal-swoop-2d1.notion.site/state-5fa898c18b67471db1e90db8d2ffce17?pvs=4

## 조건부로 반환하는 JSX

```tsx
function Item({ name, isPacked }) {
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

Item 컴포넌트는 isPacked가 boolean 값으로 나눠져 props를 받고 있습니다.

isPacked 가 true일때 패킹된 아이템에서 체크 표시를 추가하고 싶다면 어떻게 할까요?

```tsx
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

간단하게 이런 식으로 작성할 수 있습니다.

물론 잘 작동합니다!

컴포넌트 자체가 js 함수 덩어리니까 js로 처리되겠죠?

## null을 사용해 조건부로 아무것도 반환하지 않기

컴포넌트가 null을 반환하길 원할 수 있습니다.

예를들어, 포장되었다면, 아이템을 표시하지않고 싶다면

```tsx
if (isPacked) {
  return null;
}
return <li className="item">{name}</li>;
```

같은 분기처리를 통해 나타낼 수 있습니다.

컴포넌트에서 null 을 반환하면, 컴포넌트가 나타나지 않고 잘 동작합니다.

**그러나,** 같이 일하는 개발자들은 null을 싫어합니다. 갑자기 null 이 반환된다고 하면 놀랄 수 있습니다!

## 조건을 포함한 JSX

예제를 살펴보면, isPacked 된 Item 컴포넌트는 체크해주고 아니면 없는 것을 제외하곤 모두 같은 형태입니다.

```tsx
<li className="item">...</li>
```

의 같은 구조를 반환합니다.

```tsx
if (isPacked) {
  return <li className="item">{name} ✔</li>;
}
return <li className="item">{name}</li>;
```

그래서 이렇게 작성하면?

**문제점**

- 중복된 내용이 많아보입니다.
- 하나의 변경사항이 생긴다면? 공통된 모든 분기에서 처리해줘야할 것 입니다.
- 컴포넌트가 커진다면 return문이 굉장히 길어질 것 같습니다.

조건문을 조금 변경해서 **[DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)하게** 만들 수 있습니다.

## 조건(삼항) 연산자( ? : )

```tsx
return <li className="item">{isPacked ? name + " ✔" : name}</li>;
```

삼항연산자를 사용해서 반복된 li태그를 하나로 줄이고, 분기된 부분만 처리하면 됩니다.

- 😮삼항연산자 주의 경험
  https://developer.mozilla.org/ko/docs/Web/JavaScript/Reference/Operators/Operator_precedence
  ```tsx
  가끔 연산자 우선순위를 헷갈려서 문제가 될 수 있습니다.
  return (
    <li className="item">
      {isPacked ? name + ' ✔' : name}
    </li>
  );

  isOkay ? 바보.천재.어쩌구.저쩌구 : ref(getValue()[prop].mapType.머시기저시기) + 'string',

  이때 하고싶었던 건 조건이 틀리다면 바보.천재.어쩌구.저쩌구 + 'string'을 붙이고
  아니라면 ref(getValue()[prop].mapType.머시기저시기) + 'string'을 붙여라 라는 느낌으로 작성했습니다.

  변명을 붙이자면.. 내부가 굉장히 길어지면서 햇갈렸습니다.

  당시에는 해당 조회 과정에서 무조건 'string'을 붙이는게목적이었는데
  isOkay가 늘 false였기 때문에 +'string'값이 나와서 문제도 없었죠..
  근데.. 데이터가 변경되면서 앞에게 나오기 시작했습니다.. 그리고 뒤에 붙은 'string'도 사라져서 문제가됐었습니다.
  ```

## Deep 두 예제는 완전히 동일할까요?

```jsx
function comp1(){
	if (isPacked) {
	  return <li className="item">{name} ✔</li>;
	}
	return <li className="item">{name}</li>;
}
function comp2(){
	return (
	  <li className="item">
	    {isPacked ? name + ' ✔' : name}
	  </li>
	);
}
	babel 변환 시
if (isPacked) {
    return /*#__PURE__*/React.createElement("li", {
      className: "item"
    }, name, " \u2714");
  }
  return /*#__PURE__*/React.createElement("li", {
    className: "item"
  }, name);

return /*#__PURE__*/React.createElement("li", {
    className: "item"
  }, isPacked ? name + ' ✔' : name);

JSX 요소는 내부 state를 보유하지 않음:
JSX 요소는 단순히 UI를 나타내는 템플릿 또는 청사진으로 사용됩니다.
이 예제에서의 <li> 요소는 실제 DOM 노드가 아니며,
내부 상태를 가지지 않습니다.
따라서 "인스턴스"라는 개념이 적용되지 않습니다.

동등한 결과물 생성:
두 예제 모두 isPacked 변수에 따라 조건부로 표시되는 내용이 달라집니다.
하지만 두 경우 모두 <li> 요소는 조건에 따라 텍스트가 추가되거나 추가되지 않습니다.
따라서 이 코드는 두 가지 다른 "인스턴스"를 생성하는 것이 아니라,
 동일한 요소를 서로 다른 내용으로 렌더링하는 것입니다.

상태 관리 및 재설정:
JSX 요소는 내부 상태를 가지지 않으며, 이러한 상태를 관리하지 않습니다.
객체 지향 프로그래밍에서 클래스 인스턴스는 내부 상태를 가지고 있고 이를 관리할 수 있지만,
JSX 요소는 상태 관리의 역할을 하지 않습니다.
```

- 체크된 항목에 <del >테그를 넣고싶다면
  ```jsx
  function Item({ name, isPacked }) {
    return (
      <li className="item">
        {isPacked ? (
          **<del>
            {name + ' ✔'}
          </del>**
        ) : (
          name
        )}
      </li>
    );
  }
  간단히 <del> 태그로 감싸면 됩니다.

  이 스타일은 간단한 조건엔 적합하지만
  복잡해진다면 -> 새로운 컴포넌트로 추출하거나, 함수를 사용해 정리해보세요.
  ```

## 논리 AND 연산자(&&)

React 컴포넌트 내에서 조건이 **참** 일때, 일부 JSX를 렌더링하거나,

**그렇지 않을땐 아무것도 렌더링 하지 않을때** 사용합니다.

- && 예시
  ```jsx
  return (
    <li className="item">
      {name} { '✔' && isPacked}
    </li>
  );
  isPacekd가 false라면 && 를 실행하지 않고 전체를 false로 만듭니다.
  하지만 true라면 우측 연산을 실행합니다.

  && 와 || 연산자를 익히고 갑시다.
  &&는 falsy를 찾는 연산이고
  ||는 truthy를 찾는 연산입니다.

  &&는 falsy를 찾는 연산이기 떄문에 falsy한 값이 없다면, 마지막 값이 출력됩니다.
  falsy한 값이 있다면, 거기서 바로 출력하게 됩니다.
  ||는 truthy를 찾는 연산이기 때문에 true한 값이 있다면, 바로 출력하고 넘어갑니다.
  (이를 단락 평가라고하고 뒤에 나오는 값은 확인하지 않습니다.)
  우선순위는 &&가 더 높습니다.

  if (null || 1 && false || true) alert( 'hi' );는 어떻게 될까요?

  ```
- **함정 - &&의 왼쪽에 숫자를 넣지 마세요.**
  위에서 나오듯이, && 연산자를 통해 0은 falsy한 값이므로 0으로 출력이 됩니다.
  여기까지는 괜찮으나, react에서 컴포넌트 위치에 해당 값이 나온다면
  그냥 0 으로 출력되어 난감해질 수 있습니다.
  조건문 처리하도록 합시다
  ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/c25eb7fa-fce5-4067-a2c4-bec8874fa401/12b86cd7-7587-42f2-a92e-02e45dac0986/Untitled.png)

## 변수에 조건부로 JSX 할당하기

let을 통해 변수에 값을 선언하고, 할당 함으로써 해결해보세요!

```jsx
function Item({ name, isPacked }) {
  **let itemContent = name; // 선언
  if (isPacked) {
    itemContent = name + " ✔"; // isPacked가 true일때 name + '체크' 할당
  }**
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}
```

이 뿐 아니라, 할당을 **컴포넌트**로 해도 괜찮습니다.

```jsx
function Item({ name, isPacked }) {
  **let itemContent = name;**
  if (isPacked) {
    **itemContent = (
      <del>
        {name + " ✔"}
      </del>
    );**
  }
  return (
    <li className="item">
      {itemContent}
    </li>
  );
}
```

## 도전과제

- ? : 미완료 항목 아이콘 표시
  ```jsx
  function Item({ name, isPacked }) {
    return (
      <li className="item">
        {name} {isPacked && '✔'}
      </li>
    );
  }

  isPacked가 true던 false던 값에 '✔'밖에 못넣어주니 '❌'를 넣어줍시다.

  true일땐 '✔' false 일땐 '❌' 를 넣어주면되겠죠?

  function Item({ name, isPacked }) {
    return (
      <li className="item">
        {name} { isPacked ? '✔' : '❌'}
      </li>
    );
  }
  ```
- &&로 항목의 중요도 표시
  ```jsx
  function Item({ name, importance }) {
    return (
      <li className="item">
        {name}
      </li>
    );
  }

  중요도가 0이 아니라면 나와줘야 한다면 importance를 가지고 처리해줘야겠죠.

  더 다양한 값이라면, 분기처리를 해줘야 하겠지만, 0은 falsy한 값이므로
  !!importance로 처리해주도록 합시다.

  function Item({ name, importance }) {
    return (
      <li className="item">
        {name}  {!!importance && '( importance: '+ importance +' )'}
      </li>
    );
  }
  공백도 잘 주도록 합시다.
  ```
- 일련의 ? : 를 if 로 리팩토링
  ```jsx
  function Drink({ name }) {
    return (
      <section>
        <h1>{name}</h1>
        <dl>
          <dt>Part of plant</dt>
          <dd>{name === 'tea' ? 'leaf' : 'bean'}</dd>
          <dt>Caffeine content</dt>
          <dd>{name === 'tea' ? '15–70 mg/cup' : '80–185 mg/cup'}</dd>
          <dt>Age</dt>
          <dd>{name === 'tea' ? '4,000+ years' : '1,000+ years'}</dd>
        </dl>
      </section>
    );
  }

  export default function DrinkList() {
    return (
      <div>
        <Drink name="tea" />
        <Drink name="coffee" />
      </div>
    );
  }
  컴포넌트 내부에 분기처리가 너무 많아서 좋지않다. 를 수정해달라고 합니다.
  if문으로 리팩토링한다면,
  그냥 name을 가지고 jsx문 전에 분기처리해서 할당해주고 UI요소로 나타내면 되겠죠?

  function Drink({ name }) {
    let partOfPlant ='leaf',
      CaffeineContent= '15–70 mg/cup',
      age='4,000+ years'
    if(name === 'coffee') {
      partOfPlant = 'bean'
      CaffeineContent ='80–185 mg/cup'
      age = '1,000+ years'
    }
    return (
      <section>
        <h1>{name}</h1>
        <dl>
          <dt>Part of plant</dt>
          <dd>{partOfPlant}</dd>
          <dt>Caffeine content</dt>
          <dd>{CaffeineContent}</dd>
          <dt>Age</dt>
          <dd>{age}</dd>
        </dl>
      </section>
    );
  }

  이걸 더 보기 편하게 한다면 어떻게 할까요?

  Drink.js
  function Drink({ type, part, caffeine, age}) {

    return (
      <section>
        <h1>{type}</h1>
        <dl>
          <dt>Part of plant</dt>
          <dd>{part}</dd>
          <dt>Caffeine content</dt>
          <dd>{caffeine}</dd>
          <dt>Age</dt>
          <dd>{age}</dd>
        </dl>
      </section>
    );
  }

  DrinkList.js
  const drinks = {
    tea: {
  		type: 'tea',
      part: 'leaf',
      caffeine: '15–70 mg/cup',
      age: '4,000+ years'
    },
    coffee: {
  		type: 'coffee',
      part: 'bean',
      caffeine: '80–185 mg/cup',
      age: '1,000+ years'
    }
  };
  export default function DrinkList() {
    return (
      <div>
        <Drink {...drinks.tea}/>
        <Drink {...drinks.coffee}/>
      </div>
    );
  }
  객체로 따로 정리해서 넣어주는 방법
  ```

# 목록 렌더링

filter() 와 map()으로 필처링하고, 컴포넌트 배열로 변환하겠습니다.

## 배열에서 데이터 렌더링하기

```jsx
<ul>
  <li>Creola Katherine Johnson: mathematician</li>
  <li>Mario José Molina-Pasquel Henríquez: chemist</li>
  <li>Mohammad Abdus Salam: physicist</li>
  <li>Percy Lavon Julian: chemist</li>
  <li>Subrahmanyan Chandrasekhar: astrophysicist</li>
</ul>
```

예시를 통해 진행해보겠습니다.

```jsx
//step1 데이터를 배열로 **이동**
const people = [
  "Creola Katherine Johnson: mathematician",
  "Mario José Molina-Pasquel Henríquez: chemist",
  "Mohammad Abdus Salam: physicist",
  "Percy Lavon Julian: chemist",
  "Subrahmanyan Chandrasekhar: astrophysicist",
];

//step2 JSX nodes에 매핑
const listItems = people.map((person) => <li>{person}</li>);

//step3 반환
return <ul>{listItems}</ul>;

//step4 키가 필요합니다 ! ! 뒤에서 다루도록 하겠습니다
```

- 😶 **배열**로 정의된 컴포넌트는 어떻게 렌더링 될 수 있을까요?
  ```jsx
  실제 저 예시의 매핑을 실행해본다면,
  <ul>
  	{[
  	 <li>'Creola Katherine Johnson: mathematician'</li>,
     <li>'Mario José Molina-Pasquel Henríquez: chemist'</li>,
     <li>'Mohammad Abdus Salam: physicist'</li>,
     <li>'Percy Lavon Julian: chemist'</li>,
     <li>'Subrahmanyan Chandrasekhar: astrophysicist'</li>
  ]}
  </ul>
  형태가 될 것 입니다.

  근데 뭔가 [] 안에 있는게 어색합니다.
  저는 처음에 이게 어색해서 join을 해서 오류가 난 경험이 있습니다.
  [Object object]의 형태로 나타나게 됩니다.
  왜그럴까요?
  좀만 더 생각해보면, React.createElement('li',null,'...')의 형태로 실행,
  5개의 객체가 생기고, join이 되면서 묶이면서 각 객체가 문자열이되고 뒤에 ","가 생깁니다.
  그러면서 Object object가 생긴것입니다.

  혹시 저 배열을 하나의 소괄호고 묶는다면 결과가 어떻게 날까요?
  그렇다면 마지막 li태그만 나올것입니다.

  함수 내부를 디버깅하면서 살펴보면,
  function validateChildKeys(node, parentType) {
    if (typeof node !== 'object') {
      return;
    }

    if (isArray(node)) {
      for (var i = 0; i < node.length; i++) {
        var child = node[i];

        if (isValidElement(child)) {
          validateExplicitKey(child, parentType);
        }
      }
    }
    ....
  }
  함수를 통해서 children이 배열인지 확인하는 과정을 거칩니다.

  type ReactNode =
          | ReactElement
          | string
          | number
          | Iterable<ReactNode>
          | ReactPortal
          | boolean
          | null
          | undefined
          | DO_NOT_USE_OR_YOU_WILL_BE_FIRED_EXPERIMENTAL_REACT_NODES[keyof DO_NOT_USE_OR_YOU_WILL_BE_FIRED_EXPERIMENTAL_REACT_NODES];

  function createElement<P extends HTMLAttributes<T>, T extends HTMLElement>(
          type: keyof ReactHTML,
          props?: ClassAttributes<T> & P | null,
          ...children: ReactNode[]): DetailedReactHTMLElement<P, T>;

  createElement을 어떻게 정의해놧는지 살펴보면,
  type을 넣고, props를 객체로 넣고나면 나머지 인자들을 모두 children으로 받는데,
  저희가 값을 배열로 넣게된다면, children으로 Array<ReactNode>가 될 것이고,
  spread를 묶으면 children 은 Array<Array<ReactNode>> 형태가 될 것입니다.
  하지만 ReactNode자체가 Iterable<ReactNode>이기 때문에 괜찮습니다.
  재귀적으로 Iterable임을 체크하기 때문에 내부적으로 배열을 받던, 그냥 하나의 값으로 받던
  돌릴 수 있게 구현해놓지 않았나 싶습니다.
  ```
  - React Children을 만들기 위해 flat하게
    [React Children 과 친해지기 | 카카오엔터테인먼트 FE 기술블로그](https://fe-developers.kakaoent.com/2021/211022-react-children-tip/)
- 내부적으로 customTag인지 아닌지 찾는 부분
  ```tsx
  커스텀 태그일 경우
  function getComponentNameFromType(type) {
    if (type == null) {
      // Host root, text node or just invalid type.
      return null;
    }

    {
      if (typeof type.tag === 'number') {
        error('Received an unexpected object in getComponentNameFromType(). ' + 'This is likely a bug in React. Please file an issue.');
      }
    }

    **if (typeof type === 'function') {
      return type.displayName || type.name || null;
    }//여기에 걸리게된다.**

    **if (typeof type === 'string') {
      return type;
    } //그냥 html및 소문자로 구현시 구현했을때**

    switch (type) {
      case REACT_FRAGMENT_TYPE:
        return 'Fragment';

      case REACT_PORTAL_TYPE:
        return 'Portal';

      case REACT_PROFILER_TYPE:
        return 'Profiler';

      case REACT_STRICT_MODE_TYPE:
        return 'StrictMode';

      case REACT_SUSPENSE_TYPE:
        return 'Suspense';

      case REACT_SUSPENSE_LIST_TYPE:
        return 'SuspenseList';

    }

    if (typeof type === 'object') {
      switch (type.$$typeof) {
        case REACT_CONTEXT_TYPE:
          var context = type;
          return getContextName(context) + '.Consumer';

        case REACT_PROVIDER_TYPE:
          var provider = type;
          return getContextName(provider._context) + '.Provider';

        case REACT_FORWARD_REF_TYPE:
          return getWrappedName(type, type.render, 'ForwardRef');

        case REACT_MEMO_TYPE:
          var outerName = type.displayName || null;

          if (outerName !== null) {
            return outerName;
          }

          return getComponentNameFromType(type.type) || 'Memo';

        case REACT_LAZY_TYPE:
          {
            var lazyComponent = type;
            var payload = lazyComponent._payload;
            var init = lazyComponent._init;

            try {
              return getComponentNameFromType(init(payload));
            } catch (x) {
              return null;
            }
          }

        // eslint-disable-next-line no-fallthrough
      }
    }

    return null;
  }

  function createElement(type, props, rootContainerElement, parentNamespace) {
    var isCustomComponentTag; // We create tags in the namespace of their parent container, except HTML
    // tags get no namespace.

    var ownerDocument = getOwnerDocumentFromRootContainer(rootContainerElement);
    var domElement;
    var namespaceURI = parentNamespace;

    if (namespaceURI === HTML_NAMESPACE) {
      namespaceURI = getIntrinsicNamespace(type);
    }

    if (namespaceURI === HTML_NAMESPACE) {
      {
        isCustomComponentTag = isCustomComponent(type, props); // Should this check be gated by parent namespace? Not sure we want to
        // allow <SVG> or <mATH>.

        if (!isCustomComponentTag && type !== type.toLowerCase()) {
          error('<%s /> is using incorrect casing. ' + 'Use PascalCase for React components, ' + 'or lowercase for HTML elements.', type);
        }
      }

      if (type === 'script') {
        // Create the script via .innerHTML so its "parser-inserted" flag is
        // set to true and it does not execute
        var div = ownerDocument.createElement('div');

        div.innerHTML = '<script><' + '/script>'; // eslint-disable-line
        // This is guaranteed to yield a script element.

        var firstChild = div.firstChild;
        domElement = div.removeChild(firstChild);
      } else if (typeof props.is === 'string') {
        // $FlowIssue `createElement` should be updated for Web Components
        domElement = ownerDocument.createElement(type, {
          is: props.is
        });
      } else {
        // Separate else branch instead of using `props.is || undefined` above because of a Firefox bug.
        // See discussion in https://github.com/facebook/react/pull/6896
        // and discussion in https://bugzilla.mozilla.org/show_bug.cgi?id=1276240
        domElement = ownerDocument.createElement(type); // Normally attributes are assigned in `setInitialDOMProperties`, however the `multiple` and `size`
        // attributes on `select`s needs to be added before `option`s are inserted.
        // This prevents:
        // - a bug where the `select` does not scroll to the correct option because singular
        //  `select` elements automatically pick the first item #13222
        // - a bug where the `select` set the first item as selected despite the `size` attribute #14239
        // See https://github.com/facebook/react/issues/13222
        // and https://github.com/facebook/react/issues/14239

        if (type === 'select') {
          var node = domElement;

          if (props.multiple) {
            node.multiple = true;
          } else if (props.size) {
            // Setting a size greater than 1 causes a select to behave like `multiple=true`, where
            // it is possible that no option is selected.
            //
            // This is only necessary when a select in "single selection mode".
            node.size = props.size;
          }
        }
      }
    } else {
      domElement = ownerDocument.createElementNS(namespaceURI, type);
    }

    **{
      if (namespaceURI === HTML_NAMESPACE) {
        if (!isCustomComponentTag && Object.prototype.toString.call(domElement) === '[object HTMLUnknownElement]' && !hasOwnProperty.call(warnedUnknownTags, type)) {
          warnedUnknownTags[type] = true;

          error('The tag <%s> is unrecognized in this browser. ' + 'If you meant to render a React component, start its name with ' + 'an uppercase letter.', type);
        }
      }
    } // 여기서 upper로 쓰라고 오류를 발생시킴**

    return domElement;
  }
  ```

## 항목 배열 필터링하기

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

이 중에서 profession이 ‘chemist’인 사람만 원한다면,

1. people을 filter
2. filter된 값을 map
3. 반환

```tsx
1;
const chemists = people.filter((person) => person.profession === "chemist");

2;
const listItems = chemists.map((person) => (
  <li>
    <img src={getImageUrl(person)} alt={person.name} />
    <p>
      <b>{person.name}:</b>
      {" " + person.profession + " "}
      known for {person.accomplishment}
    </p>
  </li>
));

3;
return <ul>{listItems}</ul>;
```

- 다른 방법으론 forEach, reduce도 쓸 수 있겠죠?
  ```tsx
  //forEach
  import { people } from './data.js';
  import { getImageUrl } from './utils.js';

  export default function List() {
    const chemists = []
    const others =[]
    const itemBuilder = (person,idx) =>(
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
    people.forEach(person =>{
      if( person.profession === 'chemist') {
        chemists.push(itemBuilder(person))
      } else {
        others.push(itemBuilder(person))
      }
    });

    return <ul>{others}</ul>;
  }

  //reduce

  export default function List() {
    const itemBuilder = (person,idx) =>(
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
    const {chemists,others} = people.reduce((res,person) =>{
      if( person.profession === 'chemist') {
        res.chemists.push(itemBuilder(person))
      } else {
        res.others.push(itemBuilder(person))
      }
      return res
    },{chemists: [],others:[]});

    return <ul>{chemists}</ul>;
  }

  reduce가 깔끔한 맛은 있는 것 갖습니다.
  ```

## 함정 - 화살표함수 return

화살표 함수는 ⇒ 뒤에 표현식을 암시적으로 반환하므로 return문이 필요 없습니다. 중괄호가 올때만 return을 명시적으로 작성해야 합니다.

return이 없다면 무엇을 반환할까요? 아무것도 반환하지 않습니다.

즉 undefined가 반환됩니다.

```tsx
a.map((item) => {
  if (item === 3) {
    return "good";
  }
})[(undefined, undefined, "good", undefined, undefined)];
```

## key로 목록의 항목 순서 유지하기

_`Warning: Each child in a list should have a unique “key” prop.`_

_`경고: 목록의 각 자식에는 고유한 “key” prop이 있어야 합니다.`_

위의 항목들에서 발생한 오류입니다.

각 배열 항목에는 해당 배열의 항목들 사이에서 **고유하게 식별**할 수 있는 문자열 또는 숫자인 key를 부여해야합니다.

ex: `<li key={[person.id](http://person.id/)}>...</li>`

_Note:_ 호출 내부의 JSX 요소에는 항상 key가 필요합니다!!

🤔key는 각 컴포넌트가 **어떤 배열 항목에 해당하는지 React에 알려주어 나중에 매칭**할 수 있도록 합니다. 어떻게?

- 어떻게
  [The mystery of React Element, children, parents and re-renders](https://www.developerway.com/posts/react-elements-children-parents)
  [React Children And Iteration Methods — Smashing Magazine](https://www.smashingmagazine.com/2021/08/react-children-iteration-methods/)
  [React) children prop에 대한 고찰(feat. 렌더링 최적화)](https://velog.io/@2ast/React-children-prop에-대한-고찰feat.-렌더링-최적화)
  children은 부모 component의 props이다. react는 부모에서 자식으로 흘러가는 구조를 갖고있는데 children의 변경이 어떻게 부모에게 영향을 끼칠까? children이 부모의 props이고 이게 변경되면, 자식의

이는 배열 항목이 **이동,삽입,삭제**될 수 있는 경우 중요합니다.

잘 만들어진 **key**는 React가 정확히 무슨 일이 일어났는지 추론하고 DOM 트리를 올바르게 업데이트 하는 데 도움이 됩니다.

**Deep** 목록의 각 항목에 여러 개의 DOM 노드 표시하기

각 항목이 하나가 아니라 여러 개의 DOM 노드를 렌더링 할땐 어떻게할까?

**`<></>`** 로는 키를 줄 수 없어서, `<Fragment >`를 함시다.

## key를 얻을 수 있는 곳

데이터 소스에 따라 서로 다른 key 소스를 제공합니다.

**데이터베이스의 데이터:** db에서 데이터를 가져오는 경우, 고유한 db의 key/ID를 사용할 수 있습니다

**로컬에서 생성된 데이터:** 데이터가 로컬에서 생성되고 유지되는 경우, 패키지를 통해 생성된 아이디를 사용할 수 있습니다.

ex) idx같은걸로 key를 만들때 삭제하거나, 했을때 key값이 바뀌면서 리렌더링을 유발할 수 있습니다.

## Key 규칙

key는 형제간에 고유해야 합니다. 다른 배열의 JSX 노드에는 동일한 key를 사용해도 괜찮습니다.

**key가 변경되지 않아야 합니다.**

## React에 key가 필요한 이유가 무엇일까요?

key가 있어야 고유하게 식별할 수 있습니다.

재정렬로 어떤 항목의 위치가 변경되더라도, 해당 항목이 사라지지 않는 한, **React는 key를 통해 그 항목을 식별할 수 있습니다.**

**함정 idx, Math.random을 쓰지마세요! 컴포넌트는 key를 prop으로 받지않습니다.**

[[React] 배열의 index를 key로 쓰면 안되는 이유](https://medium.com/sjk5766/react-배열의-index를-key로-쓰면-안되는-이유-3ce48b3a18fb)

```tsx
[1,2,3].map((v,i)=> {
}
같은걸로 만든경우 해당 항목이 삭제됐을때, 원래 1이었을 i 가 0 이되면서,
문제가 발생할 수 있습니다.

Math.random()은 같은 값이 나올 확률이 있습니다.
for(let i =0; i<10000;i++){
    if((Math.random()*1000)>>0 == (Math.random()*1000) >>0 ){
        console.log(true)
    }
}
```

## 도전과제

- \***\*목록을 둘로 나누세요\*\***
  ```tsx
  import { people } from "./data.js";
  import { getImageUrl } from "./utils.js";
  //위에서 처리한 reduce를 잘 사용해 마무리해줍시다
  export default function List() {
    const itemBuilder = (person, idx) => (
      <li key={person.id}>
        <img src={getImageUrl(person)} alt={person.name} />
        <p>
          <b>{person.name}:</b>
          {" " + person.profession + " "}
          known for {person.accomplishment}
        </p>
      </li>
    );
    const { chemists, others } = people.reduce(
      (res, person) => {
        if (person.profession === "chemist") {
          res.chemists.push(itemBuilder(person));
        } else {
          res.others.push(itemBuilder(person));
        }
        return res;
      },
      { chemists: [], others: [] }
    );
    return (
      <article>
        <h1>Scientists</h1>
        <h2>chemists</h2>
        <ul>{chemists}</ul>
        <h2>others</h2>
        <ul>{others}</ul>
      </article>
    );
  }
  ```
- \***\*중첩 목록\*\***
  ```tsx
  key를 넣기 위해 Fragment를 사용해 줍시다.
  import { recipes } from './data.js';
  import {Fragment} from 'react'
  export default function RecipeList() {
    const recipeList = recipes.map(recipe =>(
      <Fragment key={recipe.id}>
        <h2>{recipe.id}</h2>
        <ul>
          {recipe.ingredients.map((ingredient,idx) => <li key={idx}>ingreient</li>)}
          </ul>
      </Fragment>
    ))
    return (
      <div>
        <h1>Recipes</h1>
        {recipeList}
      </div>
    );
  }
  ```
- \***\*목록 항목 컴포넌트 추출하기\*\***
  ```tsx
  이건 그냥 Fragmet부분만 떼서 만들고 props만 받으면 되겠죠?
  function Recipe({ id, name, ingredients }) {
    return (
      <div>
        <h2>{name}</h2>
        <ul>
          {ingredients.map(ingredient =>
            <li key={ingredient}>
              {ingredient}
            </li>
          )}
        </ul>
      </div>
    );
  }
  ```
- \***\*구분자가 있는 목록\*\***
  ```tsx
  import { Fragment } from "react";
  const poem = {
    lines: [
      "I write, erase, rewrite",
      "Erase again, and then",
      "A poppy blooms.",
    ],
  };

  export default function Poem() {
    return (
      <article>
        {poem.lines.map((line, i) => (
          <Fragment key={i}>
            <p>{line}</p>
            {i < poem.lines.length - 1 && <hr />}
          </Fragment>
        ))}
      </article>
    );
  }
  ```

# 컴포넌트 순수성 유지

순수함수로 작성하면, 당황스러운 버그와 예측할 수 없는 동작을 피할 수 있습니다!

하지만, 몇가지 규칙이 필요합니다.

## 순수성: 수식으로서의 컴포넌트

- 컴퓨터 과학에서, 순수함수
  A Pure Function is **a function (a block of code) that always returns the same result if the same arguments are passed**
  - **자신의 일에만 신경씁니다.** 호출되기 전에 존재했던 객체나 변수를 변경하지 않습니다.
  - **동일 입력, 동일 출력.** 동일한 입력이 주어지면 항상 동일한 결과를 반환해야 합니다.
  호출되기 전엔 아무런 영향을 끼치지 않고, 같은 값을 받으면, 같은값만 내놓는 것 입니다.
  y = 2x 는 순수함수 입니다.
  2를 넣고 123을 받을 경우는없겠죠? 늘 4만 받을 것입니다.
  ```tsx
  function double(number) {
    return 2 * number;
  }
  ```
  `double`은 순수함수입니다.

**리액트는 작성된 컴포넌트가 모두 순수함수라고 가정합니다.**

```tsx
function Recipe({ drinkers }) {
  return (
    <ol>
      <li>Boil {drinkers} cups of water.</li>
      <li>Add {drinkers} spoons of tea and {0.5 * drinkers} spoons of spice.</li>
      <li>Add {0.5 * drinkers} cups of milk to boil and sugar to taste.</li>
    </ol>
  );
}

export default function App() {
  return (
    <section>
      <h1>Spiced Chai Recipe</h1>
      <h2>For two</h2>
      <Recipe drinkers={2} />
      <h2>For a gathering</h2>
      <Recipe drinkers={4} />
    </section>
  );
}

Recipe 컴포넌트에서 늘 같은 값을 넣으면, 같은 값이 나옵니다.
```

컴포넌트는 레시피

레시피대로 재료를 넣고 요리하면, 늘 같은 요리가 나옵니다

“요리”는 렌더링에 반응하기 위해 제공하는 JSX입니다.

## 사이드 이펙트: 의도하지 (않은) 결과

리액트의 렌더링 프로세스는 **항상 순수**해야합니다!

컴포넌트는 오직 JSX만을 반환해야하고, 렌더링 전에 객체나 변수를 변경해서는 안됩니다.

```tsx
순수하지 않은 컴포넌트
let guest = 0;

function Cup() {
  // Bad: changing a preexisting variable!
  // 나쁨: 기존 변수를 변경합니다!
  guest = guest + 1;
  return <h2>Tea cup for guest #{guest}</h2>;
}

export default function TeaSet() {
  return (
    <>
      <Cup />
      <Cup />
      <Cup />
    </>
  );
}
```

**이 컴포넌트는 호출할떄마다 다른 JSX가 생성됩니다.**
Cup 함수를 3번 실행했는데 늘 다른 값을 return 합니다!

다른 컴포넌트가 guest를 읽으면, 렌더링된 시점에 따라 JSX도 다르게 생성됩니다!

- guest를 prop으로 전달하면 이 컴포넌트를 고칠 수 있습니다.
  ```tsx
  function Cup({ guest }) {
    return <h2>Tea cup for guest #{guest}</h2>;
  }

  export default function TeaSet() {
    return (
      <>
        <Cup guest={1} />
        <Cup guest={2} />
        <Cup guest={3} />
      </>
    );
  }
  ```

일반적으로 컴포넌트가 **특정 순서로 렌더링** 될 것이라고 기대해선 안됩니다. 순수하다면, 순서도 상관 없습니다

**컴포넌트간의 의존하게 하지말고**, “스스로 생각”하게 해야합니다.

렌더링은 컴포넌트가 스스로 JSX를 계산해야 합니다.

- **Deep StrictMode로 순수하지 않은 계산 감지하기**
  React에서 렌더링하는 동안, 읽을 수 있는 입력 3가지
  **props, state, context**는 항상 읽기 전용으로 취급해야합니다.
  사용자 입력에 대한 응답을 무언가 *변경*하려면 변수에 쓰는 대신, **state**를 설정해야 합니다. 컴포넌트가 렌더링되는 동안 기존 변수나 객체를 **절대 변경해서는 안 됩니다.**
  React는 개발 환경에서 각 컴포넌트의 함수를 두 번 호출하는 “**Strict Mode**”를 제공합니다.
  “Strict Mode”는 2번 호출 함으로써, 이러한 규칙을 위반하는 컴포넌트를 찾아내는 데 도움이 됩니다.
  순수함수는 계산을 2번해도 100번해도 달라지지 않아야 하는데,
  Guest는 2번 호출하니 Guest#1은 Guest#2가 됐고 다들 2배가 된 경우를 봤습니다.
  Strict Mode는 production에서는 아무런 영향이 없습니다!
  strict mode를 사용하려면 root컴포넌트를 감싸면 됩니다.
  - Strict mode 예시와 검사내용
    - 예시코드
      ```tsx
      var ReactStrictModeWarnings = {
        recordUnsafeLifecycleWarnings: function (fiber, instance) {},
        flushPendingUnsafeLifecycleWarnings: function () {},
        recordLegacyContextWarning: function (fiber, instance) {},
        flushLegacyContextWarning: function () {},
        discardPendingWarnings: function () {}
      };

      {
        var findStrictRoot = function (fiber) {
          var maybeStrictRoot = null;
          var node = fiber;

          while (node !== null) {
            if (node.mode & StrictLegacyMode) {
              maybeStrictRoot = node;
            }

            node = node.return;
          }

          return maybeStrictRoot;
        };

        var setToSortedString = function (set) {
          var array = [];
          set.forEach(function (value) {
            array.push(value);
          });
          return array.sort().join(', ');
        };

        var pendingComponentWillMountWarnings = [];
        var pendingUNSAFE_ComponentWillMountWarnings = [];
        var pendingComponentWillReceivePropsWarnings = [];
        var pendingUNSAFE_ComponentWillReceivePropsWarnings = [];
        var pendingComponentWillUpdateWarnings = [];
        var pendingUNSAFE_ComponentWillUpdateWarnings = []; // Tracks components we have already warned about.

        var didWarnAboutUnsafeLifecycles = new Set();

        ReactStrictModeWarnings.recordUnsafeLifecycleWarnings = function (fiber, instance) {
          // Dedupe strategy: Warn once per component.
          if (didWarnAboutUnsafeLifecycles.has(fiber.type)) {
            return;
          }

          if (typeof instance.componentWillMount === 'function' && // Don't warn about react-lifecycles-compat polyfilled components.
          instance.componentWillMount.__suppressDeprecationWarning !== true) {
            pendingComponentWillMountWarnings.push(fiber);
          }

          if (fiber.mode & StrictLegacyMode && typeof instance.UNSAFE_componentWillMount === 'function') {
            pendingUNSAFE_ComponentWillMountWarnings.push(fiber);
          }

          if (typeof instance.componentWillReceiveProps === 'function' && instance.componentWillReceiveProps.__suppressDeprecationWarning !== true) {
            pendingComponentWillReceivePropsWarnings.push(fiber);
          }

          if (fiber.mode & StrictLegacyMode && typeof instance.UNSAFE_componentWillReceiveProps === 'function') {
            pendingUNSAFE_ComponentWillReceivePropsWarnings.push(fiber);
          }

          if (typeof instance.componentWillUpdate === 'function' && instance.componentWillUpdate.__suppressDeprecationWarning !== true) {
            pendingComponentWillUpdateWarnings.push(fiber);
          }

          if (fiber.mode & StrictLegacyMode && typeof instance.UNSAFE_componentWillUpdate === 'function') {
            pendingUNSAFE_ComponentWillUpdateWarnings.push(fiber);
          }
        };
      ```
    - 검사내용
      1. **`recordUnsafeLifecycleWarnings(fiber, instance)`**: Strict Mode에서 안전하지 않은 라이프사이클 메서드 사용에 관한 경고 메시지를 기록하는 함수입니다. **`fiber`**는 컴포넌트의 Fiber 노드를 나타내며, **`instance`**는 컴포넌트의 인스턴스입니다.
      2. **`flushPendingUnsafeLifecycleWarnings()`**: 렌더링 중에 기록된 안전하지 않은 라이프사이클 경고 메시지를 처리하고 출력하는 함수입니다.
      3. **`recordLegacyContextWarning(fiber, instance)`**: Strict Mode에서 레거시 컨텍스트 사용에 관한 경고 메시지를 기록하는 함수입니다.
      4. **`flushLegacyContextWarning()`**: 렌더링 중에 기록된 레거시 컨텍스트 경고 메시지를 처리하고 출력하는 함수입니다.
      5. **`discardPendingWarnings()`**: 미처리된 경고 메시지를 폐기하는 함수입니다

## 지역 변이(mutation): 컴포넌트의 작은 비밀

위의 예시에서 컴포넌트가 렌더링 하는 동안 *기존 변수*를 변경하는 것이 문제였습니다. 더 무섭게 보이기 위해 **변이**(mutation)으로 쓰도록 합니다.

**렌더링하는 동안 ‘방금’ 생성한 변수와 객체를 변경하는 것은 완전히 괜찮습니다.**

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

cups 변수나 []배열이 `TeaGathering` 외부에 생성되었다면, 이는 큰 문제가 될 것입니다. 해당 배열에 항목을 밀어 넣음으로 **기존 객체**를 변경하게 될 것이기 떄문입니다.

하지만 `TeaGathering` 내부에서 동일한 렌더링 중에 생성했기 때문에 괜찮습니다. **_지역 변이_**라고 하며 컴포넌트의 작은 비밀과 같습니다.

지역변수와 비슷한 개념이고, 관리할 수 없는 변수를 갖다가 쓰지 말라는 것 같습니다.

## 사이드 이펙트를 일으킬 수 있는 곳

함수형 프로그래밍은 순수성에 크게 의존하지만, 화면 업데이트, 애니메이션 시작, 데이터 변경 같은 **사이드 이펙트**가 일어나며, 렌더링 중에 일어나는 것이 아니라 “부수적”으로 일어나는 일입니다.

- 렌더링
  React에서 렌더링이란, 컴포넌트가 props와 state를 통해 UI를 어떻게 구성할지 컴포넌트에게 **요청**하는 작업을 말한다.
  [https://yceffort.kr/2022/04/deep-dive-in-react-rendering#렌더링이란-무엇인가](https://yceffort.kr/2022/04/deep-dive-in-react-rendering#%EB%A0%8C%EB%8D%94%EB%A7%81%EC%9D%B4%EB%9E%80-%EB%AC%B4%EC%97%87%EC%9D%B8%EA%B0%80)
  1. **가상 DOM 빌드**: 컴포넌트가 렌더링되면 해당 컴포넌트의 가상 DOM 트리가 구성됩니다. 이 트리는 React 엔진에 의해 생성되고 업데이트됩니다.
  2. **상태 및 속성 변경**: 컴포넌트의 상태나 속성(props)가 변경되면 React는 이를 감지하고 새로운 상태나 속성을 사용하여 컴포넌트를 다시 렌더링합니다.
  3. **렌더링 결과 출력**: 컴포넌트가 렌더링되면 그 결과가 실제 화면에 출력됩니다. 이 과정에서 React는 가상 DOM과 실제 DOM 간의 비교를 통해 변경된 부분만 업데이트합니다.
  - 렌더링 프로세스
    렌더링이 일어나는 동안, 리액트는 컴포넌트의 루트에서 시작하여 아래쪽으로 쭉 훑어 보면서, 업데이트가 필요하다고 플래그가 지정되어 있는 모든 컴포넌트를 찾는다. 만약 플래그가 지정되어 있는 컴포넌트를 만난다면, 클래스 컴포넌트의 경우 **`classComponentInstance.render()`**를, 함수형 컴포넌트의 경우 **`FunctionComponent()`**를 호출하고, 렌더링된 결과를 저장한다.
    컴포넌트의 렌더링 결과물은 일반적으로 JSX 문법으로 구성되어 있으며, 이는 js가 컴파일되고 런타임 시점에 **`React.createElement()`**를 호출하여 변환된다. **`createElement`**는 UI 구조를 설명하는 일반적인 JS 객체인 React Element를 리턴한다. 아래 예제를 살펴보자.
    ```jsx
    // 일반적인 jsx문법return <SomeComponent a={42} b="testing">Text here</SomeComponent>// 이것을 호출해서 변환된다.return React.createElement(SomeComponent, {a: 42, b: "testing"}, "Text Here")

    // 호출결과 element를 나타내는 객체로 변환된다.{type: SomeComponent, props: {a: 42, b: "testing"}, children: ["Text Here"]}
    ```
    전체 컴포넌트에서 이러한 렌더링 결과물을 수집하고, 리액트는 새로운 오브젝트 트리 (가상돔이라고 알려져있는)와 비교하며, 실제 DOM을 의도한 출력처럼 보이게 적용해야 하는 모든 변경 사항을 수집한다. 이렇게 비교하고 계산하는 과정을 리액트에서는 **`reconciliation`**이라고 한다.
    그런 다음, 리액트는 계산된 모든 변경사항을 하나의 동기 시퀀스로 DOM에 적용한다.

**React에서 사이드 이펙트는 보통 이벤트 핸들러에 속합니다**

이벤트 핸들러는 사용자가 동작을 수행할때 React가 실행하는 함수

이벤트 핸들러가 컴포넌트 **_내부_**에 정의되어 있지만, 렌더링 도중에는 실행되지 않습니다. **따라서 이벤트 핸들러는 순수할 필요가 없습니다 .**

useEffect의 실행 순서는 렌더링이 끝난 후 이기 때문에, JSX에 이벤트 핸들러를 첨부할 수 있습니다. 하지만 최후의 수단으로 사용해야 합니다.

🤔데이터 패칭해서 핸들러에서 호출하는거 같은걸 얘기하는지 잘 모르겠다..

최대한 렌더링만으로 로직을 표현하고자 노력해보자!

Deep 왜 리액트는 **순수성**을 중요시할까?

- 입력의 변경이 없다면 렌더링 건너뛰기를 통해 성능을 향상 시킬 수 있습니다.
- 컴포넌트를 다른 환경에서 실행할 수 있습니다.
- 렌더링도중 다시 렌더링을 할때, 바로 리렌더를 할 수 있습니다.

## 도전과제

- 고장난 시계 고치기
  ```tsx

  export default function Clock({ time }) {
    let hours = time.getHours();
    if (hours >= 0 && hours <= 6) {
      document.getElementById('time').className = 'night';
    } else {
      document.getElementById('time').className = 'day';
    }
    return (
      <h1 id="time">
        {time.toLocaleTimeString()}
      </h1>
    );
  }

  변경해야할 값은 h1의 class이니까, class를 변경하도록 합시다.
  class말고 className을 써야하겠죠?
  dom 직접 변경은 지양합니다.

  export default function Clock({ time }) {
    let hours = 7 or 5 //넣어보고 테스트
    let className
    if (hours >= 0 && hours <= 6) {
      className = 'night';
    } else {
      className = 'day';
    }
    return (
      <h1 className ={className} id="time">
        {time.toLocaleTimeString()}
      </h1>
    );
  }
  ```
- 깨진 프로필을 고치세요
  ```tsx
  import Panel from './Panel.js';
  import { getImageUrl } from './utils.js';

  let currentPerson;

  export default function Profile({ person }) {
    currentPerson = person;
    return (
      <Panel>
        <Header />
        <Avatar />
      </Panel>
    )
  }

  function Header() {
    return <h1>{currentPerson.name}</h1>;
  }

  function Avatar() {
    return (
      <img
        className="avatar"
        src={getImageUrl(currentPerson)}
        alt={currentPerson.name}
        width={50}
        height={50}
      />
    );
  }

  Avatar와 Header에서 외부의 let currentPerson;을 참조합니다.
  각 함수에서 props로 값을 받고 쓴다면 순수한 함수를 만들것입니다.
  근데 여기서 오히려 좀 신기한건
  안에 있는 값들이 바뀌어있는데, 변경되지 않았다는 것입니다.
  왜그럴까요? 렌더링 되지 않았기 때문입니다.
  하지만, 다시 Panel의 children으로 재 랜더링될때, 바뀐 값으로
  나오기 때문에 바뀐값이 나온 것입니다.

  import Panel from './Panel.js';
  import { getImageUrl } from './utils.js';

  export default function Profile({ person }) {
    return (
      <Panel>
        <Header currentPerson= {person} />
        <Avatar currentPerson= {person} />
      </Panel>
    )
  }

  function Header({currentPerson}) {
    return <h1>{currentPerson.name}</h1>;
  }

  function Avatar({currentPerson}) {
    return (
      <img
        className="avatar"
        src={getImageUrl(currentPerson)}
        alt={currentPerson.name}
        width={50}
        height={50}
      />
    );
  }
  ```
- 깨진 스토리 트레이를 고치세요
  ```tsx
  export default function StoryTray({ stories }) {
    stories.push({
      id: 'create',
      label: 'Create Story'
    });

    return (
      <ul>
        {stories.map(story => (
          <li key={story.id}>
            {story.label}
          </li>
        ))}
      </ul>
    );
  }
  일단 props를 변경하는 것은 불변성을 깨는 행위입니다.
  props로 받은 stories를 push를 하면
  App.js에서 stories의 state가 변경되고, 그러면
  재랜더링이 일어나면서 StoryTray가 실행되면서 순환하게됩니다.
  순수함수가 아니어서 2번씩 늘어나기도 합니다.

  stories를 받아서 얕은복사를 한 뒤 컴포넌트 내에서 쓰도록합니다.

  export default function StoryTray({ stories }) {

    const currentStories = [...stories]

    currentStories.push({
      id: 'create',
      label: 'Create Story'
    });

    return (
      <ul>
        {currentStories.map(story => (
          <li key={story.id}>
            {story.label}
          </li>
        ))}
      </ul>
    );
  }
  ```

# 이벤트에 응답하기

## 이벤트 핸들러 추가하기

아무런 기능이 없는 button에 prop으로 전달하도록 합시다.

```tsx
export default function Button() {
  return <button>I don't do anything</button>;
}
```

1. `Button`컴포넌트 안에 `handleClick`이라는 함수를 선언
2. 해당 함수 내부의 로직을 구현
3. JSX의 `<button>`에 `onClick={handleClick}`를 추가

```tsx
export default function Button() {
  function handleClick() {
    alert("You clicked me!");
  }

  return <button onClick={handleClick}>Click me</button>;
}
```

- 🤔여기서 props로 전달된 onClick은 실제 HTML에서 button에 어떻게 전달되는 걸까요?
  ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/c25eb7fa-fce5-4067-a2c4-bec8874fa401/8a6406b2-981a-4cd0-9c3e-f20d2c0dde3a/Untitled.png)
  확인해보면 아무런 값이 없습니다. (?)
  근데 button.click()으로 실행해보면 잘 동작 합니다!! ?
  ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/c25eb7fa-fce5-4067-a2c4-bec8874fa401/aae4fd6e-1872-4135-ae0c-0f1dc82db29e/Untitled.png)
  왜그럴까요?
  [Web: React의 Event 시스템 내부 구현 자세히 알아보기 (React v18.2.0)](https://medium.com/hcleedev/web-react의-event-시스템-내부-구현-자세히-알아보기-react-v18-2-0-39d59ab45bec)
  js에서는 `addEventListener`를 사용해서 이벤트를 등록하는게 일반적인데, React에서는 그냥 바로 등록하는 것도 이 글을 보면서 생각 해보며 아차 했습니다.
  아래 내용은 윗 글을 거의 복붙한것입니다.
  **Native Event에 대한 Listener는 root**
  유저가 클릭하거나, 커서를 옮기거나 같은 이벤트들을 DOM에서 발생하는 Native Event로 칭한다.
  이 Native Event를 인지하기 위한 Listener는 React로 만들어진 가장 상위의 `<div id="root">` 에 붙게 된다.
  ![1_nn-2oPgWUKcITlTI4rfi6g.webp](https://prod-files-secure.s3.us-west-2.amazonaws.com/c25eb7fa-fce5-4067-a2c4-bec8874fa401/cda0b4b2-5710-4de9-baf0-d01991ba56d8/1_nn-2oPgWUKcITlTI4rfi6g.webp)
  리액트 17 이전에는 `document`에 `Listener`를 붙였지만, 17 부터는 React App 이 돌아가는 root요소에 `Listener` 붙인다.
  root에 붙이는 것은 하위 컴포넌트에 있는 수많은 이벤트 핸들러가아니다.
  예를 들어, `<button onClick={() => console.log('btn')}>` 이런 컴포넌트가 있다 하더라도, root에 `rootNode.addEventListener('click', () => console.log('btn'))` 처럼 붙이는게 아니라는 말이다.
  여기서는 **Native Event를 Listen하고, 그 이벤트에 따라 ‘알맞는 타겟의 핸들러를 찾아서 실행시키는’ 핸들러를 포함한 Lisnener를 root에 등록**한다.
  - 코드 및 설명
    ```tsx
    // https://github.com/facebook/react/blob/v18.2.0/packages/react-dom/src/client/ReactDOM.js
    export function createRoot(
      container: Element | Document | DocumentFragment,
      options?: CreateRootOptions,
    ): RootType {
      // 생략 ...

      const rootContainerElement: Document | Element | DocumentFragment =
        container.nodeType === COMMENT_NODE
          ? (container.parentNode: any)
          : container;
      listenToAllSupportedEvents(rootContainerElement);

      return new ReactDOMRoot(root);
    }

    // https://github.com/facebook/react/blob/v18.2.0/packages/react-dom/src/events/DOMPluginEventSystem.js
    export function listenToAllSupportedEvents(rootContainerElement: EventTarget) {
      if (!(rootContainerElement: any)[listeningMarker]) {
        (rootContainerElement: any)[listeningMarker] = true;
        allNativeEvents.forEach(domEventName => {
          // We handle selectionchange separately because it
          // doesn't bubble and needs to be on the document.
          if (domEventName !== 'selectionchange') {
            if (!nonDelegatedEvents.has(domEventName)) {
              listenToNativeEvent(domEventName, false, rootContainerElement);
            }
            listenToNativeEvent(domEventName, true, rootContainerElement);
          }
        });

        // ... 후략
      }
    }
    view rawcreateRootAndListenToAllSupportedEvents.js hosted with ❤ by GitHub
    ```
    `createRoot` 에서 ReactDOMRoot가 만들어지기 직전에 `listenToAllSupportedEvents` 를 호출하고 있다.
    해당 메서드를 자세히 보면, `allNativeEvents.forEach` 를 통해 모든 종류의 Native Event를 root에 붙이는 작업을 진행한다. `listenToNativeEvent` 는 내부에서 `rootContainerElement` 에다가 `addEventListener` 를 이용해 Listener를 붙이는 역할을 한다.
    여기서 `nonDelegatedEvents` , 즉 버블링이 안되는 이벤트의 경우에는 캡처링 상태에 대한 Listener만 붙이고, 아니면 버블링이 되는 경우에는 버블링에 대한 Listener, 캡처링에 대한 Listener 각각 하나씩 붙인다.
    이 Listener를 붙이는 과정은 `createRoot`라는 이름에서 알 수 있듯 처음 root가 만들어질 때, 즉 앱이 켜질 때 진행된다.

handleClick 함수를 정의하고, 이를 button에 prop으로 전달합니다.

**이벤트 핸들러 함수:**

일반적으로 컴포넌트 안에 정의되고, 해당 태그에 prop으로 전달합니다.

handle로 시작하는 이름 뒤에 이벤트 이름이 옵니다.

(물론 함수명이아 어떻든 동작합니다, 관례나 회사 컨벤션을 따릅니다.)

ex) 익명함수, 화살표 함수

```tsx
<button onClick={function handleClick() {
  alert('You clicked me!');
}}>

<button onClick={() => {
  alert('You clicked me!');
}}>
```

정말 동일할까요?

화살표 함수는 매 렌더링마다 새로운 함수를 생성합니다. 미미하지만, 쌓이면 큰 결과 차이가 있을 수 있습니다.

**함정 : 이벤트 핸들러에 전달되는 함수는 호출이 아니라 전달입니다**

```tsx
<button onClick={handleClick()}>
이 코드는 button의 props를 전달할때 handleClick()의 return값을 전달하겠죠?
물론 return 값이 () => alert('click') 같은 것이라면 동작은 하겠지만 굳이? 싶죠

handleClick함수를 onClick 이벤트 핸들러로 전달합니다.
버튼을 클릭할때만 호출하도록 합니다.

<button onClick={alert('...')}>
이러한 코드는 렌더링 시에 alert()가 호출되고 결과값이 없으니
onClick에 undefined가 전달되겠죠

이벤트 핸들러에는 함수를 전달 합시다
```

## 이벤트 핸들러에서 props 읽기

이벤트 핸들러는 컴포넌트 내부에서 선언되기 때문에 컴포넌트의 props에 접근할 수 있습니다.

```tsx
function AlertButton({ message, children }) {
  return (
    <button onClick={() => alert(message)}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <AlertButton message="Playing!">
        Play Movie
      </AlertButton>
      <AlertButton message="Uploading!">
        Upload Image
      </AlertButton>
    </div>
  );
}
이렇게하면 AlertButton에서 message와 children을 함수에 전달 후
실행하면 잘 전될 되서 사용되는걸 볼 수 있습니다.
```

## 이벤트 핸들러를 props에 전달하기

이벤트 핸들러를 부모에서 받아서 써야할 때, 전달합니다.

```tsx
function Button({ onClick, children }) {
  return (
    <button onClick={onClick}>
      {children}
    </button>
  );
}

function PlayButton({ movieName }) {
  function handlePlayClick() {
    alert(`Playing ${movieName}!`);
  }

  return (
    <Button onClick={handlePlayClick}>
      Play "{movieName}"
    </Button>
  );
}

function UploadButton() {
  return (
    <Button onClick={() => alert('Uploading!')}>
      Upload Image
    </Button>
  );
}

export default function Toolbar() {
  return (
    <div>
      <PlayButton movieName="Kiki's Delivery Service" />
      <UploadButton />
    </div>
  );
}

PlayButton에 props로 movieName을 전달,
PlayButton에서 받은 props를 movieName에 할당 후,
Button 컴포넌트의 props로 children과 onClick에 함수를 전달합니다.

```

## 이벤트 핸들러 props 이름 정하기

button 및 div 같은 기본 제공 컴포넌트는 onClick과 같은 브라우저 이벤트 이름만 지원합니다. 하지만 자체 컴포넌트를 빌드할 때는 이벤트 핸들러 prop의 이름을 원하는 방식으로 지정할 수 있습니다.

관례상 이벤트 핸들러 props는 on으로 시작하는 카멜케이스를 띕니다.

컴포넌트가 여러 상호작용을 지원한다면, 앱별 개념에 따라 이벤트 핸들러 props의 이름을 지정할 수 있습니다.

결론적으로, **기존의 html태그에 전달**할때는, **해당 태그에 맞는 props**만 전달 할 수 있지만, 커**스텀 컴포넌트**는 이름은 **마음대**로 가능하다! 입니다.

```tsx
export default function App() {
  return (
    <Toolbar
      on무비보기={() => alert('Playing!')}
      on영화보기={() => alert('Uploading!')}
    />
  );
}

function Toolbar({ on무비보기, on영화보기 }) {
  return (
    <div>
      <Button on클릭={on무비보기}>
        Play Movie
      </Button>
      <Button on클릭={on영화보기}>
        Upload Image
      </Button>
    </div>
  );
}
이렇게 이름을 막 지어도 상관 없습니다. 물론 한글 쓰지맙시다..!
onClick에만 잘 전달하면 됩니다.

function Button({ on클릭, children }) {
  return (
    <button onClick={on클릭}>
      {children}
    </button>
  );
}

App 컴포넌트는 Toolbar가 함수로 어떤 작업을 할지 알 필요가 없습니다.
onPlayMovie같은걸로 이름을 지어서 함수를 유기적으로 변동시켜,
잘 전달해서 쓸 수 있습니다.
```

**Note 이벤트 핸들러에 적절한 HTML 태그를 사용해야 합니다.**

DOMPluginEventSystem.js를 보니 그냥 시작할 때 전역에서 실행되는 코드가 있다. 여기 있는 `registerEvents` 가 이제 `click` 을 `onClick` 같은 React식 명명법으로 변환할 수 있도록 Map에 매핑해주는 역할을 한다. 예를 들어서 SimpleEventPlugin의 registerEvents에 들어가보자.

- 나중에 체크
  ```tsx
  function isLikelyComponentType(type) {
    {
      switch (typeof type) {
        case "function": {
          // First, deal with classes.
          if (type.prototype != null) {
            if (type.prototype.isReactComponent) {
              // React class.
              return true;
            }

            var ownNames = Object.getOwnPropertyNames(type.prototype);

            if (ownNames.length > 1 || ownNames[0] !== "constructor") {
              // This looks like a class.
              return false;
            } // eslint-disable-next-line no-proto

            if (type.prototype.__proto__ !== Object.prototype) {
              // It has a superclass.
              return false;
            } // Pass through.
            // This looks like a regular function with empty prototype.
          } // For plain functions and arrows, use name as a heuristic.

          var name = type.name || type.displayName;
          return typeof name === "string" && /^[A-Z]/.test(name);
        }

        case "object": {
          if (type != null) {
            switch (getProperty(type, "$$typeof")) {
              case REACT_FORWARD_REF_TYPE:
              case REACT_MEMO_TYPE:
                // Definitely React components.
                return true;

              default:
                return false;
            }
          }

          return false;
        }

        default: {
          return false;
        }
      }
    }
  }
  ```

## 이벤트 전파

[🌐 한눈에 이해하는 이벤트 흐름 제어 (버블링 & 캡처링)](https://inpa.tistory.com/entry/JS-📚-버블링-캡쳐링)

이벤트 핸들러는 컴포넌트에 있을 수 있는 모든 하위 컴포넌트의 이벤트도 포착합니다. 이벤트가 트리 위로 ‘버블’ 또는 ‘전파’ 되는 것을 이벤트 발생한 곳에서 트리 위로 올라간다고 합니다.

```tsx
export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('You clicked on the toolbar!');
    }}>
      <button onClick={() => alert('Playing!')}>
        Play Movie
      </button>
      <button onClick={() => alert('Uploading!')}>
        Upload Image
      </button>
    </div>
  );
}

아래 button을 클릭하면 button과 div의 onClick이 동작됩니다.
toolbar만 누르면, div태그의 onClick만 동작합니다.
```

함정 첨부한 JSX 태그에서만 작동하는 `onScroll`을 제외한 모든 이벤트는 React에서 전파됩니다.

## 전파 중지하기

```tsx
function Button({ onClick, children }) {
  return (
    <button onClick={e => {
      e.stopPropagation();
      onClick();
    }}>
      {children}
    </button>
  );
}

export default function Toolbar() {
  return (
    <div className="Toolbar" onClick={() => {
      alert('You clicked on the toolbar!');
    }}>
      <Button onClick={() => alert('Playing!')}>
        Play Movie
      </Button>
      <Button onClick={() => alert('Uploading!')}>
        Upload Image
      </Button>
    </div>
  );
}
e.stopPropagation();를 통해서 버블링을 막을 수 있습니다.
```

**Deep 캡처 단계 이벤트**

하위 요소에서 전파가 중지된 경우에도 모든 이벤트를 포착해야 하는 경우에 쓰면 됩니다.

이벤트명에 Capture를 쓰면됩니다.

```tsx
<div onClickCapture={() => { /* this runs first | 먼저 실행됨 */ }}>
  <button onClick={e => e.stopPropagation()} />
  <button onClick={e => e.stopPropagation()} />
</div>
아래로 이동하면서 모든 onClickCapture 핸들러를 호출합니다.
클릭한 요소의 onClick 핸들러를 실행합니다.
상위로 이동하면서 모든 onClick 핸들러를 호출합니다.

일반 앱 코드가 아닌 라우터에서 유용합니다.
```

## 전파의 대안으로 핸들러 전달하기

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

부모 onClick 이벤트 핸들러 호출 전에 이 핸들러에 코드를 더 추가할 수 있습니다. 자식 컴포넌트가 **이벤트 처리 + 부모 컴포넌트가 몇가지 추가 동작을 지정**하게 해줍니다.

전파에 의존하고 있고, 핸들러가 실행되고 **왜 실행되는지 추적이 어려운 경우**에 사용하십시오!

## 기본 동작 방지

```tsx
export default function Signup() {
  return (
    <form onSubmit={() => alert("Submitting!")}>
      <input />
      <button>Send</button>
    </form>
  );
}
```

정말 주구장창 쓰는 것 같습니다.

뭔가 함수를 만들때 자동반사적으로 쓰기도하는데, 왜 그럴까요?

기본동작을 막는 행위를 자동반사적으로 쓰는게 참 아이러니합니다.

- 🤔왜 생겼을까?
  초기 웹 페이지에서는 링크를 클릭하거나 폼을 제출할 때 페이지가 새로고침되는 것이 일반적이었습니다. 이는 사용자 경험을 저해할 수 있었고, 특히 `AJAX`와 같은 **비동기 통신 기**술이 개발되면서 더욱 불편해졌습니다. 따라서 이러한 기본 동작을 중단하고 페이지 리로드를 방지하기 위해 **`e.preventDefault()`**가 필요했습니다.

```tsx
export default function Signup() {
  return (
    <form onSubmit={e => {
      e.preventDefault();
      alert('Submitting!');
    }}>
      <input />
      <button>Send</button>
    </form>
  );
}
막지 않으면 새로고침된 페이지가 나옴
```

## 이벤트 핸들러에 부작용이 생길 수 있나요?

물론!

렌더링 함수와 달리 이벤트 핸들러는 **순수할 필요가 없습니다.**

**무언가를 변경하기 좋은 곳입니다.**

먼저 정보를 저장하는 **state**와 함께 씁시다!

## 도전과제

- 1
  ```tsx
  export default function LightSwitch() {
    function handleClick() {
      let bodyStyle = document.body.style;
      if (bodyStyle.backgroundColor === 'black') {
        bodyStyle.backgroundColor = 'white';
      } else {
        bodyStyle.backgroundColor = 'black';
      }
    }

    return (
      <button onClick={handleClick()}>
        Toggle the lights
      </button>
    );
  }
  앗! 함수를 호출하고있습니다!~
  전달해야겠죠?
  button 내부를 수정합시다.
  export default function LightSwitch() {
    function handleClick() {
      let bodyStyle = document.body.style;
      if (bodyStyle.backgroundColor === 'black') {
        bodyStyle.backgroundColor = 'white';
      } else {
        bodyStyle.backgroundColor = 'black';
      }
    }

    return (
      <button onClick={handleClick}>
        Toggle the lights
      </button>
    );
  }
  ```
- 2
  ```tsx
  export default function ColorSwitch({
    onChangeColor
  }) {
    return (
      <button>
        Change color
      </button>
    );
  }

  import { useState } from 'react';
  import ColorSwitch from './ColorSwitch.js';

  export default function App() {
    const [clicks, setClicks] = useState(0);

    function handleClickOutside() {
      setClicks(c => c + 1);
    }

    function getRandomLightColor() {
      let r = 150 + Math.round(100 * Math.random());
      let g = 150 + Math.round(100 * Math.random());
      let b = 150 + Math.round(100 * Math.random());
      return `rgb(${r}, ${g}, ${b})`;
    }

    function handleChangeColor() {
      let bodyStyle = document.body.style;
      bodyStyle.backgroundColor = getRandomLightColor();
    }

    return (
      <div style={{ width: '100%', height: '100%' }} onClick={handleClickOutside}>
        <ColorSwitch onChangeColor={handleChangeColor} />
        <br />
        <br />
        <h2>Clicks on the page: {clicks}</h2>
      </div>
    );
  }

  이 문제는 위에까지 봐야 편안합니다.
  이벤트 버블링이 일어난걸 확인할 수 있습니다.
  버블링을 차단하고, 함수를 실행시켜 줍시다.

  export default function ColorSwitch({
    onChangeColor
  }) {
    return (
      <button onClick={(e)=>{
        e.stopPropagation()
        onChangeColor()
      }}>
        Change color
      </button>
    );
  }
  ```

# State: 컴포넌트의 메모리

컴포넌트는 인터렉션의 결과로 화면의 내용을 변경해야합니다.

ex) “_폼에 입력하면 입력 필드가 업데이트 되고, 이미지 캐러셀에서 ‘다음’을 클릭하면 표시되는 이미지가 변경되어야 하며, ‘구매’를 클릭하면 제품이 장바구니에 담겨야 합니다.”_

컴포넌트는 현재 입력값, 현재 이미지, 장바구니와 같은 것들을 ‘**기억**’해야 합니다. React에서는 이러한 컴포넌트별 메모리를 state라고 부릅니다.

## 일반 변수로 충분하지 않은 경우

```tsx
import { sculptureList } from "./data.js";

export default function Gallery() {
  let index = 0;

  function handleClick() {
    index = index + 1;
    console.log(index);
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

조각상 이미지 렌더링 컴포넌트 입니다. Next 버튼을 클릭하면 index를 1,2 로 변경하며 다음 조각상을 보여주어야 하는데, 변화가 없습니다.

**그러나 작동하지 않습니다.**

동작은 하고 있습니다. index값도 커지고 있지만, 렌더링 되지 않습니다.

- 커지는 index값
  ![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/c25eb7fa-fce5-4067-a2c4-bec8874fa401/dd5a2c9c-975b-439e-8099-a8f3d229713d/Untitled.png)

클릭 이벤트가 index를 업데이트 하지만 **2가지 이유**로 안됩니다.

1. **지역 변수는 렌더링간에 유지되지 않습니다.** 리액트는 두번째로 렌더링할 때 지역 변수에 대한 변경사항은 고려하지 않습니다.
   → 렌더링을 하면, state가 아닌 지역변수는 초기화됩니다.
2. **지역 변수를 변경해도 렌더링을 발동시키지 않습니다.** 리액트는 새로운 컴포넌트를 다시 렌더링해야 한다는 것을 인식하지 못합니다.

컴포넌트를 **새 데이터로 업데이트** 하려면,

1. 렌더링 사이에 데이터 유지
2. 새로운 데이터로 컴포넌트를 렌더링 하도록 리액트를 **Trigger**

**useState**가 이걸 가능하게 합니다.

1. 렌더링 사이에 데이터 유지를 위해 **state 변수**
2. 변수를 업데이트하고 리액트가 다시 렌더링하도록 trigger해주는 **setState**

- 🤔어떻게 이게 가능할까요???
  - 처음 useState
    처음 useState를 봤을때가 생각나는데, 좀 이상하단 생각을 했습니다.const라고 선언을 해놨으면 고정 해놓는다는거 아닌가? 객체나 클로저 개념도 이해하지 못하고 잇었고 그냥 그런가 보다 하고 넘어갔었는데,
    나중에 객체 내부를 밴경하는걸 이해하고, 아 그럼 객체 내부를 수정하고 그 값을 받아오나 싶었는데 const를 console로 찍어보면 그냥 값이 나옵니다.
    아 그냥 객체로 구현된 값 useState로 [value,setValue]형태를 만들어 뿌리는 건가보다 했습니다.
    그러다가 이전에 준일님 블로그를 보고 useState내부구현 하신걸 보고 좀 감탄한 기억이 납니다.
    바닐라js 로 useState구현하기인데 참 좋은 자료라고 생각합니다.
    [Vanilla Javascript로 React UseState Hook 만들기 | 개발자 황준일](https://junilhwang.github.io/TIL/Javascript/Design/Vanilla-JS-Make-useSate-hook/#_4-모듈화)
  https://goidle.github.io/react/in-depth-react-hooks_1/
  useState를 타고 들어가면
  ```tsx

  function useState(initialState) {
    var dispatcher = resolveDispatcher();
    return dispatcher.useState(initialState);
  }
  resolveDispatcher를 따라가보면,
  function resolveDispatcher() {
    var dispatcher = ReactCurrentDispatcher.current;

    {
      if (dispatcher === null) {
        error('Invalid hook call. Hooks can only be called inside of the body of a function component. This could happen for' + ' one of the following reasons:\n' + '1. You might have mismatching versions of React and the renderer (such as React DOM)\n' + '2. You might be breaking the Rules of Hooks\n' + '3. You might have more than one copy of React in the same app\n' + 'See https://reactjs.org/link/invalid-hook-call for tips about how to debug and fix this problem.');
      }
    } // Will result in a null access error if accessed outside render phase. We
    // intentionally don't throw our own error because this is in a hot path.
    // Also helps ensure this is inlined.

    return dispatcher;
  }
  ReactCurrentDispatcher는
  var ReactCurrentDispatcher = {
    /**
     * @internal
     * @type {ReactComponent}
     */
    current: null
  };
  const ReactSharedInternals = {
    ReactCurrentDispatcher,
    ReactCurrentCache,
    ReactCurrentBatchConfig,
    ReactCurrentOwner,
  };
  이걸 요약해보면, 저희가 쓰는 useState는
  ReactSharedInternals에서 관리하는
  ReactCurrentDispatcher객체에 current 프로퍼티 입니다.

  ReactSharedInternals는 ReactSharedInternals.js에 있습니다.
  ReactCurrentDispatcher.current를 찾아봅시다.

  ReactCurrentDispatcher$1 = ReactSharedInternals.ReactCurrentDispatcher
  function renderWithHooks(current, workInProgress, Component, props, secondArg, nextRenderLanes) {
  ...중략...
    {
      if (current !== null && current.memoizedState !== null) {
        ReactCurrentDispatcher$1.current = HooksDispatcherOnUpdateInDEV;
      } else if (hookTypesDev !== null) {
        // This dispatcher handles an edge case where a component is updating,
        // but no stateful hooks have been used.
        // We want to match the production code behavior (which will use HooksDispatcherOnMount),
        // but with the extra DEV validation to ensure hooks ordering hasn't changed.
        // This dispatcher does that.
        ReactCurrentDispatcher$1.current = HooksDispatcherOnMountWithHookTypesInDEV;
      } else {
        ReactCurrentDispatcher$1.current = HooksDispatcherOnMountInDEV;
      }
    }
  ...중략...
    }

  }
  currnet,memoizedState가 없을때 HooksDispatcherOnUpdateInDEV를 넣고
  hookTypesDev가 없을땐  HooksDispatcherOnMountWithHookTypesInDEV를 넣는다
  HooksDispatcherOnMountWithHookTypesInDEV ={
  ...중략
  useState: function (initialState) {
        currentHookNameInDev = 'useState';
        updateHookTypesDev();
        var prevDispatcher = ReactCurrentDispatcher$1.current;
        ReactCurrentDispatcher$1.current = InvalidNestedHooksDispatcherOnMountInDEV;

        try {
          return mountState(initialState);
        } finally {
          ReactCurrentDispatcher$1.current = prevDispatcher;
        }
      },
  ...중략...
  }

  HooksDispatcherOnUpdateInDEV = {
      ...중략
      useEffect: function (create, deps) {
        currentHookNameInDev = 'useEffect';
        updateHookTypesDev();
        return updateEffect(create, deps);
      },
      useState: function (initialState) {
        currentHookNameInDev = 'useState';
        updateHookTypesDev();
        var prevDispatcher = ReactCurrentDispatcher$1.current;
        ReactCurrentDispatcher$1.current = InvalidNestedHooksDispatcherOnUpdateInDEV;

        try {
          return updateState(initialState);
        } finally {
          ReactCurrentDispatcher$1.current = prevDispatcher;
        }
      }
  ...중략...
      unstable_isNewReconciler: enableNewReconciler
    };

  function updateHookTypesDev() {
    {
      var hookName = currentHookNameInDev;

      if (hookTypesDev !== null) {
        hookTypesUpdateIndexDev++;

        if (hookTypesDev[hookTypesUpdateIndexDev] !== hookName) {
          warnOnHookMismatchInDev(hookName);
        }
      }
    }
  }

  function mountState(initialState) {
    var hook = mountWorkInProgressHook();

    if (typeof initialState === 'function') {
      // $FlowFixMe: Flow doesn't like mixed types
      initialState = initialState();
    }

    hook.memoizedState = hook.baseState = initialState;
    var queue = {
      pending: null,
      interleaved: null,
      lanes: NoLanes,
      dispatch: null,
      lastRenderedReducer: basicStateReducer,
      lastRenderedState: initialState
    };
    hook.queue = queue;
    var dispatch = queue.dispatch = dispatchSetState.bind(null, currentlyRenderingFiber$1, queue);
    return [hook.memoizedState, dispatch];
  }

  mountState를 살펴보면 저희가 찾던 배열이 나옵니다.
  hook.meomizedState, dispatch가 나오게됩니다.

  hook은
  function mountWorkInProgressHook() {
    var hook = {
      memoizedState: null,
      baseState: null,
      baseQueue: null,
      queue: null,
      next: null
    };

    if (workInProgressHook === null) {
      // This is the first hook in the list
      currentlyRenderingFiber$1.memoizedState = workInProgressHook = hook;
    } else {
      // Append to the end of the list
      workInProgressHook = workInProgressHook.next = hook;
    }

    return workInProgressHook;
  }
  의 결과입니다.
  memoizedState, baseState, baseQueue 등 값을 갖고있습니다.
  만약 진행되고 있는 hook이 있다면, next값을 넣어서 쓰는 링크드리스트입니다.

  결국 useState는 hook의 memoizedState입니다.
  렌더링때 최적화를 위해 쓰는 것 같습니다.
  ```

## 첫 번째 훅 만나기

React에서 use가 붙으면 훅이라고 부릅니다.

훅은 React가 **렌더링** 중일 때만 사용할 수 있는 함수입니다.

이게 참 중요합니다.

**함정 훅은 “컴포넌트 최상위 레벨” 또는 커스텀 훅에서만 호출할 수 있습니다.**

조건문, 반복문 또는 기타 중첩된 함수 내부에서는 훅을 호출할 수 없습니다. → 순수함수를 추구하기 때문일까요? 렌더링 도중이라면, 이벤트 함수가 호출되기 전 순수함수를 추구할 때 입니다. 근데 조건문을 사용하면, 늘 같은 결과를 기대할 수 없어 그런게 아닐까요? 렌더링 순서를 정확하게 하고싶은 리액트에서는 조건문이나 반복문에서 사용되는 훅은 부정확한 렌더링을 유발하고 이는 문제를 일으킬 수 있습니다.

## useState 해부하기

useState를 호출하는 것은, React에게 이 컴포넌트가 **무언가를 기억하기를 원한다**고 말하는 것입니다.

`const [index, setIndex] = useState(0);`

이 경우 React가 index를 기억하기 원합니다.

**Note: useState의 이름은 [명사, set명사]가 국룰입니다.**

useState의 유일한 인수는 state변수의 **초기값**입니다.

컴포넌트가 렌더링 될 떄마다 useState는 2개의 값을 포함하는 배열을 제공합니다.

1. **저장한 값**을 가진 state 변수
2. **state변수를 업데이트**하고 React의 **렌더링을 촉발**하는 state 설정자 함수

`const [index, setIndex] = useState(0);`

실제 작동 방식은

1. **컴포넌트가 처음 렌더링** [0, setIndex] 가 useState값으로 반환됩니다.
2. **state를 업데이트 합니다.** 사용자가 setIndex(값)을 실행시키면, index를 변환시키고, **렌더링을 촉발**합니다.
3. **컴포넌트가 2번째로 렌더링됩니다.** React가 여전히 useState(0)을 바라보지만, index를 1로 설정한 것을 기억하기 때문에 [1, setIndex]를 반환합니다.
4. 이런식의 반복입니다.

## 컴포넌트에 여러 state 변수 지정하기

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

하나의 컴포넌트에 원하는 만큼 유형의 state변수를 가질 수 있습니다.

이 예쩨에서 index, showMore처럼 연관이 없는 경우 여러개의 state변수를 갖도록 합니다.

하나의 값이 동시에 움직이면, 객체로 묶어서 사용합시다.

**Deep React는 어떤 state를 반환할지 어떻게 알 수 있을까요?**

useState 호출이 어떤 state 변수를 참조하는지에 대한 정보를 받지 못합니다. **useState에 전달되는 “식별자”가 없는데 어떻게 state 변수를 반환할지 알 수 있을까요?**

간결한 구문을 구현하기 위해 **훅은 동일한 컴포넌트의 모든 렌더링에서 안정적인 호출 순서에의존합니다.**

위의 규칙을 따르면, 훅은 항상 같은 순서로 호출되기 떄문에 실제로 잘 작동합니다.

내부적으로 React는 모든 컴포넌트에대해 한 쌍의 state 배열을 가집니다. 또한 렌더링 전에 0으로 설정된 쌍 인덱스를 유지합니다.

- custom으로 useState만들기
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
  useState의 동작을 보면, customHooks에 currentHookIndex을 1올리고
  pair값이 있다면, pair전달, 아니면 pair에 값과 setState를 넣습니다.
  setState내부에는 state값 변경과 DOM 변경을 갖고있습니다.
  그 값을 컴포넌트Hooks내부에 넣고 index를 1 올려
  다음 호출에선 index값으로 호출하면 없는 값이 나오며 새로 만들게ㅐ됩니다.

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
      more: `${showMore ? 'Hide' : 'Show'} details`,
      description: showMore ? sculpture.description : null,
      imageSrc: sculpture.url,
      imageAlt: sculpture.alt
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
      description.style.display = '';
    } else {
      description.style.display = 'none';
    }
  }

  let nextButton = document.getElementById('nextButton');
  let header = document.getElementById('header');
  let moreButton = document.getElementById('moreButton');
  let description = document.getElementById('description');
  let image = document.getElementById('image');
  let sculptureList = [{
    name: 'Homenaje a la Neurocirugía',
    artist: 'Marta Colvin Andrade',
    description: 'Although Colvin is predominantly known for abstract themes that allude to pre-Hispanic symbols, this gigantic sculpture, an homage to neurosurgery, is one of her most recognizable public art pieces.',
    url: 'https://i.imgur.com/Mx7dA2Y.jpg',
    alt: 'A bronze statue of two crossed hands delicately holding a human brain in their fingertips.'
  }, {
    name: 'Floralis Genérica',
    artist: 'Eduardo Catalano',
    description: 'This enormous (75 ft. or 23m) silver flower is located in Buenos Aires. It is designed to move, closing its petals in the evening or when strong winds blow and opening them in the morning.',
    url: 'https://i.imgur.com/ZF6s192m.jpg',
    alt: 'A gigantic metallic flower sculpture with reflective mirror-like petals and strong stamens.'
  }, {
    name: 'Eternal Presence',
    artist: 'John Woodrow Wilson',
    description: 'Wilson was known for his preoccupation with equality, social justice, as well as the essential and spiritual qualities of humankind. This massive (7ft. or 2,13m) bronze represents what he described as "a symbolic Black presence infused with a sense of universal humanity."',
    url: 'https://i.imgur.com/aTtVpES.jpg',
    alt: 'The sculpture depicting a human head seems ever-present and solemn. It radiates calm and serenity.'
  }, {
    name: 'Moai',
    artist: 'Unknown Artist',
    description: 'Located on the Easter Island, there are 1,000 moai, or extant monumental statues, created by the early Rapa Nui people, which some believe represented deified ancestors.',
    url: 'https://i.imgur.com/RCwLEoQm.jpg',
    alt: 'Three monumental stone busts with the heads that are disproportionately large with somber faces.'
  }, {
    name: 'Blue Nana',
    artist: 'Niki de Saint Phalle',
    description: 'The Nanas are triumphant creatures, symbols of femininity and maternity. Initially, Saint Phalle used fabric and found objects for the Nanas, and later on introduced polyester to achieve a more vibrant effect.',
    url: 'https://i.imgur.com/Sd1AgUOm.jpg',
    alt: 'A large mosaic sculpture of a whimsical dancing female figure in a colorful costume emanating joy.'
  }, {
    name: 'Ultimate Form',
    artist: 'Barbara Hepworth',
    description: 'This abstract bronze sculpture is a part of The Family of Man series located at Yorkshire Sculpture Park. Hepworth chose not to create literal representations of the world but developed abstract forms inspired by people and landscapes.',
    url: 'https://i.imgur.com/2heNQDcm.jpg',
    alt: 'A tall sculpture made of three elements stacked on each other reminding of a human figure.'
  }, {
    name: 'Cavaliere',
    artist: 'Lamidi Olonade Fakeye',
    description: "Descended from four generations of woodcarvers, Fakeye's work blended traditional and contemporary Yoruba themes.",
    url: 'https://i.imgur.com/wIdGuZwm.png',
    alt: 'An intricate wood sculpture of a warrior with a focused face on a horse adorned with patterns.'
  }, {
    name: 'Big Bellies',
    artist: 'Alina Szapocznikow',
    description: "Szapocznikow is known for her sculptures of the fragmented body as a metaphor for the fragility and impermanence of youth and beauty. This sculpture depicts two very realistic large bellies stacked on top of each other, each around five feet (1,5m) tall.",
    url: 'https://i.imgur.com/AlHTAdDm.jpg',
    alt: 'The sculpture reminds a cascade of folds, quite different from bellies in classical sculptures.'
  }, {
    name: 'Terracotta Army',
    artist: 'Unknown Artist',
    description: 'The Terracotta Army is a collection of terracotta sculptures depicting the armies of Qin Shi Huang, the first Emperor of China. The army consisted of more than 8,000 soldiers, 130 chariots with 520 horses, and 150 cavalry horses.',
    url: 'https://i.imgur.com/HMFmH6m.jpg',
    alt: '12 terracotta sculptures of solemn warriors, each with a unique facial expression and armor.'
  }, {
    name: 'Lunar Landscape',
    artist: 'Louise Nevelson',
    description: 'Nevelson was known for scavenging objects from New York City debris, which she would later assemble into monumental constructions. In this one, she used disparate parts like a bedpost, juggling pin, and seat fragment, nailing and gluing them into boxes that reflect the influence of Cubism’s geometric abstraction of space and form.',
    url: 'https://i.imgur.com/rN7hY6om.jpg',
    alt: 'A black matte sculpture where the individual elements are initially indistinguishable.'
  }, {
    name: 'Aureole',
    artist: 'Ranjani Shettar',
    description: 'Shettar merges the traditional and the modern, the natural and the industrial. Her art focuses on the relationship between man and nature. Her work was described as compelling both abstractly and figuratively, gravity defying, and a "fine synthesis of unlikely materials."',
    url: 'https://i.imgur.com/okTpbHhm.jpg',
    alt: 'A pale wire-like sculpture mounted on concrete wall and descending on the floor. It appears light.'
  }, {
    name: 'Hippos',
    artist: 'Taipei Zoo',
    description: 'The Taipei Zoo commissioned a Hippo Square featuring submerged hippos at play.',
    url: 'https://i.imgur.com/6o5Vuyu.jpg',
    alt: 'A group of bronze hippo sculptures emerging from the sett sidewalk as if they were swimming.'
  }];

  // Make UI match the initial state.
  updateDOM();
  ```

## state는 격리되고 프라이빗합니다.

state는 화면의 컴포넌트 인스턴스에 지역적입니다.

즉, 동일한 컴포넌트를 두군데에서 렌더링하면, 각 **사본은 완전히 격리된 state를 갖게 됩니다.**

컴포넌트를 몇번 호출해도 내부에 호출된 useState값은 다른 값이고, 영향을 서로 미치지 않습니다.

- 예시
  ```tsx
  import Gallery from "./Gallery.js";

  export default function Page() {
    return (
      <div className="Page">
        <Gallery />
        <Gallery />
      </div>
    );
  }
  Gallery.js;
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
      <section>
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
      </section>
    );
  }
  ```

이것이 일반 변수와 state의 차이입낟. state는 특정 함수 호출에 묶이지 않고, 코드의 특정 위치에 묶이지도 않지만, 화면상의 특정 지역에 **지역적** 입니다.

Page 컴포넌트는 Gallery의 state에 대해 전혀 모릅니다. props와 달리, **state는 이를 선언하는 컴포넌트 외에는 완전히 비공개입니다. 부모 컴포넌트는 이를 변경할 수 없습니다.**

이 두개의 state를 동기화 하려면 어떻게 해야할까요??

자식에서 부모로 올리면 되겠죠?? 그에 아니라면, context API를 쓰는것도 방법이 될 수있습니다.

## 도전과제

- 갤러리 완성하기
  ```tsx
  import { useState } from 'react';
  import { sculptureList } from './data.js';

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
        <button onClick={handleNextClick}>
          Next
        </button>
        <h2>
          <i>{sculpture.name} </i>
          by {sculpture.artist}
        </h2>
        <h3>
          ({index + 1} of {sculptureList.length})
        </h3>
        <button onClick={handleMoreClick}>
          {showMore ? 'Hide' : 'Show'} details
        </button>
        {showMore && <p>{sculpture.description}</p>}
        <img
          src={sculpture.url}
          alt={sculpture.alt}
        />
      </>
    );
  }
  내부적으로 Next를 막으면 될 것 같습니다.
  이전은 Prev를 만들면 될 것 같습니다.
  Next에서 list의 마지막 다음으로 넘어가는걸 막읍시다.
  undefined를 쓰면서 에러가 나겠죠?

  import { useState } from 'react';
  import { sculptureList } from './data.js';

  export default function Gallery() {
    const [index, setIndex] = useState(0);
    const [showMore, setShowMore] = useState(false);
    const hasPrev = (page) => (page >0)
    const hasNext = (page) => (page<sculptureList.length-1)
    function handlPrevClick() {
      console.log(index)
      setIndex(index-1);
    }
    function handleNextClick() {

      setIndex(index+ 1);
    }

    function handleMoreClick() {
      setShowMore(!showMore);
    }

    let sculpture = sculptureList[index];
    return (
      <>
        <button disabled={!hasPrev(index)} onClick={()=> handlPrevClick(index)}>
          Prev
        </button>
        <button disabled={!hasNext(index)} onClick={()=>handleNextClick(index)}>
          Next
        </button>
        <h2>
          <i>{sculpture.name} </i>
          by {sculpture.artist}
        </h2>
        <h3>
          ({index + 1} of {sculptureList.length})
        </h3>
        <button onClick={handleMoreClick}>
          {showMore ? 'Hide' : 'Show'} details
        </button>
        {showMore && <p>{sculpture.description}</p>}
        <img
          src={sculpture.url}
          alt={sculpture.alt}
        />
      </>
    );
  }

  ```
- input 입력 불가 문제 해결
  ```tsx
  export default function Form() {
    let firstName = '';
    let lastName = '';

    function handleFirstNameChange(e) {
      firstName = e.target.value;
    }

    function handleLastNameChange(e) {
      lastName = e.target.value;
    }

    function handleReset() {
      firstName = '';
      lastName = '';
    }

    return (
      <form onSubmit={e => e.preventDefault()}>
        <input
          placeholder="First name"
          value={firstName}
          onChange={handleFirstNameChange}
        />
        <input
          placeholder="Last name"
          value={lastName}
          onChange={handleLastNameChange}
        />
        <h1>Hi, {firstName} {lastName}</h1>
        <button onClick={handleReset}>Reset</button>
      </form>
    );
  }
  렌더링이 안되는 문제입니다.
  변경이 일어나도, 변화를 렌더링 하지 않으니 아무 변화가 없어보입니다.

  import {useState} from 'react'
  export default function Form() {
    const [firstName, setFirstName] = useState('')
    const [lastName, setLastName] = useState('')

    function handleFirstNameChange(e) {
      setFirstName(e.target.value)

    }

    function handleLastNameChange(e) {
      setLastName(e.target.value)

    }
  ...중략
  }
  으로 수정합시다.
  ```
- 충돌고치기
  ```tsx
  import { useState } from 'react';

  export default function FeedbackForm() {
    const [isSent, setIsSent] = useState(false);
    if (isSent) {
      return <h1>Thank you!</h1>;
    } else {
      // eslint-disable-next-line
      const [message, setMessage] = useState('');
      return (
        <form onSubmit={e => {
          e.preventDefault();
          alert(`Sending: "${message}"`);
          setIsSent(true);
        }}>
          <textarea
            placeholder="Message"
            value={message}
            onChange={e => setMessage(e.target.value)}
          />
          <br />
          <button type="submit">Send</button>
        </form>
      );
    }
  }
  훅은 최상단입니다!
  const [message, setMessage] = useState('');을 밖으로 뺍시다.

  ```
- 불필요한 state 제거하기
  ```tsx
  import { useState } from 'react';

  export default function FeedbackForm() {
    const [name, setName] = useState('');

    function handleClick() {
      setName(prompt('What is your name?'));
      alert(`Hello, ${name}!`);
    }

    return (
      <button onClick={handleClick}>
        Greet
      </button>
    );
  }
  state가 불필요하니 삭제하고
  handleClick을 수정합시다.

  import { useState } from 'react';

  export default function FeedbackForm() {

    function handleClick() {
      alert(prompt('What is your name?'));
    }

    return (
      <button onClick={handleClick}>
        Greet
      </button>
    );
  }

  여기서 왜 state가 필요하지 않을까요?
  렌더링을 요구하지 않기 떄문입니다.
  값이 변동되고 렌더링을 요하지 않고 어떠한 props로 넘기는 값도
  아닙니다.

  ```
