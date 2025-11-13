## 8.1 ESLint를 활용한 정적 코드 분석
### 8.1.1 ESLint 살펴보기
  - ESLint 코드 분석 원리
    - 자바스크립트 코드를 문자열로 읽음
    - 자바스크립트 코드를 분석할 숭 있는 파서로 코드를 구조화
    - 구조화 트리(AST- Abstract Syntax Tree)를 기준으로 각종 규칙 대조
    - 대조 위반 시 report 및 fix 수행
  - 분석 원리 살펴보기
    - ESLint는 기본값으로 `espree`를 사용한다
      - ATS explorer를 통해 자바스크립트/타입스크립트 분석 가능
      - typescript는 `@typescript-eslint/typescript-estree` 파서로 분석 가능
  ```
  function hello(str) {}

  // 파서 실행 하면 한줄 밖에 안되지만 여러 정보가 들어 있고 eslint나 prettier는 이런 정밀 정보가 필요하다.
  {
    "type": "Program",
    "start": 0,
    "end": 22,
    "body": [
      {
        "type": "FunctionDeclaration",
        "start": 0,
        "end": 22,
        "id": {
          "type": "Identifier",
          "start": 9,
          "end": 14,
          "name": "hello"
        },
        "expression": false,
        "generator": false,
        "async": false,
        "params": [
          {
            "type": "Identifier",
            "start": 15,
            "end": 18,
            "name": "str"
          }
        ],
        "body": {
          "type": "BlockStatement",
          "start": 20,
          "end": 22,
          "body": []
        }
      }
    ],
    "sourceType": "module"
  }
  ```

  - no-debugger 규칙 적용해서 espree로 분석해보기
  ```
  // test 코드 작성
  function test() {
    debugger;
  }

  // no-debugger 규칙 적용하기
  module.exports = {
    meta: {
      type: 'problem',
      docs: {
        description: 'Disallow the use of `debugger`',
        recommended: true,
        url: 'https://eslint.org/docs/rules/no-debugger',
      },
      fixable: null,
      schema: [],
      messages: {
        unexpected: "Unexpected 'debugger' statement.",
      },
    },

    create(context) {
      return {
        DebuggerStatement(node) {
          context.report({
            node,
            messageId: 'unexpected',
          });
        },
      };
    },
  };

  // debugger만 있는 코드 espree 분석
  {
    "type": "Program",
    "start": 0,
    "end": 30,
    "loc": {
      "start": {
        "line": 1,
        "column": 0
      },
      "end": {
        "line": 3,
        "column": 1
      }
    },
    "comments": [],
    "range": [
      0,
      30
    ],
    "sourceType": "module",
    "body": [
      {
        "type": "FunctionDeclaration",
        "start": 0,
        "end": 30,
        "loc": {
          "start": {
            "line": 1,
            "column": 0
          },
          "end": {
            "line": 3,
            "column": 1
          }
        },
        "id": {
          "type": "Identifier",
          "start": 9,
          "end": 13,
          "loc": {
            "start": {
              "line": 1,
              "column": 9
            },
            "end": {
              "line": 1,
              "column": 13
            },
            "identifierName": "test"
          },
          "name": "test",
          "range": [
            9,
            13
          ],
          "_babelType": "Identifier"
        },
        "generator": false,
        "expression": false,
        "async": false,
        "params": [],
        "body": {
          "type": "BlockStatement",
          "start": 16,
          "end": 30,
          "loc": {
            "start": {
              "line": 1,
              "column": 16
            },
            "end": {
              "line": 3,
              "column": 1
            }
          },
          "body": [
            {
              "type": "DebuggerStatement",
              "start": 19,
              "end": 28,
              "loc": {
                "start": {
                  "line": 2,
                  "column": 1
                },
                "end": {
                  "line": 2,
                  "column": 10
                }
              },
              "range": [
                19,
                28
              ],
              "_babelType": "DebuggerStatement"
            }
          ],
          "range": [
            16,
            30
          ],
          "_babelType": "BlockStatement"
        },
        "range": [
          0,
          30
        ],
        "_babelType": "FunctionDeclaration",
        "defaults": []
      }
    ]
  }

  // 결과
  // Unexpected 'debugger' statement. (at 2:2)
      debugger;
  // -^

  // Fixed output follows:
  // --------------------------------------------------------------------------------
  function test() {
    debugger;
  }
  ```
### 8.1.2 `eslint-plugin`과 `eslint-config`
  - `eslint-plugin`: 규칙 패키지, 리액트 규칙은 `eslint-plugin-react` 예를 들면 jsx 배열 키 선언를 사용 안하면 `react/jsx-key`에 의해 경고 발생
  - `eslint-config`: 특정 프레임워크나 도메인과 관련된 규칙을 묶어서 제공하는 패키지이다. 

  - `eslint` 커스텀 규칙 만들어보기
  ```
  rules: {
    'no-restricted-imports': [
      'error',
      {
        paths: [
          name: 'lodash',
          message: 'lodash'는 CommonJS로 작성되어 트리쉐이킹이 되지 않아 번들 사이즈를 크게합니다.
        ]
      }
    ]
  }
  ```

  - 완전히 새로운 규칙 만들기
    - new Date를 금지시켜 ServerDate라는 함수를 사용하게 만들자.
      - ExpressionStatement.expression은 어떤 표현이 들어가있는지 확인
      - ExpressionStatement.expression.type 은 해당 표현이 어떤 타입인지 (생성자 이므로 NewExpression)
      - ExpressionStatement.expression.callee 은 생성자의 이름을 나타냄
      - ExpressionStatement.expression.arguments 은 생성자에 전달하는 인수
  ```
  {
    "type": "Program",
    "start": 0,
    "end": 10,
    "body": [
      {
        "type": "ExpressionStatement",
        "start": 0,
        "end": 10,
        "expression": {
          "type": "NewExpression",
          "start": 0,
          "end": 10,
          "callee": {
            "type": "Identifier",
            "start": 4,
            "end": 8,
            "name": "Date"
          },
          "arguments": []
        }
      }
    ],
    "sourceType": "module"
  }


  // eslint 커스텀
  /**
  * @fileoverview yceffort
  * @author yceffort
  */
  "use strict";

  // ------------------------------------------------------------------------------
  // Rule Definition
  // ------------------------------------------------------------------------------

  /**
  *
  * @type {import('eslint').Rule.RuleModule}
  */
  module.exports = {
      meta: {
          type: "suggestion",
          docs: {
              description: "disallow use of the new Date()",
              recommended: false,
          },
          fixable: "code",
          schema: [],
          messages: {
              message:
                  "new Date()는 클라이언트에서 실행시 해당 기기의 시간에 의존적이라 정확하지 않습니다. 현재 시간이 필요하다면 ServerDate()를 사용해주세요.",
          },
      },
      create: function (context) {
          return {
              NewExpression: function (node) {
                  if (
                      node.callee.name === "Date" &&
                      node.arguments.length === 0
                  ) {
                      context.report({
                          node,
                          messageId: "message",
                          fix: function (fixer) {
                              return fixer.replaceText(node, "ServerDate()");
                          },
                      });
                  }
              },
          };
      },
  };
  ```
### 8.1.4 ESLint 주의할 점
  - `Prettier`와의 충돌
    - eslint와 pettier의 목적을 명확히 다르다. eslint는 코드의 잠재적인 문제가 될 수 있는 부분을 분석해 준다면, pettier는 포매팅과 관련된 작업을 담당한다.
    - 충돌이 되지 않게끔 규칙을 잘 설정하는것
    - 자바스크립트나 타입스크립트는 ESLint에 그외는 Prettier에게 맡기기 (추가적으로 필요하면 eslint-plugin-prettier를 사용)

## 8.2 리액트 팀이 권장하는 리액트 테스트 라이브러리
 - React Testing Library ?
  - DOM Testing Library로 jsdom 기반으로 이루어져 있으며, 순수한 자바스크립트로 작성되었다.
  - 실제 리액트 컴포넌트를 렌더링 하지 않고 확인 가능함


## 8.2.3 리액트 컴포넌트 테스트 코드 작성하기
  - 원본 소스
  ```
  // App.tsx
  import logo from "./logo.svg";
  import "./App.css";
  import StaticComponent from "./components/StaticComponent";

  function App() {
      return (
          <div className="App">
              <header className="App-header">
                  <img src={logo} className="App-logo" alt="logo" />
                  <p>
                      Edit <code>src/App.tsx</code> and save to reload.
                  </p>
                  <a
                      className="App-link"
                      href="https://reactjs.org"
                      target="_blank"
                      rel="noopener noreferrer"
                  >
                      Learn React
                  </a>
              </header>
          </div>
      );
  }

  export default App;
  ```

  - 테스트 소스
    - `*.test.{t|s}jsx`를 준수하여 작성
    - 특정한 무언가를 지닌 HTML 요소가 있는지 여부 하기 위한 방법 3가지
      - getBy...: 인수 조건에 맞는 요소를 반환 해당 요소가 없거나 두개 이상은 에러 발생. 복수 개는 getAllBy...
      - findBy: 위와 비슷하지만 Promise를 반환 즉, 비동기를 찾음. 기본적으로 1000ms의 타임아웃. 복수개는 findByAll...
      - queryBy: 인수 조건에 맞는 요소를 반환하는 대신 찾지 못하면 null 반환. 에러 반환과 달리 에러 발생이 필요없는 경우 사용 복수개는 queryAllBy...
  ```
  import { render, screen } from "@testing-library/react";

  import App from "./App";

  test("renders learn react link", () => {
      render(<App />);
      // 'learn react'라는 문자열을 가진 DOM을 찾는다.
      const linkElement = screen.getByText(/learn react/i);
      // linkElement요소가 document내부에 있는지 확인한다.
      expect(linkElement).toBeInTheDocument();
  });
  ```

  - 별도의 상태가 존재하지 않아 항상 같은 결과를 반환하는 정적 컴포넌트 테스트
    - beforeEach: 각 테스트(it)를 수행하기 전에 실행하는 함수
    - describe: 비슷한 속성을 가진 테스트를 하나의 그룹으로 묶는 역할을 한다.
    - it: test의 축약어로 테스트 코드 항목이다
    - testId: dom 요소에 testId 데이터셋(= 임의의 정보를 추가 할 수 있는 HTML 속성 data-) 선언 할때 `getByTestId`, `findByTestId` 등으로 선택할 수 있다
  ```
  import { memo } from "react";

  const AnchorTagComponent = memo(function AnchorTagComponent({
      name,
      href,
      targetBlank,
  }: {
      name: string;
      href: string;
      targetBlank?: boolean;
  }) {
      return (
          <a
              href={href}
              target={targetBlank ? "_blank" : undefined}
              rel="noopener noreferrer"
          >
              {name}
          </a>
      );
  });

  export default function StaticComponent() {
      return (
          <>
              <h1>Static Component</h1>
              <div>유용한 링크</div>
              <!-- testId 선언 -->
              <ul data-testid="ul" style={{ listStyleType: "square" }}>
                  <li>
                      <AnchorTagComponent
                          targetBlank
                          name="리액트"
                          href="https://reactjs.org"
                      />
                  </li>
                  <li>
                      <AnchorTagComponent
                          targetBlank
                          name="네이버"
                          href="https://www.naver.com"
                      />
                  </li>
                  <li>
                      <AnchorTagComponent
                          name="블로그"
                          href="https://yceffort.kr"
                      />
                  </li>
              </ul>
          </>
      );
  }

  // 테스트 코드 작성

  import { render, screen } from "@testing-library/react";

  import StaticComponent from "./index";

  beforeEach(() => {
      render(<StaticComponent />);
  });

  describe("링크 확인", () => {
      // test 코드
      it("링크가 3개 존재한다.", () => {
          const ul = screen.getByTestId("ul");
          expect(ul.children.length).toBe(3);
      });

      it("링크 목록의 스타일이 square다.", () => {
          const ul = screen.getByTestId("ul");
          expect(ul).toHaveStyle("list-style-type: square;");
      });
  });

  describe("리액트 링크 테스트", () => {
      it("리액트 링크가 존재한다.", () => {
          const reactLink = screen.getByText("리액트");
          expect(reactLink).toBeVisible();
      });

      it("리액트 링크가 올바른 주소로 존재한다.", () => {
          const reactLink = screen.getByText("리액트");

          expect(reactLink.tagName).toEqual("A");
          expect(reactLink).toHaveAttribute("href", "https://reactjs.org");
      });
  });

  describe("네이버 링크 테스트", () => {
      it("네이버 링크가 존재한다.", () => {
          const naverLink = screen.getByText("네이버");
          expect(naverLink).toBeVisible();
      });

      it("네이버 링크가 올바른 주소로 존재한다.", () => {
          const naverLink = screen.getByText("네이버");

          expect(naverLink.tagName).toEqual("A");
          expect(naverLink).toHaveAttribute("href", "https://www.naver.com");
      });
  });

  describe("블로그 링크 테스트", () => {
      it("블로그 링크가 존재한다.", () => {
          const blogLink = screen.getByText("블로그");
          expect(blogLink).toBeVisible();
      });

      it("블로그 링크가 올바른 주소로 존재한다.", () => {
          const blogLink = screen.getByText("블로그");

          expect(blogLink.tagName).toEqual("A");
          expect(blogLink).toHaveAttribute("href", "https://yceffort.kr");
      });

      it("블로그는 같은 창에서 열려야 한다.", () => {
          const blogLink = screen.getByText("블로그");
          expect(blogLink).not.toHaveAttribute("target");
      });
  });
  ```

  - 동적 컴포넌트 테스트
    - input은 최대 20자까지, 한글 입력만 가능하도록 제한한다.
    - 버튼은 글자가 있을때만 able처리가 된다.
    - 버튼을 클릭하면 alert창을 띄운다.
  ```
  import { useState } from "react";

  export function InputComponent() {
      const [text, setText] = useState("");

      function handleInputChange(event: React.ChangeEvent<HTMLInputElement>) {
          const rawValue = event.target.value;
          const value = rawValue.replace(/[^A-Za-z0-9]/gi, "");
          setText(value);
      }

      function handleButtonClick() {
          alert(text);
      }

      return (
          <>
              <label htmlFor="input">아이디를 입력하세요.</label>
              <input
                  aria-label="input"
                  id="input"
                  value={text}
                  onChange={handleInputChange}
                  maxLength={20}
              />
              <button onClick={handleButtonClick} disabled={text.length === 0}>
                  제출하기
              </button>
          </>
      );
  }


  import { fireEvent, render } from "@testing-library/react";
  import userEvent from "@testing-library/user-event";

  import { InputComponent } from ".";

  describe("InputComponent 테스트", () => {
      // 모든 테스트에 필요한 컴포넌트를 렌더링 한 뒤에 필요로 하는 값을 리턴해준다.
      const setup = () => {
          const screen = render(<InputComponent />);
          const input = screen.getByLabelText("input") as HTMLInputElement;
          const button = screen.getByText(/제출하기/i) as HTMLButtonElement;
          return {
              input,
              button,
              ...screen,
          };
      };

      it("input의 초기값은 빈 문자열이다.", () => {
          const { input } = setup();
          expect(input.value).toEqual("");
      });

      it("input의 최대길이가 20자로 설정되어 있다.", () => {
          const { input } = setup();
          expect(input).toHaveAttribute("maxlength", "20");
      });

      it("영문과 숫자만 입력된다.", () => {
          const { input } = setup();
          const inputValue = "안녕하세요123";
          // userEvent.type 사용자 타이핑 흉내 메소드
          userEvent.type(input, inputValue);
          expect(input.value).toEqual("123");
      });

      it("아이디를 입력하지 않으면 버튼이 활성화 되지 않는다.", () => {
          const { button } = setup();
          expect(button).toBeDisabled();
      });

      it("아이디를 입력하면 버튼이 활성화 된다.", () => {
          const { button, input } = setup();

          const inputValue = "helloworld";
          userEvent.type(input, inputValue);

          expect(input.value).toEqual(inputValue);
          expect(button).toBeEnabled();
      });

      it("버튼을 클릭하면 alert가 해당 아이디로 뜬다.", () => {
          // spyOn: 어떠한 특정 메서드를 오염시키지 않고 실행이 됐는지, 어떤 인수로 실행됐는지 등 실행과 관련된 정보만 얻고 싶을 때 사용
          const alertMock = jest
              .spyOn(window, "alert")
              .mockImplementation((_: string) => undefined); // 해당 메서드에 대한 모킹 구현을 도와준다. 모의 함수로 구현된 함수이지만 이 함수가 실행됐는지 등의 정보는 확인할 수 있도록 도와준다.

          const { button, input } = setup();
          const inputValue = "helloworld";

          userEvent.type(input, inputValue);
          fireEvent.click(button);

          expect(alertMock).toHaveBeenCalledTimes(1);
          expect(alertMock).toHaveBeenCalledWith(inputValue);
      });
  });
  ```

  - 비동기 이벤트가 발생하는 컴포넌트
    - 데이터를 불러오는 데 성공하면 응답값 중 하나를 노출하지만, 실패하면 에러 문구를 노출하는 컴포넌트를 테스트 하자
    - spyOn에서 window에 기본적으로 제공하는 fetch를 사용하게 되면 모든 시나리오에 대한 ok, status, json 값을 전부 테스트 하기에 코드가 길고 복잡함으로 MSW를 사용한다.
  ```
  import { MouseEvent, useState } from "react";

  interface TodoResponse {
      userId: number;
      id: number;
      title: string;
      completed: false;
  }

  export function FetchComponent() {
      const [data, setData] = useState<TodoResponse | null>(null);
      const [error, setError] = useState<number | null>(null);

      async function handleButtonClick(e: MouseEvent<HTMLButtonElement>) {
          const id = e.currentTarget.dataset.id;

          const response = await fetch(`/todos/${id}`);

          if (response.ok) {
              const result: TodoResponse = await response.json();
              setData(result);
          } else {
              setError(response.status);
          }
      }

      return (
          <div>
              <p>{data === null ? "불러온 데이터가 없습니다." : data.title}</p>

              {error && (
                  <p style={{ backgroundColor: "red" }}>에러가 발생했습니다</p>
              )}

              <ul>
                  {Array.from({ length: 10 }).map((_, index) => {
                      const id = index + 1;
                      return (
                          <button
                              key={id}
                              data-id={id}
                              onClick={handleButtonClick}
                          >
                              {`${id}번`}
                          </button>
                      );
                  })}
              </ul>
          </div>
      );
  }
  ```
  - setupServer: MSW에서 제공하는 메서드로 이름 그대로 서버를 만드는 역할
  - afterEach, resetHandlers: 각 테스트가 진행될 때마다 서버를 종료시킨다. setupServer의 기본 설정으로 되돌리는 역할을 한다.
  ```
  import { fireEvent, render, screen } from '@testing-library/react'
  import { rest } from 'msw'
  import { setupServer } from 'msw/node'

  import { FetchComponent } from '.'

  const MOCK_TODO_RESPONSE = {
    userId: 1,
    id: 1,
    title: 'delectus aut autem',
    completed: false,
  }

  // MSW에서 제공하는 메서드로 이름 그대로 서버를 만드는 역할
  const server = setupServer(
    rest.get('/todos/:id', (req, res, ctx) => {
      const todoId = req.params.id

      if (Number(todoId)) {
        return res(ctx.json({ ...MOCK_TODO_RESPONSE, id: Number(todoId) }))
      } else {
        return res(ctx.status(404))
      }
    }),
  )

  beforeAll(() => server.listen())
  // 각 테스트가 진행될 때마다 서버를 종료시킨다.
  afterEach(() => server.resetHandlers())
  afterAll(() => server.close())

  beforeEach(() => {
    render(<FetchComponent />)
  })

  describe('FetchComponent 테스트', () => {
    it('데이터를 불러오기 전에는 기본 문구가 뜬다.', async () => {
      const nowLoading = screen.getByText(/불러온 데이터가 없습니다./)
      expect(nowLoading).toBeInTheDocument()
    })

    it('버튼을 클릭하면 데이터를 불러온다.', async () => {
      const button = screen.getByRole('button', { name: /1번/ })
      fireEvent.click(button)

      const data = await screen.findByText(MOCK_TODO_RESPONSE.title)
      expect(data).toBeInTheDocument()
    })

  // 서버에 실패한 경우 임의로 상태를 변경하므로 리셋이 필요하다.
    it('버튼을 클릭하고 서버요청에서 에러가 발생하면 에러문구를 노출한다.', async () => {
      server.use(
        rest.get('/todos/:id', (req, res, ctx) => {
          return res(ctx.status(503))
        }),
      )

      const button = screen.getByRole('button', { name: /1번/ })
      fireEvent.click(button)

      const error = await screen.findByText(/에러가 발생했습니다/)
      expect(error).toBeInTheDocument()
    })
  })
  ```

  - 사용자 정의 훅 테스트
    - `react-hooks-testing-library` 아래와 같은 단점으로 사용하여 해당 훅의 비지니스를 테스트 하자.
      - 사용자 정의 훅이 들어가 있는 컴포넌트 테스트 시 테스트 케이스가 많아 전부 확인이 어려운점
      - 사용자 정의 훅을 사용하는 모든 컴포넌트를 찾아야하는 단점
      - 아래 글을 써놨지만 18버전 부터는 `"@testing-library/react"`로 통합됨
  ```
  import { useEffect, useRef } from "react";

  export type Props = Record<string, unknown>;

  export const CONSOLE_PREFIX = "[useEffectDebugger]";

  export default function useEffectDebugger(
      componentName: string,
      props?: Props
  ) {
      const prevProps = useRef<Props | undefined>();

      useEffect(() => {
          // process.env.NODE_ENV === ‘production’ 인 경우에는 로깅을 하지 않는다.
          if (process.env.NODE_ENV === "production") {
              return;
          }

          const prevPropsCurrent = prevProps.current;

          if (prevPropsCurrent !== undefined) {
              // 이전 props와 현재 props들의 모든 key를 가져온다.
              const allKeys = Object.keys({ ...prevProps.current, ...props });

              // 키들을 통해서 props를 얕은 비교를 통해 같은지 확인
              const changedProps: Props = allKeys.reduce<Props>((result, key) => {
                  const prevValue = prevPropsCurrent[key];
                  const currentValue = props ? props[key] : undefined;

                  // 같지 않다면 전에 값과 현재값을 담은 result를 내보내줘 무엇이 렌더링 되었는지 확인시켜준다.
                  if (!Object.is(prevValue, currentValue)) {
                      result[key] = {
                          before: prevValue,
                          after: currentValue,
                      };
                  }
                  return result;
              }, {});

              if (Object.keys(changedProps).length) {
                  // eslint-disable-next-line no-console
                  console.log(CONSOLE_PREFIX, componentName, changedProps);
              }
          }

          prevProps.current = props;
      });
  }
  ```

  - 만약 프로젝트가 18 버전 미만이라면 `@testing-library/react-hooks`를 사용한다.
  - 18 이후 부터는 `"@testing-library/react"`로 통합
  ```

  import { renderHook } from "@testing-library/react";

  import useEffectDebugger, { CONSOLE_PREFIX } from "./useEffectDebugger";

  const consoleSpy = jest.spyOn(console, "log");
  const componentName = "TestComponent";

  describe("useEffectDebugger", () => {
      afterAll(() => {
          // eslint-disable-next-line @typescript-eslint/ban-ts-comment
          // @ts-ignore
          process.env.NODE_ENV = "development";
      });

      it("props가 없으면 호출되지 않는다.", () => {
          renderHook(() => useEffectDebugger(componentName));

          expect(consoleSpy).not.toHaveBeenCalled();
      });

      it("최초에는 호출되지 않는다.", () => {
          const props = { hello: "world" };

          renderHook(() => useEffectDebugger(componentName, props));

          expect(consoleSpy).not.toHaveBeenCalled();
      });

      it("props가 변경되지 않으면 호출되지 않는다.", () => {
          const props = { hello: "world" };

          const { rerender } = renderHook(() =>
              useEffectDebugger(componentName, props)
          );

          expect(consoleSpy).not.toHaveBeenCalled();

          rerender();

          expect(consoleSpy).not.toHaveBeenCalled();
      });

      it("props가 변경되면 다시 호출한다.", () => {
          const props = { hello: "world" };

          const { rerender } = renderHook(
              ({ componentName, props }) =>
                  useEffectDebugger(componentName, props),
              {
                  initialProps: {
                      componentName,
                      props,
                  },
              }
          );

          const newProps = { hello: "world2" };

          rerender({ componentName, props: newProps });

          expect(consoleSpy).toHaveBeenCalled();
      });

      it("props가 변경되면 변경된 props를 정확히 출력한다", () => {
          const props = { hello: "world" };

          const { rerender } = renderHook(
              ({ componentName, props }) =>
                  useEffectDebugger(componentName, props),
              {
                  initialProps: {
                      componentName,
                      props,
                  },
              }
          );

          const newProps = { hello: "world2" };

          rerender({ componentName, props: newProps });

          expect(consoleSpy).toHaveBeenCalledWith(
              CONSOLE_PREFIX,
              "TestComponent",
              {
                  hello: { after: "world2", before: "world" },
              }
          );
      });

      it("객체는 참조가 다르다면 변경된 것으로 간주한다", () => {
          const props = { hello: { hello: "world" } };
          const newProps = { hello: { hello: "world" } };

          const { rerender } = renderHook(
              ({ componentName, props }) =>
                  useEffectDebugger(componentName, props),
              {
                  initialProps: {
                      componentName,
                      props,
                  },
              }
          );

          rerender({ componentName, props: newProps });

          // 이후 호출
          expect(consoleSpy).toHaveBeenCalled();
      });

      it("process.env.NODE_ENV가 production이면 호출되지 않는다", () => {
          // eslint-disable-next-line @typescript-eslint/ban-ts-comment
          // @ts-ignore
          // 환경변수 값 강제로 주입
          process.env.NODE_ENV = "production";

          const props = { hello: "world" };

          const { rerender } = renderHook(
              ({ componentName, props }) =>
                  useEffectDebugger(componentName, props),
              {
                  initialProps: {
                      componentName,
                      props,
                  },
              }
          );

          const newProps = { hello: "world2" };

          rerender({ componentName, props: newProps });

          expect(consoleSpy).not.toHaveBeenCalled();
      });
  });
  ```