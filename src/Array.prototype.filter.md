## 필터 함수 구현

```js
Array.prototype.myFilter = function (callbackFn, thisArg) {
  const len = this.length;
  const newArray = [];

  for (let k = 0; k < len; k++) {
    const value = this[k];
    if (Object.hasOwn(this, k) && callbackFn.call(thisArg, value, k, this)) {
      newArray.push(value);
    }
  }
  return newArray;
};
```

1. Object.hasOwn(this, k) - Sparse Array 처리

- Object.hasOwn 대신

  - k in this // 프로토타입 체인까지 검사, in 연산자는 객체 자신의 속성 + 상속받은 속성 모두를 검사
  - this.hasOwnProperty(k) //Object.hasOwn()의 구버전, 오버라이드 될 수 있음
    - 자바스크립트는 다음의 순서로 메서드를 조회한다.
    1. arr 자체에 hasOwnProperty가 있나?
    2. Array.prototype에 있나?
    3. Object.prototype에 있나?
  - Object.prototype.hasOwnProperty.call(this, k) // 오버라이드 문제를 피하는 안전한 방식, Object.hasOwn()이 나오기 전 polyfill에서 주로 사용

2. callbackFn.call(thisArg, value, k, this) - 콜백 실행

- callbackFn()로도 호출 가능
  - callbackFn(value, k, this) 로 인자는 똑같이 넘어가지만 콜백 함수 내에서 this가 뭐인지에 대한 차이
  - arr.filter(callbackFn, thisArg) 이걸 완벽히 지원하기 위해서는 .call()로 호출해야함

```js
const obj = { min: 10 };

function myCallback(value, index, array) {
  console.log("value:", value);
  console.log("this:", this);
}

// 일반 호출
myCallback(5, 0, [5, 15, 20]);
// value: 5
// this: undefined (strict mode) 또는 window (non-strict)

// .call() 호출
myCallback.call(obj, 5, 0, [5, 15, 20]);
// value: 5
// this: { min: 10 } , obj가 this가 됨
```

```js
Array.prototype.myFilter = function (callbackFn, thisArg) {
  //                                              ↑ 사용자가 넘긴 값

  // 여기서 this = myFilter를 호출한 배열

  callbackFn.call(thisArg, value, k, this);
  //              ↑                   ↑
  //              │                   └─ 원본 배열 (myFilter의 this)
  //              └─ 콜백 내부에서 this로 쓸 값 (사용자가 넘긴 것)
};

//콜백 함수 안에서
function(num) {
  // this = obj = { min: 10 }  ← thisArg가 this가 됨
  // num = 15                  ← value
  // (index = 1)               ← k
  // (array = [5,15,20])       ← 원본 배열

  return num > this.min;  // 15 > 10 → true
}
```

- thisArg 가 필요한 경우

```js
class NumberFilter {
  constructor(min) {
    this.min = min;
  }

  isAboveMin(num) {
    return num > this.min; // this.min에 접근
  }
}

const filter = new NumberFilter(10);
const numbers = [5, 15, 8, 20];

// this가 undefined
numbers.filter(filter.isAboveMin);

// thisArg로 filter 객체 넘기기
numbers.filter(filter.isAboveMin, filter); // [15, 20]
```

- 요즘엔 arrow function으로 사용

```js
// 1: Arrow function
numbers.filter((num) => filter.isAboveMin(num));

// 2: bind
numbers.filter(filter.isAboveMin.bind(filter));

// 3: Arrow function으로 메서드 정의
class NumberFilter {
  isAboveMin = (num) => {
    // arrow function이라 this 바인딩됨
    return num > this.min;
  };
}
```

- 일반 함수와 arrow function의 차이

```js
class Person {
  name = "Alice";

  // 일반 함수 - this가 호출 방식에 따라 변함
  normalGreet() {
    console.log(this.name);
  }

  // Arrow function - this가 정의 시점에 고정됨
  arrowGreet = () => {
    console.log(this.name);
  };
}

const person = new Person();
const normalFn = person.normalGreet;
const arrowFn = person.arrowGreet;

normalFn(); // undefined ❌ (this가 풀림)
arrowFn(); // "Alice" ✅ (this가 바인딩되어 있음)
```

- 일반 함수 this 의 규칙

```js
obj.method(); // this = 점(.) 왼쪽에 있는 것
method(); // this = undefined (strict) / window (non-strict)
method.call(x); // this = x

const obj = {
  name: "Alice",
  greet() {
    console.log(this.name);
  },
};

obj.greet(); // "Alice"  ← 점 왼쪽 = obj
obj["greet"](); // "Alice"  ← 같은 의미

const fn = obj.greet;
fn(); // undefined ← 점이 없음
fn.call(obj); // "Alice"  ← call로 강제 지정
```

- arrow function은 점(.)을 무시하고 정의될 때 this 가 뭐였는지만 확인한다.

### class vs function

```js
// 클래스: this로 접근
class Counter {
  count = 0;
  increment() {
    this.count++; // this 필요
  }
}

// 함수형: 클로저로 접근
const useCounter = () => {
  const [count, setCount] = useState(0);
  const increment = () => setCount(count + 1); // 그냥 접근
  return { count, increment };
};
```

- 함수 -> 변수 -> 클로저

```js
function outer() {
  const name = "Alice"; // 변수

  return () => console.log(name); // 클로저로 접근
}

//클로저는 정의 시점의 값을 캡처, useCallback의 의존성 배열이 필요

//클로저란? 함수가 자신이 정의된 곳의 변수를 기억하는것

function outer() {
  const name = "Alice"; // outer의 변수

  function inner() {
    console.log(name); // 바깥 변수에 접근
  }

  return inner;
}

const fn = outer(); // outer는 끝났지만
fn(); // "Alice" ← name을 아직 기억

//렉시컬 스코프=규칙, 클로저=결과
//렉시컬 스코프의 규칙은 "변수는 코드가 작성된 위치 기준으로 찾는다", 이 덕분에 함수가 바깥 변수를 기억할 수 있다.
```

- 클래스 -> 속성 -> this

```js
class Person {
  name = "Alice"; // 속성

  greet() {
    //메서드 안에서도 변수는 변수
    const greeting = "Hello";
    console.log(greeting); // 변수 접근 (클로저)
    console.log(this.name); // this로 접근
  }
}
```
