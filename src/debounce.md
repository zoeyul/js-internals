## Debounce

- debounce 는 마지막 호출로부터 N초 후 실행, 호출이 계속되면 미뤄짐 // ex: 검색 자동완성, 폼 자동저장
  - 일상 생활에서는 화면 자동 꺼짐, 자동 로그아웃, 엘레베이터문
- throttle의 경우 N초마다 한번씩 실행됨, 호출이 계속되도 주기적으로 실행된다.// ex: 스크롤 위치 추적, 무한 스크롤

```js
/**
 * @param {Function} func
 * @param {number} wait
 * @return {Function}
 */
export default function debounce(func, wait) {
  let timeoutId;

  // 여기서 this === 호출 주체, 호출 시점의 this를 받기 위해
  return function (...args) {
    clearTimeout(timeoutId);

    // arrow는 this를 캡처, 그 this를 유지하기 위해
    timeoutId = setTimeout(() => {
      timeoutId = null;
      //func()는 “그냥 함수 호출”이고, func.apply(this, …)는 원래 이 함수가 객체 메서드로 호출된 것처럼 복원하는 행위
      func.apply(this, args);
    }, wait);
  };
}
```

- JS에서 함수는 컨텍스트를 잃을 수 있음
- 유틸 함수는 호출자의 입장을 보존
- debounce는 함수를 실행하지 않고, 대신 “실행을 위임"
- JS가 “함수와 메서드를 구분하지 않는 언어”이기 때문에 다음을 신경써야함

함수를 호출할 때 JS는 항상 두 가지를 결정:

1. 무엇을 실행할지 → func
2. 누가 호출했는지 (this) → 호출 방식

```js
func();
```

- 이 함수는 어떤 객체에도 속하지 않은 상태로 호출하여 이전에 func가 어디서 왔는지는 전혀 중요하지 않음

```js
const obj = {
  x: 1,
  print() {
    console.log(this.x);
  },
};

const f = obj.print;
f(); // undefined (또는 window.x)
```

- print가 원래 obj에 있었던 건 무시됨, debounce가 "분리"ㅋ

```js
obj.method = debounce(obj.method, 100);
```

- 이순간 obj.method → 함수 값만 꺼내지고 객체와의 연결이 끊어짐

```js
const originalFunc = func;
```

- debounce는 내부에서 해당 함수를 저장하고 나중에 실행함

```js
obj.debounced();
```

- debounce가 반환한 함수는 이렇게 호출됨, 반환 함수의 this === obj
- 하지만 내부에서 그냥 func()를 호출하면?

```js
func(); // this 날아감

//컨텍스트 증발
obj.debounced()   // this = obj
   ↓
setTimeout(...)
   ↓
func()            // this = undefined
```

- 그래서 수동으로 다시 this를 붙여야함

```js
func.apply(this, args);
```

- debounce를 호출한 **바로 그 객체(this)**를 func의 호출 주체로 강제로 지정
- 즉 JS 엔진에게 이 함수는 그냥 함수가 아니라 this를 가진 메서드처럼 실행하라고 명시

```js
func.call(this, arg1, arg2);
func.apply(this, args);
```

- apply 와 call 둘다 가능하지만 call 은 인자 하나씩, apply 는 배열로 들어가서 debounce는 가변 인자를 받기 때문에 apply 사용
