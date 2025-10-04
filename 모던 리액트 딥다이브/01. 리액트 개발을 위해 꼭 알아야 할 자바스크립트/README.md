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

## React의 클로저 `useState`
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

## 클로저 주의점
클로저를 사용할 경우 어디에 사용하는지 상관없이 해당 내용을 기억(클로저는 공짜가 아니다)해야 둬야 하기 때문에 메모리에 올려야 한다.
즉, 메모리를 사용하므로 클로저를 사용할때는 적절한 스코프로 가둬둬야 한다.

## 이벤트 루프와 비동기 통신의 이해






















