## 5.1 상태 관리는 왜 필요한가?
  - 상태란
    - 어떤 의미를 지닌 값으로 애플리케이션 시나리오 따라 지속적으로 변경되는 값이다.
      - UI
      - URL
      - FORM
      - 서버에서 가저온 값

## 5.1.1 리액트 상태 관리의 역사
  - Flux 패턴의 등장
    - 순수 리액트에서 전역 상태 관리 수단으로서 props에 대한 주입을 대신하여 Context API 가 그 역할을 해왔다.
      - 엄밀히 말하면 전역 상태 관리가 아니다.
    - 기존 MVC에서는 데이터가 Model과 View에서 양방향 형태로 반영되어 시나리오가 복잡해 짐에 따라 Flux 패턴을 통해 단방향 흐름으로 데이터를 전달 하는데 시작이 되었다.
    - Flux 패턴의 장점은 데이터 흐름을 파악하기 쉬우나 단점은 데이터 갱신에 따른 화면 업데이트에 따라 코드의 양이 많아 개발자가 수고로워진다.
  - Redux의 등장
    - `Elm` 아키텍처의 영향을 받아 개발되었다.
    -  `action`, `dispatch`, `reducer`를 통해 글로벌 상태 관리로 하위 컴포넌트에 전파하는데 있다 props drilling 문제를 해결했다.
    - 코드의 양이 많아 단순한 컴포넌트 사용에 쓰기 모호 하다.
  - Context API와 useContext
    - 16.3 버전 부터 하위 컴포넌트로 주입 할 수 있는 Context API를 출시 하였고 Context Provider가 주입하는 상태를 사용 할 수 있다.
    - 이전 버전에서는 context가 존재 했으며, getChildContext()가 대체하였다.
      - 첫 번째로 상위 컴포넌트 렌더 시 getChildContext 호출과 함께 shouldComponentUpdate가 항상 true로 작동하여 불필요한 렌더링이 발생된다.
      ```
      class MyComponent extends React.Component {
        static childContextTypes = {
          name: PropTypes.string,
          age: PropTypes.number,
        }

        getChildContext() {
          return {
            name: 'foo',
            age: 30,
          }
        }

        render() {
          return <ChildComponent />
        }
      }

      function ChildComponent(props, context ) {
        return (
          <div>
            <p>{context.name}</p>
            <p>{context.age}</p>
          </div>
        )
      }

      ChildComponent.contextTypes = {
        name: PropTypes.string,
        age: PropTypes.number,
      }
      ```

## 5.2 리액트 훅으로 시작하는 상태 관리

### 5.2.1 useState와 useReducer
    - 훅을 사용할 때마다 컴포넌트별로 초기화되므로 컴포넌트에 따라 서로 다른 상태를 가질 수 밖에 없다. 결론적으로 지역 상태는 컴포넌트 내에서만 유효하다는 한계가 있다.
    
    ```
    function useCounter(initCounter) {
      const [counter, setCounter] = useState()

      function inc() {
        setCounter((prev) => prev + 1)
      }

      return {counter, inc}
    }

    function Counter1() {
      const { counter, inc } = useCounter();

      return (
        <>
          <h3>Counter1: {counter}</h3>
          <button onClick={inc}>+</button>
        </>
      );
    }

    function Counter2() {
      const { counter, inc } = useCounter();

      return (
        <>
          <h3>Counter2: {counter}</h3>
          <button onClick={inc}>+</button>
        </>
      );
    }
    ```

    ```
    // 1️⃣ useCounter 커스텀 훅 정의
    function useCounter() {
      const reducer = (state, action) => {
        switch (action.type) {
          case "INCREMENT":
            return { counter: state.counter + 1 };
          default:
            return state;
        }
      };

      const [state, dispatch] = useReducer(reducer, { counter: 0 });

      const inc = () => dispatch({ type: "INCREMENT" });

      return { counter: state.counter, inc };
    }

    // 2️⃣ Counter1 컴포넌트
    function Counter1() {
      const { counter, inc } = useCounter();

      return (
        <>
          <h3>Counter1: {counter}</h3>
          <button onClick={inc}>+</button>
        </>
      );
    }

    // 3️⃣ Counter2 컴포넌트
    function Counter2() {
      const { counter, inc } = useCounter();

      return (
        <>
          <h3>Counter2: {counter}</h3>
          <button onClick={inc}>+</button>
        </>
      );
    }
    ```
### 5.2.2 지역 상태의 한계를 벗어나보자: useState의 상태를 바깥으로 분리하기
  - 억지로 코드 만들어 보기
    - 일단 한쪽의 업데이트로 인해 다른쪽 컴포넌트는 렌더링 되지 않는다. 당연히 다른 컴포넌트는 리렌더링 되지 않기 때문이다. 
    - 같은 상태를 바라보지만 컴포넌트 내부에 useState라는 별도의 상태로 다시 관리해야 하는 비효율적인 방식이다.
  ```
  export type State = { counter: number };

  let state: State = {
    counter: 0,
  }

  export function get(): State {
    return state;
  }

  type Initializer<T> = T extends any ? T | ((prev: T) => T) : never;

  export function set<T>(nextSTate: Initializer<T>) {
    state = typeof nextState === 'function' ? nextState(state) : nextState
  }

  function Counter1() {
    const [count, setCount] = useState(state);

    function handleClick() {
        set((prev: State) => { 
          const newState = { counter: prev.counter + 1 }

          setCount(newState);
          return newState;
        });
    }

    return (
      <>
        <h3>{state.counter}</h3>
        <button onClick={handleClick}>>+</button>
      </>
    )
  }

  function Counter2() {
    const [count, setCount] = useState(state);

    function handleClick() {
        set((prev: State) => { 
          const newState = { counter: prev.counter + 1 }

          setCount(newState);
          return newState;
        });
    }

    return (
      <>
        <h3>{state.counter}</h3>
        <button onClick={handleClick}>>+</button>
      </>
    )
  }
  ```
  - 외부의 상태를 참조하면서 리렌더링까지 자연스럽게 일어나려면 다음과 같은 조건을 만족해야 한다.
    - `window`나 `global`이 아니더라도 컴포넌트 외부 어딘가에 상태를 두고 여러 컴포넌트가 같이 쓸 수 있어야 한다.
    - 외부의 상태를 사용하는 컴포넌트는 상태의 변화를 알아채어 변화가 일어날 때 리렌더링이 일어나서 최신 상태값 기준으로 렌더링해야 한다. (상태를 참조하는 모든 컴포넌트에서도 마찬가지)
    - 상태가 객체인 경우에는 객체 내부의 property중 변하지 않는 값을 참조중인 컴포넌트에서는 리렌더링이 일어나서는 안된다.
      - 코드 새로 작성 해보기
      ```
      type Initializer<T> = T extends any ? T | ((prev): T => T): never

      type Store<State> = {
        get: () => State  //  변수 대신 함수로 만들어서 항상 새로운 값을 가져오게 함
        set: (action: Initializer<State>) => State // useState와 동일하게 값 또는 함수를 받을 수 있게 함
        subscribe:(callback: () => void) => () => void // store의 변경을 감지하고 싶은 컴포넌트들이 자신의 callback 함수를 등록해 두는 곳
      }

      export const createStore = <State extends unknown>( initialState: Initializer<State>): Store<State> => {
	
        // 스토어 내부에 상태를 정의해서 관리
        let state = typeof initialState !== 'function' ? initialState : initialState();

        // 자료형에 관계없이 유니크한 값을 저장할 수 있는 Set을 사용
        const callbacks = new Set<() => void>()

        const get = () => state;

        const set = (nextState: State | ((prev: State) => State)) => {
          state = typeof nextState === 'function' ? (nextState as (prev: State) => State)(state) : nextState;
          
          // 값의 업데이트가 일어나면 콜백 목록의 전체를 실행한다.
          callbacks.forEach((callback) => callback());

          return state;
        }

        // 받은 콜백을 추가한다.
        const subscribe = (callback: () => void) => {
          callbacks.add(callback);

          /// 클린업 실행시 기존의 콜백은 제거한다.
          return () => {
            callbacks.delete(callback)
          }
        }

        return { get, set, subscribe };
      }
      ```

      - useStore
        - 아래 코드는 객체가 전부 리렌더링 되기 때문에 수정이 필요하다.
        - `setState(store.get())` 객체 비교 시 `Object.is` 시 새로운 값을 계속 생성 시킨다.
      ```
      import { useState, useEffect } from 'react';
      import type { Store } from './store';

      export const useStore = <State extends unknown>(store: Store<State>) => {
        const [state, setState] = useState<State>(() => store.get());

        useEffect(() => {
          const unsubscribe = store.subscribe(() => {
            setState(store.get());
          });

          return unsubscribe;
        },[store])

        return [state, store.set] as const;
      }
      ```

      - useStoreSelector
        - `useStore`를 보완 한 코드로 실제 꺼내고자 하는 `selector(store.get())` store의 값을 가져와서 setState를 각각 반영 해주면 이전에 같은 `값만 `!!! 비교 하기 때문에 전역으로 사용 시 리렌더링을 막을 수 있다.
        - 현재는 react에서 useSyncExternalStore 로 이러한 내용을 통해 구현된 훅이 있다.
      ```
      import { useState, useEffect } from 'react';
      import type { Store } from './store';

      export const useStoreSelector = <State extends unknown, Value extends unknown>(
        store: Store<State>,
        selector: (state: State) => Value
      ) => {
        const [state, setState] = useState(() => selector(store.get()));

        useEffect(() => {
          const unsubscribe = store.subscribe(() => {
            const value = selector(store.get())
            setState(value);
          })
        
          return unsubscribe;
        },[store, selector])

        return state;
      }
      ```
      - 5.2.3 useState와 Context를 동시에 사용해 보기
        - 위에서 만들었던 훅은 단점이 있는데, 훅과 스토어를 사용하는 구조는 반드시 하나의 스토어만 가지게 된다는 것이다. 
        - 만약 훅을 사용하는 서로 다른 스코프에서 스토어 구조는 동일하되, 여러 개의 서로 다른 데이터를 공유하고 싶다면?
          - 반복적으로 스토어를 생성시점과 스토어의 공유 범위를 보면 효율적이지 않다.
          ```
          const store1 = createStore({ count: 0 });
          const store2 = createStore({ count: 0 });
          const store3 = createStore({ count: 0 });

          function Counter1() {
            const counter = useStoreSelector(store1, useCallback((state) => state.count, []);

            // 생략...
          }
          ```
        - 위와 같은 문제로 하나의 인스턴스 내에서 독립적인 스코프를 가지게 하려면 `Context`를 통해 하위 컴포넌트로 주입 시켜 접근하게 한다.
          - Context 영역을 잘 나눠 부모와 자식간 책임과 역할을 분리하여 코드 작성이 용이 해진다.
        ```
        export type CounterStore = {
            count: number;
            text: string;
        }

        const defaultStore = createStore<CounterStore>({ count: 0, text: "hello" });
        export const CounterStoreContext = createContext<Store<CounterStore>>(defaultStore);

        export const CounterProvider = ({ 
            initialState, children 
        }: PropsWithChildren<{ initialState: CounterStore }>) => {
            const storeRef = useRef<Store<CounterStore>>()

            if (!storeRef.current) {
                storeRef.current = createStore(initialState);
            }

            return (
                <CounterStoreContext.Provider value={storeRef.current}>
                    {children}
                </CounterStoreContext.Provider>
            )
        };

        // https://www.npmjs.com/package/use-subscription
        export const useCounterContextSelector = <State extends unknown>(
            selector: (state: CounterStore) => State
        ) => {
            const store = useContext(CounterStoreContext);
            const subscribe = useSubscription(
                useMemo(() => {
                    return {
                        getCurrentValue: () => selector(store.get()),
                        subscribe: store.subscribe,
                    }
                }, [store, selector]),
            );

            return [subscribe, store.set] as const;
        }

        function App() {
          return (
            <>
              {/* 0 */}
              <ContextCounter />
              {/* hi */}
              <ContextInput />
              <CounterProvider initialState={{ count: 10, text: "hello" }}>
                {/* 10 */}
                <ContextCounter />
                {/* hello */}
                <ContextInput />
              </CounterProvider>
              <CounterProvider initialState={{ count: 20, text: "world" }}>
                {/* 20 */}
                <ContextCounter />
                {/* world */}
                <ContextInput />
              </CounterProvider>
            </>
          )
        }
        ```
      - 5.2.4 상태 관리 라이브러리 Recoil, Jotai, Zustand 살펴보기
        - Recoil, Jotai : 스토어가 가지는 클로저를 기반으로 생성
        - Zustand: Redux와 비슷하게 큰 소토어를 기반으로 상태를 관리

          - Recoil
            - 현재는 0.7.7 이후 패치 이력이 없다.
            - 핵심 기능 알아보기
              - RecoilRoot
                ```
                function notInAContext() {
                  throw err('This component must be used inside a <RecoilRoot> component.');
                }

                // 외부에서는 RecoilRoot로 감싸지 않으면 오류를 호출하게 함
                const defaultStore: Store = Object.freeze({
                  storeID: getNextStoreID(), // 스토어 아이디 값
                  getState: notInAContext, // 스토어 값 가져옴
                  replaceState: notInAContext, // 스토어 수정
                  getGraph: notInAContext,
                  subscribeToTransactions: notInAContext,
                  addTransactionMetadata: notInAContext,
                });

                const AppContext = React.createContext<StoreRef>({current: defaultStore});
                const useStoreRef = (): StoreRef => useContext(AppContext);

                function RecoilRoot(props: Props): React.Node {
                  const {override, ...propsExceptOverride} = props;

                  // AppContext가 가진 스토어
                  const ancestorStoreRef = useStoreRef();

                  // 상위에 이미 RecoilRoot가 있고, 나는 그 store를 override(덮어쓰기)하지 않겠다고 지정했다면 새 store를 만들지 말고 상위 store를 그대로 써라.
                  if (override === false && ancestorStoreRef.current !== defaultStore) {
                    // If ancestorStoreRef.current !== defaultStore, it means that this
                    // RecoilRoot is not nested within another.
                    return props.children;
                  }

                  return <RecoilRoot_INTERNAL {...propsExceptOverride} />;
                }
                ```
                - replaceState
                  - 변경된 상태를 하위 컴포넌트로 전파해 컴포넌트에 리렌더링을 일으키는 notifyComponents가 있다.
                ```
                const replaceState = (replacer: TreeState => TreeState) => {
                  startNextTreeIfNeeded(storeRef.current);
                  // Use replacer to get the next state:
                  const nextTree = nullthrows(storeStateRef.current.nextTree);
                  let replaced;
                  try {
                    stateReplacerIsBeingExecuted = true;
                    replaced = replacer(nextTree);
                  } finally {
                    stateReplacerIsBeingExecuted = false;
                  }
                  if (replaced === nextTree) {
                    return;
                  }

                  // Save changes to nextTree and schedule a React update:
                  storeStateRef.current.nextTree = replaced;
                  if (reactMode().early) {
                    notifyComponents(storeRef.current, storeStateRef.current, replaced);
                  }
                };
                ```

                - notifyComponents
                  -  스토어를 사용하고 있는 모든 하위 의존성(하위 atom)을 검색한다음 해당 컴포넌트들의 콜백을 모두 실행
                    - atom: 상태를 나타내는 Recoil의 최소 상태 단위
                    - useRecoilValue: atom의 값을 읽어오는 훅
                    - useRecoilState: 단순히 atom의 값을 가져오기도 하고 값을 변경도 할 수 있는 훅
                ```
                function notifyComponents(
                  store: Store,
                  storeState: StoreState,
                  treeState: TreeState,
                ): void {
                  const dependentNodes = getDownstreamNodes(
                    store,
                    treeState,
                    treeState.dirtyAtoms,
                  );
                  for (const key of dependentNodes) {
                    const comps = storeState.nodeToComponentSubscriptions.get(key);
                    if (comps) {
                      for (const [_subID, [_debugName, callback]] of comps) {
                        callback(treeState);
                      }
                    }
                  }
                }
                ```