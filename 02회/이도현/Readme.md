[2화] UI표현하기

# 개요
1. React는 UI를 렌더링하기위한 js 라이브러리이다. 프레임워크가 아니다.
2. UI는 버튼,텍스트,이미지,비디오 와 같은 작은 요소로 구성되는데 React를 통해 컴포넌트화 시킬수 있고
3. 컴포넌트를 통해 효율적으로 React를 작업할 수 있을 것이다.

# 첫번째 컴포넌트
1. React의 컴포넌트는 마크업으로 뿌릴수 있는 **js함수** 입니다.
2. React의 컴포넌트의 이름은 대문자로 시작해야 한다. [파스칼 케이스](https://htc-refactor.tistory.com/entry/%EC%BC%80%EC%9D%B4%EC%8A%A4-%EC%8A%A4%ED%83%80%EC%9D%BCCase-Styles-%EC%B9%B4%EB%A9%9C%EC%8B%9D-%EC%BC%80%EB%B0%A5%EC%8B%9D-%ED%8C%8C%EC%8A%A4%EC%B9%BC%EC%8B%9D-%EC%8A%A4%EB%84%A4%EC%9D%B4%ED%81%AC%EC%8B%9D)라고도 부른다고 한다.
3. 챌린지 2번에 리턴문 고치기 -> 줄 위치 차이로 화면에 이미지가 나오지 않는 이유?

# 컴포넌트를 import하거나 export하는 방법

1. export 와 default export의 차이

**export**
1. named export는 여러 값을 내보낼때 유용 해야 합니다. 단 내보낸이름을 편집하는것이 불가능합니다 ex) import ... as ... 불가능
2. 내보니기 시에 이름을 변경하는 방법이 있다.

**default**
1. default는 어떤이름으로도 가져올수 있습니다. 단 하나만 내보낼수 있다.
2. export default를 사용할 때 var,let,const는 사용하지 못합니다.

# jsx로 마크업 작성하기
- 컴포넌트를 통한 렌더링 로직과 마크업 알아보기
1. 버튼의 렌더링로직과 마크업이 함께 있어 변화가 생길때마다 서로 동기화 상태 유지 
2. 버튼과 사이드바의 경우는 컴포넌트를 분리하여 사용해 개별적으로 사용한다.

- jsx태그를 하나의 태그(<div> 와 Fragments)로 감싸는 이유
1. jsx -> 일반 js객체로 변환되기때문에 하나의 함수에서 두개의 객체를 반환할수 없다.

- 클래스는 HTML 속성이고, className은 DOM 속성입니다.

# 중괄호가 있는 jsx안에서 자바스크립트 사용하기
- {}를 통하여 마크업 안에서 객체를 참고를 할수 있습니다.
- {{}}는 jsx {}(jsx중괄호)안에 있는 js객체입니다.

# 컴포넌트에 props에 전달하기
- 구조분해할당 
1. 배열이나 객체의 속성을 해체한 값을 개별변수에 담는 js표현식이다.
2. ... Rest 속성은 열거형 패턴의 구조분해패턴으로 걸러지지 않는 나머지항목들을 모은다. 
```js

export default function Profile() {
    return (
        <Avatar
            person={{ name: 'Lin Lanying', imageId: '1bX5QH6' }}
            size={100}
        />
    );
}
//서로 다른 파일
   function Avatar(props) {
   let person = props.person;
   let size = props.size;
   // ...
   }
   // Rest속성 예시
   
let { a, b, ...rest } = { a: 10, b: 20, c: 30, d: 40 };
//...rest 는 c와 d의 값을 갇는다.

  ```