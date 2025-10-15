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
### 3.1.23`useMemo`
  - 비용이 큰 연산에 대한 결과를 저장하고 이 저장된 값을 반환하는 훅

### 3.1.23`useCallback`
  - useMemo가 값을 기억했다면, useCallback은 인수로 넘겨받은 콜백를 기억함. 특정함수를 새로만들지 않고 다시 재사용
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
  
  