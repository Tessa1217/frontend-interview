# Vue 핵심 개념

[← 목차로 돌아가기](../README.md)

---

## 목차

- [Composition API vs Options API](#composition-api-vs-options-api)
- [ref vs reactive](#ref-vs-reactive)
- [toRefs와 toRef](#torefs와-toref)
- [Computed vs Watch](#computed-vs-watch)
- [라이프사이클 훅](#라이프사이클-훅)

---

## Composition API vs Options API

### Q. Composition API와 Options API의 차이점과, Composition API가 등장한 배경은?

**답변:**

**Options API:**

- Vue 2의 전통적인 방식
- data, methods, computed, watch 등을 객체 옵션으로 정의
- React의 클래스 컴포넌트와 유사

**Composition API:**

- Vue 3에서 도입
- `setup()` 함수 내에서 로직을 구성
- React Hooks와 유사한 패러다임

### 1. Options API 방식

```vue
<script>
export default {
  data() {
    return {
      count: 0,
      user: null,
      loading: false,
    };
  },

  computed: {
    doubleCount() {
      return this.count * 2;
    },
  },

  methods: {
    increment() {
      this.count++;
    },

    async fetchUser() {
      this.loading = true;
      this.user = await api.getUser();
      this.loading = false;
    },
  },

  mounted() {
    this.fetchUser();
  },
};
</script>
```

**Options API의 한계:**

```javascript
// ❌ 관련 로직이 여기저기 흩어짐
export default {
  data() {
    return {
      // 기능 A 관련
      searchQuery: "",
      searchResults: [],

      // 기능 B 관련
      userList: [],
      selectedUser: null,

      // 기능 A 관련
      searchHistory: [],
    };
  },

  methods: {
    // 기능 A
    search() {
      /*...*/
    },
    saveSearchHistory() {
      /*...*/
    },

    // 기능 B
    selectUser() {
      /*...*/
    },
    loadUsers() {
      /*...*/
    },
  },

  mounted() {
    // 기능 A 초기화
    this.loadSearchHistory();
    // 기능 B 초기화
    this.loadUsers();
  },
};

// 기능이 커질수록 코드 추적이 어려워짐!
```

**Mixin의 문제:**

```javascript
// ❌ Mixin Hell
export default {
  mixins: [userMixin, authMixin, analyticsMixin],

  data() {
    return {
      // 어느 mixin에서 온 데이터인지 불명확
      loading: false, // 충돌 가능성
      error: null,
    };
  },

  methods: {
    // 같은 이름의 메서드가 여러 mixin에 있다면?
    fetchData() {
      /*...*/
    },
  },
};
```

### 2. Composition API 방식

```vue
<script setup>
import { ref, computed, onMounted } from "vue";

// 기능별로 명확하게 분리!
const count = ref(0);
const doubleCount = computed(() => count.value * 2);

const increment = () => {
  count.value++;
};

// 사용자 관련 로직
const user = ref(null);
const loading = ref(false);

const fetchUser = async () => {
  loading.value = true;
  user.value = await api.getUser();
  loading.value = false;
};

onMounted(() => {
  fetchUser();
});
</script>
```

**로직 재사용 - Composables:**

```javascript
// composables/useSearch.js
import { ref } from "vue";

export function useSearch() {
  const searchQuery = ref("");
  const searchResults = ref([]);
  const searchHistory = ref([]);

  const search = async () => {
    searchResults.value = await api.search(searchQuery.value);
    searchHistory.value.push(searchQuery.value);
  };

  return {
    searchQuery,
    searchResults,
    searchHistory,
    search,
  };
}

// composables/useUserList.js
export function useUserList() {
  const userList = ref([]);
  const selectedUser = ref(null);

  const loadUsers = async () => {
    userList.value = await api.getUsers();
  };

  const selectUser = (user) => {
    selectedUser.value = user;
  };

  return {
    userList,
    selectedUser,
    loadUsers,
    selectUser,
  };
}

// 컴포넌트에서 사용
<script setup>
  import {useSearch} from './composables/useSearch' import {useUserList} from
  './composables/useUserList' // 깔끔하게 분리된 기능! const{" "}
  {(searchQuery, searchResults, search)} = useSearch() const{" "}
  {(userList, selectedUser, loadUsers)} = useUserList()
</script>;
```

### 3. setup() 실행 타이밍

```javascript
// 라이프사이클 순서
setup()              // ⬅️ 가장 먼저!
    ↓
beforeCreate         // (Composition API에서는 불필요)
    ↓
created              // (Composition API에서는 불필요)
    ↓
beforeMount
    ↓
mounted
    ↓
beforeUpdate
    ↓
updated
    ↓
beforeUnmount
    ↓
unmounted
```

```vue
<script>
export default {
  setup() {
    console.log("1. setup");

    // setup에서 사용 가능한 훅
    onMounted(() => {
      console.log("3. mounted");
    });

    return {};
  },

  beforeCreate() {
    console.log("❌ 실행 안 됨");
  },

  created() {
    console.log("❌ 실행 안 됨");
  },

  mounted() {
    console.log("4. mounted (Options)");
  },
};

// 출력:
// 1. setup
// 3. mounted (Composition)
// 4. mounted (Options)
</script>
```

---

## ref vs reactive

### Q. ref와 reactive의 차이점은 무엇인가요?

**답변:**

| 특징          | ref            | reactive                         |
| ------------- | -------------- | -------------------------------- |
| **타입**      | 원시값 + 객체  | 객체만 (Object, Array, Map, Set) |
| **접근 방식** | `.value` 필요  | 직접 접근                        |
| **재할당**    | ✅ 가능        | ❌ 불가능                        |
| **구조분해**  | ✅ 반응성 유지 | ❌ 반응성 상실                   |

### 1. ref 사용

```vue
<script setup>
import { ref } from "vue";

// 원시값
const count = ref(0);
const name = ref("Alice");

// 접근 시 .value 필요
console.log(count.value); // 0
count.value++; // 1

// 객체도 가능
const user = ref({ name: "Alice", age: 25 });

// 재할당 가능 ✅
user.value = { name: "Bob", age: 30 };

// 중첩 접근
console.log(user.value.name); // 'Bob'
user.value.name = "Charlie";
</script>

<template>
  <!-- template에서는 .value 생략 -->
  <div>{{ count }}</div>
  <div>{{ user.name }}</div>
</template>
```

### 2. reactive 사용

```vue
<script setup>
import { reactive } from "vue";

// 객체만 가능
const state = reactive({
  count: 0,
  user: {
    name: "Alice",
    age: 25,
  },
});

// 직접 접근
console.log(state.count); // 0
state.count++; // 1

// 재할당 불가 ❌
// state = { count: 10 }  // Error!

// 중첩 접근
console.log(state.user.name); // 'Alice'
state.user.name = "Bob";
</script>

<template>
  <div>{{ state.count }}</div>
  <div>{{ state.user.name }}</div>
</template>
```

### 3. 구조분해 함정

```vue
<script setup>
import { ref, reactive } from "vue";

// ref: 구조분해 가능 ✅
const count = ref(0);
// count 자체가 ref 객체이므로 분해 의미 없음

// reactive: 구조분해 시 반응성 상실 ❌
const state = reactive({
  count: 0,
  name: "Alice",
});

const { count: stateCount, name } = state;

// ❌ 반응성 없음!
stateCount++; // UI 업데이트 안 됨
name = "Bob"; // UI 업데이트 안 됨

console.log(state.count); // 0 (변경 안 됨!)
console.log(state.name); // 'Alice' (변경 안 됨!)
</script>
```

### 4. 언제 무엇을 사용?

```javascript
// ✅ ref 사용: 원시값 또는 재할당 필요
const count = ref(0);
const isLoading = ref(false);
const user = ref(null); // 나중에 객체 할당

user.value = await fetchUser(); // 재할당 가능!

// ✅ reactive 사용: 관련된 상태 그룹화
const formState = reactive({
  username: "",
  password: "",
  email: "",
  errors: {},
});

// this처럼 사용
formState.username = "test";
formState.errors.username = "Too short";

// ✅ 혼합 사용
const form = reactive({
  data: {
    username: "",
    email: "",
  },
  loading: false,
  error: null,
});

// 또는
const formData = reactive({ username: "", email: "" });
const loading = ref(false);
const error = ref(null);
```

---

## toRefs와 toRef

### Q. toRefs와 toRef는 무엇이고 언제 사용하나요?

**답변:**

**toRefs**: reactive 객체의 모든 속성을 ref로 변환  
**toRef**: reactive 객체의 특정 속성만 ref로 변환

### 1. toRefs 사용

```javascript
import { reactive, toRefs } from "vue";

const state = reactive({
  count: 0,
  name: "Alice",
  age: 25,
});

// ❌ 구조분해 시 반응성 상실
const { count, name } = state;
count++; // 반응성 없음!

// ✅ toRefs로 해결
const { count, name, age } = toRefs(state);
count.value++; // 반응성 유지! ✅

console.log(state.count); // 1 (연결됨!)
```

### 2. toRef 사용 (개별 속성)

```javascript
import { reactive, toRef } from "vue";

const state = reactive({
  count: 0,
  name: "Alice",
  age: 25,
});

// 특정 속성만 ref로
const countRef = toRef(state, "count");
const nameRef = toRef(state, "name");

countRef.value++;
console.log(state.count); // 1 (연결됨!)

nameRef.value = "Bob";
console.log(state.name); // 'Bob' (연결됨!)
```

### 3. Composable에서의 활용

```javascript
// composables/useUser.js
import { reactive, toRefs } from "vue";

export function useUser() {
  const state = reactive({
    user: null,
    loading: false,
    error: null,
  });

  const fetchUser = async (id) => {
    state.loading = true;
    try {
      state.user = await api.getUser(id);
    } catch (e) {
      state.error = e.message;
    } finally {
      state.loading = false;
    }
  };

  // ✅ toRefs로 반응성 유지하며 구조분해 가능하게
  return {
    ...toRefs(state),
    fetchUser,
  };
}

// 사용
const { user, loading, error, fetchUser } = useUser();

// .value로 접근
console.log(user.value);
if (loading.value) {
  /*...*/
}
```

### 4. 선택적 노출

```javascript
// composables/useAuth.js
import { reactive, toRef } from "vue";

export function useAuth() {
  const state = reactive({
    user: null,
    token: null, // 🔒 내부용
    refreshToken: null, // 🔒 내부용
    isAuthenticated: false,
  });

  const login = async (credentials) => {
    const response = await api.login(credentials);
    state.user = response.user;
    state.token = response.token;
    state.refreshToken = response.refreshToken;
    state.isAuthenticated = true;
  };

  // ✅ 필요한 것만 노출
  return {
    user: toRef(state, "user"),
    isAuthenticated: toRef(state, "isAuthenticated"),
    // token, refreshToken은 노출하지 않음 🔒
    login,
  };
}
```

---

## Computed vs Watch

### Q. computed와 watch의 차이점은?

**답변:**

| 특징          | computed              | watch               |
| ------------- | --------------------- | ------------------- |
| **목적**      | 파생 상태 계산        | 부수 효과 실행      |
| **반환값**    | 있음 (캐싱됨)         | 없음                |
| **실행 시점** | 의존성 변경 + 접근 시 | 의존성 변경 시 즉시 |
| **용도**      | 값 변환/계산          | API 호출, 로깅      |

### 1. computed 사용

```vue
<script setup>
import { ref, computed } from "vue";

const firstName = ref("John");
const lastName = ref("Doe");

// ✅ computed: 파생 상태
const fullName = computed(() => {
  console.log("computed 실행");
  return `${firstName.value} ${lastName.value}`;
});

// 같은 값 여러 번 접근해도 1번만 계산 (캐싱)
console.log(fullName.value); // "computed 실행" → "John Doe"
console.log(fullName.value); // "John Doe" (캐시)
console.log(fullName.value); // "John Doe" (캐시)

// 의존성 변경 시에만 재계산
firstName.value = "Jane";
console.log(fullName.value); // "computed 실행" → "Jane Doe"
</script>

<template>
  <!-- template에서 여러 번 사용해도 1번만 계산 -->
  <div>{{ fullName }}</div>
  <div>{{ fullName }}</div>
  <div>{{ fullName }}</div>
</template>
```

### 2. watch 사용

```vue
<script setup>
import { ref, watch } from "vue";

const searchQuery = ref("");
const searchResults = ref([]);

// ✅ watch: 부수 효과 (API 호출)
watch(searchQuery, async (newQuery, oldQuery) => {
  console.log(`검색어 변경: ${oldQuery} → ${newQuery}`);

  if (newQuery.length > 2) {
    searchResults.value = await api.search(newQuery);
  }
});

// 즉시 실행 옵션
watch(
  searchQuery,
  async (query) => {
    searchResults.value = await api.search(query);
  },
  { immediate: true }
); // 마운트 시 즉시 실행

// 깊은 감시
const state = ref({ user: { name: "Alice" } });

watch(
  state,
  (newState) => {
    console.log("state 변경");
  },
  { deep: true }
); // 중첩 객체 변경도 감지
</script>
```

### 3. watchEffect (자동 의존성 추적)

```vue
<script setup>
import { ref, watchEffect } from "vue";

const firstName = ref("John");
const lastName = ref("Doe");

// ✅ 의존성 자동 감지
watchEffect(() => {
  // firstName, lastName 자동으로 추적됨
  console.log(`Full name: ${firstName.value} ${lastName.value}`);
});

// 정리 함수
watchEffect((onCleanup) => {
  const timer = setTimeout(() => {
    console.log("타이머 실행");
  }, 1000);

  // 컴포넌트 언마운트 또는 재실행 전 정리
  onCleanup(() => {
    clearTimeout(timer);
  });
});
</script>
```

---

## 라이프사이클 훅

### Q. Vue 3 Composition API의 라이프사이클 훅은?

**답변:**

| Options API   | Composition API | 실행 시점                      |
| ------------- | --------------- | ------------------------------ |
| beforeCreate  | ❌ setup()      | 컴포넌트 인스턴스 생성 전      |
| created       | ❌ setup()      | 컴포넌트 인스턴스 생성 후      |
| beforeMount   | onBeforeMount   | DOM 마운트 전                  |
| mounted       | onMounted       | DOM 마운트 후                  |
| beforeUpdate  | onBeforeUpdate  | 반응형 데이터 변경 → 리렌더 전 |
| updated       | onUpdated       | 리렌더 후                      |
| beforeUnmount | onBeforeUnmount | 컴포넌트 언마운트 전           |
| unmounted     | onUnmounted     | 컴포넌트 언마운트 후           |

```vue
<script setup>
import {
  onBeforeMount,
  onMounted,
  onBeforeUpdate,
  onUpdated,
  onBeforeUnmount,
  onUnmounted,
} from "vue";

console.log("setup 실행 (beforeCreate + created)");

onBeforeMount(() => {
  console.log("DOM 마운트 전");
});

onMounted(() => {
  console.log("DOM 마운트 완료");
  // API 호출, DOM 조작, 이벤트 리스너 등록
});

onBeforeUpdate(() => {
  console.log("리렌더링 전");
});

onUpdated(() => {
  console.log("리렌더링 완료");
  // DOM 업데이트 후 작업
});

onBeforeUnmount(() => {
  console.log("언마운트 전");
  // 정리 작업 준비
});

onUnmounted(() => {
  console.log("언마운트 완료");
  // 이벤트 리스너 제거, 타이머 정리 등
});
</script>
```

**실전 예시:**

```vue
<script setup>
import { ref, onMounted, onUnmounted } from "vue";

const windowWidth = ref(window.innerWidth);

const handleResize = () => {
  windowWidth.value = window.innerWidth;
};

onMounted(() => {
  // 이벤트 리스너 등록
  window.addEventListener("resize", handleResize);
});

onUnmounted(() => {
  // 정리: 메모리 누수 방지
  window.removeEventListener("resize", handleResize);
});
</script>
```

---

[← 목차로 돌아가기](../README.md)
