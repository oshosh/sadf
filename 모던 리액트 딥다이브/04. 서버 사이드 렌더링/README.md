## 4.1 서버 사이드 렌더링이란?

### 싱글 페이지 애플리케이션이란?
  - 렌더링과 라우팅에 필요한 대부분의 기능을 서버가 아닌 브라우저의 자바스크립트에 의존하는 방식을 의미한다. 
  - 최초의 페이지에서 데이터를 불러온 이후 페이지 전환은 자바스크립트와 브라우저의 history.pushState 와 history.replaceState로 이뤄진다.
  - 사이트의 소스 보기로 HTML코드를 보면 내부에 아무런 코드가 없다
  - 최초에 데이터를 불러온 이후에 자바스크립트 리소스와 브라우저 API를 기반으로 모든 작동이 이뤄진다.

### 서버 사이드 렌더링이란?
  - 최초에 사용자에게 보여줄 페이지를 서버에서 렌더링해 빠르게 사용자에게 화면을 제공하는 방식이다.
  - 웹페이지 렌더링의 책임에 따라 다르다.
    - SPA: 사용자에게 제공되는 자바스크립트 번들에서 렌더링 담당
      ```
      // 예를 들면...
      <script src="/static/js/main.js"></script>
      ```
    - SSR: 렌더링에 필요한 작업은 모두 서버에서 수행
  - 장점
    - 최초 페이지 진입이 비교적 빠르다.
      - 사용자 자원 혹은 HTML에 요청 의존적이거나 렌더링 해야할 HTML 크기가 큰 경우에 이점이 있는 것이다.
    - 검색 엔진 최적화
      - 검색 엔진 로봇 페이지 진입 (로봇은 정적 정보만 가져온다.)
      - 로봇이 HTML 다운로드를 하고 자바스크립트 실행은 하지 않음
      - 다운로드 기반의 오픈 그래프나 메타 태그 정보를 검색 엔진에 저장
    - CLS (누적 레이아웃 이동) 발생이 적다.
      - SPA에서는 콘텐츠 API 요청에 의존하는 경우가 많기 때문에 예상치 않게 데이터가 기존 레이아웃에 밀려 부정적인 경험이 나오는 경우가 있다.
    - 서버 렌더링 수행으로 인해 리소스 부담이 줄어 디바이스 성능에 비교적 자유롭다.
    - 보안이 안전하다.
      - `JAM 스택(Javascript, API, Markup)`의 공통적 문제 점으로 브라우저 활동이 모두 노출이 되고 API 호출로 인해 민감 정보 노출이 되어 방지할 준비가 되어 있어야한다. 하지만 SSR의 경우는 서버에서 수행으로 보안 위협을 피할 수 있다.
  - 단점
    - 소스코드 작성시 항상 서버 고려해야 한다.
      - `windows` 객체 혹은 `ssessionStorage`와 같은 브라우저 전역 객체 사용 시 문제가 된다.
    - 적절한 서버가 구축돼 있어야 한다.
      - SPA에 비해 물리적 가용량, 복구 전략, 요청 분산, 프로세스 매니저 등 다양한 환경이 필요함.
    - 서비스 지연에 따른 문제
      - `최초에 사용자에게 보여줄 페이지를 서버에서 렌더링해 빠르게 사용자에게 화면을 제공하는 방식`이라고 했지만 지연 작업 및 규모가 커지고 복잡해지면 의미있는 데이터를 먼저 보여줄 때까지 어떤 정보도 제공 될 수 없다. 반면 SPA의 경우는 로딩중 이나 혹은 지연 호출등 다양한 처리가 가능하다.
    - 결론
      - SPA와 SSR을 적절하게 사용해야 병목 현상을 줄이거나 접근성 등을 높일 수 있다.
## 4.2 서버 사이드 렌더링을 위한 리액트 API 살펴보기?
### 4.2.1 `renderToString`
  - React 컴포넌트를 렌더링 하여 HTML 문자열로 반환하는 함수이다.
  - 아래 코드를 살펴 보면 `useEffect`, `handleClick` 과 같은 이벤트는 포함되지 않는다.
    - 자바스크립트 코드는 HTML 생성된 결과 별도로 브라우저에 제공 되며 때문에 HTML만 제공되어 초기 렌더링에서 뛰어난 성능을 보일 수 있다.
    - `data-reactroot`은 자바스크립트를 실행하기 위한 `hydrate` 함수 식별 기준점이 된다.
    ```
    import ReactDOMServer from 'react-dom/server'
    import { useEffect } from 'react'
    
    function ChildrenComponent({ fruits }: { fruits: Array<string> }) {
      useEffect(() => {
        console.log(fruits)
      }, [fruits])
    
      function handleClick() {
        console.log('hello')
      }
    
      return (
        <ul>
          {fruits.map((fruit) => (
            <li key={fruit} onClick={handleClick}>
              {fruit}
            </li>
          ))}
        </ul>
      )
    }
    
    function SampleComponent() {
      return (
        <>
          <div>hello</div>
          <ChildrenComponent fruits={['apple', 'banana', 'peach']} />
        </>
      )
    }
  
    // 부모 컴포넌트인 SampleComponent를 렌더링 한다.
    const result = ReactDOMServer.renderToString(
      // <div id="root" /> 에서 실행
      React.createElement('div', { id: 'root' }, <SampleComponent />)
    )
    
    console.log(result)
  
    // 렌더링 결과
    <div id="root" data-reactroot="">
      <div>hello</div>
      <ul>
        <li>apple</li>
        <li>banana</li>
        <li>peach</li>
      </ul>
    </div>
    ```

### 4.2.2 `renderToStaticMarkup`
  - `renderToString`과 매우 유사 하며, HTML 문자열을 만든다는 점에 동일하다.
  - 리액트에서만 사용하는 `data-reactroot`과 같은 추가적인 DOM 속성을 만들지 않아 HTML 크기를 경량화 시킬 수 있다.
    - 이벤트 리스너가 필요 없는 순수 정적 HTML을 만들 때 사용된다.
    ```
    // 이하 생략................
    const result = ReactDOMServer.renderToStaticMarkup(
      // <div id="root" /> 에서 실행
      React.createElement('div', { id: 'root' }, <SampleComponent />)
    )
    // 렌더링 결과
    <div id="root">
      <div>hello</div>
      <ul>
        <li>apple</li>
        <li>banana</li>
        <li>peach</li>
      </ul>
    </div>
    ```
### 4.2.3 `renderToNodeStream` (리액트 서버 사이드 렌더링 프레임워크는 이걸 채택함! => `Next.js`)
  => 현재는 `Deprecated`가 되었고 `renderToPipeableStream`으로 변경 되었다.
  - `renderToString`과 결과물이 완전히 동일 하지만 차이점 두가지가 존재한다.
    - Node, Deno, Bun과 같은 서버 환경에서만 사용된다.
      - `renderToString` 또한 물론 브라우저에 사용 될 일이 없겠지만 브라우저 자체에서 실행이 되긴한다.
    - 결과물의 타입 반환 자체가 다르다.
      - `Node.js`의 경우 `ReadableStream`의 `utf-8`인코딩 된 바이트 스트림으로 내려오고 string을 얻기 위해 추가적 처리가 필요하다.
      - `ReadableStream` 자체는 브라우저에서 사용 할 수 있으나 만드는 과정이 브라우저에서 불가능하게 구현돼 있다.
  - 사용하는 이유는?
    - 동영상을 보기 위해 유튜브가 있다 치면 유튜브 영상을 보기 위해 다운로드 할때 까지 기다리지 않는다. 스트림은 큰 데이터를 다룰 때 데이터를 청크(chunk, 작은 단위)로 분할해 조금씩 가져오게 되며, HTML의 크기가 매우 큰 경우 서버에 부담을 줄일 수 있어 순차적 처리에 장점을 지닌다.
    - `renderToString`의 경우 또한 생각해보면 작은 크기을 굳이 분할해서 가져올 필요가 없고 부담 되는 크기가 있다면 굳이 `renderToString`로 보낼 필요가 없는 것이다.

### 4.2.4 `renderToStaticNodeStream`
  - 위 결과물과 같다. 리액트 속성을 제공하지 않는 순수 HTML 결과물이 필요할 때 사용 한다.

### 4.2.5 `hydrate`
  - `renderToString`, `renderToNodeStream`의 자바스크립트 이벤트, 핸들러를 붙이는 역할을 한다.
  - SPA React render의 일반적인 모습
    ```
    const rootElement = document.getElementById('root')
    // 클라이언트에서는 HTML 요소를 인수로 받아 렌더링과 이벤트 핸들러 추가 등 온전한 웹페이지를 만드는데 모든 작업을 수행 하고 있음.
    ReactDOM.render(<App />, rootElement)
    ```
  - `hydrate` render
    ```
    // containerId를 가리키는 element는 서버에서 렌더링된 HTML의 특정 위치를 의미한다.
    const element = document.getElementById(containerId) // 특정 위치 찾는다.
    // 해당 element 기준으로 이벤트 핸들러를 붙인다.
    ReactDOM.hydrate(<App />, element)
    ```
  - `hydrate` render 시 서버로 부터 받은 HTML 정보가 같지 않으면 발생되는 문제
    - 일단 서버와 클라이언트 시도가 같다면 정상적으로 SSR이 동작하고 기존 DOM을 유지한 채 이벤트만 붙임
      ```
      <!-- 서버에서 렌더된 HTML -->
      <div id="root">
        <span>안녕하세요</span>
      </div>
  
      <!-- 클라이언트에서 hydrate 시도 -->
      function App() {
        return <span>안녕하세요</span>;
      }
      
      const rootElement = document.getElementById("root");
      
      // hydrate 시도
      ReactDOM.hydrate(<App />, rootElement);
      ```
    - React는 “서버에서 이미 <span>이 있을 거라고 가정했는데 없네?” 즉, CSR로 전환되어 새로 렌더링.
       ```
        <!-- 서버에서 렌더된 HTML -->
        <div id="root">
          .............. 없음................
        </div>
    
        <!-- 클라이언트에서 hydrate 시도 -->
        function App() {
          return <span>안녕하세요</span>;
        }
        
        const rootElement = document.getElementById("root");
        
        // hydrate 시도
        // Warning: Expected server HTML to contain a matching <span> in <div>.
        ReactDOM.hydrate(<App />, rootElement);
       ```
    - 시간 난수 사용으로 인한 불일치 문제
      - 클라이언트 사이드와 서버사이드의 분기를 정확히 파악 하여 빌드하게 만들거나...
      - ssr 옵션을 포기하는 조건
      - suppressHydrationWarning를 잠시 무시하도록 한다. (클라이언트 사이드로 다시 랜더링 하기 때문에 깜박임이 있음)
      ```
        <!-- 서버에서 렌더된 HTML -->
        <div id="root">
          <div>Server: 1676641135288</div>
        </div>
    
        <!-- 클라이언트에서 hydrate 시도 -->
        function App() {
          return <div>Client: {Date.now()}</div>;
        }

        // 불일치 경고를 잠시 무시하고 싶은 경우는 suppressHydrationWarning를 사용하게 되지만 근본적인 문제 해결은 안됨...
        function App() {
          return <div suppressHydrationWarning>{Date.now()}</div>;
        }
        
        const rootElement = document.getElementById("root");
        
        // hydrate 시도
        // Warning: Text content did not match. Server: "1676641135288" Client: "1676641137261"
        ReactDOM.hydrate(<App />, rootElement);
      ```





















