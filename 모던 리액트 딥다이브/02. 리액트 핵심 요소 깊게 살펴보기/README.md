## 2.1 JSX 란
- JSX 내부에 트리 구조로 표현하고 싶은 다양한 것들을 작성하고 트랜스파일 과정을 거쳐 자바스크립트가 이해 할 수 있는 코드로 변경하는 것.

### 2.2.1 JSX의 정의
- `JSXElement`
  - JSX의 가장 기본적 요소, HTML의 요소와 비슷한 역할.
  - 리액트 `JSXElement`는 반드시 대문자로 시작 할 것.
    ```
    // JSXOpeningElement
    <JSXElement JSXAttributes(optional)>
    
    // JSXClosingElement
    </JSXElement>
    
    // JSXSelfClosingElement
    <JSXElement JSXAttributes(optional) />
    
    // JSXFragment
    <>JSXChildren(optional)</>
    ```
  - `JSXElementName`: `JSXElement`의 요소 이름으로 쓸 수 있는 것을 의미.
    - JSXIdentifier
      ```
      function Valid1() {
        return <$_$ />     // ✅ 가능
      }
      function Valid2() {
        return <_X_ />     // ✅ 가능
      }
      // ❌ 숫자로 시작 불가
      function Invalid1() {
        // return <1x />   // 문법 에러
        return null
      }
      ```
    - JSXNamespacedName  (React에서는 중요도는 낮음)
      ```
      function ValidNS() {
        return <foo:bar />    // ⚠️ 문법상 ‘가능’하지만, 보통 React 빌드/런타임에선 에러/미지원
      }
      
      // ❌ 콜론 두 번 이상은 식별자 아님
      function InvalidNS() {
        // return <foo:bar:baz /> // 문법 에러
        return null
      }
      ```
    - JSXMemberExpression (React에서는 중요도는 낮음)
      ```
      const foo = {
        bar: () => <div>bar</div>,
        baz: {
          qux: () => <div>qux</div>,
        },
      };
      
      function ValidM1() {
        return <foo.bar />    // ✅ 멤버 접근
      }
      function ValidM2() {
        return <foo.baz.qux /> // ✅ 여러 단계도 가능
      }
      
      // ⚠️ 멤버 접근과 콜론을 섞는 건 불가/미지원
      function InvalidM() {
        // return <foo.bar:baz /> // 문법/도구에 따라 에러
        return null
      }
      ```
- JSXAttributes : `JSXElement`에 부여할 수 있는 속성을 의미 함
  - JSXSpreadAttributes
    ```
    const props = { id: 'foo', className: 'bar' }
    return <div {...props}></div>
    ```
  - JSXAttributeName
    ```
    function Valid() {
      return <foo.bar foo:bar="baz"></foo.bar> // ⚠️ JSXNamespacedName은 React에선 비권장
    }
    ```
  - JSXAttributeValue
    | 표현 형태      | 예시                                  | 설명               |
    | ---------- | ----------------------------------- | ---------------- |
    | 큰따옴표 문자열   | `<div title="hi" />`                | 문자열 리터럴          |
    | 작은따옴표 문자열  | `<div title='hi' />`                | 문자열 리터럴          |
    | JS 표현식     | `<div title={user.name} />`         | 중괄호 안에 JS 값      |
    | JSXElement | `<div attr={<span>Hello</span>} />` | 속성값으로 JSX 자체 가능  |
    | 생략         | `<input disabled />`                | 존재 자체가 `true` 의미 |

- JSXChildren : `JSXElement`의 내부의 자식 값을 의미
  - JSXChild 종류
    | 형태                     | 예시                           | 설명                 |
    | ---------------------- | ---------------------------- | ------------------ |
    | **JSXText**            | `<div>hello</div>`           | 단순 문자열             |
    | **JSXElement**         | `<div><span>hi</span></div>` | JSX 요소 포함          |
    | **JSXFragment**        | `<><li>1</li><li>2</li></>`  | 빈 태그 형태 (Fragment) |
    | **JSXChildExpression** | `<div>{user.name}</div>`     | 중괄호 안 JS 표현식       |
  - 코드
    ```
    export default function App() {
        return (
          <>
            {/* JSXText */}
            <p>Hello</p>
      
            {/* JSXElement */}
            <div><strong>bold</strong></div>
      
            {/* JSXFragment */}
            <>
              <li>1</li>
              <li>2</li>
            </>
      
            {/* JSXChildExpression */}
            <div>{(() => 'foo')()}</div> {/* 출력: foo */}
          </>
        )
      }

    ```
- JSXStrings: HTML과 JSX 간의 호환성을 위해 HTML에서 허용되는 모든 문자열을 JSX에서도 그대로 사용할 수 있습니다.
```
<!-- HTML -->
<button>\를 사용할 수 있다.</button>

// JSX — 백슬래시는 JS 문자열 규칙을 따름
let escape1 = "\"";  // ❌ SyntaxError (이스케이프 필요)
let escape2 = "\\";  // ✅ OK
```

### 2.2.3 JSX는 어떻게 자바스크립트에서 변환될까?
- `@babel/plugin-transform-react-jsx`의 이해가 일단 필요함.
- 예시 코드
  - JSX 원본
    ```
    const ComponentA = <A required={true}>Hello World</A>;
    const ComponentB = <>Hello World</>;
    const ComponentC = (
      <div>
        <span>hello world</span>
      </div>
    );
    ```
  - React 16/고전(runtime: classic) – React.createElement로 변환
    ```
    'use strict';

    var ComponentA = React.createElement(
      A,
      { required: true },
      'Hello World'
    );
    
    var ComponentB = React.createElement(React.Fragment, null, 'Hello World');
    
    var ComponentC = React.createElement(
      'div',
      null,
      React.createElement('span', null, 'hello world')
    );
    ```
  - React 17+
    ```
    'use strict';
    import * as _jsxRuntime from 'custom-jsx-library/jsx-runtime';
    
    const ComponentA = _jsxRuntime.jsx(A, {
      required: true,
      children: 'Hello World',
    });
    
    const ComponentB = _jsxRuntime.jsx(_jsxRuntime.Fragment, {
      children: 'Hello World',
    });
    
    const ComponentC = _jsxRuntime.jsxs('div', {
      children: [_jsxRuntime.jsx('span', { children: 'hello world' })],
    });
    ```
- 결론
  - JSX 변환 특성을 활용한다면 다음과 같이 같결하게 처리가 가능하고 불필요한 코드 중복을 제거 할 수 있다. 예를 들면 삼항 연산 처리
    ```
    function Greeting({ isLogin, children }: PropsWithChildren<{ isLogin: boolean }>) {
      return (
        <div>
          {isLogin ? <h1 className="text">Welcome back!</h1> : <span className="text">Please sign in</span>}
        </div>
      );
    }

    function Greeting({ isLogin, children }: PropsWithChildren<{ isLogin: boolean }>) {
      return createElement(
        isLogin ? 'h1' : 'span',
        {className: 'text'},
        children
      );
    }
    ```
## 2.2 가상 DOM과 리액트 파이버
### 2.2.1 DOM과 브라우저 렌더링 과정
- DOM: 웹페이지에 대한 인터페이스로 브라우저가 웹페이지의 콘텐츠와 구조를 어떻게 보여줄지 정보가 담겨있음.
- 브라우저 렌더링 과정은 생략 p129 참고

### 2.2.2 가상DOM의 탄생 배경
- 브라우저에서 사용자 인터렉션에 따라 크기가 만약 조정되었다고 할때 레이아웃 과정과 리페인트 과정을 거치게 되고 DOM의 많은 자식 요소 또한 덩달아 변경되기 때문에 브라우저의 비용의 증가로 인해 React에서 메모리에 저장이 가능한 가상돔을 제공한다.
- 가상돔은 실제 변경에 대한 준비가 완료되었을때 실제 브라우저 DOM에 반영되며, DOM보다 무조건 빠르진 않다.

### 2.2.3 가상 DOM을 위한 아키텍처, 리액트 파이버
- 가상 DOM과 실제 DOM을 비교해 변경 사항을 수집, 차이 발생 시 파이버 기준 화면 렌더링 요청 (재조정 - reconciliation)
  - 작업을 작은 단위로 분할하고 쪼갠 다음, 우선순위를 매긴다.
  - 이러한 작업을 일시 중지하고 나중에 다시 시작할 수 있다.
  - 이전 했던 작업을 다시 재사용하거나 필요하지 않은 경우에는 폐기할 수 있다.
- 과거 리액트 조정 알고리즘(DFS- 스택 알고리즘)과 달리 위 과정은 전부 비동기로 발생 시킨다.
- 사전 요약
  | 구분          | 설명                                                   |
  | ----------- | ---------------------------------------------------- |
  | **Fiber란?** | React Element를 나타내는 단위 객체로, 렌더링 작업 단위                |
  | **1:1 관계**  | 하나의 JSX(React Element)가 하나의 FiberNode로 대응하며, 그 관계는 [tag](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactWorkTags.js) 속성을 통해 알 수 있다.            |
  | **구조**      | `child`, `sibling`, `return` 연결로 트리(혹은 단일 연결 리스트) 구성 |
  | **재사용**     | 렌더링마다 Fiber를 새로 만들지 않고 `alternate`로 교체               |
  | **역할**      | 렌더링 작업 단위 관리 + 변경 사항 추적 + 커밋 단계에서 DOM 반영             |
- 파이버 구조의 개념 JSX -> 파이버 관계도 표현
  | 속성        | 역할                      |
  | --------- | ----------------------- |
  | `child`   | 첫 번째 자식 파이버 (트리 아래 방향)  |
  | `sibling` | 다음 형제 파이버 (같은 레벨의 옆 방향) |
  | `return`  | 부모 파이버 (트리 위 방향)        |
  
  ```
  <ul>
    <li>하나</li>
    <li>둘</li>
    <li>셋</li>
  </ul>

  // 대략적 관계
  const l3 = {
    return: ul,
    index: 2,
  }
  
  const l2 = {
    sibling: l3,
    return: ul,
    index: 1,
  }
  
  const l1 = {
    sibling: l2,
    return: ul,
    index: 0,
  }
  
  const ul = {
    // ...
    child: l1,
  }

   // 좀 더 시각적으로 표현
   ul
   └── child → l1
                ├── sibling → l2
                │             └── sibling → l3
                │
                └── (각 li의 return → ul)
  ```

- 리액트 파이버 트리 : 두개 존재하고 하나는 현재 모습 파이버 트리, 다른 하나는 workInProgress 트리가 있으며 작업이 끝나면 현재 트리를 workInProgress 트리로 변경 시키며 이러한 기술을 더블 버퍼링이라고 하며 커밋 단계에서 수행 된다.
  <img width="747" height="458" alt="image" src="https://github.com/user-attachments/assets/6774c16c-8628-47e0-ac1e-981947c2b521" />


- 파이버 작업 순서
  - beginWork() 함수로 파이버 작업 수행. 자식 없는 파이버 만날때까지 수행.
  - completeWork() 함수로 파이버 작업 완료.
  - 순회 방식 :
    - 최상위 React 요소에서 순회를 시작, 이에 대한 Fiber 노드 생성
    - 자식 요소로 이동하여 요소에 대한 Fiber 노드 생성
    - 형제 있으면 형제로 넘어가고, 형제 하위 트리 순회
    - 형제 없는 경우 return (부모) 로 돌아가 작업 완료됐음을 알린다.
    - 루트 완성되어 commitWork 수행.
 - 리택트 코드를 파이버 트리로 표현 하면 어떻게 될까?
   ```
   <A1>
     </B1>
     <B2>
       <C1>
         <D1 />
         <D2 />
       </C1>
     </B2>
     <B3 />
   </A1>
   ```

  - 파이버 트리 표현
<img width="464" height="399" alt="image" src="https://github.com/user-attachments/assets/afee8909-edbd-4ed7-a44b-598e7f704d35" />




