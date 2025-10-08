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

## 2.3 클래스 컴포넌트와 함수 컴포넌트
### 2.3.1 클래스 컴포넌트
- 클래스 컴포넌트 예제
  - 마운트: 컴포넌트 생성 시점
  - 업데이트: 이미 생성된 컴포넌트의 내용 변경 시점
  - 언마운트: 컴포넌트가 더 이상 존재하지 않는 시점점
    <img width="931" height="695" alt="image" src="https://github.com/user-attachments/assets/2cc1980c-6918-480f-abfa-1a0ea6bf9e67" />

  ```
  import React from 'react'
  
  // ✅ props 타입 선언
  interface SampleProps {
    required?: boolean
    text: string
  }
  
  // ✅ state 타입 선언
  interface SampleState {
    count: number
    isLimited?: boolean
    isMounted: boolean
    // ✅ props.text를 state로 동기화해 보관 (필요 시 제거 가능)
    syncedText: string
    // ✅ 윈도우 크기 추적 (리사이즈 전/후 비교용)
    windowWidth: number
    windowHeight: number
  }
  
  // ✅ 스냅샷 타입 (업데이트 직전 DOM 상태 넘겨주기)
  type Snapshot = {
    // 스크롤 보정용
    prevScrollTop: number
    prevScrollHeight: number
    wasAtBottom: boolean
    // 리사이즈 비교용
    prevWidth: number
    prevHeight: number
  }
  
  // ✅ props, state 순으로 제네릭 선언
  class SampleComponent extends React.Component<SampleProps, SampleState> {
    private mountTimer?: number
    private containerRef = React.createRef<HTMLDivElement>() // ✅ 스크롤 컨테이너 참조
  
    // ✅ 생성자로 부터 props를 넘겨주고 state를 정의한다. ES2022 부터는 constructor 없이 state 정의 가능
    constructor(props: SampleProps) {
      super(props) // React.Component 상위 접근 가능
      this.state = {
        count: 0,
        isLimited: false,
        isMounted: false,
        syncedText: props.text, // ✅ 초기 props로 동기화
        windowWidth: typeof window !== 'undefined' ? window.innerWidth : 0,
        windowHeight: typeof window !== 'undefined' ? window.innerHeight : 0,
      }
      this.handleDecreaseClick = this.handleDecreaseClick.bind(this)
      this.handleWindowResize = this.handleWindowResize.bind(this)
    }
  
    // ✅ props 변화에 따라 state를 동기화 (static 이므로 this 사용 불가)
    static getDerivedStateFromProps(nextProps: SampleProps, prevState: SampleState) {
      if (nextProps.text !== prevState.syncedText) {
        return { syncedText: nextProps.text }
      }
      return null // 변경 없으면 그대로 유지
    }
  
    // ✅ state 변경이 가능하며, 컴포넌트가 마운트되고 준비되는 즉시 실행
    componentDidMount() {
      console.log('✅ componentDidMount 실행')
  
      // 1초 뒤 isMounted 상태 변경
      this.mountTimer = window.setTimeout(() => {
        this.setState({ isMounted: true })
        console.log('✅ isMounted 상태가 true로 변경되었습니다.')
      }, 1000) // 1초 뒤 마운트 완료 처리
  
      // ✅ 윈도우 리사이즈 이벤트 등록
      window.addEventListener('resize', this.handleWindowResize)
    }
  
    // ✅ 업데이트 직전의 DOM 상태를 스냅샷으로 캡처하여 componentDidUpdate로 전달
    // 스크롤 위치/스크롤 높이, 리사이즈 이전 크기 등을 기록
    getSnapshotBeforeUpdate(prevProps: SampleProps, prevState: SampleState): Snapshot | null {
      const el = this.containerRef.current
      if (!el) {
        return {
          prevScrollTop: 0,
          prevScrollHeight: 0,
          wasAtBottom: false,
          prevWidth: prevState.windowWidth,
          prevHeight: prevState.windowHeight,
        }
      }
  
      const { scrollTop, scrollHeight, clientHeight } = el
      const wasAtBottom = scrollTop + clientHeight >= scrollHeight - 4 // 바닥 근사치 판정
      return {
        prevScrollTop: scrollTop,
        prevScrollHeight: scrollHeight,
        wasAtBottom,
        prevWidth: prevState.windowWidth,
        prevHeight: prevState.windowHeight,
      }
    }
  
    // ✅ 업데이트 감지 (이전 props, state, snapshot 비교 가능)
    componentDidUpdate(prevProps: SampleProps, prevState: SampleState, snapshot: Snapshot | null) {
      // count 변경 감지
      if (prevState.count !== this.state.count) {
        console.log(`🔁 [Update] count가 ${prevState.count} → ${this.state.count} 로 변경되었습니다.`)
      }
      // syncedText 변경 감지 (getDerivedStateFromProps 반영 결과)
      if (prevState.syncedText !== this.state.syncedText) {
        console.log(
          `🔁 [Update] syncedText가 "${prevState.syncedText}" → "${this.state.syncedText}" 로 변경되었습니다.`
        )
      }
  
      // ✅ 스냅샷을 사용한 스크롤 위치 보정 & 리사이즈 로그
      if (snapshot) {
        const el = this.containerRef.current
        // 스크롤 보정: 바닥에 붙어 있었으면 업데이트 후에도 바닥 유지
        // 아니었다면 증가한 scrollHeight 만큼 상대 위치 보정
        if (el) {
          const added = el.scrollHeight - snapshot.prevScrollHeight
          if (snapshot.wasAtBottom) {
            el.scrollTop = el.scrollHeight // 바닥 고정
          } else if (added !== 0) {
            el.scrollTop = snapshot.prevScrollTop + added // 상대 위치 유지
          }
        }
  
        // 리사이즈 전/후 비교 로그
        if (
          prevState.windowWidth !== this.state.windowWidth ||
          prevState.windowHeight !== this.state.windowHeight
        ) {
          console.log(
            `🖥️ [Resize] ${snapshot.prevWidth}×${snapshot.prevHeight} → ${this.state.windowWidth}×${this.state.windowHeight}`
          )
        }
      }
    }
  
    // ✅ 언마운트 시 타이머 클리어
    componentWillUnmount() {
      console.log('❌ componentWillUnmount 실행')
      if (this.mountTimer) {
        clearTimeout(this.mountTimer)
      }
      // ✅ 리스너 해제
      window.removeEventListener('resize', this.handleWindowResize)
    }
  
    // ✅ shouldComponentUpdate: 리렌더링 여부 제어
    // PureComponent로 대체가 가능하며 1뎁스의 얕은 비교만 가능
    shouldComponentUpdate(nextProps: SampleProps, nextState: SampleState) {
      // 1. count나 isMounted가 바뀐 경우만 렌더링 허용 (+ syncedText/크기 비교)
      if (
        nextState.count !== this.state.count ||
        nextState.isMounted !== this.state.isMounted ||
        nextState.isLimited !== this.state.isLimited ||
        nextState.syncedText !== this.state.syncedText ||
        nextState.windowWidth !== this.state.windowWidth ||
        nextState.windowHeight !== this.state.windowHeight ||
        nextProps.text !== this.props.text ||
        nextProps.required !== this.props.required
      ) {
        console.log('✅ [shouldComponentUpdate] 렌더링 허용')
        return true
      }
  
      // 2. 나머지는 렌더링 막기
      console.log('🚫 [shouldComponentUpdate] 렌더링 차단')
      return false
    }
  
    // ✅ render 내부에 쓰일 함수 (화살표 함수 상위 스코프에서 this가 결정되기 때문에 굳이 바인딩이 필요없음)
    private handleIncreaseClick = () => {
      const newValue = this.state.count + 1
      this.setState({
        count: newValue,
        isLimited: newValue >= 10, // ✅ 문법 수정: "> =" → ">="
      })
    }
  
    // ✅ 화살표 함수가 아닌 경우 this를 현재 클래스로 바인딩 해야함
    private handleDecreaseClick() {
      this.setState((prev) => ({ count: prev.count - 1 }))
    }
  
    // ✅ 윈도우 리사이즈 핸들러
    private handleWindowResize() {
      this.setState({
        windowWidth: window.innerWidth,
        windowHeight: window.innerHeight,
      })
    }
  
    // ✅ 렌더링 정의
    // (this.setState를 절대 호출하면 안됨 => 순수 함수로 항상 같은 결과물을 반환해야하기 때문, 생명주기를 활용하자.)
    render() {
      // ✅ props와 state를 this로 부터 꺼낸다.
      const {
        props: { required, text },
        state: { count, isLimited, isMounted, syncedText, windowWidth, windowHeight },
      } = this
  
      // ✅ 마운트 준비 중일 때 로딩 표시
      if (!isMounted) {
        return <div>⏳ 컴포넌트 준비 중...</div>
      }
  
      // ✅ 렌더링 (스크롤 테스트용으로 컨테이너에 고정 높이/overflow 부여)
      return (
        <div>
          <h2>Sample Component</h2>
          <div>{required ? '필수' : '필수아님'}</div>
          <div>문자: {text}</div>
          <div>문자 (props→state 동기화): {syncedText}</div>
          <div>현재 카운트: {count}</div>
          <div>윈도우 크기: {windowWidth} × {windowHeight}</div>
  
          <button onClick={this.handleIncreaseClick} disabled={isLimited}>
            증가
          </button>
          {/*
           * ✅ 이 방법은 지양하도록 한다.
           * 렌더링 시 새로운 함수 생성으로 인해 최적화 수행이 어려움
           *
           * <button onClick={() => this.handleIncreaseClick()} disabled={isLimited}>
           *   증가
           * </button>
           */}
          <button onClick={this.handleDecreaseClick}>감소</button>
  
          {/* ✅ 스냅샷/스크롤 보정 확인용 컨테이너 */}
          <div
            ref={this.containerRef}
            style={{
              height: 160,
              overflow: 'auto',
              marginTop: 12,
              border: '1px solid #ddd',
              padding: 8,
              lineHeight: 1.6,
            }}
          >
            {/* 스크롤 테스트용 더미 콘텐츠 */}
            {Array.from({ length: 30 + count }).map((_, i) => (
              <div key={i}>항목 #{i + 1}</div>
            ))}
          </div>
        </div>
      )
    }
  }
  
  export default SampleComponent

  ```
- 에러 바운더리 예제
  ```
  import React, { ErrorInfo, PropsWithChildren } from 'react'

  type Props = PropsWithChildren<{
    name?: string // ✅ ErrorBoundary 구분용 (예: parent, child)
  }>
  
  type State = {
    hasError: boolean
    errorMessage: string
  }
  
  export default class ErrorBoundary extends React.Component<Props, State> {
    constructor(props: Props) {
      super(props)
      this.state = {
        hasError: false,
        errorMessage: '',
      }
    }
  
    // ✅ 자식 컴포넌트에서 오류 발생 시, 새로운 state 반환하여 자식으로 부터 어떤 렌더링을 결정하는 용도로 사용함
    // Render 단계에서 실행
    static getDerivedStateFromError(error: Error): State {
      return {
        hasError: true,
        errorMessage: error.toString(),
      }
    }

    // Commit 단계에서 실행
    // ✅ 실제 에러 정보 (error + info.componentStack)를 잡아서 로깅
    // 자식 컴포넌트에서 에러 발생 시 실행 -> getDerivedStateFromError에서 state를 경정한 이후에 실행
    componentDidCatch(error: Error, info: ErrorInfo) {
      console.group(`🚨 [${this.props.name ?? 'ErrorBoundary'}] 에러 발생`)
      console.error(error)
      console.info(info.componentStack)
      console.groupEnd()
    }
  
    render() {
      // ✅ 에러 발생 시 대체 UI 렌더링
      if (this.state.hasError) {
        return (
          <div style={{ border: '1px solid red', padding: 12, margin: 8 }}>
            <h2>⚠️ 오류가 발생했습니다.</h2>
            <p>{this.state.errorMessage}</p>
            <p>({this.props.name ?? 'unknown boundary'})</p>
          </div>
        )
      }
  
      // ✅ 정상 렌더링 시 children 반환
      return this.props.children
    }
  }


  import React, { useState } from 'react'

  // 익명 함수로 쓰는 경우는 ErrorBoundary의 프로덕션 레벨에서 windows 함수 전파가 되지 않아 추적이 어려움으로 function 함수로 써서 하자 혹은 익명 함수는 displayName을 명시 해야한다.
  export default function Child() {
    const [error, setError] = useState(false)
  
    const handleClick = () => {
      setError((prev) => !prev)
    }
  
    // ✅ 버튼 클릭 시 에러 발생
    if (error) {
      throw new Error('Error has been occurred.')
    }
  
    return (
      <button onClick={handleClick} style={{ margin: 8 }}>
        에러 발생
      </button>
    )
  }

  import React from 'react'
  import ErrorBoundary from './ErrorBoundary'
  import Child from './Child'
  
  export default function App() {
    return (
      <ErrorBoundary name="parent">
        {/* ✅ 하위 ErrorBoundary가 별도로 예외를 처리 */}
        <ErrorBoundary name="child">
          <Child />
        </ErrorBoundary>
      </ErrorBoundary>
    )
  }
  ```
- 클래스 컴포넌트 한계
  - 애플리케이션 내부 로직의 재사용이 어렵다.
    - `componentDidMount`, `componentWillUnmount`, `this.setState` 등 `React.Component`의 외부적 종속으로 인해 분리가 어려움
    ```
      // 두 컴포넌트 모두 윈도우 크기를 추적한다고 가정
      class WindowSizeA extends React.Component {
        state = { width: window.innerWidth }
      
        componentDidMount() {
          window.addEventListener('resize', this.handleResize)
        }
      
        componentWillUnmount() {
          window.removeEventListener('resize', this.handleResize)
        }
      
        handleResize = () => this.setState({ width: window.innerWidth })
      
        render() {
          return <div>Width: {this.state.width}</div>
        }
      }
      
      class WindowSizeB extends React.Component {
        state = { width: window.innerWidth }
      
        componentDidMount() {
          window.addEventListener('resize', this.handleResize)
        }
      
        componentWillUnmount() {
          window.removeEventListener('resize', this.handleResize)
        }
      
        handleResize = () => this.setState({ width: window.innerWidth })
      
        render() {
          return <div>너비: {this.state.width}</div>
        }
      }

      // HOC를 통한 우회책으로 wrapper hell에 빠질 수 있고 흐름을 쫓아야 하는 문제로 복잡도 증가 문제가 있어 보임.
      // 현재는 hooks로 분리가 가능함.
      function withWindowSize(WrappedComponent) {
        return class extends React.Component {
          state = { width: window.innerWidth }
          componentDidMount() {
            window.addEventListener('resize', this.handleResize)
          }
          componentWillUnmount() {
            window.removeEventListener('resize', this.handleResize)
          }
          handleResize = () => this.setState({ width: window.innerWidth })
          render() {
            return <WrappedComponent width={this.state.width} {...this.props} />
          }
        }
      }
    ```
  - 코드 크기를 최적화 하기 어렵다.
    - 번들링 최적화가 잘 되지 않아 트리 쉐이킹이 되지 않고 handleChange을 번들 그대로 나타난 모습을 볼 수 있다.
    ```
    import React from 'react'

    export class Example extends React.PureComponent {
      state = { count: 1 }
    
      handleClick = () => {
        console.log('clicked')
        this.setState({ count: this.state.count + 1 })
      }
    
      handleChange = () => {
        console.log('changed')
      }
    
      render() {
        return (
          <div>
            Count: {this.state.count}
            <button onClick={this.handleClick}>+</button>
          </div>
        )
      }
    }

    function Example(e) {
      var n;
    
      // 클래스 호출 보장(ES5 변환 시 보일 수 있는 패턴)
      return (
        (function (e, n) {
          if (!(e instanceof n)) throw new TypeError('Cannot call a class as a function');
        })(this, Example),
    
        // super 호출과 비슷한 헬퍼
        ((n = SomeSuper.call(this, e)).handleClick = function () {
          console.log('handleClick!');
          n.setState({ count: 1 });
        }),
    
        // ❗ 사용하지 않아도 인스턴스에 메서드가 남아 있음
        (n.handleChange = function () {
          console.log('handleChanged!');
        }),
    
        // 초기 state
        (n.state = { count: 1 }),
    
        n
      );
    }
    ```
  - 핫 리로딩에도 불리하다.
    - 클래스 함수 특성상 인스턴스 기반으로 동작하기 때문에 코드 수정 시 새로운 인스턴스를 생성하여 상태 유지가 되지 않는다.
    - 함수 컴포넌트의 `useState`의 경우 클로저로 되어 있기 때문에 함수 정의만 교체되며 클로저 상태가 복원된다.




















