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