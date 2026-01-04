# JavaScript 기초

[← 목차로 돌아가기](../README.md)

---

## 목차

- [undefined vs null](#undefined-vs-null)
- [var, let, const 차이](#var-let-const-차이)
- [호이스팅 (Hoisting)](#호이스팅-hoisting)
- [클로저 (Closure)](#클로저-closure)
- [이벤트 루프 (Event Loop)](#이벤트-루프-event-loop)
- [마이크로태스크 vs 매크로태스크](#마이크로태스크-vs-매크로태스크)

---

## undefined vs null

### Q. `undefined`와 `null`의 차이점은?

**답변:**

- **undefined**: 변수가 선언되었지만 초기화되지 않은 상태 (시스템이 자동 할당)
- **null**: 개발자가 의도적으로 "값이 없음"을 명시하는 빈 값

```javascript
let a; // undefined (초기화 안 됨)
let b = null; // null (명시적으로 빈 값 설정)

console.log(a); // undefined
console.log(b); // null
```

**타입 체크:**

```javascript
typeof undefined; // "undefined"
typeof null; // "object" ❗ (JavaScript 초기 버그)

undefined == null; // true (느슨한 비교)
undefined === null; // false (엄격한 비교)
```

**중요한 버그 - `typeof null`:**

`typeof null`이 `"object"`를 반환하는 것은 JavaScript 초기 구현의 버그입니다.

JavaScript 초기 버전에서 값들은 타입 태그와 함께 저장되었는데:

- 객체는 타입 태그가 `000`
- `null`은 NULL 포인터(대부분 플랫폼에서 `0x00`)로 표현

그래서 `null`의 타입 태그가 객체와 같아서 `"object"`로 인식되는 버그가 생겼고, 이미 너무 많은 코드가 이 동작에 의존하고 있어서 수정되지 않고 그대로 유지되고 있습니다.

**실무 활용:**

```javascript
// 좋은 예: 명시적으로 빈 값 표현
let user = null; // 아직 로드되지 않음을 명시

function getUser() {
  // 사용자를 찾지 못함
  return null; // 의도적으로 "없음"을 반환
}

// 나쁜 예: undefined 직접 할당
let user = undefined; // ❌ 권장하지 않음
```

---

## var, let, const 차이

### Q. `var`, `let`, `const`의 차이점을 설명하세요

**답변:**

| 특징         | var                           | let                          | const                        |
| ------------ | ----------------------------- | ---------------------------- | ---------------------------- |
| **스코프**   | 함수 스코프                   | 블록 스코프                  | 블록 스코프                  |
| **호이스팅** | 선언만 호이스팅 → `undefined` | 선언은 호이스팅되나 TDZ 존재 | 선언은 호이스팅되나 TDZ 존재 |
| **재할당**   | 가능                          | 가능                         | 불가능                       |
| **재선언**   | 가능                          | 불가능                       | 불가능                       |

**1. 호이스팅 차이:**

```javascript
// var - 선언만 호이스팅, 할당은 그대로
console.log(x); // undefined (에러 아님!)
var x = 5;

// 위 코드는 실제로 이렇게 동작:
var x; // 선언만 끌어올려짐
console.log(x); // undefined
x = 5; // 할당은 원래 위치

// let/const - TDZ(Temporal Dead Zone)
console.log(y); // ReferenceError!
let y = 5;
```

**2. 스코프 차이:**

```javascript
// 함수 스코프 (var)
function example1() {
  var x = 1;
  if (true) {
    var x = 2; // 같은 변수!
    console.log(x); // 2
  }
  console.log(x); // 2 (if 안의 x와 같은 변수)
}

// 블록 스코프 (let/const)
function example2() {
  let x = 1;
  if (true) {
    let x = 2; // 다른 변수!
    console.log(x); // 2
  }
  console.log(x); // 1 (if 밖의 x)
}
```

**3. 재할당 & 재선언:**

```javascript
// var - 재선언, 재할당 모두 가능
var x = 1;
var x = 2; // ✅ 재선언 가능
x = 3; // ✅ 재할당 가능

// let - 재할당만 가능
let y = 1;
// let y = 2; // ❌ 재선언 불가능
y = 2; // ✅ 재할당 가능

// const - 재선언, 재할당 모두 불가능
const z = 1;
// const z = 2; // ❌ 재선언 불가능
// z = 2;       // ❌ 재할당 불가능
```

**const와 객체:**

```javascript
// const는 "재할당"만 막음, 객체 내부 변경은 가능
const obj = { name: "John" };
obj.name = "Jane"; // ✅ 가능 (재할당 아님)
obj.age = 25; // ✅ 가능 (재할당 아님)
// obj = {};       // ❌ 불가능 (재할당)

// 완전 불변 객체가 필요하면 Object.freeze
const immutableObj = Object.freeze({ name: "John" });
// immutableObj.name = 'Jane'; // ❌ 무시됨 (strict mode에서 에러)
```

**권장 사용법:**

1. 기본적으로 `const` 사용
2. 재할당이 필요한 경우에만 `let` 사용
3. `var`는 사용하지 않기 (레거시 코드 유지보수 시에만)

---

## 호이스팅 (Hoisting)

### Q. 호이스팅이란 무엇인가요?

**답변:**

JavaScript에서 선언부가 스코프의 최상단으로 끌어올려진 것처럼 동작하는 현상입니다. JavaScript 엔진이 코드를 실행하기 전에 변수, 함수 선언을 미리 메모리에 등록하는 과정에서 발생합니다.

**변수 호이스팅:**

```javascript
console.log(x); // undefined
var x = 5;

// 실제 동작:
var x; // 선언만 호이스팅
console.log(x); // undefined
x = 5; // 할당은 원래 위치
```

**함수 호이스팅:**

```javascript
// 함수 선언식 - 전체가 호이스팅
foo(); // "A" 출력 (정상 동작!)
function foo() {
  console.log("A");
}

// 함수 표현식 - 변수 선언만 호이스팅
bar(); // TypeError: bar is not a function
var bar = function () {
  console.log("B");
};

// 실제 동작:
var bar; // 선언만 호이스팅
bar(); // undefined()로 호출 시도 → TypeError
bar = function () {
  console.log("B");
};
```

**let/const의 TDZ (Temporal Dead Zone):**

```javascript
// TDZ 시작
console.log(x); // ReferenceError

let x = 5; // TDZ 끝, 초기화
console.log(x); // 5

// let/const도 호이스팅은 되지만,
// TDZ로 인해 선언 이전 접근이 막힘
```

**클래스 호이스팅:**

```javascript
// const p = new Person(); // ReferenceError

class Person {
  constructor(name) {
    this.name = name;
  }
}

const p = new Person("John"); // ✅ 정상 동작
```

**호이스팅 우선순위:**

```javascript
var foo;
function foo() {
  return 1;
}

console.log(typeof foo); // "function"
// 함수 선언이 변수 선언보다 우선!
```

---

## 클로저 (Closure)

### Q. 클로저란 무엇이고, 어떻게 동작하나요?

**답변:**

클로저는 함수가 선언된 **렉시컬 환경(Lexical Environment)**을 기억하여, 외부 함수의 실행이 종료된 후에도 외부 함수의 변수에 접근할 수 있는 메커니즘입니다.

**1. 기본 클로저:**

```javascript
function outerFunction() {
  let count = 0; // 외부 함수의 변수

  function innerFunction() {
    count++; // 외부 변수 접근!
    console.log(count);
  }

  return innerFunction;
}

const counter = outerFunction();
// outerFunction 실행 종료되었지만...

counter(); // 1 - count에 접근 가능!
counter(); // 2
counter(); // 3
```

**왜 가능할까?**

- `innerFunction`이 `count`를 참조
- 가비지 컬렉터가 `count`를 해제하지 않음 (참조 카운트 > 0)
- `counter` 변수가 살아있는 한 `count`도 메모리에 유지

**2. 클로저의 활용 - 은닉화:**

```javascript
function createBankAccount(initialBalance) {
  let balance = initialBalance; // private 변수

  return {
    deposit(amount) {
      balance += amount;
      return balance;
    },
    withdraw(amount) {
      if (balance >= amount) {
        balance -= amount;
        return balance;
      }
      return "잔액 부족";
    },
    getBalance() {
      return balance;
    },
  };
}

const account = createBankAccount(1000);
console.log(account.getBalance()); // 1000
account.deposit(500); // 1500
account.withdraw(200); // 1300

// balance에 직접 접근 불가능!
console.log(account.balance); // undefined
```

**3. 메모리 누수 위험:**

```javascript
// ❌ 나쁜 예: 대용량 객체 클로저
function createHeavyClosure() {
  const hugeArray = new Array(1000000).fill("data");

  return function () {
    console.log("작은 함수");
    // hugeArray를 사용하지 않지만 메모리에 계속 유지됨!
  };
}

const leak = createHeavyClosure();
// hugeArray가 GC되지 않음 - 메모리 누수!
```

**해결 방법:**

```javascript
// ✅ 좋은 예: 필요한 것만 참조
function createOptimizedClosure() {
  const hugeArray = new Array(1000000).fill("data");
  const summary = hugeArray.length; // 필요한 정보만 추출

  return function () {
    console.log(`배열 크기: ${summary}`);
    // summary만 참조 - hugeArray는 GC 가능!
  };
}
```

**4. WeakMap으로 메모리 누수 방지:**

```javascript
function createCache() {
  const cache = new WeakMap();

  return {
    set(key, value) {
      cache.set(key, value);
    },
    get(key) {
      return cache.get(key);
    },
  };
}

const cache = createCache();
let obj = { id: 1 };

cache.set(obj, { data: "heavy" });
obj = null; // obj 참조 제거 → cache의 엔트리도 GC됨!
```

**5. 클로저 활용 - 메모이제이션:**

```javascript
function memoize(fn) {
  const cache = {};

  return function (...args) {
    const key = JSON.stringify(args);

    if (key in cache) {
      console.log("캐시에서 반환");
      return cache[key];
    }

    console.log("계산 수행");
    const result = fn(...args);
    cache[key] = result;
    return result;
  };
}

const factorial = memoize(function (n) {
  return n <= 1 ? 1 : n * factorial(n - 1);
});

console.log(factorial(5)); // 계산 수행: 120
console.log(factorial(5)); // 캐시에서 반환: 120
```

**6. 클로저 주의사항 - 루프:**

```javascript
// ❌ 나쁜 예
for (var i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i); // 3, 3, 3 (모두 같은 i 참조!)
  }, 100);
}

// ✅ 해결 1: let 사용 (블록 스코프)
for (let i = 0; i < 3; i++) {
  setTimeout(function () {
    console.log(i); // 0, 1, 2
  }, 100);
}

// ✅ 해결 2: 즉시 실행 함수 (IIFE)
for (var i = 0; i < 3; i++) {
  (function (j) {
    setTimeout(function () {
      console.log(j); // 0, 1, 2
    }, 100);
  })(i);
}
```

---

## 이벤트 루프 (Event Loop)

### Q. 자바스크립트의 이벤트 루프에 대해 설명하세요

**답변:**

이벤트 루프는 JavaScript의 비동기 처리를 관리하는 메커니즘입니다.

**구성 요소:**

- **Call Stack**: 실행 중인 함수들의 스택
- **Web APIs**: 브라우저가 제공하는 비동기 API (setTimeout, fetch 등)
- **Task Queue (Macro Task)**: setTimeout, setInterval 콜백
- **Microtask Queue**: Promise, queueMicrotask 콜백
- **Event Loop**: 큐와 스택을 모니터링하고 조율

**시각적 구조:**

```
┌───────────────┐
│  Call Stack   │
└───────────────┘
        ↑
        │
┌───────┴───────┐
│  Event Loop   │
└───────┬───────┘
        │
        ↓
┌───────────────────────────┐
│   Microtask Queue         │
│   (Promise, queueMicro)   │
└───────────────────────────┘
        ↓
┌───────────────────────────┐
│   Task Queue (Macro)      │
│   (setTimeout, setInt)    │
└───────────────────────────┘
```

**동작 순서:**

1. Call Stack의 동기 코드 실행
2. Call Stack이 비면 **Microtask Queue** 확인 → 모두 실행
3. Microtask Queue가 비면 **Task Queue**에서 하나 실행
4. 1번으로 돌아가서 반복

**예제:**

```javascript
console.log("1"); // 동기

setTimeout(() => console.log("2"), 0); // Macro Task

Promise.resolve().then(() => console.log("3")); // Micro Task

console.log("4"); // 동기

// 출력 순서: 1 → 4 → 3 → 2
```

**실행 순서 상세:**

```javascript
// 1. 동기 코드
console.log("1"); // → "1"

// 2. setTimeout → Task Queue에 등록
setTimeout(() => console.log("2"), 0);

// 3. Promise → Microtask Queue에 등록
Promise.resolve().then(() => console.log("3"));

// 4. 동기 코드
console.log("4"); // → "4"

// 5. Call Stack 비움
//    → Microtask Queue 확인
//    → console.log('3') 실행 → "3"

// 6. Microtask Queue 비움
//    → Task Queue 확인
//    → console.log('2') 실행 → "2"
```

**복잡한 예제:**

```javascript
console.log("script start");

setTimeout(() => {
  console.log("setTimeout");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("promise1");
  })
  .then(() => {
    console.log("promise2");
  });

console.log("script end");

// 출력 순서:
// script start
// script end
// promise1
// promise2
// setTimeout
```

---

## 마이크로태스크 vs 매크로태스크

### Q. 마이크로태스크 큐와 매크로태스크 큐의 차이는?

**답변:**

| 특징          | Microtask                                 | Macrotask (Task)                 |
| ------------- | ----------------------------------------- | -------------------------------- |
| **우선순위**  | 높음                                      | 낮음                             |
| **예시**      | Promise, queueMicrotask, MutationObserver | setTimeout, setInterval, I/O     |
| **실행 시점** | 매크로태스크 실행 후 즉시 모두 실행       | 하나씩 실행                      |
| **중첩 실행** | 큐가 빌 때까지 계속 실행                  | 다음 이벤트 루프 사이클로 넘어감 |

**핵심 규칙:**

- **매크로태스크 1개 실행 후 → 모든 마이크로태스크 실행**
- Microtask가 Macro Task보다 **항상 우선순위 높음**

**예제 1: 기본 동작**

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0); // Macro
Promise.resolve().then(() => console.log("3")); // Micro

console.log("4");

// 출력: 1 → 4 → 3 → 2
```

**예제 2: 중첩된 마이크로태스크**

```javascript
console.log("start");

setTimeout(() => {
  console.log("timeout");
}, 0);

Promise.resolve()
  .then(() => {
    console.log("promise1");

    // 마이크로태스크 내부에서 또 마이크로태스크 생성
    Promise.resolve().then(() => {
      console.log("promise2");
    });
  })
  .then(() => {
    console.log("promise3");
  });

console.log("end");

// 출력:
// start
// end
// promise1
// promise2  ← 마이크로태스크가 계속 실행됨
// promise3  ← 마이크로태스크가 계속 실행됨
// timeout   ← 모든 마이크로태스크 끝난 후
```

**예제 3: async/await**

```javascript
console.log("1");

setTimeout(() => console.log("2"), 0);

async function foo() {
  console.log("3");
  await Promise.resolve();
  console.log("4"); // await 이후는 마이크로태스크!
}

foo();

Promise.resolve().then(() => console.log("5"));

console.log("6");

// 출력: 1 → 3 → 6 → 4 → 5 → 2
```

**마이크로태스크 큐가 막히는 경우:**

```javascript
// ⚠️ 주의: 무한 마이크로태스크 생성
function infiniteMicrotasks() {
  Promise.resolve().then(() => {
    console.log("micro");
    infiniteMicrotasks(); // 재귀!
  });
}

infiniteMicrotasks();

setTimeout(() => {
  console.log("macro"); // 절대 실행되지 않음!
}, 0);

// 마이크로태스크가 계속 생성되어
// 매크로태스크가 실행될 기회가 없음!
```

**실무 활용:**

```javascript
// 브라우저 렌더링 차단 방지
function heavyTask() {
  // 무거운 연산을 작은 단위로 쪼개기
  let chunk1 = () => {
    /* ... */
  };
  let chunk2 = () => {
    /* ... */
  };
  let chunk3 = () => {
    /* ... */
  };

  // 마이크로태스크: 렌더링 차단
  Promise.resolve().then(chunk1).then(chunk2).then(chunk3);

  // 매크로태스크: 렌더링 기회 제공
  setTimeout(chunk1, 0);
  setTimeout(chunk2, 0);
  setTimeout(chunk3, 0);
}
```

---

[← 목차로 돌아가기](../README.md)
