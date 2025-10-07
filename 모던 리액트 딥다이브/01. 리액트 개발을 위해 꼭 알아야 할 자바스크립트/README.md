# 1장 리액트 개발을 위해 꼭 알아야 할 자바스크립트

## 자바스크립트의 데이터 타입

### **자바스크립트의 원시타입**
```
string, null, undefined, boolean, number, symbol, bigint
```

### **자바스크립트의 객체타입**
```
object
```

### 값을 저장하는 방식의 차이
- 같은 객체라 하더라도 내부 값이 같더라도 결과는 true가 아니다 참조하는 객체의 주소가 다르기 때문
- 원시 값 내부 속성 값은 비교하면 동일하다. 이유는 원시값 타입은 stack에서 관리 되지만 값은 heap에서 주소로 참조하고 있음.

#### 자바스크립트 엔진(V8 등)의 메모리 구조 관점에서 살펴보기
```
var car = {
    hello: 'hi'
};

var car2 = {
    hello2: 'hi'
}

var car3 = {
    hello: 'hi2'
}

var car4 = {
    hello: 'hi'
}

console.log(car === car4) // false
console.log(car.hello === car4.hello) // true

Heap:
 ┌─────────────┐
 │ "hi"  (str) │◄───────────────┐
 └─────────────┘                │
 ┌─────────────┐                │
 │ "hi2" (str) │◄───────────┐   │
 └─────────────┘            │   │
                            │   │
car   ──► { hello: ─────────┘ } │
car2  ──► { hello2: ────────────┘ }
car4  ──► { hello: ─────────────┘ }
car3  ──► { hello: ─────────────► "hi2" }

```

### 자바스크립트의 또 다른 비교 공식, `Object.is`

<table>
 <colgroup>
    <col style="width:10rem;">
    <col style="width:20rem;">
    <col style="width:20rem;">
  </colgroup>
<thead>
    <tr>
        <th scope="col" style="text-align: center;">비교 방법</th>
        <th scope="col" style="text-align: center;">설명</th>
        <th scope="col" style="text-align: center;">예시</th>
    </tr>
</thead>
<tbody>
    <tr>
        <td>
            ==
        </td>
        <td>느슨한 비교로, 타입을 강제 변환 후 비교</td>
        <td>'1' == 1 (true)</td>
    </tr>
    <tr>
        <td>
            ===
        </td>
        <td>엄격한 비교로, 타입 변환 없이 비교</td>
        <td>'1' === 1 (false), +0 === -0(true), NaN === NaN(false), ({}) === ({}) (false)</td>
    </tr>
    <tr>
        <td>
            Object.is
        </td>
        <td>===과 비슷하고 객체 비교의 원리는 같으나, +-0 비교 및 NaN 비교에 대한 처리가 다름</td>
        <td>Object.is('1', 1) (false), Object.is(+0, -0) (false), Object.is(NaN, NaN) (true), Object.is({}, {}) (false)</td>
    </tr>
</tbody>
</table>

### 리액트에서의 동등 비교
 - 객체의 depth을 추정 할 수 없기 때문에 얕은 비교로 비교 할 수 밖에 없고 재귀적 비교를 하게 되면 성능에 악영향이 있을 수 있다.
 
```
function shallowEqual(objA, objB) {
  // 같은 참조면 true
  // React에서는 폴리필용으로 따로 함수를 더 만들어 놓았다.
  if (Object.is(objA, objB)) return true;

  // null 이나 객체가 아니면 false
  if (
    typeof objA !== "object" ||
    objA === null ||
    typeof objB !== "object" ||
    objB === null
  ) {
    return false;
  }

  const keysA = Object.keys(objA);
  const keysB = Object.keys(objB);

  // key 개수가 다르면 바로 false
  if (keysA.length !== keysB.length) return false;

  // 각 key 값이 같은지 검사 (얕은 비교)
  for (let i = 0; i < keysA.length; i++) {
    const key = keysA[i];
    // React에서는 hasOwnProperty.call을 유틸 함수로 불렀는데 재정의 한 이유는 객체안에 hasOwnProperty를 명시적으로 사용 했을때 문제점으로 인해 재선언을 한 것으로 추정된다. 
    if (!Object.prototype.hasOwnProperty.call(objB, key)) {
      return false;
    }
    if (!Object.is(objA[key], objB[key])) {
      return false;
    }
  }

  return true;
}

console.log(shallowEqual({ a: 1 }, { a: 1 }));      // true
console.log(shallowEqual({ a: 1 }, { b: 1 }));      // false
console.log(shallowEqual({ a: 1 }, { a: 1, b: 2 })); // false
console.log(shallowEqual({ b: 2, a: 1 }, { a: 1, b: 2 })); // true
  console.log(shallowEqual({ b: 2, a: { num: 1 } }, { a: { num: 1 }, b: 2 })); // false
```

## 클래스

### 인스턴스 메서드
- 인스턴스 메서드란 클래스 내부에 선언한 메서드를 말한다. 인스턴스 메서드들은 자바스크립트의 prototype 에 선언되어서 프로토타입 메서드로 불리기도 한다.

```
const myCar = new Car('자동차')
myCar.hello() // 인스턴스 매서드 호출 => 프로토타입 체이닝에 의해서 Car.prototype에서 hello을 찾아서 호출 할 수 있는 것임.

// {constructor: f, hello: f}
Object.getPrototypeOf(myCar) === Car.prototype // true

myCar.__proto__ === Car.prototype // true

myCar.hello();  // this === myCar → this.name === "자동차"

Object.getPrototypeOf(myCar).hello();
// Object.getPrototypeOf(myCar) === Car.prototype
// this === Car.prototype → Car.prototype.name === undefined

Car.prototype.name = "프로토타입 자동차";

Object.getPrototypeOf(myCar).hello(); 
// "안녕하세요, 저는 프로토타입 자동차입니다"
```

### 결론
- MyComponent의 클래스 컴포넌트가 존재할때, `React.Component`나 `React.PureCompoent`는 MyComponent.prototype는 기본 render 만 제공 되고 나머지는 React.Component.prototype에서 setState 나 lifecycle를 물려 받게 된다.

## 클로저
- 선언된 함수은 작성 순간에 정적으로 결정되어 내부 어디서 선언에 의한 환경 조합이다.
- 이러한 유효 범위는 스코프라 한다.
```
function add() {
  const a = 10;
  function innerAdd() {
    const b = 20;
    console.log(a + b);
  }
  innerAdd()
}

add() // 30
```

### React의 클로저 `useState`
- 아래 코드는 물론 `useState` 구현이 아니다. 쉽게 접근하기 위한 pseudo 코드입니다.
- `value`는 내부적으로 `useState` 안에 은닉 시켜 외부로 부터의 변수 변경을 막을 수 있음.
```
function useState(initialValue) {
  let value = initialValue; // 외부에서 접근 불가, 내부에만 존재

  function setValue(newValue) {
    value = newValue;       // 이 변수(value)를 계속 기억하는 클로저
  }

  function getValue() {
    return value;
  }

  return [getValue, setValue];
}

const [getCount, setCount] = useState(0);
console.log(getCount()); // 0
setCount(5);
console.log(getCount()); // 5
```

### 클로저 주의점
클로저를 사용할 경우 어디에 사용하는지 상관없이 해당 내용을 기억(클로저는 공짜가 아니다)해야 둬야 하기 때문에 메모리에 올려야 한다.
즉, 메모리를 사용하므로 클로저를 사용할때는 적절한 스코프로 가둬둬야 한다.

## 이벤트 루프와 비동기 통신의 이해
- 자바스크립트 => 싱글 스레드 작동 => 한 번에 하나의 작업만 동기 처리.
- 이벤트 루트를 통해 비동기식 처리가 가능하다.

### 싱글 스레드 자바스크립트
- 자바스크립트는 설계 당시 지금과 같은 복잡한 처리를 예상 하지 못했기 때문에 단순한 작업 환경 처리를 위해 싱글 스레드로 설계됨.
- `"Run-to-completion"` 자바스크립트는 하나의 작업이 끝나기 전까지 다른 작업 실행되지 않는다. (임의적 멈춤도 존재하지 않음)

### 이벤트 루트, 테스크 큐
- V8 기준, 자바스크립트 런타임 환경 외부에서 비동기 실행을 돕기 위한 장치.
- 이벤트 루트
  - 콜 스택이 비었다면, 태스크 큐에 있는 콜백 함수를 처리한다.
- 태스크 큐
  - 이벤트 루프는 하나 이상의 태스크 큐를 갖는다.
  - 태스크 큐는 태스크의 `Set`이다.
  - 이벤트 루프가 큐의 첫 번째 태스크를 가져오는 것이 아니라, 태스크 큐에서 실행 가능한(runnable) 첫 번째 태스크를 가져오는 것이다.
```
function bar() {
    console.log('bar')
}

function baz() {
    console.log('baz')
}

function foo() {
    console.log('foo')
    setTimeout(bar, 0)
    baz()
}

foo()
```
- 흐름 중심 설명
```
[시작]
Call Stack           Web APIs                 Task Queue
-----------          --------                 ----------
(global)                                       (빈 큐)

1) foo() 호출
foo
(global)

foo 실행
console.log('foo')

2) setTimeout(bar,0) → 타이머 Web API로 등록
foo                   [타이머(bar, 0ms)]
(global)

(0ms 후) Web API가 타이머 만료 → 콜백 bar를 Task Queue로 보냄
foo                                          [bar]
(global)

3) baz() 호출
baz
foo
(global)

console.log('baz') → 출력 후 baz 종료
foo
(global)

4) foo 종료 → Stack 비움
(global)

5) 이벤트 루프: Stack이 비었고, Task Queue에 bar 있음 → Dequeue
bar
(global)

console.log('bar') → 출력 후 bar 종료
(global)

[끝]
```
### 마이크로 태스크 큐
- 이벤트 루프는 하나의 마이크로 태스크 큐를 갖는다.
- 테스트 큐 < 마이크로 태스크 큐 가 더 우선적 처리를 가짐
  - 태스크 큐: `setTimeout`, `setInterval`, `setImmediate`
  - 마이크로 태스크 큐: `process.nextTick`, `Promise`, `queueMicroTask`, `MutationObserver`

```
function bar() {
    console.log('bar')
}

function baz() {
    console.log('baz')
}

function foo() {
    console.log('foo')
}

setTimeout(foo, 0)
Promise.resolve().then(bar).then(baz)

// bar
// baz
// foo
```
- macrotask < requestAnimationFrame < mircrotask < 동기
  - 브라우저 렌러딩 하는 작업은 마이크로 태스크 큐와 태스크 큐 사이에서 발생.
```
console.log('a') // 동기

setTimeout(() => {
    console.log('b') 
}, 0) // macrotask

Promise.resolve().then(()=> {
    console.log('c') // mircrotask
})

windows.requestAnimationFrame(() => {
    console.log('d')
})  // 브라우저 랜더링

// a
// c
// d
// b
```
## 리액트에서 자주 사용하는 자바스크립트 문법
- 바벨은 다양한 브라우저 환경, 최신 문법을 작성 하고 싶은 개발자의 요구를 해결하기 위해 만들어졌으며, 트랜스파일된 코드 파악이 중요함.

### 구조 분해 할당 (배열 구조/객체 구조 분해 할당)
- 배열 구조 분해 할당은 ES6, 객체 구조 분해 할당은 ECMA 2018 등.
- `useState`, component의 props 등에서 쓰임.
- 객체 구조 분해 할당은 생각보다 번들링 크기가 상대적으로 크기 때문에 `...rest` 사용 시 외부 라이브러리 사용도 고려 해볼만함.

- 배열 구조 분해 할당
```
// 트랜스파일하기 전
const array = [1, 2, 3, 4, 5]
const [first, second, third, ...arrayRest] = array

// 트랜스파일된 결과
var array = [1, 2, 3, 4, 5]
var first = array[0],
    second = array[1],
    third = array[2],
    arrayRest = array.slice(3)
```
- 객체 구조 분해 할당
```
// 트랜스파일하기 전
const object = {
    a:1,
    b:1,
    c:1,
    d:1,
    e:1,
}
const {a, b, ...rest} = object

// Babel이 생성하는 헬퍼들 (Symbols/열거 가능 속성까지 처리)
function _objectWithoutProperties(source, excluded) {
  if (source == null) return {};
  // 해당 객체에서 특정 키를 제외 함.
  var target = _objectWithoutPropertiesLoose(source, excluded);
  var key, i;
  // 객체 내부에 Symbols 존재 여부 판단 getOwnPropertySymbols
  if (Object.getOwnPropertySymbols) {
    var sourceSymbolKeys = Object.getOwnPropertySymbols(source);
    for (i = 0; i < sourceSymbolKeys.length; i++) {
      key = sourceSymbolKeys[i];
      if (excluded.indexOf(key) >= 0) continue;
      if (!Object.prototype.propertyIsEnumerable.call(source, key)) continue;
      target[key] = source[key];
    }
  }
  return target;
}

function _objectWithoutPropertiesLoose(source, excluded) {
  if (source == null) return {};
  var target = {};
  var sourceKeys = Object.keys(source);
  var key, i;
  for (i = 0; i < sourceKeys.length; i++) {
    key = sourceKeys[i];
    if (excluded.indexOf(key) >= 0) continue;
    target[key] = source[key];
  }
  return target;
}

var object = {
    a:1,
    b:1,
    c:1,
    d:1,
    e:1,
}

var a = object.a,
    b = object.b,
    rest = __objectWithoutProperties(object, ['a', 'b'])
```

### 전개 구문
- 배열 혹은 객체, 문자열과 같이 순회 할 수 있는 값에 대해 간결하게 전개 할 수 있음.
- 배열은 ES6, 객체은 ECMA 2018
- 객체 구조 분해 할당과 마찬가지로 객체 전개 구문은 생각보다 번들링 크기가 상대적으로 크다.

- 배열 전개 구문
```
// 트랜스파일하기 전 (ES6+)
const arr1 = ['a', 'b'];
const arr2 = [...arr1, 'c', 'd', 'e'];

// 트랜스파일된 결과 (ES5)
var arr1 = ['a', 'b'];
var arr2 = [].concat(arr1, ['c', 'd', 'e']);
```

- 객체 전개 구문
```
// 트랜스파일하기 전 (ES6+)
const obj1 = {
    a: 1,
    b: 2,
}
const obj2 = {
    c: 3,
    d: 4,
}
const newObj = {...obj1, ...obj2}

// 트랜스파일된 결과 (ES5)
function ownKeys(object, enumerableOnly) {
  var keys = Object.keys(object);
  if (Object.getOwnPropertySymbols) {
    var symbols = Object.getOwnPropertySymbols(object);
    if (enumerableOnly) {
      symbols = symbols.filter(function (sym) {
        return Object.getOwnPropertyDescriptor(object, sym).enumerable;
      });
    }
    keys.push.apply(keys, symbols);
  }
  return keys;
}

function _objectSpread(target) {
  for (var i = 1; i < arguments.length; i++) {
    var source = arguments[i] != null ? arguments[i] : {};
    if (i % 2) {
      // 홀수 번째 인수: 열거 가능한 키만 복사
      ownKeys(Object(source), true).forEach(function (key) {
        _defineProperty(target, key, source[key]);
      });
    } else if (Object.getOwnPropertyDescriptors) {
      // 전체 디스크립터 단위 복사
      Object.defineProperties(target, Object.getOwnPropertyDescriptors(source));
    } else {
      // 디스크립터 미지원 환경용 폴리백
      ownKeys(Object(source)).forEach(function (key) {
        Object.defineProperty(
          target,
          key,
          Object.getOwnPropertyDescriptor(source, key)
        );
      });
    }
  }
  return target;
}

function _defineProperty(obj, key, value) {
  if (key in obj) {
    Object.defineProperty(obj, key, {
      value: value,
      enumerable: true,
      configurable: true,
      writable: true,
    });
  } else {
    obj[key] = value;
  }
  return obj;
}

// ---- 사용 예시 (이미지와 동일) ----
var obj1 = {
  a: 1,
  b: 2,
};

var obj2 = {
  c: 3,
  d: 4,
};

// const newObj = { ...obj1, ...obj2 } 의 트랜스파일 버전
var newObj = _objectSpread(_objectSpread({}, obj1), obj2);

console.log(newObj); // { a: 1, b: 2, c: 3, d: 4 }
```

### 객체 초기자
- ECMAScript 2015부터 도입됨
- 객체 선언 시 객체에 넣고자 하는 키와 값을 가지고 있는 변수가 존재하면 간결하게 처리 할 수 있음.
- 키와 값 할당 형식이 변경, 트랜스파일 이후 부담이 없음.

  
```
// 트랜스파일하기 전 (ES6+)
const a = 1;
const b = 2;

const obj = {
  a,
  b,
};

// 트랜스파일된 결과 (ES5)
var a = 1;
var b = 2;

var obj = {
  a: a,
  b: b,
};
```
## 선택이 아닌 필수, 타입스크립트
- 타입 확장 연산자 `typeof`의 번거로움으로 타입 체크를 정적으로 런타임이 아닌 빌드 타임(트랜스파일)에 수행할 수 있게 해준다.

### 리액트 코드를 효과적으로 작성하기 위한 타입스크립트 활용법
- `any` 대신 `unknown` 사용
```
function doSometiong(callback: any) {
    callback()
}
// 타입스크립트 에러 발생 x, 실행시 에러 발생
doSometiong(1)

// unknown 사용 시
function doSometiong(callback: unknown) {
    if(typeof callback === 'function') {
        callback()
        return
    }
    throw new Error('callback은 function 입니다.')
}
doSometiong(1)
```

- 어떠한 타입도 존재 만족시킬 수 없는 경우는 `never`를 선언하자.
```
// string이 키지만 값은 never — 즉, 어떤 값도 올 수 없다.
type Props = Record<string, never>;
type State = {
  counter: 0;
};

class SampleComponent extends React.Component<Props, State> {
  constructor(props: Props) {
    super(props);
    this.state = {
      counter: 0,
    };
  }

  render() {
    return <>...</>;
  }
}

export default function App() {
  return (
    <>
      {/* ✅ OK */}
      <SampleComponent />

      {/* ❌ Type 'string' is not assignable to type 'never' */}
      <SampleComponent hello="world" />
    </>
  );
}
```

- 타입 가드를 사용해서 최대한 타입을 좁히고 자료형 확인 시 `typeof`를 사용하자.
```
// 사용자 정의 에러 클래스
class UnAuthorizedError extends Error {
  constructor() {
    super();
  }
  get message() {
    return '인증에 실패했습니다.';
  }
}

class UnExpectedError extends Error {
  constructor() {
    super();
  }
  get message() {
    return '예상치 못한 에러가 발생했습니다.';
  }
}

// 원시 타입을 검사하는 헬퍼 함수
function logging(value: string | undefined) {
  if (typeof value === 'string') {
    console.log('로그:', value);
  }
  if (typeof value === 'undefined') {
    console.warn('값이 없습니다.');
    return;
  }
}

// API 호출 예제
async function fetchSomething() {
  try {
    const response = await fetch('/api/something');
    const data = await response.json();

    // 정상적인 문자열 응답 로깅
    logging(data.message);

    return data;
  } catch (e: unknown) {
    // e는 unknown이므로 타입가드 필요

    // ✅ 1️⃣ 사용자 정의 에러 타입 좁히기
    if (e instanceof UnAuthorizedError) {
      console.error('UnAuthorizedError:', e.message);
      return;
    }

    // ✅ 2️⃣ 또 다른 에러 타입
    if (e instanceof UnExpectedError) {
      console.error('UnExpectedError:', e.message);
      return;
    }

    // ✅ 3️⃣ 문자열/undefined일 수 있는 상황에서 typeof 사용
    if (typeof e === 'string') {
      console.error('string error:', e);
      return;
    }

    if (typeof e === 'undefined') {
      console.warn('정의되지 않은 오류입니다.');
      return;
    }

    // ✅ 4️⃣ 알 수 없는 예외 처리
    console.error('Unknown error:', e);
  }
}

// 실행 예시
(async () => {
  await fetchSomething();
})();
```

- 어떤 객체에 키가 존재하는지 확인 할때는 `in`을 사용하자.
```
type Student = {
  age: number;
  score: number;
};

type Teacher = {
  name: string;
  subject: string;
};

function doSchool(person: Student | Teacher) {
  // 🧩 'in' 연산자를 사용하면 특정 프로퍼티 존재 여부로 타입을 좁힐 수 있다.
  if ('age' in person) {
    // person은 Student로 좁혀짐
    console.log('학생 나이:', person.age);
    console.log('점수:', person.score);
  }

  if ('name' in person) {
    // person은 Teacher로 좁혀짐
    console.log('교사 이름:', person.name);
    console.log('과목:', person.subject);
  }
}
```

- 다양한 타입에 대한 처리가 필요한 경우 제네릭을 선언하자.
  - 적절한 네이밍으로 명확히 하자.
  - 리액트에서는 대표적으로 `useState`
    - 기본값을 넘기지 않고 사용시 `undefined`로 추론하는 문제가 발생하는데 이를 방지함.
```
function getFirstAndLast<T>(list: T[]): [T, T] {
  return [list[0], list[list.length - 1]];
}

const [nFirst, nLast] = getFirstAndLast([1, 2, 3, 4, 5]);
// nFirst: number, nLast: number

const [sFirst, sLast] = getFirstAndLast(['a', 'b', 'c', 'd', 'e']);
// sFirst: string, sLast: string

// 리액트에서는 대표적으로 `useState`
function Component() {
    // state: string
    const [state, setState] = useState<string>('');
}

// 적절한 네이밍 부여
function multipleGenric<First, Last>(a: First, a2: Last): [First, Last] {
    return [a1, a2]
}

const [a, b] = multipleGenric<string, number>('hi', 12345)
```

- 인덱스 시그니처 활용 방법 및 객체의 키는 동적 선언을 최대한 지양해야하며 객체의 타입도 필요에 따라 좁혀야한다.
```
type Hello = Record<"hello" | "hi", string>;
type Hello2 = {
  [key in "hello" | "hi"]: string;
};

const hello: Hello = {
  hello: "hello",
  hi: "hi",
};

const hello2: Hello2 = {
  hello: "hello",
  hi: "hi",
};

// Object.keys는 타입스크립트 덕 타이핑에 의해 어떤 객체가 필요한 변수 메서드만 지니고 있어도 해당 타입에 속하도록 인정해주기 때문에 예상과 다른 결과를 얻을 수 있음.
const result = Object.keys(hello); // string[]....??????

Object.keys(hello).forEach((key) => {
  const value = hello[key]; // value -> any, key -> string ????????????
  return value;
});

Object.keys(hello2).forEach((key) => {
  const value = hello2[key]; // value -> any, key -> string ????????????
  return value;
});

console.log(result);

function KeysOf<T extends object>(obj: T): Array<keyof T> {
  return Array.from(Object.keys(obj)) as Array<keyof T>; // 타입 단언이 필요함...
}

const result2 = KeysOf(hello).map((key) => {
  const value = hello[key]; // value -> string, key -> 'hi' | 'hello' 
  return value;
});

console.log(result2);
```
- 덕 타이핑은 내부적으로 필요한 형태를 지니고 있다면 동작을 하게 되어 있다. `car: Car` 가 가능한 이유
```
type Car = { name: string };
type Truck = Car & { power: number };

// truck의 Truck 타입만 가능 할 것 같지만 Truck 타입은 Car의 name을 포함 하기 때문에 가능함.
function horn(car: Car) {
  console.log(`${car.name}이 경적을 울립니다! 빵빵~`);
}

const truck: Truck = {
  name: '비싼차',
  power: 100,
};

// 정상적으로 작동한다!
horn(truck);
```
