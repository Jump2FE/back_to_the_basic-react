
# [8회차] EscapeHatches(3)(이벤트와 Effect 분리하기 ~ 커스텀 훅으로 로직 재사용하기)
>링크 : https://woozy-stool-6eb.notion.site/8-EscapeHatches-3-Effect-d4d1d253a49b48ccb3c08b1eea270c36?pvs=4

# **Separating Events from Effects**

> **이벤트와 Effect 분리하기**
> 

**이벤트 핸들러** : 같은 상호작용을 다시 수행할 때만 재실행 된다.

**Effect** : prop 또는 state 변수 값이 마지막 렌더링 때와 달라지면 다시 동기화 된다.

때로는 일부 값의 변화에 따라 재실행되는 Effect와 그렇지 않은 Effect의 혼합이 필요할 때도 있다.

<aside>
💡 학습목표

- 이벤트 핸들러와 Effect 중에서 선택하는 방법
- Effect는 반응형이고 이벤트 핸들러는 반응형이 아닌 이유
- Effect 코드의 일부가 반응하지 않기를 원할 때 해야 할 일
- Effect Event란 무엇이며 Effect에서 추출하는 방법
- Effect Event를 사용하여 Effect에서 최신 props 및 state를 읽는 방법
</aside>

## **Choosing between event handlers and Effects**

> **이벤트 핸들러와 Effect 중 선택하기**
> 

이벤트 핸들러와 Effects 중 어느 것을 사용해야할지 결정할 때는 [코드가 실행되어야하는 *이유*](https://react-ko.dev/learn/synchronizing-with-effects#what-are-effects-and-how-are-they-different-from-events)를 생각해야 한다.

### **Event handlers run in response to specific interactions**

> **이벤트 핸들러는 특정 상호 작용에 대한 응답으로 실행됩니다.**
> 

사용자의 반응에 따라 무언가 동작을 해야 한다면 이벤트 핸들러를 사용한다.

### **Effects run whenever synchronization is needed**

> **Effect는 동기화가 필요할 때마다 실행됩니다**
> 

코드 실행의 이유가 특정 상호작용이 아닐 경우 Effect를 사용한다.

## **Reactive values and reactive logic**

> **반응형 값 및 반응형 로직**
> 

이벤트 핸들러는 사용자의 동작에 의해 ‘수동’으로 촉발되고 Effect는 코드의 목적과 실행이유에 의해 ‘자동’으로 동기화 상태를 유지한다.

컴포넌트 내부에는 리렌더링으로 인해 값이 변경될 수 있는 props, state, 변수와 같은 반응형 값이 존재한다. 

- **이벤트 핸들러 내부의 로직은 반응형이 아니다.**
    
    사용자가 동일한 상호작용을 수행하지 않으면 다시 실행되지 않으며 변경에 반응하지 않고 반응 값을 읽을 수 있다.
    
- **Effect 내부의 로직은 반응형이다.**
    
    Effect에서 반응형 값을 읽는 경우 의존성으로 지정해야한다. 이 후 리렌더링으로 인해 해당 반응형 값이 변경되면 변경된 값으로 Effect 로직을 다시 실행한다.
    

### **Logic inside event handlers is not reactive**

> **이벤트 핸들러 내부의 로직은 반응형이 아닙니다**
> 

이벤트 핸들러의 내부 코드는 반응형 값이 변경 되었다는 이유만으로 다시 실행 되서는 안된다. 

### **Logic inside Effects is reactive**

> **Effect 내부의 로직은 반응형입니다**
> 

반응형 값의 변경에 따라 재실행하길 원하는 코드는 Effect에 속한다.

## **Extracting non-reactive logic out of Effects**

> **Effect에서 비반응형 로직 추출하기**
> 

반응형 로직과 비반응형 로직을 함께 사용하기

**e.g., 사용자가 채팅에 연결할 때 알람을 표시하는 기능**

props 에서 현재 테마(dark or light)를 읽어 테마에 맞는 색상으로 알람을 표시하려고 한다.

```jsx
**function ChatRoom({ roomId, theme }) {**
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      **showNotification('Connected!', theme);**
    });
    connection.connect();
    return () => {
      connection.disconnect()
    };
  **}, [roomId, theme]);  //** Effect에서 읽는 모든 반응형 값은 의존성으로 선언해야 한다.
  // ...
```

Effect에서 읽는 모든 반응형 값은 의존성으로 선언해야 한다. theme 역시 리렌더링의 결과로 변경될 수 있는 반응형 값이다.

https://codesandbox.io/embed/friendly-banach-vxw9h1?fontsize=14&hidenavigation=1&theme=dark

공식문서의 예시 코드를 확인해보면 roomId가 변경될 때마다 채팅이 다시 연결될 뿐만 아니라, theme 역시 반응형 값이므로 테마를 전환할 때도 채팅이 다시 연결되며 불필요한 알람이 보인다.

```jsx
	// ...
  showNotification('Connected!', theme);
  // ...
```

다시 말해, 위 코드는 반응형 Effect 안에 있어도 반응형으로 동작하길 원하지 않는다.

### **Declaring an Effect Event**

> **Effect Event 선언하기**
> 

<aside>
⚠️ **이 섹션에서는 아직 안정되지 않은 API 에 대해 설명한다.**

</aside>

`**useEffectEvent` Hook를 사용하면 비반응형 로직을 Effect에서 추출**할 수 있다.

```jsx
import { useEffect, useEffectEvent } from 'react';

function ChatRoom({ roomId, theme }) {
  const onConnected = **useEffectEvent**(() => {
    showNotification('Connected!', theme);
  });
  // ...
```

onConnect는 Effect Event가 되고, Effect 로직의 일부지만 이벤트 핸들러처럼 동작한다. Effect Event의 내부 로직은 반응형으로 동작하지 않고 항상 porps 와 state의 최신 값을 확인하다. 

```jsx
function ChatRoom({ roomId, theme }) {
  **const onConnected = useEffectEvent(() => {
    showNotification('Connected!', theme);
  });**

  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.on('connected', () => {
      **onConnected();**
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); 
  // ...
```

Effect Event로 작성된 코드는 더 이상 반응형 이벤트가 아니므로 의존 배열에서 제거해줘야 한다. 

**Effect Event와 이벤트 핸들러의 차이**

이벤트 핸들러는 사용자 상호작용에 대한 응답으로 실행된다. 반면 Effect Event는 Effect에서 사용자가 촉발한다. 

# **Reading latest props and state with Effect Events**

> **Effect Event로 최신 props 및 state 읽기**
> 

<aside>
⚠️ **이 섹션에서는 아직 안정되지 않은 API 에 대해 설명한다.**

</aside>

장바구니에 있는 품목의 수에 대한 정보를 포함하여 페이지 방문 기록을 하기 위한 코드를 작성한다고 가정하자.

```jsx
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  useEffect(() => {
    **logVisit(url, numberOfItems);**
  }, [url]); // 🔴 React Hook useEffect has a missing dependency: 'numberOfItems'
  // ...
}
```

Effect 내부에서 numberOfItems 를 사용했기 때문에 Linter는 이를 의존성 배열에 추가하라고 요청한다. 하지만 logVisit 호출은 numberOfItems 에 대해 반응하길 원하지 않는다. 

페이지를 방문 했을 때 방문 기록과 함께 장바구니 품목의 수를 저장하는게 목적이지, 장바구니 품목의 수가 변경 되었다고 해서 방문 기록을 남겨서는 안된다. 즉 logVisit 호출이 numberOfItems에 대해 반응하길 원하지 않는다. 페이지 방문은 어떤 의미에서 ‘이벤트’, 사용자의 상호작용으로 볼 수 있다. 

```jsx
function Page({ url }) {
  const { items } = useContext(ShoppingCartContext);
  const numberOfItems = items.length;

  **const onVisit = useEffectEvent(visitedUrl => {**
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    **onVisit(url);**
  }, [url]); // ✅ All dependencies declared
  // ...
}
```

⇒ url이 변경될 때마다 logVisit을 호출하고 항상 최신 numberOfItems를 읽게 된다. 그러나 numberOfItems가 변경된다고 해서 logVisit 이 호출되지는 않는다. 

<aside>
💡 **Note**

매개변수 없이 onVisit() 를 호출하고 그 안에 있는 url을 읽을 수 있을까?

```jsx
...
const onVisit = useEffectEvent(() => {
    logVisit(url, numberOfItems);
  });

  useEffect(() => {
    onVisit();
  }, [url]);
...
```

동작은 하지만 url을 Effect Event에 명시적으로 전달하는 것이 좋다. 

```jsx
...
const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    onVisit(url);
  }, [url]);
...
```

Effect Event에 **인자로 url을 전달하면 다른 url을 가진 페이지를 방문하는 것이 사용자 관점에서 별도의 ‘이벤트’를 구성한다는 의미**이다.

이는 Effect 내부에 비동기 로직이 있는 경우 특히 중요하다. 

```jsx
...
const onVisit = useEffectEvent(visitedUrl => {
    logVisit(visitedUrl, numberOfItems);
  });

  useEffect(() => {
    setTimeout(() => {
      onVisit(url);
    }, 5000); // Delay logging visits
  }, [url]);
...
```

`onVisit` 내부의 `url`은 (이미 변경되었을 수 있는) 최신 `url`에 해당하지만 `visitedUrl`은 원래 이 Effect(및 `onVisit` 호출)를 실행하게 만든 `url`에 해당하기 때문이다.

</aside>

## **Limitations of Effect Events**

> **Effect Event의 제한사항**
> 

<aside>
⚠️ **이 섹션에서는 아직 안정되지 않은 API 에 대해 설명한다.**

</aside>

**Effect Event 를 사용할 수 있는 방법은 매우 제한적이다.**

- **Effect 내부에서만 호출할 수 있습니다.**
- **다른 컴포넌트나 Hook에 전달하지 마세요.**

Effect Event는 Effect 코드의 비반응형 “조각”이다. Effect Event는 이를 사용하는 Effect 옆에 있어야 **한다.**

# **Removing Effect Dependencies**

> **Effect 의존성 제거하기**
> 

linter는 Effect의 의존성 배열에 Effect가 사용하는 모든 반응형 값을 포함하도록 하는데, 이렇게 하면 Effect가 최신 props 및 state와 동기화 상태를 유지할 수 있다. 반면 불필요한 의존성으로 인해 Effect가 너무 자주 실행되거나 무한 루프를 생성할 수도 있다. 

<aside>
💡 **학습목표**

- Effect 의존성 무한 루프를 수정하는 방법
- 의존성을 제거하고자 할 때 해야 할 일
- Effect에 “반응”하지 않고 Effect에서 값을 읽는 방법
- 객체와 함수 의존성을 피하는 방법과 이유
- 의존성 린터를 억제하는 것이 위험한 이유와 대신 할 수 있는 일
</aside>

## **Dependencies should match the code**

> **의존성은 코드와 일치해야 합니다**
> 

Effect를 작성할 때는 원하는 작업을 시작하고 중지하는 방법을 지정해야 하며 의존성 배열이 비어있다면 린터가 올바른 의존성을 제안한다.

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  useEffect(() => {
    const connection = createConnection(serverUrl, roomId);
    connection.connect();
    return () => connection.disconnect();
  	// ...
}
```

## **To remove a dependency, prove that it’s not a dependency**

> **의존성을 제거하려면 의존성이 아님을 증명하세요**
> 

Effect의 의존성은 선택할 수 없다. Effect에서 사용되는 모든 반응형 값은 의존성 목록에 선언되어야 한다. 

의존성을 제거하려면 해당 컴포넌트가 의존성이 필요하지 않다는 것을 증명해야 한다. 

## **To change the dependencies, change the code**

> **의존성을 변경하려면 코드를 변경하세요**
> 

Effect 사용 순서

1. 먼저 Effect의 코드 또는 반응형 값 선언 방식을 **변경한**다.
2. 그런 다음, **변경한 코드에 맞게** 의존성을 조정한다.
3. 의존성 목록이 마음에 들지 않으면 **첫 번째 단계로 돌아가서** 코드를 다시 변경한다. (그리고 코드를 다시 변경한다).

의존성을 변경하려면 주변 코드 변경이 선행되어야 한다. 

<aside>
💡 **Pitfall | 함정**

```jsx
useEffect(() => {
  // ...
  // 🔴 Avoid suppressing the linter like this:
  // eslint-ignore-next-line react-hooks/exhaustive-deps
}, []);
```

이처럼 린터를 억제하는 Effect가 있을 수 있는데, 의존성이 코드와 일치하지 않으면 버그 발생 가능성이 매우 높아진다.

</aside>

<aside>
🌟 **DEEP DIVE | 의존성 린터를 억제하는 것이 위험한 이유**

린터를 억제하면 직관적이지 않은 버그를 찾아서 수정하기 어렵다.

**의존성 린트 오류는 컴파일 오류로 처리하는 것이 좋다. 이를 억제하지 않으면 이와 같은 버그가 발생하지 않는다.**

</aside>

## **Removing unnecessary dependencies**

> **불필요한 의존성 제거하기**
> 

의존성 목록에 여러 개의 값이 존재할 때, 의존성 중 하나라도 변경되면 Effect를 재실행 하도록 하는 것이 합리적이지 않을 때도 있다. 

- 다른 조건에서 Effect의 *다른 부분*을 다시 실행하고 싶을 수도 있다.
- 일부 의존성의 변경에 “반응”하지 않고 “최신 값”만 읽고 싶을 수도 있다.
- 의존성은 객체나 함수이기 때문에 *의도치 않게* 너무 자주 변경될 수 있다.

이런 경우에 사용할 수 있는 해결책을 찾기 위한 몇 가지 질문

### **Should this code move to an event handler?**

> **이 코드를 이벤트 핸들러로 옮겨야 하나요?**
> 

가장 먼저 고려할 것은 코드를 Effect 내부에 작성할 지에 대한 여부

특정 상호작용에 대한 응답으로 코드를 실행하려면 해당 로직을 해당 이벤트 핸들러에 직접 넣어야 한다.

### **Is your Effect doing several unrelated things?**

> **Effect가 여러 관련 없는 일을 하고 있나요?**
> 

각 Effect는 독립적인 동기화 프로세스를 나타내야 한다. 따라서 Effect 내부의 코드가 서로 관련이 없다면 각각 서로 다른 useEffect로 분리하는 것이 좋다. 

### **Are you reading some state to calculate the next state?**

> **다음 state를 계산하기 위해 어떤 state를 읽고 있나요?**
> 

Effect에서 반응형 값을 사용한다고 해도 내부 로직에서 반응형 값을 사용하지 않는다면 의존성 목록에 반응형 값을 추가하지 않고 반응형 값만 업데이트 하도록 업데이터 함수를 전달하여 구현할 수 있다.

```jsx
function ChatRoom({ roomId }) {
  const [messages, setMessages] = useState([]);
  useEffect(() => {
    const connection = createConnection();
    connection.connect();
    connection.on('message', (receivedMessage) => {
      **setMessages(msgs => [...msgs, receivedMessage]);**
    });
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

### **Do you want to read a value without “reacting” to its changes?**

> **값의 변경에 ‘반응’하지 않고 값을 읽고 싶으신가요**
> 

<aside>
⚠️ **이 섹션에서는 아직 안정되지 않은 API 에 대해 설명한다.**

</aside>

Effect에서 반응해서는 안되는 로직은 Effect Event를 사용하여 추출하면 된다.

### **Wrapping an event handler from the props**

> **props를 이벤트 핸들러로 감싸기**
> 

부모로부터 함수(이벤트 핸들러)를 props로 전달받은 경우 부모 컴포넌트가 렌더링할 때마다 다른 함수를 전달하므로 매번 Effect가 동기화된다. 이럴 때도 Effect Event를 사용하면 된다.

### **Separating reactive and non-reactive code**

> **반응형 코드와 비반응형 코드 분리**
> 

비반응형 코드 역시 Effect Event로 분리하는 것이 좋다. 

### **Does some reactive value change unintentionally?**

> **일부 반응형 값이 의도치 않게 변경되나요?**
> 

**JavaScript에서는 새로 생성된 객체와 함수가 다른 모든 객체와 완벽하게 동일한 값을 가지고 있다고 하더라도 완벽히 구별되는 것으로 간주된다.** 

따라서 객체 및 함수 의존성으로 인해 Effect 가 필요 이상으로 재동기화될 수 있다. 따라서 객체와 함수는 웬만하면 의존성으로 사용하지 않는 것이 좋다.

### **Move static objects and functions outside your component**

> **정적 객체와 함수를 컴포넌트 외부로 이동**
> 

객체가 props 및 state에 의존하지 않는다면 해당 객체를 컴포넌트 외부로 이동 시키는 것이 좋다. 

### **Move dynamic objects and functions inside your Effect**

> **Effect 내에서 동적 객체 및 함수 이동**
> 

만약 객체가 roomId 프로퍼티처럼 리렌더링의 결과로 변경될 수 있는 반응형 값에 의존하는 경우, 컴포넌트를 외부로 끌어낼 수는 없지만 Effect 코드 내부로 이동시킬 수는 있다. 

```jsx
const serverUrl = 'https://localhost:1234';

function ChatRoom({ roomId }) {
  const [message, setMessage] = useState('');

  useEffect(() => {
    **const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };**
    const connection = createConnection(options);
    connection.connect();
    return () => connection.disconnect();
  }, [roomId]); // ✅ All dependencies declared
  // ...
```

Effect 내부에서 선언된 객체는 Effect의 의존성이 아니다. 여기서 유일한 반응형 값은 roomId 이며 객체나 함수가 아니므로 의도치 않게 달라지지 않을 것이다. 

### **Read primitive values from objects**

> **객체에서 원시 값 읽기**
> 

props로 객체를 받는 경우도 있는데, 이 경우 렌더링 중에 부모 컴포넌트가 객체를 생성한다. 따라서 부모 컴포넌트가 리렌더링 할 때마다 Effect가 다시 연결된다.  → Effect 외부의 객체에서 정보를 읽음으로써 객체 및 함수 의존성을 피할 수 있다.

```jsx
function ChatRoom({ options }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = options;
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
  // ...
```

⇒ 부모 컴포넌트에 의해 의도치 않게 객체가 다시 생성된 경우 채팅이 다시 연결되지 않지만 `options.roomId` 또는 `options.serverUrl`이 실제로 다른 경우 채팅이 다시 연결된다.

### **Calculate primitive values from functions**

> **함수에서 원시값 계산**
> 

함수에 대해서도 동일한 방식으로 접근할 수 있다. 부모 컴포넌트가 props로 함수를 전달할 때 의존성을 만들고 싶지 않다면 Effect 외부에서 호출하면 된다.

```jsx
function ChatRoom({ getOptions }) {
  const [message, setMessage] = useState('');

  const { roomId, serverUrl } = getOptions();
  useEffect(() => {
    const connection = createConnection({
      roomId: roomId,
      serverUrl: serverUrl
    });
    connection.connect();
    return () => connection.disconnect();
  }, [roomId, serverUrl]); // ✅ All dependencies declared
  // ...
```

만약 함수가 이벤트 핸들러이지만 변경 사항으로 인해 Effect가 다시 동기화 되는 것을 원치 않으면 대신 Effect Event로 감싸면 된다.

# **Reusing Logic with Custom Hooks**

> **커스텀 훅으로 로직 재사용하기**
> 

useState, useContext, useEffect 와 같이 React에서 기본적으로 제공하는 훅 이외에 별도로 구체적인 목적을 지닌 훅이 필요하다면 커스텀 훅을 만들어 사용할 수 있다.

<aside>
🌟 **학습 내용**

- 커스텀 훅이란 무엇이며, 직접 작성하는 방법
- 컴포넌트 간에 로직을 재사용하는 방법
- 커스텀 훅의 이름을 만들고 구조화하는 방법
- 커스텀 훅을 추출해야 하는 시기와 이유
</aside>

## **Custom Hooks: Sharing logic between components**

> **커스텀 훅: 컴포넌트간의 로직 공유**
> 

네트워크 연결이 끊어진 경우 사용자에게 주의를 주는 기능을 구현하고 싶다고 가정하자.

이럴 경우 컴포넌트에는 2가지가 필요하다.

1. 네트워크가 온라인 상태인지 여부를 추적하는 state
2. 전역 online 및 offline 이벤트를 구독하고, state를 업데이트하는 Effect

아래 예시의 StatusBar 컴포넌트와 SaveButton 컴포넌트는 네트워크 온라인 상태에 따라 state를 업데이트하고 state 업데이트에 따라서 컴포넌트를 업데이트 한다. 두 컴포넌트는 네트워크 온라인 상태에 대한 중복되는로직을 갖고 있다.

```jsx
import { useState, useEffect } from 'react';

export default function StatusBar() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

export default function SaveButton() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}
```

## **Extracting your own custom Hook from a component**

> **컴포넌트에서 커스텀 훅 추출하기**
> 

네트워크 상태를 추적할 수 있는 state를 커스텀 훅으로 만들면 중복된 코드를 줄일 수 있다.

```jsx
**// useOnlineStatus 커스텀 훅**
function useOnlineStatus() {
  const [isOnline, setIsOnline] = useState(true);
  useEffect(() => {
    function handleOnline() {
      setIsOnline(true);
    }
    function handleOffline() {
      setIsOnline(false);
    }
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  return isOnline;
}

**// useOnlineStatus 활용하여 중복된 로직을 제거**
function StatusBar() {
  const isOnline = useOnlineStatus();
  return <h1>{isOnline ? '✅ Online' : '❌ Disconnected'}</h1>;
}

function SaveButton() {
  const isOnline = useOnlineStatus();

  function handleSaveClick() {
    console.log('✅ Progress saved');
  }

  return (
    <button disabled={!isOnline} onClick={handleSaveClick}>
      {isOnline ? 'Save progress' : 'Reconnecting...'}
    </button>
  );
}

export default function App() {
  return (
    <>
      <SaveButton />
      <StatusBar />
    </>
  );
}
```

반복된 로직을 제거하면서 컴포넌트 내부 코드는 무엇을 할 것인지 컴포넌트의 목적을 위한 코드만 작성하면 된다.

(컴포넌트의 코드는 구현이 아니라 의도를 표현한다.)

## **Hook names always start with `use`**

> **훅의 이름은 언제나 `use`로 시작됩니다.**
> 

React 컴포넌트는 2가지 명명 규칙을 따라야 한다.

1. React 컴포넌트 이름은 대문자로 시작해야 한다.
2. 훅의 이름은 use로 시작해야 하며 그 다음 첫글자는 대문자여야 한다.

use로 시작하지 않는 함수는 내부에 React state를 포함할 수 없다는 것을 의미한다.

<aside>
🌟 **DEEP DIVE | 렌더링 시에 호출되는 모든 함수에 use 접두사를 써야 하나요?**

훅을 사용하지 않는 함수는 use 접두사를 사용할 필요가 없다. 훅을 호출하지 않으면 일반 함수로 작성하면 된다.

반드시 강제되는 원칙은 아니지만 혼란을 야기할 수 있으므로 권장을 따르는 것이 좋다.

</aside>

## **Custom Hooks let you share stateful logic, not state itself**

> **커스텀 훅은 state 자체가 아닌 상태적인 로직(stateful logic)을 공유합니다.**
> 

앞선 예제에서 StatusBar 컴포넌트와 SaveButton 컴포넌트는 모두 내부에 useOnlineStatus 커스텀 훅을 사용하고 있다. 두 컴퍼넌트에서 사용되는 isOnline state는 공유되는 값이 아니라 독립적인 state 변수이다. 

단지 네트워크 여부에 따라 동일한 외부 값과 동기화 되었을 뿐이다. 

**커스텀 훅을 사용하면 *상태 로직(stateful logic)은 공유할 수 있지만 state 자체는 공유할 수 없다.* 각 훅 호출은 동일한 훅에 대한 다른 모든 호출과 완전히 독립적이다.**

여러 컴포넌트 간에 state 자체를 공유해야 하는 경우, 대신 [끌어올려 전달하기](https://react-ko.dev/learn/sharing-state-between-components)를 사용하자.

## **Passing reactive values between Hooks**

> **훅 사이에 반응형 값 전달하기**
> 

공식 문서 참조 - 커스텀 훅 생성하여 사용하는 예시

### **Passing event handlers to custom Hooks**

> **커스텀훅에게 이벤트 핸들러 전달하기**
> 

<aside>
⚠️ **이 섹션에서는 아직 안정되지 않은 API 에 대해 설명한다.**

</aside>

```jsx
export function useChatRoom({ serverUrl, roomId }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
		**// 메시지가 도착했을 때 무엇을 해야 하는지에 대한 로직**
    **connection.on('message', (msg) => {
      showNotification('New message: ' + msg);
    });**
    return () => connection.disconnect();
  }, [roomId, serverUrl]);
}
```

위 로직에서 동작을 실행하는 이벤트 핸들러를  props 로 받아서 처리할 수 있다. 

```jsx
**// 먼저 부모 컴포넌트에서 이벤트 핸들러를 선언하고, 커스텀 훅에게 props로 전달한다.**
export default function ChatRoom({ roomId }) {
  const [serverUrl, setServerUrl] = useState('https://localhost:1234');

  useChatRoom({
    roomId: roomId,
    serverUrl: serverUrl,
    **onReceiveMessage(msg) {
      showNotification('New message: ' + msg);
    }**
  });
  // ...

**// 커스텀 훅에서 props로 전달받은 이벤트 핸들러를 사용한다.** 
export function useChatRoom({ serverUrl, roomId, **onReceiveMessage** }) {
  useEffect(() => {
    const options = {
      serverUrl: serverUrl,
      roomId: roomId
    };
    const connection = createConnection(options);
    connection.connect();
    connection.on('message', (msg) => {
      **onReceiveMessage(msg);**
    });
    return () => connection.disconnect();
  }, [roomId, serverUrl, onReceiveMessage]); // ✅ All dependencies declared
                                             // ✅ 모든 의존성이 선언됨
}
```

### **When to use custom Hooks**

> **언제 커스텀 훅을 사용할 것인가**
> 

중복되는 모든 코드를 커스텀훅으로  추출할 필요는 없다. 다만 Effect를 작성할 때마다 커스텀 훅으로 감싸는 것이 더 명확할 지 고려해볼 필요는 있다. Effect를 커스텀 훅으로 감싸면 의도와 데이터 흐름 방식을 정확하게 전달할 수 있다. 

서로 다른 동기화를 해야한다면 하나의 Effect 가 아니라 개별 커스텀 훅으로 추출하여 단순화 하는게 좋다.

<aside>
🌟 **DEEP DIVE | 커스텀 훅은 구체적인 고수준 사용 사례에 집중하세요**

커스텀 생명주기와 연관된 훅을 생성하거나 사용하지 말 것

Effect를 사용할 것이라면 React API를 직접 사용하는 것을 권장한다.

</aside>

### **Custom Hooks help you migrate to better patterns**

> **커스텀 훅은 더 나은 패턴으로 마이그레이션하는데 도움을 줍니다.**
> 

Effect는 탈출구로써 React 프레임워크에서 벗어나 빌트인 솔루션으로 해결할 수 없는 솔루션을 제공하기 위해 사용한다. 

**커스텀 훅으로 Effect를 감싸면 얻을 수 있는 장점**

1. Effect와의 데이터 흐름을 매우 명확하게 만들 수 있다.
2. 컴포넌트가 Effect의 정확한 구현보다는 의도에 집중할 수 있다.
3. React가 새로운 기능을 추가할 때 컴포넌트를 변경하지 않고도 해당 Effect를 제거할 수 있다.

### **There is more than one way to do it**

> **여러가지 방법이 있습니다**
> 

각각의 Effect 로직을 커스텀 훅으로 분리할 수 있다. 얼마나 세세하게 커스텀 훅을 쪼갤지는 사용자가 결정한다. 뿐만 아니라 **Effect에 로직을 유지하는 대신 대부분의 명령형 로직을 JavaScript 클래스로 관리**할 수 도 있다. 

Effect간의 연결 관계가 많이 필요할 수록 Effect와 훅에서 각 로직을 완전히 추출하는 것이 합리적이다. 그러면 추출한 코드가 “외부 시스템”이 된다. 이런 구조로 작성하면 React 외부로 이동한 시스템으로 메시지를 보내기만 하면 되므로 Effect를 단순하게 유지할 수 있다.