# 번들러: Vite vs Webpack

[← 목차로 돌아가기](../README.md)

---

## 목차

- [Vite vs Webpack 비교](#vite-vs-webpack-비교)
- [Vite가 빠른 이유](#vite가-빠른-이유)
- [Pre-bundling](#pre-bundling)
- [ES Modules](#es-modules)
- [번들러 선택 가이드](#번들러-선택-가이드)

---

## Vite vs Webpack 비교

### Q. Vite가 Webpack보다 빠른 이유는 무엇인가요?

**답변:**

| 특징               | Webpack                       | Vite                     |
| ------------------ | ----------------------------- | ------------------------ |
| **개발 서버 시작** | 전체 번들링 후 시작 (느림 🐢) | 즉시 시작 (빠름 ⚡)      |
| **모듈 시스템**    | 모든 걸 번들로 변환           | ES Modules 네이티브 사용 |
| **HMR 속도**       | 앱 크기에 비례해 느려짐       | 항상 빠름                |
| **프로덕션 빌드**  | Webpack                       | Rollup                   |
| **트랜스파일러**   | Babel (JavaScript)            | esbuild (Go)             |

### 1. 개발 서버 시작 속도

**Webpack:**

```
$ npm run dev

1. entry 파일 분석
   ↓
2. 의존성 그래프 생성 (모든 import 추적)
   ↓
3. 전체 번들링
   - 모든 .js 파일 변환
   - 모든 .vue 파일 변환
   - 모든 .css 파일 처리
   ↓
4. 메모리에 번들 저장
   ↓
5. 개발 서버 시작 (수십 초 소요 🐢)
   ↓
6. 브라우저 접속 가능

대규모 프로젝트: 30초~2분
```

**Vite:**

```
$ npm run dev

1. 개발 서버 즉시 시작 (< 1초 ⚡)
   ↓
2. 브라우저 접속 가능
   ↓
3. 브라우저가 모듈 요청 시
   ↓
4. 요청된 파일만 on-demand 변환
   - App.vue 요청? → 즉시 변환 후 응답
   - router.js 요청? → 즉시 변환 후 응답

프로젝트 크기 무관: < 1초
```

### 2. HMR (Hot Module Replacement) 속도

**Webpack HMR:**

```
파일 변경 (예: Header.vue 수정)
    ↓
변경된 모듈 + 의존하는 모듈 재번들링
    ↓
WebSocket으로 브라우저에 전송
    ↓
브라우저가 모듈 교체

소규모 프로젝트: 1-2초
대규모 프로젝트: 3-10초 🐢
```

**Vite HMR:**

```
파일 변경 (예: Header.vue 수정)
    ↓
Header.vue만 변환 (초고속!)
    ↓
WebSocket으로 브라우저에 전송
    ↓
ES Modules로 정확한 모듈만 교체

프로젝트 크기 무관: < 100ms ⚡
```

---

## Vite가 빠른 이유

### 1. esbuild (Go 기반)

```javascript
// JavaScript 번들러 (Webpack, Rollup)
// - 싱글 스레드
// - 인터프리터 언어

function bundleFiles(files) {
  for (const file of files) {
    parse(file); // 순차 처리
    transform(file); // 순차 처리
    minify(file); // 순차 처리
  }
}
```

```go
// Go 번들러 (esbuild)
// - 네이티브 멀티스레드
// - 컴파일 언어

func BundleFiles(files []File) {
  var wg sync.WaitGroup
  for _, file := range files {
    wg.Add(1)
    go func(f File) {       // 병렬 처리!
      defer wg.Done()
      parse(f)
      transform(f)
      minify(f)
    }(file)
  }
  wg.Wait()
}
```

**속도 비교:**

| 작업             | Babel | esbuild               |
| ---------------- | ----- | --------------------- |
| 10,000 파일 변환 | 30초  | 0.3초 (100배 빠름 ⚡) |

### 2. ES Modules 네이티브 사용

**Webpack 방식:**

```javascript
// 원본 코드
import App from "./App.vue";
import router from "./router";

// Webpack이 변환
(function (modules) {
  function __webpack_require__(moduleId) {
    // ...
  }
  return __webpack_require__(0);
})([
  function (module, exports, require) {
    var App = require(1);
    var router = require(2);
  },
]);

// 결과: bundle.js (수 MB)
// 브라우저: bundle.js 1개 로드
```

**Vite 방식:**

```html
<!-- Vite가 보내는 HTML -->
<script type="module">
  import App from "/src/App.vue";
  import router from "/src/router";
</script>

<!-- 브라우저가 직접 ES Modules 실행 -->
<!-- 변환 없이 네이티브로! -->
```

---

## Pre-bundling

### Q. Vite의 Pre-bundling은 무엇이고 왜 필요한가요?

**답변:**

**Pre-bundling은 개발 서버에서만 사용되는 최적화 기법입니다.**

### 1. 무엇을 Pre-bundle?

```javascript
// main.js
import { createApp } from "vue"; // ✅ Pre-bundle
import { debounce } from "lodash-es"; // ✅ Pre-bundle
import axios from "axios"; // ✅ Pre-bundle
import MyComponent from "./MyComponent.vue"; // ❌ 소스 코드

// node_modules → Pre-bundle ✅
// src/ → On-demand 변환 ❌
```

**파일 위치:**

```
프로젝트/
├── node_modules/          ← Pre-bundle 대상
│   ├── vue/
│   ├── lodash-es/
│   └── axios/
│
├── node_modules/.vite/    ← Pre-bundle 결과
│   └── deps/
│       ├── vue.js         (캐시)
│       ├── lodash-es.js   (캐시)
│       └── axios.js       (캐시)
│
└── src/                   ← On-demand 변환
    ├── main.js
    ├── App.vue
    └── components/
```

### 2. 왜 필요한가?

**이유 1: CJS → ESM 변환**

```javascript
// axios는 CommonJS 사용
// node_modules/axios/lib/axios.js
module.exports = axios;

// Vite가 ESM으로 변환
// .vite/deps/axios.js
export default axios;
```

**이유 2: 수백 개 파일 → 1개로**

```javascript
// lodash-es: 600개 파일!
import { debounce } from 'lodash-es'

// Pre-bundle 없으면:
GET /node_modules/lodash-es/debounce.js
  → import './internal/...'
    → import './internal/...'
      → 600개 HTTP 요청! 💥

// Pre-bundle 후:
GET /.vite/deps/lodash-es.js
  → 1개 요청 ✅
```

**이유 3: 브라우저 네트워크 제한**

```
브라우저 동시 연결 제한: 6~10개

600개 파일:
- 6개씩 순차 로딩
- 총 100번의 왕복
- 매우 느림 🐢

1개 파일:
- 1번의 왕복
- 매우 빠름 ⚡
```

### 3. Pre-bundle 과정

```bash
$ npm run dev

# Vite가 하는 일:
1. package.json 스캔
2. node_modules의 의존성 발견
3. esbuild로 pre-bundle (1초 이내)
4. .vite/deps/에 캐시
5. 서버 시작

# 브라우저 요청:
GET /@vite/deps/vue.js
  → 캐시된 파일 응답 (매우 빠름)

GET /src/App.vue
  → on-demand 변환 후 응답
```

### 4. 프로덕션에서는?

```bash
$ npm run build

# ❌ Pre-bundling 사용 안 함!
# ✅ Rollup으로 전체 번들링

dist/
└── assets/
    ├── index.js       ← 전체 앱 + 의존성
    ├── vendor.js      ← vue, axios 등
    └── style.css
```

**개발 vs 프로덕션:**

```
개발 환경:
- Pre-bundle (esbuild)
- ES Modules
- .vite/deps/ 사용

프로덕션 환경:
- Rollup 번들링
- Tree-shaking
- Code splitting
- .vite/deps/ 사용 안 함
```

---

## ES Modules

### Q. Vite가 모던 브라우저에 적합한 이유는?

**답변:**

Vite는 **ES Modules를 네이티브로 사용**하기 때문입니다.

### 1. 브라우저 지원

```html
<!-- 모던 브라우저 (Chrome 61+, Firefox 60+, Safari 11+) -->
<script type="module">
  import { createApp } from "vue"; // ✅ 네이티브 지원
</script>

<!-- 구형 브라우저 (IE11) -->
<script type="module">
  import { createApp } from "vue"; // ❌ SyntaxError
</script>
```

| 브라우저    | ES Modules 지원 |
| ----------- | --------------- |
| Chrome 61+  | ✅              |
| Firefox 60+ | ✅              |
| Safari 11+  | ✅              |
| Edge 16+    | ✅              |
| **IE11**    | ❌              |

### 2. Webpack vs Vite 브라우저 지원

**Webpack:**

```javascript
// 원본 (ES2015+)
const double = (x) => x * 2;

// Webpack + Babel → ES5 변환
var double = function (x) {
  return x * 2;
};

// 결과: IE11에서도 실행 가능 ✅
```

**Vite 개발 서버:**

```html
<!-- Vite가 보내는 HTML -->
<script type="module" src="/src/main.js">

<!-- IE11: type="module" 자체를 이해 못함 💥 -->
```

### 3. Vite에서 구형 브라우저 지원

```javascript
// vite.config.js
import { defineConfig } from "vite";
import legacy from "@vitejs/plugin-legacy";

export default defineConfig({
  plugins: [
    legacy({
      targets: ["ie >= 11"],
      additionalLegacyPolyfills: ["regenerator-runtime/runtime"],
    }),
  ],
});
```

**빌드 결과:**

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- 모던 브라우저용 -->
    <script type="module" src="/assets/index.js"></script>

    <!-- 구형 브라우저용 (폴백) -->
    <script nomodule src="/assets/index-legacy.js"></script>
  </head>
</html>

<!-- 
모던 브라우저: type="module" 실행, nomodule 무시
구형 브라우저: type="module" 무시, nomodule 실행
-->
```

---

## 번들러 선택 가이드

### Q. 어떤 상황에서 어떤 번들러를 선택해야 하나요?

**답변:**

| 상황                        | 추천                          | 이유                            |
| --------------------------- | ----------------------------- | ------------------------------- |
| **신규 Vue/React 프로젝트** | Vite                          | 빠른 개발, 모던 기본값          |
| **레거시 프로젝트**         | Webpack                       | 안정성, IE 지원, 기존 설정 유지 |
| **라이브러리 개발**         | Rollup (또는 Vite lib 모드)   | Tree-shaking, 다양한 포맷       |
| **Monorepo**                | Turborepo + Vite              | 빌드 캐싱, 빠른 속도            |
| **SSR/SSG**                 | Next.js(Webpack) / Nuxt(Vite) | 프레임워크 통합                 |

### 시나리오별 선택

**시나리오 1: 신규 Vue 3 프로젝트 (팀 5명)**

- 목표: 빠른 개발 속도
- 레거시 브라우저 지원 불필요

```
✅ Vite 선택

이유:
- 개발 서버 < 1초
- HMR < 100ms
- 팀 생산성 향상
- Vue 3 공식 권장
```

**시나리오 2: 대규모 레거시 프로젝트**

- 기존 Webpack 설정 1000줄+
- 많은 커스텀 플러그인
- IE11 지원 필수

```
✅ Webpack 유지

이유:
- 이관 비용 > 이점
- IE11 필수
- 안정성 보장
- 팀 익숙함
```

**시나리오 3: NPM 라이브러리 개발**

- Tree-shaking 최적화 필수
- ESM, CJS, UMD 모두 지원

```
✅ Rollup (또는 Vite lib 모드)

이유:
- 최고의 Tree-shaking
- 다양한 포맷 지원
- 작은 번들 크기
```

### Webpack → Vite 마이그레이션

```markdown
## 체크리스트

### ❌ 마이그레이션 어려운 경우

- IE11 지원 필수
- require() 광범위 사용
- Webpack 전용 플러그인 다수
- 복잡한 커스텀 설정

### ✅ 마이그레이션 가능

- 모던 브라우저만 지원
- ES Modules 사용
- Vite 플러그인 대체 가능
- 새로운 도구 학습 의지
```

**점진적 마이그레이션:**

```
1단계: 새 기능부터 Vite로 개발
2단계: 개발 환경만 Vite 전환
3단계: 프로덕션은 Webpack 유지
4단계: 안정화 후 전체 전환
```

---

[← 목차로 돌아가기](../README.md)
