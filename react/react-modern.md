# React 최신 기능 (React 18/19)

[← 목차로 돌아가기](../README.md)

---

## 목차

- [React Server Components](#react-server-components)
- [Suspense 동작 원리](#suspense-동작-원리)
- [useTransition](#usetransition)
- [use() hook (React 19)](#use-hook-react-19)

---

## React Server Components

### Q. React 서버 컴포넌트가 어떻게 동작하나요?

**답변:**

**React Server Components (RSC)는 SSR과 다릅니다!**

| 특징                | SSR                 | RSC           |
| ------------------- | ------------------- | ------------- |
| **실행 위치**       | 서버                | 서버          |
| **클라이언트 번들** | 포함됨              | 포함 안 됨 ✅ |
| **Hydration**       | 필요                | 불필요 ✅     |
| **상태 관리**       | 가능 (Hydration 후) | 불가능        |
| **DB 접근**         | 불가능 (API 필요)   | 직접 가능 ✅  |

**1. 기본 구조:**

```jsx
// ServerComponent.server.jsx
// 서버에서만 실행 - 클라이언트 번들에 포함 안 됨!
async function ServerComponent() {
  // 직접 DB 접근 가능
  const data = await db.query("SELECT * FROM posts");

  // API 키 안전하게 사용
  const apiKey = process.env.SECRET_KEY;

  return (
    <div>
      {data.map((post) => (
        <ClientComponent key={post.id} post={post} />
      ))}
    </div>
  );
}

// ClientComponent.client.jsx
("use client"); // 클라이언트 컴포넌트 명시

function ClientComponent({ post }) {
  const [likes, setLikes] = useState(0);

  // 상태 관리 가능
  const handleLike = () => {
    setLikes(likes + 1);
  };

  return (
    <div>
      <h2>{post.title}</h2>
      <button onClick={handleLike}>Likes: {likes}</button>
    </div>
  );
}
```

**2. RSC의 장점:**

```javascript
// Before RSC
// 클라이언트에서 API 호출
function BlogPost({ id }) {
  const [post, setPost] = useState(null);

  useEffect(() => {
    fetch(`/api/posts/${id}`) // 별도 API 엔드포인트 필요
      .then((res) => res.json())
      .then(setPost);
  }, [id]);

  // 로딩 상태 처리, 에러 처리 등...
}

// After RSC
// 서버에서 직접 데이터 접근
async function BlogPost({ id }) {
  // DB 직접 쿼리
  const post = await db.posts.findUnique({ where: { id } });

  return <div>{post.title}</div>;
}
```

**3. 번들 크기 감소:**

```javascript
// 서버 컴포넌트에서 무거운 라이브러리 사용
import { marked } from "marked"; // Markdown 파서 (큰 라이브러리)
import Prism from "prismjs"; // 코드 하이라이터 (큰 라이브러리)

async function BlogPost({ id }) {
  const post = await getPost(id);

  // 서버에서 처리 - 클라이언트 번들에 포함 안 됨!
  const html = marked(post.content);
  const highlighted = Prism.highlight(html);

  return <div dangerouslySetInnerHTML={{ __html: highlighted }} />;
}
```

**4. 제약사항:**

```javascript
// ❌ 서버 컴포넌트에서 불가능
async function ServerComponent() {
  // const [state, setState] = useState(0); // ❌
  // useEffect(() => {}, []); // ❌
  // const handleClick = () => {}; // ❌

  // ✅ 서버 컴포넌트에서 가능
  const data = await fetchData();
  const result = heavyComputation(data);

  return <div>{result}</div>;
}
```

**5. 클라이언트 컴포넌트를 서버 컴포넌트의 자식으로:**

```javascript
// ✅ 가능: 서버 → 클라이언트
async function ServerComponent() {
  const data = await fetchData();

  return (
    <ClientComponent data={data}>
      <AnotherClientComponent />
    </ClientComponent>
  );
}

// ❌ 불가능: 클라이언트 → 서버
("use client");
function ClientComponent() {
  return (
    <div>
      {/* <ServerComponent /> */} {/* ❌ 직접 import 불가 */}
    </div>
  );
}
```

---

## Suspense 동작 원리

### Q. Suspense가 어떻게 Promise를 인지하나요?

**답변:**

Suspense는 **Promise를 throw**하는 독특한 메커니즘으로 동작합니다!

**1. React 18 이전 (Legacy 방식):**

```javascript
function fetchUser(id) {
  let status = "pending";
  let result;

  const suspender = fetch(`/api/users/${id}`)
    .then((res) => res.json())
    .then((data) => {
      status = "success";
      result = data;
    })
    .catch((error) => {
      status = "error";
      result = error;
    });

  return {
    read() {
      if (status === "pending") {
        throw suspender; // Promise를 throw!
      } else if (status === "error") {
        throw result;
      } else {
        return result;
      }
    },
  };
}

// 사용
function UserProfile({ id }) {
  const user = userResource.read(); // Promise를 throw!
  return <div>{user.name}</div>;
}

// React 내부 (단순화)
try {
  component.render();
} catch (thrown) {
  if (thrown instanceof Promise) {
    // Promise를 catch!
    thrown.then(() => {
      component.forceUpdate(); // Promise 완료 후 재렌더링
    });
    return <Fallback />; // fallback 렌더링
  }
}
```

**2. React 19 (use hook):**

```javascript
import { use } from "react";

function UserProfile({ userPromise }) {
  const user = use(userPromise); // Promise를 "소비"
  return <div>{user.name}</div>;
}

// 사용
function App() {
  const userPromise = fetch("/api/user").then((res) => res.json());

  return (
    <Suspense fallback={<Loading />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

**3. 동작 순서:**

```
1. 컴포넌트 렌더링 시작
   ↓
2. Promise 발견 (throw 또는 use)
   ↓
3. Suspense가 catch
   ↓
4. fallback 렌더링
   ↓
5. Promise 완료 대기
   ↓
6. 컴포넌트 재렌더링
   ↓
7. 실제 UI 표시
```

**4. 중첩된 Suspense:**

```javascript
function App() {
  return (
    <Suspense fallback={<PageLoader />}>
      <Header /> {/* 빠르게 로드 */}
      <Suspense fallback={<ContentLoader />}>
        <Content /> {/* 느리게 로드 */}
      </Suspense>
      <Suspense fallback={<SidebarLoader />}>
        <Sidebar /> {/* 느리게 로드 */}
      </Suspense>
    </Suspense>
  );
}

// Header가 먼저 표시되고,
// Content와 Sidebar는 각자의 로딩 상태를 가짐
```

**5. React 19의 주요 변경사항:**

```javascript
// React 18: 병렬 패칭
<Suspense fallback={<Loading />}>
  <ComponentA /> {/* fetch A */}
  <ComponentB /> {/* fetch B */}
</Suspense>
// → A와 B가 동시에 패칭됨 ✅

// React 19: Waterfall 방식
<Suspense fallback={<Loading />}>
  <ComponentA /> {/* fetch A */}
  <ComponentB /> {/* A 완료 후 fetch B */}
</Suspense>
// → A 완료 후 B 패칭 시작 ⚠️

// 해결: Route Loader나 Server Components 사용
// 렌더링 전에 데이터 패칭 시작
```

---

## useTransition

### Q. `useTransition`은 언제 사용하나요?

**답변:**

`useTransition`은 **긴급/비긴급 업데이트를 분리**하여 UI 응답성을 유지합니다.

| 특징          | 일반 업데이트 | Transition 업데이트 |
| ------------- | ------------- | ------------------- |
| **우선순위**  | 높음          | 낮음                |
| **UI 블로킹** | 블로킹        | 논블로킹            |
| **중단 가능** | 불가능        | 가능                |

**1. 기본 사용법:**

```javascript
function SearchResults() {
  const [query, setQuery] = useState("");
  const [filteredResults, setFilteredResults] = useState([]);
  const [isPending, startTransition] = useTransition();

  const handleChange = (e) => {
    const value = e.target.value;

    // 긴급: 입력값은 즉시 업데이트
    setQuery(value);

    // 비긴급: 필터링은 나중에
    startTransition(() => {
      const results = expensiveFilter(value);
      setFilteredResults(results);
    });
  };

  return (
    <div>
      <input value={query} onChange={handleChange} placeholder="Search..." />

      {isPending && <Spinner />}

      <ResultsList results={filteredResults} />
    </div>
  );
}
```

**2. 긴급 vs 비긴급:**

```javascript
// 긴급 업데이트 (즉시 처리)
- 사용자 입력 (타이핑, 클릭)
- 애니메이션
- 즉각적인 피드백

// 비긴급 업데이트 (나중에 처리)
- 검색 결과 필터링
- 대량 리스트 렌더링
- 복잡한 계산 결과
```

**3. 탭 전환 예제:**

```javascript
function TabContainer() {
  const [tab, setTab] = useState("home");
  const [isPending, startTransition] = useTransition();

  const handleTabChange = (newTab) => {
    startTransition(() => {
      setTab(newTab);
    });
  };

  return (
    <div>
      <TabButtons
        selected={tab}
        onChange={handleTabChange}
        isPending={isPending}
      />

      <TabContent tab={tab} />
    </div>
  );
}

// 탭 버튼은 즉시 응답하고,
// 탭 콘텐츠 렌더링은 나중에
```

**4. useTransition vs Suspense:**

```javascript
// useTransition: CPU 바운드 작업
function FilterResults() {
  const [isPending, startTransition] = useTransition();

  const handleFilter = () => {
    startTransition(() => {
      // 무거운 필터링 (CPU 작업)
      setResults(heavyFilter(data));
    });
  };
}

// Suspense: I/O 바운드 작업
function FetchResults() {
  return (
    <Suspense fallback={<Loading />}>
      <AsyncComponent /> {/* API 호출 (I/O 작업) */}
    </Suspense>
  );
}
```

**5. 실전 예제 - 대량 렌더링:**

```javascript
function BigList({ items }) {
  const [searchTerm, setSearchTerm] = useState("");
  const [displayedItems, setDisplayedItems] = useState(items);
  const [isPending, startTransition] = useTransition();

  const handleSearch = (e) => {
    const term = e.target.value;
    setSearchTerm(term);

    startTransition(() => {
      // 10,000개 항목 필터링 (무거운 작업)
      const filtered = items.filter((item) =>
        item.name.toLowerCase().includes(term.toLowerCase())
      );
      setDisplayedItems(filtered);
    });
  };

  return (
    <div>
      <input
        value={searchTerm}
        onChange={handleSearch}
        placeholder="Search 10,000 items..."
      />

      {isPending && <div>Filtering...</div>}

      <ul>
        {displayedItems.map((item) => (
          <li key={item.id}>{item.name}</li>
        ))}
      </ul>
    </div>
  );
}
```

---

## use() hook (React 19)

### Q. React 19의 `use` hook은 무엇인가요?

**답변:**

`use`는 **Promise와 Context를 조건부로 읽을 수 있는** 새로운 hook입니다.

**1. Promise 읽기:**

```javascript
import { use } from "react";

function UserProfile({ userPromise }) {
  // Promise를 "소비"
  const user = use(userPromise);

  return (
    <div>
      <h1>{user.name}</h1>
      <p>{user.email}</p>
    </div>
  );
}

// 사용
function App() {
  const userPromise = fetch("/api/user").then((res) => res.json());

  return (
    <Suspense fallback={<Loading />}>
      <UserProfile userPromise={userPromise} />
    </Suspense>
  );
}
```

**2. 조건부 Promise 읽기 (Hook 규칙 예외!):**

```javascript
function Profile({ userId }) {
  // 조건부로 use 호출 가능!
  if (!userId) {
    return <div>No user selected</div>;
  }

  // userId가 있을 때만 Promise 읽기
  const user = use(fetchUser(userId));

  return <div>{user.name}</div>;
}
```

**3. Context 읽기:**

```javascript
const ThemeContext = createContext("light");

function Button() {
  // 조건부 Context 읽기
  const isSpecialMode = checkSomeCondition();

  if (!isSpecialMode) {
    return <button>Normal Button</button>;
  }

  // 특별한 경우에만 Context 읽기
  const theme = use(ThemeContext);

  return (
    <button style={{ color: theme === "dark" ? "white" : "black" }}>
      Themed Button
    </button>
  );
}
```

**4. useContext와 비교:**

```javascript
// useContext: 최상위에서만 호출 가능
function Component() {
  const theme = useContext(ThemeContext); // 항상 호출

  if (someCondition) {
    return <div>No theme needed</div>;
  }

  return <div style={{ color: theme }}>Content</div>;
}

// use: 조건부 호출 가능
function Component() {
  if (someCondition) {
    return <div>No theme needed</div>;
  }

  const theme = use(ThemeContext); // 필요할 때만 호출!

  return <div style={{ color: theme }}>Content</div>;
}
```

**5. 캐싱 전략:**

```javascript
// Promise는 컴포넌트 외부에서 생성하여 캐싱
const userCache = new Map();

function getUserPromise(id) {
  if (!userCache.has(id)) {
    const promise = fetch(`/api/users/${id}`).then((res) => res.json());
    userCache.set(id, promise);
  }
  return userCache.get(id);
}

function UserProfile({ userId }) {
  // 캐시된 Promise 사용
  const user = use(getUserPromise(userId));

  return <div>{user.name}</div>;
}
```

**6. 에러 처리:**

```javascript
function UserProfile({ userPromise }) {
  try {
    const user = use(userPromise);
    return <div>{user.name}</div>;
  } catch (error) {
    // Promise가 reject되면 에러 throw
    return <div>Error: {error.message}</div>;
  }
}

// 또는 Error Boundary 사용
<ErrorBoundary fallback={<ErrorMessage />}>
  <Suspense fallback={<Loading />}>
    <UserProfile userPromise={userPromise} />
  </Suspense>
</ErrorBoundary>;
```

**7. use vs await:**

```javascript
// ❌ 컴포넌트에서 await 직접 사용 불가
async function Component() {
  const data = await fetchData(); // ❌ Error!
  return <div>{data}</div>;
}

// ✅ Server Component에서는 await 가능
async function ServerComponent() {
  const data = await fetchData(); // ✅ 서버에서만
  return <div>{data}</div>;
}

// ✅ Client Component에서는 use 사용
function ClientComponent({ dataPromise }) {
  const data = use(dataPromise); // ✅
  return <div>{data}</div>;
}
```

---

[← 목차로 돌아가기](../README.md)
