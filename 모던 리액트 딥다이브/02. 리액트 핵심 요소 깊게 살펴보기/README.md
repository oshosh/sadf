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















