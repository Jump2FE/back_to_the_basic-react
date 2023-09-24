[7화] UI표현하기 ~ 상호작용 더하기 전반부 

### 이펙트가 필요하지 않을수 있습니다.

#### ✅ 불필요한 Effect 처리
1. 렌더링을 위해 데이터를 변환하는 경우 
   - popst나 useState에 따라 바뀌는 state에서 계산할수 있는 값은 state에 넣지 말고
   - 대신 랜더링 중에 계산하세요
- 간단한 계산 
- 
```js
function Form() {
  const [firstName, setFirstName] = useState('Taylor');
  const [lastName, setLastName] = useState('Swift');

  // 🔴 이러지 마세요: 중복 state 및 불필요한 Effect
  const [fullName, setFullName] = useState('');
  useEffect(() => {
    setFullName(firstName + ' ' + lastName);
  }, [firstName, lastName]);
  // ...
    //대신 
    // ✅ 좋습니다: 렌더링 과정 중에 계산
    const fullName = firstName + ' ' + lastName;
}
```
- 복잡한 계산?
```js
function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  // ✅ getFilteredTodos()가 느리지 않다면 괜찮습니다.
  const visibleTodos = getFilteredTodos(todos, filter);
}
// 하지만 이 getfilteredTodes함수가 느리거나 갖고있는 값이 많을경우 캐싱화를 생각해보자
```
```js
import { useMemo, useState } from 'react';

function TodoList({ todos, filter }) {
  const [newTodo, setNewTodo] = useState('');
  const visibleTodos = useMemo(() => {
    // ✅ todos나 filter가 변하지 않는 한 재실행되지 않음
      // memo는 다음렌더링때도 값을 가지고 있기 때문에(캐시에 저장)
      // 의존성에 속한 값이 변할경우에만 실행이 됩니다! 그외는 기존에 저장된 데이터를 활용해요
    return getFilteredTodos(todos, filter);
  }, [todos, filter]);
  // Or 한줄로 표현하면
    const visibleTodos = useMemo(() => 
            getFilteredTodos(todos, filter), [todos, filter]);
}
```
- 하지만 이 방법은 전체시간이 1ms이상이라면 memo를 사용하는 것을 고려해봐요
- 왜냐면 memo는 첫렌더링시 기록단축에 도움이 되지 않고 재렌더링 시에만 도움이 되기 때문이에요
- 
#### ✅ prop이 변경되면 모든 state재설정 하기
```js
export default function ProfilePage({ userId }) {
  const [comment, setComment] = useState('');

  // 🔴 이러지 마세요: prop 변경시 Effect에서 state 재설정 수행
  useEffect(() => {
    setComment('');
  }, [userId]);
  // ...
}
// 이것은 ProfilePage와 그 자식들이 먼저 오래된 값으로 렌더링한 다음 새로운 값으로 다시 렌더링하기 때문에 비효율적입니다.
//또 state가 있는 모든 컴포넌트에서 이 작업을 수행해야 하므로 복잡하죠!

```
- 그 대신 key를 전달하여 정확한 자식 컴포넌트에게 전달할수 있습니다.
```js
export default function ProfilePage({ userId }) {
  return (
    <Profile
      userId={userId}
      key={userId}
    />
  );
}

function Profile({ userId }) {
  // ✅ This and any other state below will reset on key change automatically
  // ✅ key가 변하면 이 컴포넌트 및 모든 자식 컴포넌트의 state가 자동으로 재설정됨
  const [comment, setComment] = useState('');
  // ...
}
```
#### ✅ props가 변경될때 일부 state 조정하기
- 하지만 이러한 경우보다는 렌더링 과정에서 계산을 하는것이 좋습니다.
- 굳이 한다면 key로 모든 state를 재설정하거나 렌더링중 모두 계산할수 있는지 고려해보자
- 그렇지 않고 써야 한다면 무한루프를 어떻게 해결할건지(조건식으로던지?) 고려하고 써야한다.
- 일부 or 전체의 state및 props 조정은 할수 있겠지만 컴포넌트의 순수성을 유지하는것이 좋으므로 지양하는것이 좋은데ㅎㅎ;; 고려만이라도 해보자

#### 이벤트 핸들러간 로직 공유
- Effect안에 특정 이벤트 로직 존재
- 이런경우 페이지를 새로고침할때마다 effect가 실행되기 때문에 Effect내부가 아닌 
컴포넌트를 따로 빼서 사용자에게 필요한 경우에만 사용되도록 사용해야 합니다.
```js
function ProductPage({ product, addToCart }) {
  // 🔴 이러지 마세요: Effect 내부에 특정 이벤트에 대한 로직 존재
  useEffect(() => {
    if (product.isInCart) {
      showNotification(`Added ${product.name} to the shopping cart!`);
    }
  }, [product]);

  function handleBuyClick() {
    addToCart(product);
  }

  function handleCheckoutClick() {
    addToCart(product);
    navigateTo('/checkout');
  }
  // ...
}
function ProductPage({ product, addToCart }) {
   // ✅ 좋습니다: 이벤트 핸들러 안에서 각 이벤트별 로직 호출
   function buyProduct() {
      addToCart(product);
      showNotification(`Added ${product.name} to the shopping cart!`);
   }

   function handleBuyClick() {
      buyProduct();
   }

   function handleCheckoutClick() {
      buyProduct();
      navigateTo('/checkout');
   }
  
   //이렇게 하면 새로고침할때마다 가 아닌 필요한 경우에만 알람을 띄울수 있습니다.
}
```
#### POST요청 보내기

- 버튼을 눌르는 post요청도 마찬가지로 제출을 클릭하였을때 요청을 보내므로 
이벤트행핸들러를 사용하여 코드문을 작성합니다.

#### 연쇄 계산
- 이 경우는 state값에 따라서 또 다른 state값을 조정하고 싶을때가 있는데 이경우도 effect가 아닌 
이벤트 헨들러에서 적용하는것이 좋습니다. 
   - 또 비효율적입니다. 간단한 계산같은 경우 체인의 각 값의 호출 사이에 렌더링을 다시 해야하기 때문에 비효율 적이죠!
   - 공식문서의 코드문경우엔 무려 3번이나 렌더링을 해야죠 
- 하지만 드롭다운의 선택값의 따라 다음 드롭다운의 옵션이 달라지는 form을 작성한다 했을때!
이는 effect체인이 적절해 보입니다. 왜냐 선택되는 값에 따라 결과가 달리지기 때문이죠

#### 어플리케이션 초기화 하기
- 일부 로직은 앱이 로드될때 한번만 실행되어야 할때도 있는데 그경우엔 Effect가 아닌 최상위 변수로 선언을 사용해보죠
- Effect로 최상위 컴포넌트에 배치할 경우 개발중에는 두번 실행되는것을 확인할수 있습니다.
- 두번 실행될경우 예기치 못한 토큰무효화 라든지의 문제가 발생할수 있기 때문에 지양합시다
- 하지만 prd환경같은 경우엔 1번 실행되어 다시 마운트 되지 않을수도 있지만 그래도! 동일한 제약조건을 따르면 코드를 이용하고 재사용하기 편하기 때문에 사용하지 말죠!
   - window변수의 경우엔 nextjs를 다루는 환경에서는 다르니 사용에 주의하죠


#### state변경을 부모컴포넌트에 알리기
```js
function Toggle({ onChange }) {
  const [isOn, setIsOn] = useState(false);

  // 🔴 Avoid: The onChange handler runs too late
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

 
}
```
- 이 경우엔 사용자의 동작(클릭,드래그)에 의해서 이뤄지는 코드문이기에 Effect는 맞지 않습니다.또 늦습니다.
- 그래서 각 코드들은 이벤트 핸들러에서 작성합시다.

```js
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

  // 이렇게 ison과 onchange를 같이 관리하여 손쉽게 코드문을 이해할수 있게 해줄수도 있습니다.
}
```
#### 부모에게 데이터 전달하기
- 이경우엔 부모 -> 자식 으로 데이터흐름이 이어지고 있는데 자식에서 데이터를 페치하지말고 미리 부모에서 해당 데이터를 패치하고 전달하도록 하면
- 데이터흐름이 깔끔해지고 정보의 출처도 한눈에 알아볼수 있다 (자식꺼는 이리저리 찾아봐야 하니까! 자식은 전달만 받자!)

#### 외부 스토어 구독하기
- 외부에서 가져오는 데이터api의 경우 react가 모르는 사이 변경될수 있는데 그럴땐 컴포넌트에서 수동으로 데이터를 구독하도록 해야합니다.
```js
function useOnlineStatus() {
  // Not ideal: Manual store subscription in an Effect
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
- 이 경우엔 윈도우가 online일 경우에 값을 저장하라 라는 코드문인데 useEffect로 하기보다는 react에서 제공하는 훅인 useSyncExternalStore이걸 사용하자
- 요것은 훅을 공부할때 더 자세히 해보자

#### 데이터 패칭하기

```js
function SearchResults({ query }) {
  const [results, setResults] = useState([]);
  const [page, setPage] = useState(1);

  useEffect(() => {
    // 🔴 이러지 마세요: 클린업 없이 fetch 수행
    fetchResults(query, page).then(json => {
      setResults(json);
    });
  }, [query, page]);

  function handleNextPageClick() {
    setPage(page + 1);
  }
  // ...
}
```
- 이 경우 페이징의 값에따라 변경되는 페칭과 query에대한 패칭은 ?? 을 나타내는 구문입니다.
- 하지만 클린업이 없다는 점이 잘못된 점입니다.
- 그래서 ignore문을 넣어(클린없) 해결해보죠 
- 클린업을 하는 이유는 더이상 관련없는 페치가 앱에 계속 영향을 미치지 않아야 하기 때문이죠!! effect와 동기화 하기 참조!!

*경쟁상태*
- 요건 서로 다른 요청이 서로 영향을 미치는 것을 의미합니다.  패치에서는 패칭으로 불러온 값이 계속 남아있어 앱에 영향을 주는 것을 의미할수 도 있죠

## 정리

1. 렌더링중에 계산은 effect가 필요 x
2. 비용이 많이 드는(시간이 소요되는) 계산을 저장하려면 usememo하자
3. 전체 컴포넌트의 state변경은 key를 넣어 정확하게 수행하자
4. 컴포넌트에 표시되는 코드는 effect 나머지는 이벤트핸들러!!!
5. 자식컴포넌트에서 데이터를 발생시키는게 아닌 부모에서 데이터를 발생시켜 자식한테 주자
6. 부모컴포넌트에서 state를 끌어올리면 부모컴포넌트에서 더 많은 로직을 포함해야 하지만 전체적으로 걱정할 state가 줄어든다! 
    - 서로다른 state를 동기화 하려면 부모쪽에서 끌어올리기를 해보자
7. effect에서 데이터를 fetch할수 있지만 경쟁조건을 피하기 위해 클립업(ignore)를 구현해야죠