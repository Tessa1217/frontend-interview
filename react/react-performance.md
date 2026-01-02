# React 성능 최적화

[← 목차로 돌아가기](../README.md)

---

## 목차

- [Virtual DOM & Reconciliation](#virtual-dom--reconciliation)
- [리스트 렌더링과 key](#리스트-렌더링과-key)
- [React.memo, useMemo, useCallback](#reactmemo-usememo-usecallback)
- [Code Splitting](#code-splitting)

---

## Virtual DOM & Reconciliation

### Q. React가 필요한 부분만 업데이트하는 원리는?

**답변:**

React는 **Virtual DOM**과 **Reconciliation(재조정)** 알고리즘을 통해 효율적으로 UI를 업데이트합니다.

**동작 과정:**

```
상태 변경 발생
      ↓
새로운 Virtual DOM 생성
      ↓
이전 Virtual DOM과 비교 (Diffing)
      ↓
변경된 부분만 실제 DOM에 반영 (Commit)
```

**1. Virtual DOM이란?**

- 실제 DOM의 **가벼운 복사본** (JavaScript 객체)
- 메모리에만 존재
- 실제 DOM보다 훨씬 빠름

```javascript
// Virtual DOM 예시 (단순화)
{
  type: 'div',
  props: {
    className: 'container',
    children: [
      {
        type: 'h1',
        props: { children: 'Hello' }
      }
    ]
  }
}
```

**2. Diffing Algorithm의 핵심 가정:**

**가정 1: 다른 타입의 엘리먼트는 다른 트리 생성**

```javascript
// Before
<div>
  <Counter />
</div>

// After
<span>
  <Counter />
</span>

// → <div> 전체를 버리고 <span>으로 새로 만듦
// → Counter도 언마운트 후 재마운트
```

**가정 2: key를 통해 자식들의 안정성 힌트 제공**

```javascript
// key가 있으면 같은 컴포넌트로 인식
<Item key="item-1" />
```

**3. Reconciliation 과정:**

```javascript
// 이전 Virtual DOM
<ul>
  <li key="1">First</li>
  <li key="2">Second</li>
</ul>

// 새 Virtual DOM
<ul>
  <li key="1">First</li>
  <li key="2">Second (modified)</li>
  <li key="3">Third</li>
</ul>

// React의 판단:
// key="1": 변경 없음 → 유지
// key="2": 텍스트 변경 → 업데이트
// key="3": 새로운 요소 → 추가
```

**4. Fiber Architecture (React 16+):**

- 재조정 과정을 **작은 단위로 쪼개기**
- **우선순위** 부여 가능
- 작업 **중단/재개** 가능

```javascript
// 우선순위 예시
// 높음: 사용자 입력, 애니메이션
// 낮음: 데이터 패칭, 리스트 렌더링
```

---

## 리스트 렌더링과 key

### Q. 리스트 렌더링 시 key가 중요한 이유는?

**답변:**

**key는 React가 어떤 항목이 변경/추가/제거되었는지 식별하는 데 사용됩니다.**

**1. index를 key로 사용하면 안 되는 이유:**

```javascript
// 초기 상태
["A", "B", "C"][ // key: 0, 1, 2
  // 맨 앞에 'D' 추가
  ("D", "A", "B", "C")
]; // key: 0, 1, 2, 3

// React의 판단:
// key 0: 'A' → 'D' (변경으로 인식!)
// key 1: 'B' → 'A' (변경으로 인식!)
// key 2: 'C' → 'B' (변경으로 인식!)
// key 3: 새로운 'C' (추가)
// → 모든 항목이 재렌더링! 💥
```

**시각화:**

```javascript
// index를 key로 사용
{
  items.map((item, index) => <Item key={index} value={item} />);
}

// 'D' 추가 후:
// 0: <Item value="D" /> ← 'A'에서 'D'로 변경
// 1: <Item value="A" /> ← 'B'에서 'A'로 변경
// 2: <Item value="B" /> ← 'C'에서 'B'로 변경
// 3: <Item value="C" /> ← 새로 생성
```

**2. 고유한 key 사용 (올바른 방법):**

```javascript
const items = [
  { id: "a", name: "A" },
  { id: "b", name: "B" },
  { id: "c", name: "C" },
];

// 고유한 id를 key로 사용
{
  items.map((item) => <Item key={item.id} value={item.name} />);
}

// 'D' 추가 후:
const newItems = [
  { id: "d", name: "D" }, // 새로 mount
  ...items, // 재사용 (위치만 shift)
];

// React의 판단:
// key="d": 새로운 항목 → 추가
// key="a", "b", "c": 기존 항목 → 재사용
// → 'D'만 새로 렌더링! ✅
```

**3. 상태를 가진 컴포넌트의 경우:**

```javascript
function TodoItem({ todo }) {
  const [isEditing, setIsEditing] = useState(false);

  return (
    <div>
      <input type="checkbox" checked={todo.done} />
      {isEditing ? <input value={todo.text} /> : <span>{todo.text}</span>}
    </div>
  );
}

// index를 key로 사용하면:
// 순서가 바뀔 때 isEditing 상태가 엉키는 문제 발생!

// 올바른 사용:
{
  todos.map((todo) => <TodoItem key={todo.id} todo={todo} />);
}
```

**4. key 선택 가이드:**

| 상황                 | key 선택         | 예시                                       |
| -------------------- | ---------------- | ------------------------------------------ |
| **DB 데이터**        | ID 사용          | `key={user.id}`                            |
| **고유 식별자 있음** | 해당 식별자 사용 | `key={product.sku}`                        |
| **정적 리스트**      | index 허용       | `key={index}` (추가/제거/재정렬 없을 때만) |
| **동적 리스트**      | 고유 ID 생성     | `key={uuid()}` 또는 `key={nanoid()}`       |

**5. key 생성 예시:**

```javascript
import { nanoid } from "nanoid";

function TodoList() {
  const [todos, setTodos] = useState([]);

  const addTodo = (text) => {
    setTodos([
      ...todos,
      {
        id: nanoid(), // 고유 ID 생성
        text,
        done: false,
      },
    ]);
  };

  return (
    <ul>
      {todos.map((todo) => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}
```

---

## React.memo, useMemo, useCallback

### Q. React의 메모이제이션 도구들에 대해 설명하세요

**답변:**

**1. React.memo - 컴포넌트 메모이제이션:**

```javascript
// 메모이제이션 없이
function Child({ name }) {
  console.log("Child rendered");
  return <div>{name}</div>;
}

function Parent() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <button onClick={() => setCount(count + 1)}>Count: {count}</button>
      <Child name="John" /> {/* count 변경 시마다 재렌더링 */}
    </div>
  );
}

// React.memo 사용
const Child = React.memo(function Child({ name }) {
  console.log("Child rendered");
  return <div>{name}</div>;
});

// 이제 name이 변경될 때만 재렌더링!
```

**커스텀 비교 함수:**

```javascript
const Child = React.memo(
  function Child({ user }) {
    return <div>{user.name}</div>;
  },
  (prevProps, nextProps) => {
    // true 반환: 재렌더링 스킵
    // false 반환: 재렌더링
    return prevProps.user.id === nextProps.user.id;
  }
);
```

**2. useMemo - 값 메모이제이션:**

```javascript
function ExpensiveComponent({ items }) {
  // 비용이 큰 계산
  const expensiveResult = useMemo(() => {
    console.log("Computing expensive value...");
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]); // items가 변경될 때만 재계산

  return <div>Total: {expensiveResult}</div>;
}
```

**useMemo 없이:**

```javascript
function ExpensiveComponent({ items }) {
  // 매 렌더링마다 재계산! 💸
  const expensiveResult = items.reduce((sum, item) => sum + item.value, 0);

  return <div>Total: {expensiveResult}</div>;
}
```

**3. useCallback - 함수 메모이제이션:**

```javascript
function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState("");

  // useCallback 없이
  const handleClick = () => {
    console.log("Clicked");
  };
  // → 매 렌더링마다 새 함수 생성!

  // useCallback 사용
  const handleClickMemo = useCallback(() => {
    console.log("Clicked");
  }, []); // 의존성이 없으므로 한 번만 생성

  return (
    <div>
      <input value={text} onChange={(e) => setText(e.target.value)} />
      <Child onClick={handleClickMemo} />
    </div>
  );
}

const Child = React.memo(function Child({ onClick }) {
  console.log("Child rendered");
  return <button onClick={onClick}>Click me</button>;
});

// useCallback이 없으면:
// text 변경 시마다 handleClick이 새로 생성되고
// Child도 재렌더링됨 (React.memo가 있어도!)
```

**4. 비교 정리:**

| 도구            | 용도                   | 반환     | 의존성 배열 |
| --------------- | ---------------------- | -------- | ----------- |
| **React.memo**  | 컴포넌트 재렌더링 방지 | 컴포넌트 | ❌          |
| **useMemo**     | 값 재계산 방지         | 값       | ✅          |
| **useCallback** | 함수 재생성 방지       | 함수     | ✅          |

**5. 언제 사용해야 하나?**

**React.memo 사용 시기:**

- 무거운 컴포넌트
- 자주 렌더링되는 컴포넌트
- props가 자주 바뀌지 않는 컴포넌트

**useMemo 사용 시기:**

- 비용이 큰 계산 (복잡한 루프, 필터링 등)
- 객체/배열을 props로 전달할 때

**useCallback 사용 시기:**

- 자식 컴포넌트에 함수 props 전달 시
- 의존성 배열에 함수가 들어갈 때

**6. 주의사항:**

```javascript
// ❌ 나쁜 예: 모든 것을 메모이제이션
const Component = React.memo(function Component() {
  const value1 = useMemo(() => 1 + 1, []);
  const value2 = useMemo(() => "hello", []);
  const handleClick = useCallback(() => {}, []);
  // 오버 엔지니어링!
});

// ✅ 좋은 예: 필요한 곳에만 사용
function Component() {
  // 간단한 계산은 그냥 하기
  const simpleValue = 1 + 1;

  // 비용이 큰 계산만 메모이제이션
  const expensiveValue = useMemo(() => {
    return heavyComputation();
  }, [deps]);
}
```

**7. 실전 예제:**

```javascript
function UserList({ users, onUserClick }) {
  return (
    <ul>
      {users.map((user) => (
        <UserItem key={user.id} user={user} onClick={onUserClick} />
      ))}
    </ul>
  );
}

// UserItem 메모이제이션
const UserItem = React.memo(function UserItem({ user, onClick }) {
  const handleClick = useCallback(() => {
    onClick(user.id);
  }, [user.id, onClick]);

  return <li onClick={handleClick}>{user.name}</li>;
});

// 부모 컴포넌트
function App() {
  const [users, setUsers] = useState([]);

  const handleUserClick = useCallback((userId) => {
    console.log("User clicked:", userId);
  }, []);

  return <UserList users={users} onUserClick={handleUserClick} />;
}
```

---

## Code Splitting

### Q. React에서 Code Splitting은 어떻게 하나요?

**답변:**

Code Splitting은 **번들을 작은 청크로 나누어** 필요할 때만 로드하는 기법입니다.

**1. React.lazy - 컴포넌트 지연 로딩:**

```javascript
import React, { Suspense } from "react";

// 일반 import (정적)
// import HeavyComponent from './HeavyComponent';

// lazy import (동적)
const HeavyComponent = React.lazy(() => import("./HeavyComponent"));

function App() {
  return (
    <div>
      <Suspense fallback={<div>Loading...</div>}>
        <HeavyComponent />
      </Suspense>
    </div>
  );
}
```

**2. 라우트 기반 Code Splitting:**

```javascript
import { BrowserRouter, Routes, Route } from "react-router-dom";
import { lazy, Suspense } from "react";

// 각 페이지를 개별 청크로 분리
const Home = lazy(() => import("./pages/Home"));
const About = lazy(() => import("./pages/About"));
const Contact = lazy(() => import("./pages/Contact"));

function App() {
  return (
    <BrowserRouter>
      <Suspense fallback={<PageLoader />}>
        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/contact" element={<Contact />} />
        </Routes>
      </Suspense>
    </BrowserRouter>
  );
}
```

**3. 조건부 로딩:**

```javascript
function App() {
  const [showAdmin, setShowAdmin] = useState(false);

  return (
    <div>
      <button onClick={() => setShowAdmin(true)}>Show Admin Panel</button>

      {showAdmin && (
        <Suspense fallback={<Spinner />}>
          <AdminPanel /> {/* 필요할 때만 로드 */}
        </Suspense>
      )}
    </div>
  );
}

const AdminPanel = lazy(() => import("./AdminPanel"));
```

**4. Named Exports 처리:**

```javascript
// MyComponent.jsx
export function MyComponent() {
  return <div>My Component</div>;
}

// App.jsx
const MyComponent = lazy(() =>
  import("./MyComponent").then((module) => ({
    default: module.MyComponent,
  }))
);
```

**5. 에러 처리:**

```javascript
import { ErrorBoundary } from "react-error-boundary";

function App() {
  return (
    <ErrorBoundary
      fallback={<div>Failed to load component</div>}
      onError={(error) => console.error(error)}
    >
      <Suspense fallback={<Spinner />}>
        <LazyComponent />
      </Suspense>
    </ErrorBoundary>
  );
}
```

**6. Preloading (프리로딩):**

```javascript
const OtherComponent = lazy(() => import("./OtherComponent"));

function MyComponent() {
  // 마우스 호버 시 미리 로드
  const handleMouseEnter = () => {
    // Webpack의 magic comment 사용
    import(/* webpackPreload: true */ "./OtherComponent");
  };

  return <button onMouseEnter={handleMouseEnter}>Show Other Component</button>;
}
```

**7. 번들 분석:**

```bash
# webpack-bundle-analyzer 설치
npm install --save-dev webpack-bundle-analyzer

# package.json
{
  "scripts": {
    "analyze": "source-map-explorer 'build/static/js/*.js'"
  }
}
```

---

[← 목차로 돌아가기](../README.md)
