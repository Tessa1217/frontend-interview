# CSS

[← 목차로 돌아가기](../README.md)

---

## 목차

- [px, em, rem 차이](#px-em-rem-차이)
- [반응형 브레이크포인트](#반응형-브레이크포인트)
- [Flexbox vs Grid](#flexbox-vs-grid)
- [CSS-in-JS](#css-in-js)

---

## px, em, rem 차이

### Q. px, em, rem의 차이는?

**답변:**

| 단위    | 타입 | 기준                   | 특징               |
| ------- | ---- | ---------------------- | ------------------ |
| **px**  | 절대 | 고정                   | 모든 환경에서 동일 |
| **em**  | 상대 | 부모 요소의 font-size  | 중첩 시 곱셈       |
| **rem** | 상대 | 루트(html)의 font-size | 중첩 무관          |

**1. px (픽셀) - 절대 단위:**

```css
.element {
  font-size: 16px; /* 항상 16px */
  width: 200px; /* 항상 200px */
}

/* 장점: 정확한 제어 */
/* 단점: 반응형에 불리 */
```

**2. em - 부모 기준 상대 단위:**

```css
html {
  font-size: 16px;
}

.parent {
  font-size: 20px; /* 20px */
}

.child {
  font-size: 2em; /* 40px (20 * 2) */
  padding: 1em; /* 40px (자신의 font-size 기준) */
}

.grandchild {
  font-size: 2em; /* 80px (40 * 2) - 중첩! */
}
```

**중첩 문제:**

```css
.level-1 {
  font-size: 2em;
} /* 32px (16 * 2) */
.level-2 {
  font-size: 2em;
} /* 64px (32 * 2) */
.level-3 {
  font-size: 2em;
} /* 128px (64 * 2) */
/* 예상치 못한 크기! */
```

**3. rem - 루트 기준 상대 단위:**

```css
html {
  font-size: 16px;
}

.parent {
  font-size: 2rem; /* 32px (16 * 2) */
}

.child {
  font-size: 2rem; /* 32px (16 * 2) - 동일! */
}

.grandchild {
  font-size: 2rem; /* 32px (16 * 2) - 동일! */
}
/* 중첩 영향 없음 ✅ */
```

**4. 반응형 웹에서의 활용:**

```css
/* 기본 설정 */
html {
  font-size: 16px; /* 1rem = 16px */
}

/* 모바일 */
@media (max-width: 768px) {
  html {
    font-size: 14px; /* 1rem = 14px */
  }
}

/* 요소들은 rem 사용 */
.heading {
  font-size: 2rem; /* 데스크톱: 32px, 모바일: 28px */
}

.text {
  font-size: 1rem; /* 데스크톱: 16px, 모바일: 14px */
}

.small {
  font-size: 0.875rem; /* 데스크톱: 14px, 모바일: 12.25px */
}
```

**5. 실무 권장 사항:**

```css
/* ✅ rem: 텍스트, 간격, 레이아웃 */
body {
  font-size: 1rem;
  line-height: 1.5rem;
}

.container {
  padding: 2rem;
  margin-bottom: 1rem;
}

/* ✅ em: 컴포넌트 내부 비율 */
.button {
  font-size: 1rem;
  padding: 0.5em 1em; /* 버튼 크기에 비례 */
}

/* ✅ px: 세밀한 제어 */
.border {
  border: 1px solid black; /* 1px은 항상 1px */
}

.icon {
  width: 24px;
  height: 24px;
}
```

**6. 접근성 고려:**

```css
/* ❌ 나쁜 예: px 고정 */
body {
  font-size: 16px;
  /* 사용자가 브라우저 설정으로 폰트 크기를 키워도 변하지 않음 */
}

/* ✅ 좋은 예: rem 사용 */
body {
  font-size: 1rem;
  /* 브라우저 설정에 따라 자동 조정 */
}
```

---

## 반응형 브레이크포인트

### Q. 반응형 브레이크포인트를 어떻게 정하나요?

**답변:**

**현대적 접근: Content-first (콘텐츠 우선) ✅**

브레이크포인트는 **디바이스가 아닌 콘텐츠가 깨지는 지점**에 설정합니다.

**1. 일반적인 브레이크포인트 (참고용):**

```css
/* Tailwind CSS 기본값 */
/* sm */
@media (min-width: 640px) {
} /* 큰 모바일, 작은 태블릿 */
/* md */
@media (min-width: 768px) {
} /* 태블릿 */
/* lg */
@media (min-width: 1024px) {
} /* 작은 노트북 */
/* xl */
@media (min-width: 1280px) {
} /* 데스크톱 */
/* 2xl */
@media (min-width: 1536px) {
} /* 큰 화면 */

/* Bootstrap */
/* xs */
@media (min-width: 0px) {
} /* 모바일 */
/* sm */
@media (min-width: 576px) {
}
/* md */
@media (min-width: 768px) {
}
/* lg */
@media (min-width: 992px) {
}
/* xl */
@media (min-width: 1200px) {
}
/* xxl */
@media (min-width: 1400px) {
}
```

**2. Mobile First 접근:**

```css
/* 기본: 모바일 (375px 이상) */
.container {
  padding: 1rem;
  width: 100%;
}

.grid {
  display: grid;
  grid-template-columns: 1fr;
  gap: 1rem;
}

/* 태블릿 (768px 이상) */
@media (min-width: 768px) {
  .container {
    padding: 2rem;
    max-width: 720px;
    margin: 0 auto;
  }

  .grid {
    grid-template-columns: repeat(2, 1fr);
  }
}

/* 데스크톱 (1024px 이상) */
@media (min-width: 1024px) {
  .container {
    padding: 3rem;
    max-width: 960px;
  }

  .grid {
    grid-template-columns: repeat(3, 1fr);
  }
}
```

**3. Content-based 브레이크포인트:**

```css
/* 콘텐츠가 깨지는 지점에 설정 */

/* 네비게이션이 겹칠 때 (600px) */
@media (min-width: 600px) {
  .nav {
    flex-direction: row;
  }
}

/* 텍스트가 너무 길어질 때 (900px) */
@media (min-width: 900px) {
  .text-content {
    max-width: 70ch; /* 70글자 */
  }
}

/* 사이드바 여유 공간 생길 때 (1200px) */
@media (min-width: 1200px) {
  .layout {
    display: grid;
    grid-template-columns: 250px 1fr;
  }
}
```

**4. 컨테이너 쿼리 (최신):**

```css
/* 컴포넌트 자체 크기 기준 */
.card-container {
  container-type: inline-size;
}

@container (min-width: 400px) {
  .card {
    display: grid;
    grid-template-columns: 1fr 2fr;
  }
}

/* 부모 크기에 따라 레이아웃 변경 */
```

**5. 실무 예제:**

```css
/* 3단계 반응형 */
:root {
  --container-padding: 1rem;
  --grid-columns: 1;
}

/* 모바일 (기본) */
.container {
  padding: var(--container-padding);
}

.grid {
  grid-template-columns: repeat(var(--grid-columns), 1fr);
}

/* 태블릿 */
@media (min-width: 640px) {
  :root {
    --container-padding: 2rem;
    --grid-columns: 2;
  }
}

/* 데스크톱 */
@media (min-width: 1024px) {
  :root {
    --container-padding: 3rem;
    --grid-columns: 3;
  }

  .container {
    max-width: 1200px;
    margin: 0 auto;
  }
}
```

**6. 브레이크포인트 관리:**

```scss
// SCSS 변수
$breakpoints: (
  "sm": 640px,
  "md": 768px,
  "lg": 1024px,
  "xl": 1280px,
);

@mixin respond-to($breakpoint) {
  @media (min-width: map-get($breakpoints, $breakpoint)) {
    @content;
  }
}

// 사용
.element {
  font-size: 1rem;

  @include respond-to("md") {
    font-size: 1.25rem;
  }

  @include respond-to("lg") {
    font-size: 1.5rem;
  }
}
```

---

## Flexbox vs Grid

### Q. Flexbox와 Grid는 언제 사용하나요?

**답변:**

| 특징         | Flexbox                 | Grid                |
| ------------ | ----------------------- | ------------------- |
| **레이아웃** | 1차원 (행 또는 열)      | 2차원 (행 + 열)     |
| **용도**     | 컴포넌트, 작은 레이아웃 | 페이지, 큰 레이아웃 |
| **정렬**     | 한 방향 정렬            | 양방향 정렬         |

**1. Flexbox - 1차원 레이아웃:**

```css
/* 네비게이션 */
.nav {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

/* 카드 내부 */
.card {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

/* 버튼 그룹 */
.button-group {
  display: flex;
  gap: 0.5rem;
}

/* 센터 정렬 */
.center {
  display: flex;
  justify-content: center;
  align-items: center;
}
```

**2. Grid - 2차원 레이아웃:**

```css
/* 페이지 레이아웃 */
.layout {
  display: grid;
  grid-template-areas:
    "header header header"
    "sidebar main aside"
    "footer footer footer";
  grid-template-columns: 200px 1fr 200px;
  grid-template-rows: auto 1fr auto;
  min-height: 100vh;
}

.header {
  grid-area: header;
}
.sidebar {
  grid-area: sidebar;
}
.main {
  grid-area: main;
}
.aside {
  grid-area: aside;
}
.footer {
  grid-area: footer;
}

/* 카드 그리드 */
.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 2rem;
}
```

**3. 선택 가이드:**

```css
/* ✅ Flexbox 사용 */
- 한 방향 배치 (가로 또는 세로)
- 컨텐츠 크기에 따라 자동 조정
- 동적인 요소 배치

/* ✅ Grid 사용 */
- 2차원 배치 (가로 + 세로)
- 고정된 레이아웃 구조
- 복잡한 페이지 레이아웃
```

**4. 함께 사용하기:**

```html
<div class="page-grid">
  <header class="flex-header">
    <div>Logo</div>
    <nav class="flex-nav">
      <a href="#">Home</a>
      <a href="#">About</a>
      <a href="#">Contact</a>
    </nav>
  </header>

  <main>
    <div class="card-grid">
      <div class="card flex-card">
        <img src="..." />
        <h3>Title</h3>
        <p>Content</p>
      </div>
    </div>
  </main>
</div>
```

```css
/* Grid: 페이지 레이아웃 */
.page-grid {
  display: grid;
  grid-template-rows: auto 1fr;
  min-height: 100vh;
}

/* Flexbox: 헤더 내부 */
.flex-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.flex-nav {
  display: flex;
  gap: 2rem;
}

/* Grid: 카드 그리드 */
.card-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 2rem;
}

/* Flexbox: 카드 내부 */
.flex-card {
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
```

---

## CSS-in-JS

### Q. CSS-in-JS의 장단점은?

**답변:**

**CSS-in-JS 라이브러리:**

- styled-components
- Emotion
- Vanilla Extract
- Stitches
- Panda CSS

**1. 장점:**

```javascript
// ✅ 동적 스타일링
const Button = styled.button`
  background: ${(props) => (props.primary ? "blue" : "gray")};
  color: white;
  padding: ${(props) => (props.size === "large" ? "12px 24px" : "8px 16px")};
`;

// ✅ 스코프 자동 생성 (CSS 충돌 방지)
// .Button-sc-abc123 { }

// ✅ 사용하지 않는 스타일 자동 제거
// 컴포넌트가 렌더링되지 않으면 CSS도 포함 안 됨

// ✅ TypeScript 지원
interface ButtonProps {
  primary?: boolean;
  size?: "small" | "large";
}

const Button =
  styled.button <
  ButtonProps >
  `
  /* 타입 안전한 props 사용 */
`;
```

**2. 단점:**

```javascript
// ❌ 런타임 오버헤드
// JavaScript 실행 후 CSS 생성

// ❌ 번들 크기 증가
// 라이브러리 자체 크기 (8-15KB gzipped)

// ❌ SSR 복잡도
// 서버에서 스타일 수집 및 주입 필요

// ❌ DevTools 불편
// 클래스명이 해시로 생성되어 디버깅 어려움
```

**3. styled-components 예시:**

```javascript
import styled from "styled-components";

// 기본 스타일
const Button = styled.button`
  background: blue;
  color: white;
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;

  &:hover {
    background: darkblue;
  }
`;

// Props 기반 스타일링
const Card = styled.div`
  padding: ${(props) => (props.compact ? "8px" : "16px")};
  background: ${(props) => (props.dark ? "#333" : "#fff")};
  color: ${(props) => (props.dark ? "#fff" : "#333")};
`;

// 확장
const PrimaryButton = styled(Button)`
  background: green;

  &:hover {
    background: darkgreen;
  }
`;

// 사용
function App() {
  return (
    <Card dark compact>
      <Button>Normal</Button>
      <PrimaryButton>Primary</PrimaryButton>
    </Card>
  );
}
```

**4. 성능 최적화:**

```javascript
// ✅ 정적 스타일은 밖으로
const Button = styled.button`
  /* 정적 스타일 */
  padding: 8px 16px;
  border: none;
`;

// ❌ 매 렌더링마다 스타일 재생성
function Component() {
  const Button = styled.button`
    background: blue;
  `;
  return <Button />;
}

// ✅ 컴포넌트 외부로 이동
const Button = styled.button`
  background: blue;
`;

function Component() {
  return <Button />;
}
```

**5. 대안: CSS Modules**

```css
/* Button.module.css */
.button {
  background: blue;
  color: white;
  padding: 8px 16px;
}

.button:hover {
  background: darkblue;
}

.primary {
  background: green;
}
```

```javascript
import styles from "./Button.module.css";

function Button({ primary }) {
  return (
    <button className={`${styles.button} ${primary ? styles.primary : ""}`}>
      Click me
    </button>
  );
}
```

**6. 선택 가이드:**

```
CSS-in-JS 사용:
- 동적 스타일이 많음
- 컴포넌트 라이브러리 제작
- TypeScript 타입 안전성 필요

일반 CSS/Tailwind 사용:
- 정적 스타일이 많음
- 성능 최우선
- 간단한 프로젝트
```

---

[← 목차로 돌아가기](../README.md)
