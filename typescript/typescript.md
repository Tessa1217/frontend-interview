# TypeScript

[← 목차로 돌아가기](../README.md)

---

## 목차

- [type vs interface](#type-vs-interface)
- [제네릭 (Generics)](#제네릭-generics)
- [유틸리티 타입](#유틸리티-타입)

---

## type vs interface

### Q. TypeScript에서 `type`과 `interface`의 차이점은?

**답변:**

| 특징                  | type                                        | interface        |
| --------------------- | ------------------------------------------- | ---------------- |
| **선언 가능한 타입**  | 모든 타입 (원시, 객체, 유니온, 인터섹션 등) | 주로 객체 타입   |
| **확장 방식**         | `&` (intersection)                          | `extends` 키워드 |
| **선언 병합**         | ❌ 불가능                                   | ✅ 가능          |
| **computed property** | ✅ 가능                                     | ❌ 불가능        |
| **유니온 타입**       | ✅ 가능                                     | ❌ 불가능        |

**1. 기본 사용법:**

```typescript
// Type
type User = {
  name: string;
  age: number;
};

// Interface
interface User {
  name: string;
  age: number;
}
```

**2. 확장 (Extends):**

```typescript
// Type - intersection (&)
type Animal = {
  name: string;
};

type Dog = Animal & {
  breed: string;
};

// Interface - extends
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}
```

**3. 선언 병합 (Declaration Merging):**

```typescript
// Interface만 가능!
interface User {
  name: string;
}

interface User {
  age: number;
}

// 자동으로 병합됨
const user: User = {
  name: "John",
  age: 25,
};

// Type은 불가능
type User = {
  name: string;
};

// type User = {  // ❌ Error: Duplicate identifier
//   age: number;
// };
```

**4. Type만 가능한 것들:**

**유니온 타입:**

```typescript
type Status = "pending" | "success" | "error";
type ID = string | number;

// Interface로는 불가능
```

**원시 타입 별칭:**

```typescript
type Name = string;
type Age = number;
```

**Tuple:**

```typescript
type Point = [number, number];
type RGB = [number, number, number];
```

**Computed Property:**

```typescript
type Keys = "name" | "age";

type User = {
  [K in Keys]: string;
};
// { name: string; age: string; }
```

**5. 실무 선택 기준:**

**Interface 사용 권장:**

```typescript
// 객체 타입 정의
interface User {
  id: number;
  name: string;
}

// 라이브러리 API (확장 가능하게)
interface APIResponse {
  data: any;
  status: number;
}

// React 컴포넌트 Props
interface ButtonProps {
  label: string;
  onClick: () => void;
}
```

**Type 사용 권장:**

```typescript
// 유니온 타입
type Status = "idle" | "loading" | "success" | "error";

// 복잡한 타입 조합
type Result<T> = { success: true; data: T } | { success: false; error: string };

// 유틸리티 타입 활용
type ReadonlyUser = Readonly<User>;
type PartialUser = Partial<User>;
```

**6. 혼용 사용:**

```typescript
// Interface와 Type 함께 사용
interface BaseUser {
  id: number;
  name: string;
}

type UserRole = "admin" | "user" | "guest";

type User = BaseUser & {
  role: UserRole;
};
```

**7. 성능 차이:**

```typescript
// Interface가 약간 더 빠름 (내부 구현 차이)
// 하지만 대부분의 경우 체감할 수 없을 정도로 미미함

// Interface: 타입 체크가 이름 기반
interface User {
  name: string;
}

// Type: 타입 체크가 구조 기반
type User = { name: string };
```

---

## 제네릭 (Generics)

### Q. TypeScript 제네릭이란 무엇이고, 언제 사용하나요?

**답변:**

제네릭은 타입을 **파라미터화**하여 재사용 가능한 컴포넌트를 만드는 기능입니다.

**1. 기본 문법:**

```typescript
// 제네릭 없이
function identityNumber(arg: number): number {
  return arg;
}

function identityString(arg: string): string {
  return arg;
}

// 제네릭 사용
function identity<T>(arg: T): T {
  return arg;
}

// 사용
const num = identity<number>(42); // number
const str = identity<string>("hello"); // string
const auto = identity(true); // 타입 추론: boolean
```

**2. 배열과 제네릭:**

```typescript
function getFirst<T>(arr: T[]): T | undefined {
  return arr[0];
}

const numbers = [1, 2, 3];
const first = getFirst(numbers); // number | undefined

const names = ["John", "Jane"];
const firstName = getFirst(names); // string | undefined
```

**3. 제약 조건 (Constraints):**

```typescript
// length 속성을 가진 타입만 허용
interface Lengthwise {
  length: number;
}

function logLength<T extends Lengthwise>(arg: T): T {
  console.log(arg.length);
  return arg;
}

logLength("hello"); // ✅ string has length
logLength([1, 2, 3]); // ✅ array has length
logLength({ length: 10, value: 3 }); // ✅ has length
// logLength(42);       // ❌ number doesn't have length
```

**4. 인터페이스와 제네릭:**

```typescript
interface Box<T> {
  value: T;
}

const numberBox: Box<number> = { value: 42 };
const stringBox: Box<string> = { value: "hello" };

// 여러 개의 타입 파라미터
interface Pair<K, V> {
  key: K;
  value: V;
}

const pair: Pair<string, number> = {
  key: "age",
  value: 25,
};
```

**5. 클래스와 제네릭:**

```typescript
class Stack<T> {
  private items: T[] = [];

  push(item: T): void {
    this.items.push(item);
  }

  pop(): T | undefined {
    return this.items.pop();
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
// numberStack.push('3'); // ❌ Error

const stringStack = new Stack<string>();
stringStack.push("hello");
```

**6. 실무 예제 - API 응답:**

```typescript
interface APIResponse<T> {
  data: T;
  status: number;
  message: string;
}

interface User {
  id: number;
  name: string;
}

interface Post {
  id: number;
  title: string;
  content: string;
}

async function fetchData<T>(url: string): Promise<APIResponse<T>> {
  const response = await fetch(url);
  return response.json();
}

// 사용
const userResponse = await fetchData<User>("/api/user/1");
// userResponse.data는 User 타입

const postResponse = await fetchData<Post>("/api/post/1");
// postResponse.data는 Post 타입
```

**7. React 컴포넌트와 제네릭:**

```typescript
interface ListProps<T> {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
}

function List<T>({ items, renderItem }: ListProps<T>) {
  return (
    <ul>
      {items.map((item, index) => (
        <li key={index}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}

// 사용
<List
  items={[1, 2, 3]}
  renderItem={(num) => <span>{num * 2}</span>}
/>

<List
  items={['a', 'b', 'c']}
  renderItem={(str) => <span>{str.toUpperCase()}</span>}
/>
```

---

## 유틸리티 타입

### Q. TypeScript의 주요 유틸리티 타입들은?

**답변:**

TypeScript는 타입 변환을 쉽게 하는 내장 유틸리티 타입들을 제공합니다.

**1. Partial<T> - 모든 속성을 선택적으로**

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

type PartialUser = Partial<User>;
// {
//   id?: number;
//   name?: string;
//   email?: string;
// }

// 사용 예: 업데이트 함수
function updateUser(id: number, updates: Partial<User>) {
  // updates는 일부 속성만 가질 수 있음
}

updateUser(1, { name: "John" }); // ✅
updateUser(2, { email: "john@example.com" }); // ✅
```

**2. Required<T> - 모든 속성을 필수로**

```typescript
interface Config {
  host?: string;
  port?: number;
  ssl?: boolean;
}

type RequiredConfig = Required<Config>;
// {
//   host: string;
//   port: number;
//   ssl: boolean;
// }
```

**3. Readonly<T> - 모든 속성을 읽기 전용으로**

```typescript
interface User {
  id: number;
  name: string;
}

type ReadonlyUser = Readonly<User>;

const user: ReadonlyUser = { id: 1, name: "John" };
// user.name = 'Jane'; // ❌ Error: Cannot assign to 'name'
```

**4. Pick<T, K> - 특정 속성만 선택**

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type UserPreview = Pick<User, "id" | "name">;
// {
//   id: number;
//   name: string;
// }

// 사용 예: 공개 정보만 반환
function getPublicUser(user: User): UserPreview {
  return { id: user.id, name: user.name };
}
```

**5. Omit<T, K> - 특정 속성 제외**

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  password: string;
}

type UserWithoutPassword = Omit<User, "password">;
// {
//   id: number;
//   name: string;
//   email: string;
// }

// 여러 속성 제외
type PublicUser = Omit<User, "password" | "email">;
```

**6. Record<K, T> - 키-값 쌍 객체 타입**

```typescript
type Role = "admin" | "user" | "guest";

type Permissions = Record<Role, string[]>;
// {
//   admin: string[];
//   user: string[];
//   guest: string[];
// }

const permissions: Permissions = {
  admin: ["read", "write", "delete"],
  user: ["read", "write"],
  guest: ["read"],
};
```

**7. Extract<T, U> - 유니온에서 추출**

```typescript
type Status = "idle" | "loading" | "success" | "error";

type SuccessStatus = Extract<Status, "success" | "error">;
// 'success' | 'error'
```

**8. Exclude<T, U> - 유니온에서 제외**

```typescript
type Status = "idle" | "loading" | "success" | "error";

type PendingStatus = Exclude<Status, "success" | "error">;
// 'idle' | 'loading'
```

**9. ReturnType<T> - 함수 반환 타입 추출**

```typescript
function getUser() {
  return { id: 1, name: "John", email: "john@example.com" };
}

type User = ReturnType<typeof getUser>;
// {
//   id: number;
//   name: string;
//   email: string;
// }
```

**10. Parameters<T> - 함수 파라미터 타입 추출**

```typescript
function createUser(name: string, age: number) {
  return { name, age };
}

type CreateUserParams = Parameters<typeof createUser>;
// [name: string, age: number]

function wrapper(...args: CreateUserParams) {
  return createUser(...args);
}
```

**11. 실무 예제 - Form 타입:**

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
  updatedAt: Date;
}

// 생성 폼: id, 날짜 필드 제외
type CreateUserForm = Omit<User, "id" | "createdAt" | "updatedAt">;

// 수정 폼: 모든 필드 선택적 + id 필수
type UpdateUserForm = Partial<User> & Pick<User, "id">;

// 읽기 전용 뷰
type UserView = Readonly<User>;
```

**12. 실무 예제 - API 상태:**

```typescript
type LoadingState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: string };

type UserState = LoadingState<User>;

function handleUserState(state: UserState) {
  switch (state.status) {
    case "idle":
      return "Not started";
    case "loading":
      return "Loading...";
    case "success":
      return `Welcome, ${state.data.name}`;
    case "error":
      return `Error: ${state.error}`;
  }
}
```

---

[← 목차로 돌아가기](../README.md)
