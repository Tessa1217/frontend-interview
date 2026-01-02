# React 핵심 개념

[← 목차로 돌아가기](../README.md)

---

## 목차

- [라이프사이클](#라이프사이클)
- [Props vs State](#props-vs-state)
- [고차 컴포넌트 (HOC)](#고차-컴포넌트-hoc)
- [Custom Hooks](#custom-hooks)

---

## 라이프사이클

### Q. React의 라이프사이클에 대해 설명하세요

**답변:**

React 컴포넌트의 라이프사이클은 크게 3단계로 구분됩니다:

1. **Mount (마운트)**: 컴포넌트가 DOM에 삽입되는 단계
2. **Update (업데이트)**: props나 state 변경으로 재렌더링되는 단계
3. **Unmount (언마운트)**: 컴포넌트가 DOM에서 제거되는 단계

**클래스 컴포넌트 라이프사이클:**

```javascript
class MyComponent extends React.Component {
  // 1. Mount 단계
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    console.log("1. constructor");
  }

  static getDerivedStateFromProps(props, state) {
    console.log("2. getDerivedStateFromProps");
    return null;
  }

  componentDidMount() {
    console.log("4. componentDidMount - DOM 삽입 후 실행");
    // API 호출, 이벤트 리스너 등록
  }

  // 2. Update 단계
  shouldComponentUpdate(nextProps, nextState) {
    console.log("5. shouldComponentUpdate");
    return true; // false 반환 시 렌더링 스킵
  }

  componentDidUpdate(prevProps, prevState) {
    console.log("7. componentDidUpdate - 리렌더링 후 실행");
    // DOM 업데이트 후 작업
  }

  // 3. Unmount 단계
  componentWillUnmount() {
    console.log("8. componentWillUnmount - 제거 직전 실행");
    // 이벤트 리스너 제거, 타이머 정리
  }

  render() {
    console.log("3/6. render");
    return <div>{this.state.count}</div>;
  }
}
```

**함수 컴포넌트 with Hooks:**

```javascript
function MyComponent() {
  const [count, setCount] = useState(0);

  // componentDidMount + componentDidUpdate
  useEffect(() => {
    console.log("Mount 또는 Update 시 실행");
  });

  // componentDidMount만 (빈 배열)
  useEffect(() => {
    console.log("Mount 시에만 실행");

    return () => {
      console.log("Unmount 시 실행");
    };
  }, []);

  // dependency 변경 시만
  useEffect(() => {
    console.log("count 변경 시 실행");

    return () => {
      console.log("다음 effect 실행 전 cleanup");
    };
  }, [count]);

  return <div>{count}</div>;
}
```

**useEffect cleanup 함수 실행 시점:**

```javascript
useEffect(() => {
  console.log("Effect 실행");

  return () => {
    console.log("Cleanup 실행");
  };
}, [dependency]);

// Cleanup이 실행되는 시점:
// 1. 컴포넌트가 언마운트될 때
// 2. dependency가 변경되어 effect가 재실행되기 직전
```

**실행 순서 예제:**

```javascript
function Parent() {
  useEffect(() => {
    console.log("Parent mount");
    return () => console.log("Parent unmount");
  }, []);

  return <Child />;
}

function Child() {
  useEffect(() => {
    console.log("Child mount");
    return () => console.log("Child unmount");
  }, []);

  return <div>Child</div>;
}

// 출력 순서:
// Child mount
// Parent mount
// (언마운트 시)
// Parent unmount
// Child unmount
```

---

## Props vs State

### Q. Props와 State의 차이는?

**답변:**

| 특징            | Props                 | State                   |
| --------------- | --------------------- | ----------------------- |
| **소유권**      | 부모 컴포넌트         | 해당 컴포넌트           |
| **변경 가능성** | 읽기 전용 (immutable) | 변경 가능 (mutable)     |
| **변경 방법**   | 부모가 다시 렌더링    | `setState` / `useState` |
| **데이터 흐름** | 위 → 아래 (단방향)    | 컴포넌트 내부           |
| **리렌더링**    | props 변경 시         | state 변경 시           |

**1. Props - 부모에서 자식으로 데이터 전달:**

```javascript
// 부모 컴포넌트
function Parent() {
  return <Child name="John" age={25} />;
}

// 자식 컴포넌트
function Child({ name, age }) {
  // props는 읽기 전용
  // name = "Jane"; // ❌ 불가능!

  return (
    <div>
      {name} - {age}
    </div>
  );
}
```

**2. State - 컴포넌트 내부 상태:**

```javascript
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increase</button>
    </div>
  );
}
```

**3. State를 Props로 전달:**

```javascript
function Parent() {
  const [user, setUser] = useState({ name: "John", age: 25 });

  return (
    <div>
      <Child user={user} />
      <button onClick={() => setUser({ ...user, age: user.age + 1 })}>
        Increase Age
      </button>
    </div>
  );
}

function Child({ user }) {
  return (
    <div>
      {user.name} - {user.age}
    </div>
  );
}
```

**4. Props Drilling 문제:**

```javascript
// 3단계 전달... 비효율적!
function GrandParent() {
  const data = { user: "John" };

  return <Parent data={data} />;
}

function Parent({ data }) {
  return <Child data={data} />;
}

function Child({ data }) {
  return <GrandChild data={data} />;
}

function GrandChild({ data }) {
  return <div>{data.user}</div>; // 여기서만 사용
}
```

**5. 해결 방법 1: Context API**

```javascript
const DataContext = createContext();

function GrandParent() {
  const data = { user: "John" };

  return (
    <DataContext.Provider value={data}>
      <Parent /> {/* props 불필요! */}
    </DataContext.Provider>
  );
}

function Parent() {
  return <Child />;
}

function Child() {
  return <GrandChild />;
}

function GrandChild() {
  const data = useContext(DataContext); // 직접 접근!
  return <div>{data.user}</div>;
}
```

**6. 해결 방법 2: 상태 관리 라이브러리**

```javascript
// Zustand 예시
import create from "zustand";

const useStore = create((set) => ({
  user: { name: "John", age: 25 },
  setUser: (user) => set({ user }),
}));

function GrandChild() {
  const user = useStore((state) => state.user);
  const setUser = useStore((state) => state.setUser);

  return (
    <div>
      {user.name} - {user.age}
      <button onClick={() => setUser({ ...user, age: user.age + 1 })}>
        Increase Age
      </button>
    </div>
  );
}
```

**7. Props Children 패턴:**

```javascript
function Card({ children, title }) {
  return (
    <div className="card">
      <h2>{title}</h2>
      <div className="content">{children}</div>
    </div>
  );
}

// 사용
<Card title="User Info">
  <p>Name: John</p>
  <p>Age: 25</p>
</Card>;
```

---

## 고차 컴포넌트 (HOC)

### Q. 고차 컴포넌트(HOC)는 무엇이고, 언제 사용하나요?

**답변:**

고차 컴포넌트는 컴포넌트를 받아 **새로운 컴포넌트를 반환하는 함수**입니다.

**기본 패턴:**

```javascript
function withSomething(Component) {
  return function EnhancedComponent(props) {
    // 추가 로직
    return <Component {...props} />;
  };
}
```

**1. React 내장 HOC:**

```javascript
// React.memo - 메모이제이션
const MemoizedComponent = React.memo(MyComponent);

// 같은 props면 리렌더링 스킵
function MyComponent({ name }) {
  console.log("Rendering:", name);
  return <div>{name}</div>;
}

const MemoizedMyComponent = React.memo(MyComponent);
```

**2. 인증 체크 HOC:**

```javascript
function withAuth(Component) {
  return function AuthenticatedComponent(props) {
    const { isLoggedIn } = useAuth();
    const navigate = useNavigate();

    useEffect(() => {
      if (!isLoggedIn) {
        navigate("/login");
      }
    }, [isLoggedIn]);

    if (!isLoggedIn) {
      return null;
    }

    return <Component {...props} />;
  };
}

// 사용
const ProtectedPage = withAuth(MyPage);

function MyPage() {
  return <div>Protected Content</div>;
}
```

**3. 로딩 상태 HOC:**

```javascript
function withLoading(Component) {
  return function ComponentWithLoading({ isLoading, ...props }) {
    if (isLoading) {
      return <Spinner />;
    }

    return <Component {...props} />;
  };
}

// 사용
const UserListWithLoading = withLoading(UserList);

function App() {
  const [isLoading, setIsLoading] = useState(true);
  const [users, setUsers] = useState([]);

  return <UserListWithLoading isLoading={isLoading} users={users} />;
}
```

**4. 데이터 패칭 HOC:**

```javascript
function withDataFetching(Component, url) {
  return function ComponentWithData(props) {
    const [data, setData] = useState(null);
    const [loading, setLoading] = useState(true);
    const [error, setError] = useState(null);

    useEffect(() => {
      fetch(url)
        .then((res) => res.json())
        .then((data) => {
          setData(data);
          setLoading(false);
        })
        .catch((err) => {
          setError(err);
          setLoading(false);
        });
    }, []);

    if (loading) return <div>Loading...</div>;
    if (error) return <div>Error: {error.message}</div>;

    return <Component data={data} {...props} />;
  };
}

// 사용
const UserList = withDataFetching(
  ({ data }) => (
    <ul>
      {data.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  ),
  "/api/users"
);
```

**5. HOC의 문제점:**

```javascript
// Props 충돌
const Enhanced = withAuth(withLoading(withData(MyComponent)));
// 어떤 props가 어디서 오는지 불명확

// Wrapper Hell
<WithAuth>
  <WithLoading>
    <WithData>
      <MyComponent />
    </WithData>
  </WithLoading>
</WithAuth>;
```

**6. 현대적 대안: Custom Hooks ✅**

```javascript
// HOC 대신 Hook 사용
function useAuth() {
  const { isLoggedIn } = useAuthContext();
  const navigate = useNavigate();

  useEffect(() => {
    if (!isLoggedIn) {
      navigate("/login");
    }
  }, [isLoggedIn]);

  return { isLoggedIn };
}

function useDataFetching(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(url)
      .then((res) => res.json())
      .then((data) => {
        setData(data);
        setLoading(false);
      })
      .catch((err) => {
        setError(err);
        setLoading(false);
      });
  }, [url]);

  return { data, loading, error };
}

// 사용 - 훨씬 깔끔!
function MyPage() {
  const { isLoggedIn } = useAuth();
  const { data, loading, error } = useDataFetching("/api/users");

  if (!isLoggedIn) return null;
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {data.map((u) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

**7. HOC vs Hooks 비교:**

| 특징           | HOC                 | Hooks     |
| -------------- | ------------------- | --------- |
| **가독성**     | 낮음 (Wrapper Hell) | 높음      |
| **Props 충돌** | 가능                | 없음      |
| **타입 추론**  | 어려움              | 쉬움      |
| **디버깅**     | 어려움              | 쉬움      |
| **권장**       | ❌ (레거시)         | ✅ (현대) |

---

## Custom Hooks

### Q. Custom Hooks는 무엇이고, 어떻게 만드나요?

**답변:**

Custom Hooks는 **로직을 재사용하기 위해 만드는 함수**로, `use`로 시작하는 이름 규칙을 따릅니다.

**1. 기본 패턴:**

```javascript
function useCustomHook() {
  // Hooks 사용 가능
  const [state, setState] = useState();

  useEffect(() => {
    // 로직
  }, []);

  // 값이나 함수 반환
  return { state, setState };
}
```

**2. useToggle - 불리언 상태 토글:**

```javascript
function useToggle(initialValue = false) {
  const [value, setValue] = useState(initialValue);

  const toggle = useCallback(() => {
    setValue((v) => !v);
  }, []);

  return [value, toggle];
}

// 사용
function Component() {
  const [isOpen, toggleOpen] = useToggle(false);

  return (
    <div>
      <button onClick={toggleOpen}>{isOpen ? "Close" : "Open"}</button>
      {isOpen && <div>Content</div>}
    </div>
  );
}
```

**3. useLocalStorage - 로컬 스토리지 동기화:**

```javascript
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });

  const setValue = (value) => {
    try {
      const valueToStore =
        value instanceof Function ? value(storedValue) : value;

      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error(error);
    }
  };

  return [storedValue, setValue];
}

// 사용
function Component() {
  const [name, setName] = useLocalStorage("name", "John");

  return <input value={name} onChange={(e) => setName(e.target.value)} />;
}
```

**4. useFetch - 데이터 패칭:**

```javascript
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const abortController = new AbortController();

    setLoading(true);

    fetch(url, { signal: abortController.signal })
      .then((res) => res.json())
      .then((data) => {
        setData(data);
        setLoading(false);
      })
      .catch((err) => {
        if (err.name !== "AbortError") {
          setError(err);
          setLoading(false);
        }
      });

    return () => {
      abortController.abort();
    };
  }, [url]);

  return { data, loading, error };
}

// 사용
function UserList() {
  const { data, loading, error } = useFetch("/api/users");

  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;

  return (
    <ul>
      {data.map((user) => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}
```

**5. useDebounce - 디바운스:**

```javascript
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);

  return debouncedValue;
}

// 사용
function SearchInput() {
  const [searchTerm, setSearchTerm] = useState("");
  const debouncedSearchTerm = useDebounce(searchTerm, 500);

  useEffect(() => {
    if (debouncedSearchTerm) {
      // API 호출 (500ms 디바운스)
      fetch(`/api/search?q=${debouncedSearchTerm}`);
    }
  }, [debouncedSearchTerm]);

  return (
    <input
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

**6. usePrevious - 이전 값 저장:**

```javascript
function usePrevious(value) {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
}

// 사용
function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);

  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

**7. useWindowSize - 윈도우 크기:**

```javascript
function useWindowSize() {
  const [windowSize, setWindowSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight,
  });

  useEffect(() => {
    function handleResize() {
      setWindowSize({
        width: window.innerWidth,
        height: window.innerHeight,
      });
    }

    window.addEventListener("resize", handleResize);

    return () => {
      window.removeEventListener("resize", handleResize);
    };
  }, []);

  return windowSize;
}

// 사용
function Component() {
  const { width, height } = useWindowSize();

  return (
    <div>
      Window size: {width} x {height}
    </div>
  );
}
```

**8. Custom Hooks 규칙:**

- ✅ 이름이 `use`로 시작해야 함
- ✅ 다른 Hooks를 호출할 수 있음
- ✅ 컴포넌트 최상위에서만 호출
- ✅ 조건문/반복문 안에서 호출 금지
- ✅ 일반 JavaScript 함수와 달리 React의 규칙 적용

---

[← 목차로 돌아가기](../README.md)
