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
### 4.2.6 서버 사이드 렌더링 예제
- `index.tsx`
  - 앞서 살펴본 바와 같이 `hydrate` 시에는 `<App />`과 HTML로 부터 내려받는 `el`이 같아야 정상적으로 SSR이 동작하게 된다.
  ```
  import React from 'react'
  import { hydrate } from 'react-dom'
  
  import App from './components/App'
  import { fetchTodo } from './fetch'
  
  async function main() {
    const result = await fetchTodo()
  
    const app = <App todos={result} />
    const el = document.getElementById('root')

    // app = el 은 동일한 결과물이어야 한다.
    hydrate(app, el)
  }
  
  main()
  ```
- `App.tsx`, `Todo.tsx`
  - 간단한 Todo 목록 조회 및 토글 변경 이벤트를 넣어 `hydrate`가 잘 되는지 간단한 예제를 넣는다.
  ```
  // `App.tsx`
  import React, { useEffect } from 'react'
  
  import { TodoResponse } from '../fetch'
  
  import { Todo } from './Todo'
  
  export default function App({ todos }: { todos: Array<TodoResponse> }) {
    useEffect(() => {
      console.log('하이!') // eslint-disable-line no-console
    }, [])
  
    return (
      <>
        <h1>나의 할일!</h1>
        <ul>
          {todos.map((todo, index) => (
            <Todo key={index} todo={todo} />
          ))}
        </ul>
      </>
    )
  }

  // `Todo.tsx`
  import React, { useState } from 'react'

  import { TodoResponse } from '../fetch'
  
  export function Todo({ todo }: { todo: TodoResponse }) {
    const { title, completed, userId, id } = todo
    const [finished, setFinished] = useState(completed)
  
    function handleClick() {
      setFinished((prev) => !prev)
    }
  
    return (
      <li>
        <span>
          {userId}-{id}) {title} {finished ? '완료' : '미완료'}
          <button onClick={handleClick}>토글</button>
        </span>
      </li>
    )
  }
  ```
- `index.html`
  - `__placeholder__` : HTML에 대한 파싱 정보 데이터는 전부 여기로 반영 된다.
  - `unpkg`: npm 라이브러리를 CDN으로 제공하고 있고 react와 react.dom을 확인 할 수 있다.
  - `browser.js`: 클라이언트 리액트 애플리케이션 코드를 번들링 할때 제공되는 리액트 자바스크립트 코드이다.
    - `__placeholder__`에서 HTML이 삽입되면 이후 이 코드가 실행되면서 필요한 이벤트 핸들러가 붙는다.
  ```
  <!DOCTYPE html>
  <html>
    <head>
      <meta charset="utf-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0" />
      <title>SSR Example</title>
    </head>
    <body>
      __placeholder__
      <script src="https://unpkg.com/react@17.0.2/umd/react.development.js"></script>
      <script src="https://unpkg.com/react-dom@17.0.2/umd/react-dom.development.js"></script>
      <script src="/browser.js"></script>
    </body>
  </html>
  ```
- `server.ts`
```
import { createReadStream } from 'fs'
import { createServer, IncomingMessage, ServerResponse } from 'http'

import { createElement } from 'react'
import { renderToNodeStream, renderToString } from 'react-dom/server'

import indexEnd from '../public/index-end.html'
import indexFront from '../public/index-front.html'
import html from '../public/index.html'

import App from './components/App'
import { fetchTodo } from './fetch'

const PORT = process.env.PORT || 3000

// http 서버 라우트 별 동작
async function serverHandler(req: IncomingMessage, res: ServerResponse) {
  const { url } = req

  switch (url) {
    case '/': {
      // App과 el의 노출 형태를 동일 시 하기 위해선 어쩔 수 없이 두번 호출 하는 형태로 일단 진행
      const result = await fetchTodo()

      const rootElement = createElement(
        'div',
        { id: 'root' },
        createElement(App, { todos: result }),
      )
      // html 생성 및 __placeholder__ 통해 서버 응답 제공
      const renderResult = renderToString(rootElement)
      const htmlResult = html.replace('__placeholder__', renderResult)

      res.setHeader('Content-Type', 'text/html')
      res.write(htmlResult)
      res.end()
      return
    }

    case '/stream': {
      res.setHeader('Content-Type', 'text/html')
      res.write(indexFront)

      // App과 el의 노출 형태를 동일 시 하기 위해선 어쩔 수 없이 두번 호출 하는 형태로 일단 진행
      const result = await fetchTodo()
      const rootElement = createElement(
        'div',
        { id: 'root' },
        createElement(App, { todos: result }),
      )

      const stream = renderToNodeStream(rootElement)
      stream.pipe(res, { end: false })
      stream.on('end', () => {
        res.write(indexEnd)
        res.end()
      })
      return
    }
    
    case '/browser.js': {
      // 브라우저에 제공되는 리액트 코드
      res.setHeader('Content-Type', 'application/javascript')
      createReadStream(`./dist/browser.js`).pipe(res)
      return
    }

    case '/browser.js.map': {
      // 브라우저에 제공되는 리액트 코드의 소스 맵
      res.setHeader('Content-Type', 'application/javascript')
      createReadStream(`./dist/browser.js.map`).pipe(res)
      return
    }

    default: {
      res.statusCode = 404
      res.end('404 Not Found')
    }
  }
}

function main() {
  // http 서버 생성
  createServer(serverHandler).listen(PORT, () => {
    console.log(`Server has been started ${PORT}...`) // eslint-disable-line no-console
  })
}

main()

==========================================================================
`/stram 라우터` 는 실행된 console에 아래 와 같이 넣고 확인한다.
const main = async () => {
  // 크롬에서 발생한 네트워크 요청을 복사해서 가져왔다.
  const response = await fetch('http://localhost:3000/stream');
  const reader = response.body.getReader(); // fetch api를 수동으로 읽어 드림

  while (true) {
    const { value, done } = await reader.read(); // 스트림을 순차적으로 청크 단위로 읽어옴.
    const str = new TextDecoder().decode(value); // 문자열 변환
    if (done) {
      break;
    }
    console.log(`=================================`);
    console.log(str);
  }

  console.log('Response fully received');
};

main();
```
- webpack.config.js
```
// @ts-check
/** @typedef {import('webpack').Configuration} WebpackConfig **/
const path = require('path')

const nodeExternals = require('webpack-node-externals')

/** @type WebpackConfig[] */
const configs = [
  {
    entry: {
      browser: './src/index.tsx',
    },
    output: {
      path: path.join(__dirname, '/dist'),
      filename: '[name].js',
    },
    resolve: {
      extensions: ['.ts', '.tsx'],
    },
    devtool: 'source-map',
    module: {
      rules: [
        {
          test: /\.tsx?$/,
          loader: 'ts-loader',
        },
      ],
    },
    externals: {
      react: 'React',
      'react-dom': 'ReactDOM',
    },
  },
  {
    entry: {
      server: './src/server.ts',
    },
    output: {
      path: path.join(__dirname, '/dist'),
      filename: '[name].js',
    },
    resolve: {
      extensions: ['.ts', '.tsx'],
    },
    devtool: 'source-map',
    module: {
      rules: [
        {
          test: /\.tsx?$/,
          loader: 'ts-loader', 
        },
        {
          test: /\.html$/,
          use: 'raw-loader', // html을 string으로 다시 읽어 드리기게 되어 replace 시 html을 재결합 할때 쓰임
        },
      ],
    },
    target: 'node',
    externals: [nodeExternals()],
  },
]

module.exports = configs
```
## 4.3 Next.js 톺아보기 (12.2.5)
- `package.json`
  - eslint-config-next: 구글과 협업해 만든 핵심 웹 지표 규칙이 내장되어 있음.
- `next.config.js`
  - reactStrictMode: react의 엄격 모드와 같으며 `13.5` 버전 부터는 디폴트가 `true`로 되어 있어 따로 끄려면 선언이 필요함.
  - swcMinify: 번들링과 컴파일러를 더욱 빠르게 수행 할 수 있으나 `15`버전 부터는 플래그 지원이 중단이 되고 기본 제공으로 변경되었음.
- `pages/_app.tsx`
  - 시작점으로 콘솔을 넣어봤을때, 서버 사이드 렌더링 이후 클라이언트에서 _app.tsx의 렌더링이 실행된다는 것을 짐작 할 수 있음
    - 에러 바운더리를 사용해 전역 에러 처리
    - 전역 css 선언
    - 모든 페이지에 공통 사용 혹은 제공해야 하는 데이터 제공
- `pages/_document.tsx`
  - _app.tsx가 애플리케이션 초기화라고 보면 여기는 html 초기화이다.
  - SEO 정보를 담을 수 있다.
    - `getServerSideProps`, `getStaticProps` 등 서버에서 사용 가능한 데이터 불러오는 함수는 여기서 사용은 못한다.
- `pages/_error.tsx`
  - 클라이언트에서 발생하는 에러 또는 서버에서 발생하는 500 에러를 처리할 목적으로 쓰인다. 개발 모드에서는 접근이 불가능하고 프로덕션 빌드에서 확인이 가능하다.
  - `500.tsx` 도 있지만 `_error.tsx`보다 `500.tsx`의 우선순위가 높
- `pages_404.tsx`
  - 페이지 이름 그대로 404 페이지를 정의할 수 있는 파일이다.
- 다이나믹 라우트
  ```
  // pages/hi/[...props].tsx
  import { useRouter } from 'next/router'
  import { useEffect } from 'react'
  import type { GetServerSideProps, NextPage } from 'next'
  
  type PageProps = {
    // URL 세그먼트들이 전부 문자열 배열로 들어온다. (없으면 빈 배열)
    props: string[]
  }
  
  const HiAll: NextPage<PageProps> = ({ props }) => {
    const { query } = useRouter() // 클라이언트에서 꺼내는 법
  
    useEffect(() => {
      // 서버에서 내려준 props와 클라이언트 query 비교용
      console.log('query from router:', query)
      console.log('serverProps:', props)
      console.log(JSON.stringify(query.props) === JSON.stringify(props)) // true
    }, [query, props])
  
    return (
      <>
        <h1>hi!</h1>
        <ul>
          {props.map((item, i) => (
            <li key={`${i}:${item}`}>{item}</li>
          ))}
        </ul>
      </>
    )
  }
  
  export const getServerSideProps: GetServerSideProps<PageProps> = async (context) => {
    // 서버에서 값 가져오는 법 (문자열 배열 or undefined)
    const { props } = context.query as { props?: string[] | string }
  
    // next/router의 catch-all은 항상 "배열" 형태로 다루는 게 안전함
    const asArray = Array.isArray(props) ? props : props ? [props] : []
  
    return {
      props: {
        props: asArray,
      },
    }
  }
  
  export default HiAll
  ```
- 서버 라우팅과 클라이언트 라우팅의 차이
  - Next.js는 서버 사이드 렌더링을 수행하지만 동시에 SPA처럼 클라이언트 라우팅 또한 수행한다.
  ```
  export default function Hello() {
    console.log(typeof window === "undefined" ? "서버" : "클라이언트");
  
    return <>hello</>;
  }
  
  export const getServerSideProps = () => {
    return {
      props: {},
    };
  };
  
  // Home.tsx
  import type { NextPage } from "next";
  import Link from "next/link";
  
  const Home: NextPage = () => {
    return (
      <ul>
        <li>
          {/* next의 eslint 룰을 잠시 끄기 위해 추가했다. */}
          {/* eslint-disable-next-line */}
          <a href="/hello">A 태그로 이동</a>
        </li>
        <li>
          {/* 차이를 극적으로 보여주기 위해 해당 페이지의 리소스를 미리 가져오는 prefetch를 잠시 꺼두었다. */}
          <Link prefetch={false} href="/hello">
            next/link로 이동
          </Link>
        </li>
      </ul>
    );
  };
  
  export default Home;
  ```
  - a태그로 이동하는 것과 Link태그로 이동하는 것은 차이점이 있다. 
    - a 태그로 이동하는 경우에는 페이지를 만드는데 필요한 모든 리소스를 처음부터 다 가져오고, 서버와 클라이언트가 동시에 찍힌다.
    - Link 태그로 이동하는 경우에는 클라이언트에서만 필요한 자바스크립트만 불러온 뒤 클라이언트 라우팅/렌더링 방식으로 동작한다.

  - 페이지에서 getServerSideProps를 제거하면 어떻게 될까?
    - Hello컴포넌트 에 getServerSideProps를 제거한 뒤 빌드 후 실행해보면 a태그, Link태그에 상관없이 서버에 로그가 남지 않는다.
      - 정적인 페이지로 분류되어 빌드 시점에 미리 트리쉐이킹을 해버렸기 때문이다.
      - Next.js가 단순히 서버 사이드 프레임워크라 하여 모든 작업이 서버에서 일어나는 것이 아니라는 점이다.

- `/pages/api/hello.ts`
  - 서버의 API를 정의하는 폴더
  - 서버에서만 실행되고 window, document 등 브라우저에서 접근할 수 있는 코드를 작성하면 문제가 발생 (BFF, CORS 우회, 풀스택 구축 등)

### 4.3.3 Data Fetching
 - `Next.js`에서는 데이터 불러오기 전략이 있는데, 이를 Data Fetching 이라고 한다.
   - `pages/` 폴더에 있는 라우팅이 되는 파일에서만 사용이 가능
   - 예약어로 지정되어 있으며, 정해진 함수명으로 export 하여 외부로 보내야한다.
 - `getStaticPaths`와 `getStaticProps`
   - 정적으로 결정된 페이지를 보여주고자 할 때 사용되는 함수로 결과를 바탕으로 페이지에 렌더링을 한다.
   - 즉, 빌드 시점에 모든 조합을 페이지로 렌더링이 가능하다.
     - 다이나이믹 라우트로 들어오는 경우 상품과 같은 여러 이미지의 목록 항목을 미리 먼저 뿌려주고자 할때 사용 하면 된다.
   - `getStaticPaths`는 정해진 값으로만 허용
     - `fallback`
       - false: `getStaticPaths`에서 리턴하지 않은 페이지는 모두 404로 연결
       - true: getStaticPaths에서 리턴하지 않은 페이지에 접속 시,
         - 먼저 사용자에게 `fallback` 페이지를 보여줌
         - 서버에서 static하게 페이지를 생성함
         - 해당 페이지를 사용자에게 보여줌
         - 다음부터 해당 페이지로 접속하는 사용자에게는 static한 페이지를 보여줌 (캐싱)
       - blocking: getStaticPaths에서 리턴하지 않은 페이지에 접속 시, 즉 fallback 페이지나 로딩 화면이 없다.
         - 사용자에게 server side rendering한 static 페이지를 보여줌
         - 다음부터 해당 페이지로 접속하는 사용자에게는 server side rendering한 static 페이지를 보여줌
     ```
     export const getStaticPaths: getStaticPaths = async () => {
       retrun {
         // 1번과 2번을 사전 빌드를 할 수 있도록 요청 값을 정해 둔다.
         paths: [{params: {id: '1'}, {params: {id: '2'}],
         fallback: false, // false, true, blocking
       }
     }
     ```
   - `getStaticProps`는 데이터 요청 수행을 통해 props로 반환
     ```
     export const getStaticProps: getStaticProps = async ({params}) => {
       const {id} = params
       // 각 id별 데이터 가져오기
       const post = await = fetchPost(id)
     
       retrun {
         props: {post},
         revalidate: 60 * 60 // 경우에 따라 ISR로 제공해야 할 수 도 있다.
       }
     }

     export default function Post({ post }) {
       const router = useRouter()
       if(router.isFallback) {
         // fallback true 시......
         return <div>isFallbackisFallbackisFallbackisFallback</div>
       }
       return <h1>{post.title}</h1>;
     }
     ```
- `getServerSideProps`
   - 페이지 진입 전 이 함수가 실행됨
   - 응답 값에 따라 페이지의 루트 컴포넌트에 props를 반환하거나, 다른 페이지로 리다이렉트 시킬 수 있다.
     ```
     export const GetServerSideProps: GetServerSideProps = async (context) => {
       // context.query.id 를 통해 /post/[id] 같은 경로의 id 값에 접근이 가능하고 이 값을 통해 렌더링 수행
       const {
         query: { id = '' },
       } = context;
       const post = await fetchPost(id.toString())

       	if (!post) {
      		redirect: {
      			destination: '/404' // 리다이렉트 처리를 하는것은 클리이언트에서 하는 것보다 서버에서 하는것이 더 자연스럽다
      		}
      	}

       return {
         props: { post },
       }
     }
    
     export defautl function Post({ post }: { post: Post }) {
     }
     ```
   - HTML이 getSererSideProps 기반으로 렌더링되어 있고 여기서 눈여겨봐야 할 것은 id="__NEXT_DATA__" 의 script 이다.
     - 서버에서 fetch등으로 렌더링에 필요한 정보를 가져온다.
     - 가져온 정보를 기반으로 HTML을 완성
     - 위 정보를 기반으로 클라이언트에 제공
     - 클라이언트에서 hydrate작업을 한다. (DOM에 리액트 라이프사이클과 이벤트 핸들러를 추가)
     - hydrate로 만든 리액트 컴포넌트 트리와 서버에서 만든 HTML이 다르면 불일치 에러를 뱉는다.
     - fetch등을 이용해 정보를 가져온다.
     ```
       <body>
        <script id="__NEXT_DATA__" type="application/json">
          {
            "props": {
              "pageProps": {
                "post": { "title": "안녕하세요", "contents": "반갑습니다." }
              },
              "__N_SSP": true
            },
            "page": "/post/[id]",
            "query": { "id": "1" },
            "buildId": "development",
            "isFallback": false,
            "gssp": true,
            "scriptLoader": []
          }
        </script>
      </body>
      ```
    
  - 정리
    - 가져온 정보를 HTML에 script형태로 내려주어 1 ~ 6 번 작업을 반복하지 않아도 되어 불필요한 요청을 막고 시점 차이로 인한 결과물의 차이도 막을 수 있다
    - **. 즉, 6번에서 재용청하는 대신에 script 를 그대로 읽어서 동일한 데이터를 가져오는 것이다. 또한 window 객체에도 저장해 둔다.**
    - JSON으로만 props를 제공해주기에 JSON으로 직렬화할 수 없는 값(class, Date, 함수 등)은 제공이 불가능하다.
    - getServerSideProps는 매 페이지를 호출할때마다 실행되고, 이 실행이 끝나기 전까지는 사용자에게 어떠한 HTML도 보여줄 수 없기 때문에 최대한 간결하게 작성해야 한다.

- `getInitialProps`
  - 제한적 사용만 가능하며, `_app.tsx`, `_error.tsx`와 같은 일부 페이지에서 밖에 사용이 안됨.
  - 이전 예약어와 달리 `props` 객체를 반환하는 것이 아니라 객체를 반환한다.
  - 상황에 따라 클라이언트나 서버에서 둘다 호출이 된다. 
  - 최초 서버 상태의 첫 진입 시점을 판단 할 수 있다.
    - `_app.tsx`에서 판단이 가능한데 `getServerSideProps`가 존재하는 라우트에 접근할때 `_next/data/...../블라블라.json`으로 요청하게 되는데 없다면 클라이언트 렌더링 발생이라 판단하면 된다.
    ```
    import Link from "next/link";
    import { NextPageContext } from "next";

    export default function Todo({ todo }: { todo: { userId: number; id: number; title: string; completed: boolean } }) {
      return (
        <>
          <h1>{todo.title}</h1>
          <ul>
            <li>
              <Link href="/todo/1">1번</Link>
            </li>

            <li>
              <Link href="/todo/2">2번</Link>
            </li>

            <li>
              <Link href="/todo/3">3번</Link>
            </li>
          </ul>
        </>
      );
    }

    Todo.getInitialProps = async (ctx: NextPageContext) => {
      const {
        query: { id = "" },
      } = ctx;
      const response = await fetch(`https://jsonplaceholder.typicode.com/todos/${id}`);
      const result = await response.json();
      return { todo: result };
    };
    ```

### 4.3.5
