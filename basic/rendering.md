# 브라우저 동작 원리

[← 목차로 돌아가기](../README.md)

---

## 목차

- [렌더링 과정 (Critical Rendering Path)](#렌더링-과정-critical-rendering-path)
- [Reflow vs Repaint](#reflow-vs-repaint)
- [레이어 합성 (Composite)](#레이어-합성-composite)
- [성능 최적화](#성능-최적화)

---

## 렌더링 과정 (Critical Rendering Path)

### Q. 브라우저 렌더링 과정을 설명하세요

**답변:**

브라우저는 HTML, CSS, JavaScript를 화면에 그리기 위해 여러 단계를 거칩니다.

**전체 과정:**

```
1. Parsing (구문 분석)
   HTML → DOM Tree
   CSS → CSSOM Tree

2. Render Tree 생성
   DOM + CSSOM = Render Tree
   (display: none은 제외)

3. Layout (Reflow)
   각 요소의 위치, 크기 계산

4. Paint
   픽셀로 그리기

5. Composite
   레이어 합성
```

**1. Parsing 단계:**

```html
<!-- HTML 파싱 -->
<!DOCTYPE html>
<html>
  <head>
    <link rel="stylesheet" href="style.css" />
  </head>
  <body>
    <div class="container">
      <h1>Hello</h1>
    </div>
    <script src="app.js"></script>
  </body>
</html>
```

```
HTML 파싱 → DOM Tree
├── html
    ├── head
    │   └── link
    └── body
        ├── div.container
        │   └── h1
        └── script
```

**2. CSSOM Tree:**

```css
/* CSS 파싱 */
body {
  font-size: 16px;
}
.container {
  padding: 20px;
}
h1 {
  color: blue;
}
```

```
CSSOM Tree
└── body (font-size: 16px)
    └── .container (padding: 20px)
        └── h1 (color: blue)
```

**3. Render Tree (DOM + CSSOM):**

```
Render Tree (보이는 요소만)
└── body
    └── div.container
        └── h1 "Hello"

❌ 제외되는 것들:
- display: none
- <head>, <meta>, <script>
- visibility: hidden (공간은 차지, 렌더는 안 함)
```

**4. Layout (Reflow) - 위치/크기 계산:**

```javascript
// Viewport 기준으로 계산
div.container {
  x: 0, y: 0,
  width: 100%, height: auto
}

h1 {
  x: 20px, y: 20px,  // container의 padding
  width: calc(100% - 40px),
  height: 32px
}
```

**5. Paint - 픽셀로 그리기:**

```
레이어별로 픽셀 채우기:
1. 배경색 그리기
2. 보더 그리기
3. 텍스트 그리기
4. 그림자 그리기
```

**6. Composite - 레이어 합성:**

```
GPU가 레이어들을 합성
Layer 1 (background)
Layer 2 (content)
Layer 3 (transform elements)
    ↓
최종 화면
```

**중요한 포인트:**

```javascript
// JavaScript가 실행되면 렌더링 중단!
<script>
  // DOM 파싱 멈춤
  // JavaScript 실행
  // 다시 DOM 파싱 재개
</script>

// 해결: async/defer 사용
<script async src="app.js"></script>
<script defer src="app.js"></script>
```

---

## Reflow vs Repaint

### Q. Reflow와 Repaint의 차이는?

**답변:**

| 구분          | Reflow (Layout)    | Repaint          | Composite         |
| ------------- | ------------------ | ---------------- | ----------------- |
| **비용**      | 💸💸💸 (매우 높음) | 💸💸 (높음)      | 💸 (낮음)         |
| **발생 조건** | 위치/크기 변경     | 색상 변경        | transform/opacity |
| **계산**      | 레이아웃 재계산    | 픽셀 다시 그리기 | 레이어만 합성     |

**1. Reflow (Layout) - 가장 비쌈:**

```javascript
// Reflow를 유발하는 속성들
element.style.width = "100px";
element.style.height = "200px";
element.style.margin = "10px";
element.style.padding = "20px";
element.style.border = "1px solid black";
element.style.left = "50px";
element.style.top = "100px";
element.style.fontSize = "20px";

// 계산이 필요한 속성 읽기
const height = element.offsetHeight;
const width = element.clientWidth;
const rect = element.getBoundingClientRect();
```

**Reflow 과정:**

```
속성 변경
    ↓
Layout 계산 (Reflow)
    ↓
Paint (Repaint)
    ↓
Composite
```

**2. Repaint - 중간 비용:**

```javascript
// Repaint만 유발하는 속성들
element.style.color = "red";
element.style.backgroundColor = "blue";
element.style.visibility = "hidden";
element.style.outline = "1px solid red";
element.style.boxShadow = "2px 2px 5px black";
```

**Repaint 과정:**

```
속성 변경
    ↓
Paint (Repaint)
    ↓
Composite
```

**3. Composite만 - 가장 저렴:**

```javascript
// Composite만 유발하는 속성들 (GPU 가속)
element.style.transform = "translateX(100px)";
element.style.opacity = "0.5";
element.style.filter = "blur(5px)";
```

**Composite 과정:**

```
속성 변경
    ↓
Composite (GPU에서 처리)
```

**4. 성능 비교 예시:**

```javascript
// ❌ 나쁜 예: Reflow 유발 (300ms)
element.style.left = "100px";

// ✅ 좋은 예: Composite만 (16ms)
element.style.transform = "translateX(100px)";

// ❌ 나쁜 예: 여러 번 Reflow
element.style.width = "100px"; // Reflow
element.style.height = "200px"; // Reflow
element.style.margin = "10px"; // Reflow

// ✅ 좋은 예: 한 번에 처리
element.style.cssText = `
  width: 100px;
  height: 200px;
  margin: 10px;
`; // Reflow 1번만
```

**5. Layout Thrashing (레이아웃 스래싱) 방지:**

```javascript
// ❌ 나쁜 예: 읽기/쓰기 반복 (Reflow 여러 번)
elements.forEach((el) => {
  const height = el.offsetHeight; // 읽기 → Reflow
  el.style.height = height + 10 + "px"; // 쓰기 → Reflow
});

// ✅ 좋은 예: 읽기/쓰기 분리 (Reflow 2번만)
// 1. 모든 읽기
const heights = elements.map((el) => el.offsetHeight);

// 2. 모든 쓰기
elements.forEach((el, i) => {
  el.style.height = heights[i] + 10 + "px";
});
```

---

## 레이어 합성 (Composite)

### Q. 레이어 합성은 어떻게 동작하나요?

**답변:**

브라우저는 특정 조건에서 요소를 **별도의 레이어**로 분리하여 GPU로 합성합니다.

**1. 레이어 생성 조건:**

```css
/* 새 레이어를 만드는 속성들 */

/* transform 3D */
.element {
  transform: translateZ(0);
  transform: translate3d(0, 0, 0);
}

/* will-change */
.element {
  will-change: transform;
  will-change: opacity;
}

/* video, canvas */
<video></video>
<canvas></canvas>

/* position: fixed */
.element {
  position: fixed;
}

/* opacity + transform */
.element {
  opacity: 0.9;
  transform: translateX(0);
}
```

**2. 레이어 구조 예시:**

```html
<div class="page">
  <header>Header</header>
  <main>
    <div class="moving-box">Box</div>
  </main>
  <footer>Footer</footer>
</div>
```

```css
.moving-box {
  transform: translateZ(0); /* 새 레이어 생성 */
}
```

```
Layers:
├── Layer 1 (Document)
│   ├── header
│   ├── main (empty shell)
│   └── footer
└── Layer 2 (Moving Box)
    └── .moving-box
```

**3. GPU 가속 활용:**

```javascript
// ❌ CPU에서 처리 (느림)
element.style.left = '100px';

// ✅ GPU에서 처리 (빠름)
element.style.transform = 'translateX(100px)';

// 성능 차이
CPU: 30-60ms
GPU: 1-2ms
```

**4. will-change 최적화:**

```css
/* 애니메이션 시작 전 */
.button {
  will-change: transform;
}

.button:hover {
  transform: scale(1.1);
}

/* 주의: 너무 많이 사용하면 메모리 낭비 */
* {
  will-change: transform; /* ❌ 나쁨 */
}
```

**5. 레이어 디버깅:**

```javascript
// Chrome DevTools
// 1. More tools → Rendering
// 2. Layer borders 체크
// 3. 레이어 경계선 확인

// 또는 Layers 패널
// 1. More tools → Layers
// 2. 레이어 계층 구조 확인
```

---

## 성능 최적화

### Q. 렌더링 성능을 최적화하는 방법은?

**답변:**

**1. CSS 최적화:**

```css
/* ✅ 좋음: Composite만 사용 */
.box {
  transform: translateX(100px);
  opacity: 0.5;
}

/* ❌ 나쁨: Reflow 유발 */
.box {
  left: 100px;
  width: 200px;
}

/* ✅ 좋음: CSS containment */
.card {
  contain: layout style paint;
  /* 이 요소의 변경이 외부에 영향 안 줌 */
}

/* ✅ 좋음: content-visibility */
.long-content {
  content-visibility: auto;
  /* 뷰포트 밖 요소는 렌더링 스킵 */
}
```

**2. JavaScript 최적화:**

```javascript
// ✅ requestAnimationFrame 사용
function animate() {
  element.style.transform = `translateX(${position}px)`;
  requestAnimationFrame(animate);
}

// ❌ setInterval/setTimeout
setInterval(() => {
  element.style.left = position + "px";
}, 16);

// ✅ 배치 읽기/쓰기
const heights = elements.map((el) => el.offsetHeight);
elements.forEach((el, i) => {
  el.style.height = heights[i] + "px";
});

// ❌ 읽기/쓰기 섞기
elements.forEach((el) => {
  const h = el.offsetHeight;
  el.style.height = h + "px";
});
```

**3. 이미지 최적화:**

```html
<!-- Lazy loading -->
<img src="image.jpg" loading="lazy" />

<!-- Responsive images -->
<img
  srcset="small.jpg 400w, medium.jpg 800w, large.jpg 1200w"
  sizes="(max-width: 400px) 400px, (max-width: 800px) 800px, 1200px"
  src="medium.jpg"
  alt="Image"
/>

<!-- WebP with fallback -->
<picture>
  <source srcset="image.webp" type="image/webp" />
  <img src="image.jpg" alt="Image" />
</picture>
```

**4. Critical CSS:**

```html
<head>
  <!-- 인라인 Critical CSS -->
  <style>
    /* 초기 뷰에 필요한 최소 CSS */
    body {
      font-family: sans-serif;
    }
    .header {
      background: blue;
    }
  </style>

  <!-- 나머지 CSS는 비동기 로드 -->
  <link
    rel="preload"
    href="styles.css"
    as="style"
    onload="this.onload=null;this.rel='stylesheet'"
  />
</head>
```

**5. 리소스 힌트:**

```html
<!-- DNS Prefetch -->
<link rel="dns-prefetch" href="https://fonts.googleapis.com" />

<!-- Preconnect -->
<link rel="preconnect" href="https://api.example.com" />

<!-- Prefetch (다음 페이지) -->
<link rel="prefetch" href="/next-page.html" />

<!-- Preload (현재 페이지) -->
<link rel="preload" href="/critical-font.woff2" as="font" crossorigin />
```

**6. 가상 스크롤:**

```javascript
// 긴 리스트는 가상 스크롤 사용
import { FixedSizeList } from "react-window";

function BigList({ items }) {
  return (
    <FixedSizeList
      height={600}
      itemCount={items.length}
      itemSize={50}
      width="100%"
    >
      {({ index, style }) => <div style={style}>{items[index].name}</div>}
    </FixedSizeList>
  );
}
```

**7. 성능 측정:**

```javascript
// Performance API
const startTime = performance.now();
// ... 작업 수행
const endTime = performance.now();
console.log(`Execution time: ${endTime - startTime}ms`);

// PerformanceObserver
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    console.log(entry.name, entry.duration);
  }
});
observer.observe({ entryTypes: ["measure"] });

// Lighthouse
// Chrome DevTools → Lighthouse 탭
// 성능, 접근성, SEO 등 종합 분석
```

---

[← 목차로 돌아가기](../README.md)
