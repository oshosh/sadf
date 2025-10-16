## 3.1 리액트 모든 훅 파헤치기
### 3.1.1 `useState`
  - 핵심은 클로저를 사용하여 구현 되었을 것을 짐작 할 수 있음.
  - 게으른 초기화는 함수로 넘겨서 초기화 해야 리렌더 시 재계산 되는 것을 막을 수 있음 초기 한번만 실행 시키는게 좋다.
    - `const [value, setValue] = useState(()=> 복잡한 계산 함수)` 
  - 대략적인 `pseudo code` (실제 구현은 다름)
  ```
  const MyReact = (function () {
    const global = {}
    let index = 0

    function useState(initalState) {
      if(!global.states) {
        // 애플리케이션 전체의 states 배열을 초기화
        // 최초 접근 시 빈 배열 초기화
        global.states = []
      }

      // states 정보를 조회해서 현재 상태값 체크 없다면 초기값 설정
      const currentState = global.states[index] || initalState
      global.states[index] = currentState

      const setState = (fuction () {
        // 클로저! 현재 index를 기억하여 이후 계속 동일한 index에 접근이 가능하도록 함
        let currentIndex = index
  
        return function (value) {
          gloval.states[currentIndex] = value
          // 랜더 코드 발생....
        }
      })()
      index = index + 1
      return [currentState, setState]
    }
  
    function Component() {
      const [value, setValue] = useState(0)
    }
  })()
  ```
### 3.1.2 `useEffect`
  - 클린업 함순가 왜 이벤트를 등록하고 지울 때 사용 해야하는가?
    - 클린업 함수는 새로운 값을 기반으로 렌더링 뒤에 실행되만 변경된 값을 읽는게 아니라 이전 값을 보고 실행한다.
    - useEffect의 내부에서는 window 객체 접근 하는 의존 코드를 사용해도 된다.
    - 클라이언트 사이드 보장이 가능하다.
    - `Object.is`를 통해 얕은 비교로 의존성 배열의 변경 여부를 체크하고 변경 사항을 다시 저장 하여 다시 비교를 수행함.
    - 만약 비동기를 쓴다면 바로 쓰지말고 내부에 함수를 만들어 사용해라 경쟁 상태가 발생 할 수 있음.
      - 또, 클린업을 통해 직전 요청 자체를 취소하는 방법 또한 존재함.
    ```
    const MyReact = (function () {
      const global = {}
      let index = 0
  
      function useEffect(callback, dependencies) {
        const hooks = global.hooks
    
        // 이전 훅 정보가 있는지 확인한다.
        let previousDependencies = hooks[index]
    
        // 변경됐는지 확인
        // 이전 값이 있다면 이전 값을 얕은 비교로 비교해 변경이 일어났는지 확인한다.
        // 이전 값이 없다면 최초 실행이므로 변경이 일어난 것으로 간주해 실행을 유도한다.
        let isDependenciesChanged = previousDependencies
          ? dependencies.some((value, idx) => !Object.is(value, previousDependencies[idx]))
          : true
    
        // 변경이 일어났다면 첫 번째 인수인 콜백 함수를 실행한다.
        if (isDependenciesChanged) {
          callback()
        }
    
        // 현재 의존성을 훅에 다시 저장한다.
        hooks[index] = dependencies
    
        // 다음 훅이 일어날 때를 대비하기 위해 index를 추가한다.
        index++
      }
    })()
    ```
### 3.1.3`useMemo`
  - 비용이 큰 연산에 대한 결과를 저장하고 이 저장된 값을 반환하는 훅

### 3.1.4`useCallback`
  - useMemo가 값을 기억했다면, useCallback은 인수로 넘겨받은 콜백를 기억함. 특정함수를 새로만들지 않고 다시 재사용
  - 크롬 > 메모리 > 타임라인 할당에서 확인해보면 `useCallback`을 사용하지 않으면 메모리 누수가 있다.
  ```
  const ChildComponent = memo (({name, value, onChange}) => {
    // memo로 감쌌기 때문에 이전 props 데이터는 memo의 얕은 비교를 통해 메모라이징 효과가 발동 된다.
    useEffect(() => {
      console.log('rendering', name)
    })
    return (
      <>
        <h1>{name}: {value ? 'true' : 'false'}</h1>
        <button onClick={onChange}>Toggle</button>
      </>
    )
  })
  
  function App() {
    const [status1, setStatus1] = useState(false)
    const [status2, setStatus2] = useState(false)


    // const toggle1 = () => {
    //   setStatus1(!status1)
    // }
    // const toggle2 = () => {
    //   setStatus2(!status2)
    // }

    const toggle1 = useCallback(() => {
      setStatus1(!status1)
    }, [status1])

    const toggle2 = useCallback(() => {
      setStatus2(!status2)
    }, [status2])

    return (
      <>
        <!-- useCallback을 쓰는 경우 리렌더링 시 특정 함수를 재 사용하기 때문에 status1 변경 시 status2는 리렌더링을 막을 수 있음.  -->
        <ChildComponent name="status1" value={status1} onChange={toggle1} />
        <ChildComponent name="status2" value={status2} onChange={toggle2} />
      </>
    )
  }

  export default App
  ```
  - `useCallback`은 `useMemo`를 사용해서 구현할 수 있다.
    ```
    export function useCallback(callback, args) {
      currentHook = 8
      return useMemo(()=> callback, args)
    }
    ```
### 3.1.5 `useRef`
  - `useState`와 동일하게 컴포넌트 내부에서 렌더링이 일어나도 변경 가능한 상태 값을 저장하는 공통점이 있으나 차이점 두가지가 존재한다.
  - 차이점
    - 반환값인 객체 내부에 있는 `current`로 값에 접근 또는 변경할 수 있다.
    - 그 값이 변하더라도 렌더링을 발생시키지 않는다.
  - 차이점을 알기전에 `useRef` 왜 필요한가?
    - `useRef`을 사용하지 않아도 값을 공유 할 수 있다 생각하고 아래 코드와 같은 가정해보자
      - 컴포넌트가 렌더링 되지 않아도 기본적인 `value`의 값을 가지게 되는 문제가 발생된다. (메모리 누수)
      - 컴포넌트가 초기화 되는 지점이 다르더라도 하나의 값을 바라보는 경우가 아닌 다수의 컴포넌트들은 인스턴스 하나당 하나의 값을 필요로 하지만 value의 값이 공유되는 문제가 있다.
      ```
      let value = 0

      function Component() {
        function handleClick() {
          value += 1
        }
      }
      ```
  - 위와 같은 문제로 `useRef`를 사용하게 된다. 기본적인 예제 코드
    ```
    function RefComponent() {
      const inputRef = useRef()
      console.log(inputRef.current) // undefined

      useEffect(()=> {
        console.log(inputRef.current) // <input type="text" />
      }, [inputRef])

      return <input ref={inputRef} type="text" />
    }
    ```
  - 의외로 구현 또한 간단하다.
    ```
    export function useRef(initialValue) {
      currentHook = 5
      return useMemo(()=> ({current: initialValue}), [])
    }
    ```
### 3.1.6 `useContext`
  - context 란?
    - 리액트에서는 부모와 자식은 트리구조로 이루어져 있으며, 데이터 전달 시 `props`를 통해 데이터를 전달 해주지만 부모, 자식간에 데이터 깊이가 깊어질 수록 관리가 힘들다 이렇게 내려주는 기법을 `props drilling` 이라고 하며 번거로운 작업을 극복하기 위해 등장한 개념이 `context`이다.
  - 코드
    - 재사용이 어려움으로 최대한 컴포넌트를 작게 하거나 재사용되지 않을 정도의 컴포넌트로 작성하여 컨텍스트를 좁해야 한다.
    - 상태 관리 API가 아니라 콘텍스트는 상태를 주입을 해주는 API 이다.
      - 상태 관리 라이브러리는 어떠한 상태를 기반으로 다른 상태를 만들 수 있어야 하며, 필요에 따라 변화를 최적화할 수 있어야 한다.
      - 콘텍스트는 단순히 props를 하위로 전달 해줄 뿐 렌더링이 최적화되지 않기 때문에 전부 리렌더링이 발생된다. 
    ```
    const MyContext = createContext<{ hello: string } | undefined>(undefined)

    function ContextProvider({
      children,
      text,
    }: PropsWithChildren<{ text: string }>) {
      return (
        <MyContext.Provider value={{ hello: text }}>{children}</MyContext.Provider>
      )
    }
    
    function useMyContext() {
      const context = useContext(MyContext)
      if (context === undefined) {
        throw new Error(
          'useMyContext는 ContextProvider 내부에서만 사용할 수 있습니다.',
        )
      }
      return context
    }
    
    function ChildComponent() {
      // 타입이 명확히 설정돼 있어서 굳이 undefined 체크를 하지 않아도 된다.
      // 이 컴포넌트가 Provider 하위에 없다면 에러가 발생할 것이다.
      const { hello } = useMyContext()
    
      return <>{hello}</>
    }
    
    function ParentComponent() {
      return (
        <>
          <ContextProvider text="react">
            <ChildComponent />
          </ContextProvider>
        </>
      )
    }
    ```

### 3.1.7 `useReducer`
  - `useReducer`의 반환 값
    - state: 현재 useReducer 가 가지고 있는 값
    - dispatcher: state를 업데이트하는 함수. state를 변경할 수 있는 `action`을 넘기게 된다.
  - `useReducer`의 인수
    - reducer: 기본 action을 정의하는 함수
    - initialState: useReducer의 초깃값을 의미
    - init: useState의 게으른 초기화 처럼 초깃값을 지연해서 생성시키고 싶을 때 사용하는 `함수`이다. 
  - `useReducer` 와 `useState`는 상호 보완적 같은 관계 같아 서로 같은 구현이 가능하다.
    - `useReducer` -> `useState`
      ```
      function reducer(prevState, newState) {
        return typeof newState === "function" ? newState(prevState) : newState;
      }
      
      function init(initialArg: Initializer) {
        return typeof initialArg === "function" ? initialArg() : initialArg;
      }
      
      function useState(initialArg) {
        return useReducer(reducer, initialArg, init)
      }

      // ex
      const [count, setCount] = useState(0);
      setCount((prev)=> prev + 1) // function

      // useState 내부 훅...
      reducer(prevState = 0, (prev)=> prev + 1)) => (prev)=> 0 + 1 -> 1)
      ```
    - `useState` -> `useReducer`
      ```
      const useReducer = (reducer, initialArg, init) => {
        const [state, setState] = useState(init ? () => init(initialArg) : initialArg);
      
        const dispatch = useCallback((action) => setState((prev) => reducer(prev, action)), [reducer]);
      
        return useMemo(() => [state, dispatch], [state, dispatch]);
      };
      ```
### 3.1.8 `useImperativeHandle`
  - 위 훅을 학습 하기 전 `React.forwardRef`에 대하여 살펴 봐야 한다.
    - `React.forwardRef`는 상위 컴포넌트에서 접근 하고 싶은 ref을 직접 props를 통해 전달할때 명시적으로 예측 할 수 있다.
    ```
    // `React.forwardRef` 를 쓰지 않는 경우
    function ChildComponent ({parentRef}) {
      .... code ...
    }

    function ParentComponent() {
        const inputRef = useRef()

        return (
          <>
            <ChildComponent parentRef={inputRef} />
          </>
        )
    }

    // `React.forwardRef` 사용
    function ChildComponent = forwardRef((props, ref)=> {
      ...
    })
    function ParentComponent() {
        const inputRef = useRef()

        return (
          <>
            <ChildComponent ref={inputRef} />
          </>
        )
    }
    ```
  - `useImperativeHandle`는 부모로 부터 넘겨 받은 `ref`를 마음대로 수정 할 수 있다.
  ```
  import { forwardRef, useImperativeHandle, useRef, useState } from 'react';

  type ChildHandle = { handleClick: () => void };
  type ChildProps = { count: number };

  const ChildComponent = forwardRef<ChildHandle, ChildProps>((props, ref) => {
    useImperativeHandle(ref, () => ({
      handleParentClick() {
        console.log('child handleClick', props.count);
      },
    }), [props.count]);

    return (
      <div>
        자식
      </div>
    );
  });

  function App() {
    const [count, setCount] = useState(0);
    const childRef = useRef<ChildHandle>(null);

    const handleClick = () => {
      childRef.current?.handleParentClick();
    };

    const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
      setCount(Number(e.target.value));
    };

    return (
      <>
        <input type="text" value={count} onChange={handleChange} />
        <button onClick={handleClick}>부모에서 자식 메서드 호출</button>
        <ChildComponent ref={childRef} count={count} />
      </>
    );
  }
  export default App

  ```
### 3.1.9 `useLayoutEffect`
  - 브라우저가 렌더링 되기전 실행이 되며 `useLayoutEffect`가 완료 될 때까지 컴포넌트가 잠시 동안 중지되는 것(동기성)과 같은 일이 발생된다.
  - 즉, 모든 Dom 이 변경 후 동기적으로 발생
    - 리액트가 DOM을 업데이트
    - useLayoutEffect 실행
    - 브라우저 변경 사항 반영
    - useEffect 실행
  - 성능상 문제 발생의 여지가 있기 때문에 화면이 반영되기전 작업 즉, 스크롤 계산, 애니메이션 등 자연스러운 UX를 제공하는데 사용하면 된다.

### 번외
  - 고차 컴포넌트 (HOC)
    - 주의 사항: 
      - `with` prefix 사용
      - 인수로는 `Component`를 넘겨주고 `props`를 임의로 수정, 추가, 삭제하는 일이 없어야 한다.
      - 여러개의 고차 컴포넌트로 감싸는 경우 복잡성이 커지므로 최소한으로만 사용한다.













  
 