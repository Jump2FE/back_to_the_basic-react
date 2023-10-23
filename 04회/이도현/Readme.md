[4화] Adding Interactivity (상호작용 추가하기)

## 랜더링하고 커밋하기
- step 1 . 촉발!!
  - 렌더링이 일어나는데는 두가지 이유가 있다.
  1. 컴포넌트의 초기 렌더링인 경우
     - 대상 DOM 노드로 createRood를 호출 후 render메서드를 호출한다
     
  2. 컴포넌트의 state (또는 상위요소중의 하나)가 업데이트된 경우
     - state를 업데이트하면 set함수로 업데이트하여 추가렌더링을 일으킨다.(업데이트할때마다!!)

  추가. 부모에서 자식한테 prop이 없을때는?
     - **렌더링은 되지만** 바뀌는 데이터가 없으니 그대로 있는다

- step 2. 컴포넌트 렌더링
  - 촉발 후 React는 **컴포넌트를 호출**하여 화면에 표시할 내용을 파악
  - 이 과정은 재귀적으로 일어나는데
     1. 업데이트된 컴포넌트가 다른 컴포넌트를 반한 
     2. React는 다른 컴포넌트 렌더링
  - 예제는 간단해서.. 패스
    - 첫 렌더링을 하는동안 각 태그에 대한 DOM노드(html요소)를 생성
### 함정
- 렌더링은 항상 순수한 계산이어야 합니다.
  - 동일한 입력에는 동일한 출력을 나와야 한다.
  - 이전의 state를 변경해서는 안됩니다. 렌더링 전에 존재했던 객체나 변수를 변경해서는 안됩니다.
     - (이 경우는 코드의 복잡해짐을 겪는 경우가 너무 많았어서 꼭 지켜야 될거 같아요)
### 성능 최적화 (state 관리 챕터에서 깊게 다룰 예정)
- npm build를 통해 성능 최적화를 할수 있지만 성급하게 해서는 안된다 (다 따져보고 해야하는데 다음챕터에서 언급)

- step 3. React가 DOM에 변경사항을 커밋
 ```js
  export default function Clock({ time }) {
  return (
  <>
  <h1>{time}</h1>
  <input />
  </>
  );
  }
  ```
타임의 초가 변경될때마다 렌더링이 되는 모습을 확인할수 있다.
이 경우 컴포넌트를 각자 사용하면 될거 같긴 하지만 더 좋은 경우가 있을지 확인해보자

##  스냅샷으로서의 state
- 스냅샷 (한번찍은 정보에 대해 쭉 사용한다 (변경하기 전까지))
1. state를 설정하면 렌더리이 실행됨

- 인터페이스가 이벤트(불린 값에 의하던지 일정 숫자,클릭)에 반응하려면 state를 업데이트해야 합니다.
  - 렌더링은 함수를 호출한다는 뜻인데 모든 변수(prop,이벤트,로컬변수)는 렌더링 당시의 state를 사용합니다.
      ```js
      import { useState } from 'react';
    
      export default function Counter() {
      const [number, setNumber] = useState(0);
    
     return (
    <>
    <h1>{number}</h1>
    <button onClick={() => {
    setNumber(number + 1);
       setNumber(number + 1);
     setNumber(number + 1);
     }}>+3</button>
     </>
     )}
    //3개의 큐가 생성이 되지만 각각 set값에 들어갈때는 렌더링 초기의 값 (0)에 들어가므로 결과적으로 1이 출력된다.
    //strict모드에서는 2번 , 없이는 1번 렌더링 되는것을 확인할수 있다.
     ```
    - 하지만 시점을 미뤄도 그대로 number의 값이 0으로 표시가 될까?
       - 아니다! 지정된 state는 버튼을 클릭했을때 변경이 되었을수 있지만 보여지는 변수의 값은 예약되어있는 값을 출력하기 때문이다.
       - state변수의 값은 이벤트핸들러의 코드가 비동기적이라도 렌더링내에서 절대 변경되지 않습니다!!
       - 렌더링 내에서 변수를 변경하고 싶다면 다음 챕터인 state업데이트를 공부하자!
    - 이벤트 헨들러의 state관계는 렌더링된 jsx와의 관계와 동일하다고 볼수 있다.(위의 jsx문 참고)
      - 렌더링시점에서의 state변수와 jsx문 안에서의 state변수와 같다는 말

- 문제 1 
  - alert의 위치가 크게 의미가 없는 이유는 snapshot의 영향인데 바뀐 state변수의 값은 예약어 일뿐 현재 상태에서는 어떠한 영향도 끼치지 않기 때문이다.


##  여러 state 업데이트를 큐에 담기
- state변수를 여러번 업데이트 하고 싶다면 전 챕터의 "number+1"이 아닌 아래의 업데이트 함수를 이용하자
  - 아래의 방법은 number의 값을 예약하며 개별적으로 처리되는게 아닌 일괄적으로 데이터를 저장하여 처리 
  - 따라서 업데이트 함수는 "다음 state값"을 전달하는 것이 아닌 "state값을 계산하는 함수"를 "지시"하는 방법입니다.
    - 전달하는 것은 다음 전달값이 초기화 되며 값을 출력하는 대신
    - 업데이트 함수는 큐를 순회하여 업데이트되는 state값을 제공하는것이 다릅니다.
  
```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(n => n + 1);
        setNumber(n => n + 1);
        setNumber(n => n + 1);
      }}>+3</button>
    </>
  )
}
// 
```
```js
import { useState } from 'react';

export default function Counter() {
  const [number, setNumber] = useState(0);

  return (
    <>
      <h1>{number}</h1>
      <button onClick={() => {
        setNumber(number + 5);
        setNumber(n => n + 1);
      }}>Increase the number</button>
    </>
  )
}
//위와 같은 방법은 number에 저장된 state값을 함수에 전달하여 6이 나오는것을 확인할수 잇는데
// "number+5"와 "n=>n+1"의 순서를 바꾼다면 함수로 지시한 값은 초기화 되고 "1"만 출력되는 것을 확인할수 있습니다.
//이는 jsx문에서 위에서 아래로 순회한다는 것을 보여준다.
```
#### 문제 2
- for of 문과 forEach문을 사용할수 있는데 배열안의 값을 나열하여 타입비교를 통해 배치시킬수 있다.
   - 갠적으로 for문보다는 forEach문이 더 간결해 보여 좋을거 같습니다. 복사보다는 그대로 출력시키는게 좋아보입니다.

## 객체 state 업데이트
- state는 객체를 포함해서 어떠한 js값이든 저장할수 있는데!
  - state에 있는 객체를 직접 변이(mutation)해서는 안됩니다.
  - 대신 객체를 업데이트하려면 새 객체를 생성또는 복사를 통해 state를 업데이트 해야합니다.**중요**
- js객체의 값에는 number,boolean,string,symbol,null,undefined의 기본값이 있는데 이러한 종류의 값은 읽기전용의 값입니다.
  - 하지만 렌더링을 촉발하여? 값을 바꿀수 있습니다(업데이트? 복사?)

```jsx
const [x, setX) = useState(0);

setX(5);
//위와 같이 선언된 x의 값이 0에서 5로 변경은 되었지만 0(디폴트여서 인지? 예약어라서?)자체는 변경되지 않습니다. 

const [position,setposition] = useState({x:0,y:0});

position.x = 5;  // 이건 변이! 왜냐 초기값을 변경하기 때문에 //순수성을 지켜야 한다.
```
```js
import { useState } from 'react';
export default function MovingDot() {
  const [position, setPosition] = useState({
    x: 0,
    y: 0
  });
  return (
    <div
      onPointerMove={e => {
        position.x = e.clientX;
        position.y = e.clientY;
      }}
      style={{
        position: 'relative',
        width: '100vw',
        height: '100vh',
      }}>
      <div style={{
        position: 'absolute',
        backgroundColor: 'red',
        borderRadius: '50%',
        transform: `translate(${position.x}px, ${position.y}px)`,
        left: -10,
        top: -10,
        width: 20,
        height: 20,
      }} />
    </div>
  );
}
//위의 jsx문은 원이 커서를 따라다니는 문인데 position의 값이 계속 초기화가 되며 초기값을 표출하기 때문에 원이 고정되어 있는 것처럼 보인다.
//또 state함수에 할당되지 않는다면 react는 객체가 변이되었다는 사실을 모르기 때문이다.
//리렌더링을 촉발하려면 새 객체를 생성하거나 복사하여 state함수에 전달하자

// onPointerMove={e => {
//   setPosition({
//     x: e.clientX,
//     y: e.clientY
//   });
// }}
// 위와 같은 방법으로 말이다!!
//이 경우는 기존 객체의 state를 수정하는게 아닌 지역적으로 객체를 생성하여 변이한것이기 때문에 괜찮습니다!이것을 지역변이라고 합니다
```
### 또 다른 방법으로는 전개구문을 사용하여 객체를 복사하는 방법이 있습니다.
```js
import { useState } from 'react';

export default function Form() {
  const [person, setPerson] = useState({
    firstName: 'Barbara',
    lastName: 'Hepworth',
    email: 'bhepworth@sculpture.com'
  });

  function handleFirstNameChange(e) {
    person.firstName = e.target.value;
  }

  function handleLastNameChange(e) {
    person.lastName = e.target.value;
  }

  function handleEmailChange(e) {
    person.email = e.target.value;
  }

  return (
    <>
      <label>
        First name:
        <input
          value={person.firstName}
          onChange={handleFirstNameChange}
        />
      </label>
      <label>
        Last name:
        <input
          value={person.lastName}
          onChange={handleLastNameChange}
        />
      </label>
      <label>
        Email:
        <input
          value={person.email}
          onChange={handleEmailChange}
        />
      </label>
      <p>
        {person.firstName}{' '}
        {person.lastName}{' '}
        ({person.email})
      </p>
    </>
  );
}
// 위의 문 역시 기존의 State값을 변경하여 react가 인식하지 못하고 있습니다.

setPerson({
  ...person,
  firstName: e.target.value 
});
//하지만 전개구문을 사용하여 복사하고 firstname객체만 덮어씌운다면 react가 인식할것입니다.
//주의할 점은 전개구문은 얕은 복사라는 점!! 한단계 깊은 복사를 하고 싶다면 두번 이상 사용해야 합니다.
```
- 더 간단하게 하려면 중괄호를 사용하여 동적이름을 가진 프로퍼티를 지정할수 있다.
```js
  function handleChange(e) {
    setPerson({
      ...person,
      [e.target.name]: e.target.value
    });
  }
// 예시
// <label>
//   First name:
//   <input
//           name="firstName"
//           value={person.firstName}
//           onChange={handleChange}
//   />
// </label>
```
### 중복된 객체 업데이트 하기
위와 같은 방법으로 변경점이 있는 객체를 단일 이벤트 헨들러를 사용하면 더 간단히 적용시킬수 있다.
- 여기서 e.targer.name은  input tag의 name속성을 가리킨다.

더 간단히 해보자면 immer라이브러리가 있다.
- immer는 proxy라는 개체로 사용자가 수행하는 작업을 기록합니다. 그래서 뭐가 됐든 draft를 사용하여 편집내용이 포함된 모든 객체를 새로운객체로 생성된다.
```js
export default function Form() {
  const [person, updatePerson] = useImmer({
    name: 'Niki de Saint Phalle',
    artwork: {
      title: 'Blue Nana',
      city: 'Hamburg',
      image: 'https://i.imgur.com/Sd1AgUOm.jpg',
    }
  });

  function handleNameChange(e) {
    updatePerson(draft => {
      draft.name = e.target.value;
    });
  }
  function handleImageChange(e) {
    updatePerson(draft => {
      draft.artwork.image = e.target.value;
    });
  }
  return (
          <>
            <label>
              Name:
              <input
                      value={person.name}
                      onChange={handleNameChange}
              />
            </label>
            <img
                    src={person.artwork.image}
                    alt={person.artwork.title}
            />
          </>
  )};
//이러한 방법으로 깊이에 상관없이 image에 해당되는 값을 변경한다.
//immer는 redux에 사용된다고 하는데 사용되는 코드의 양이 방대해져서 그런게 아닐까 라는 생각이 드는데... 그 외에는... 언제 사용이 되는지 잘 모르겠다
```
[문제1](https://codesandbox.io/s/little-fast-tvkygl?file=/App.js)
[문제2](https://codesandbox.io/s/broken-dust-wslqtv?file=/App.js) 
[문제3](https://codesandbox.io/s/hardcore-mahavira-47k3td?file=/App.js)



## 배열 state 업데이트
- 배열연산은 배열을 직접 변이하는 메서드 보다는 기존배열 복사후 생성하는 메서드를 사용하자
  1. 추가: concat [...arr]
  2. 삭제: filter, slice (배열또는 배열의 일부를 복사)
  3. 교체: map 
  4. 정렬: 배열 복사후 다음 처리

- 배열 변경하기
  - map 함수를 이용하여 새로운 배열을 만듭니다.  ar[0] = 'bird' 와 같은 할당은 원래 배열을 변이하는 것으로
  이 경우에 map함수를 통하여 변경하도록 하자

문제1
```js
import { useState } from 'react';

const initialProducts = [{
  id: 0,
  name: 'Baklava',
  count: 1,
}, {
  id: 1,
  name: 'Cheese',
  count: 5,
}, {
  id: 2,
  name: 'Spaghetti',
  count: 2,
}];

export default function ShoppingCart() {
  const [
    products,
    setProducts
  ] = useState(initialProducts)

  function handleIncreaseClick(productId) {
    setProducts(products.map(product => {
      if (product.id === productId) {
        return {
          ...product,
          count: product.count + 1
        };
      } else {
        return product;
      }
    }))
  }
  //맵함수를 통해 배열을 재생성하고 id값을 비교 하고 맞으면 +1 클릴할때마다 전체 렌더링

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          {product.name}
          {' '}
          (<b>{product.count}</b>)
          <button onClick={() => {
            handleIncreaseClick(product.id);
          }}>
            +
          </button>
        </li>
      ))}
    </ul>
  );
}

```

문제2
```js

import { useState } from 'react';

const initialProducts = [{
  id: 0,
  name: 'Baklava',
  count: 1,
}, {
  id: 1,
  name: 'Cheese',
  count: 5,
}, {
  id: 2,
  name: 'Spaghetti',
  count: 2,
}];

export default function ShoppingCart() {
  const [
    products,
    setProducts
  ] = useState(initialProducts)

  function handleIncreaseClick(productId) {
    setProducts(products.map(product => {
      if (product.id === productId) {
        return {
          ...product,
          count: product.count + 1
        };
      } else {
        return product;
      }
    }))
  }
  function handleDecreaseClick(productId) {
    let nextProducts = products.map(product => {
      if (product.id === productId) {
        return {
          ...product,
          count: product.count - 1
        };
      } else {
        return product;
      }
    });
    nextProducts = nextProducts.filter(p =>
      p.count > 0
    );
    setProducts(nextProducts)
  }

  return (
    <ul>
      {products.map(product => (
        <li key={product.id}>
          {product.name}
          {' '}
          (<b>{product.count}</b>)
          <button onClick={() => {
            handleIncreaseClick(product.id);
          }}>
            +
          </button>
         <button onClick={() => {
            handleDecreaseClick(product.id);
          }}>
            –
          </button>
        </li>
      ))}
    </ul>
  );
}
//필터를 사용하여 0이상인 count값만 렌더링한다.
```