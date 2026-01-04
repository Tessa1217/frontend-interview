# Git 워크플로우 & CI/CD

[← 목차로 돌아가기](../README.md)

---

## 목차

- [트렁크 베이스 vs Git Flow](#트렁크-베이스-vs-git-flow)
- [Feature Flag 관리](#feature-flag-관리)
- [코드 리뷰 자동화](#코드-리뷰-자동화)

---

## 트렁크 베이스 vs Git Flow

### Q. 트렁크 베이스 개발과 Git Flow의 차이는?

**답변:**

**트렁크 베이스 개발(Trunk-Based Development)**과 **Git Flow**는 서로 다른 브랜치 전략입니다.

| 특징             | 트렁크 베이스    | Git Flow             |
| ---------------- | ---------------- | -------------------- |
| **브랜치 수명**  | 1-2일            | 1-2주+               |
| **머지 빈도**    | 하루 여러 번     | 주 1-2회             |
| **충돌 가능성**  | 낮음 (항상 최신) | 높음 (오래된 브랜치) |
| **CI/CD**        | 필수             | 선택                 |
| **Feature Flag** | 필수             | 선택                 |
| **적합한 팀**    | 빠른 배포 중시   | 계획적 릴리즈 중시   |

**1. 트렁크 베이스 개발:**

```
main (trunk)
 │
 ├─ feature/A (1-2일) ──┐
 │                      ├─→ merge (daily)
 ├─ feature/B (1-2일) ──┘
 │
 ├─ feature/C (1-2일) ──┐
 │                      ├─→ merge (daily)
 └─ feature/D (1-2일) ──┘
```

**핵심 원칙:**

- Feature 브랜치 수명: **1-2일 이내**
- 하루에 여러 번 main에 머지
- 작은 단위로 지속적 통합 (CI)
- Feature Flag로 미완성 기능 숨김

```javascript
// Feature Flag 예시
const FEATURES = {
  NEW_DASHBOARD: false, // 개발 중
  PAYMENT_V2: false, // 개발 중
};

function App() {
  return (
    <div>
      {FEATURES.NEW_DASHBOARD ? (
        <NewDashboard /> // 숨김
      ) : (
        <OldDashboard /> // 표시
      )}
    </div>
  );
}
```

**2. Git Flow:**

```
main
 │
 ├─ develop
 │   │
 │   ├─ feature/A (1-2주) ──┐
 │   ├─ feature/B (1-2주) ──┤
 │   └─ feature/C (1-2주) ──┴─→ develop
 │                              │
 ├──────────────────────────────┴─ release/v1.0
 │                                   │
 └───────────────────────────────────┴─ main (배포)
```

**핵심 원칙:**

- 장기 브랜치: main, develop
- Feature 브랜치 수명: **1-2주 이상**
- Release 브랜치로 배포 준비
- Hotfix 브랜치로 긴급 수정

**3. 대규모 팀에서의 전략:**

**시나리오 1: 대기업 (30명)**

```
# Git Flow 방식 (문제 발생)
Q1 작업 (1월~3월)
 ├─ 팀A feature/payment (3개월)
 ├─ 팀B feature/dashboard (3개월)
 └─ 팀C feature/analytics (3개월)
      ↓
3월 말: 3개 브랜치 동시 머지 → 💥 Merge Hell

# 트렁크 베이스 방식 (해결)
main (항상 배포 가능)
 ├─ 1월: 팀A 작은 PR 20개 (Feature Flag OFF)
 ├─ 2월: 팀B 작은 PR 25개 (Feature Flag OFF)
 └─ 3월: 팀C 작은 PR 30개 (Feature Flag OFF)
      ↓
3월 말: Feature Flag만 ON → 릴리즈 🎉
```

**4. 릴리즈 전략:**

```javascript
// Q1 릴리즈 준비 (트렁크 베이스)
const Q1_FEATURES = {
  PAYMENT_V2: false, // 팀A 작업 중 (50%)
  NEW_DASHBOARD: false, // 팀B 작업 중 (80%)
  ANALYTICS: false, // 팀C 작업 중 (30%)
};

// 3월 말 릴리즈 결정
const Q1_FEATURES = {
  PAYMENT_V2: true, // ✅ 완료 → 배포
  NEW_DASHBOARD: true, // ✅ 완료 → 배포
  ANALYTICS: false, // ❌ 미완성 → Q2로 연기
};
```

**5. Git Flow가 적합한 경우:**

```
의료/금융 소프트웨어:
- FDA 승인 필요
- 버전별 감사 필요
- 롤백 불가능

레거시 멀티버전 지원:
main (v3.0)
 ├─ release/v2.5 (구버전 유지보수)
 └─ release/v1.9 (레거시 지원)
```

---

## Feature Flag 관리

### Q. Feature Flag를 사용할 때의 문제점과 해결 방법은?

**답변:**

Feature Flag는 강력하지만 잘못 관리하면 **기술 부채**가 됩니다.

**1. 문제점: Flag Hell (분기 지옥)**

```javascript
// ❌ 6개월 후의 악몽
function Dashboard() {
  if (FEATURES.NEW_DASHBOARD) {
    if (FEATURES.DASHBOARD_ANALYTICS) {
      if (FEATURES.REALTIME_CHART) {
        return <NewDashboardWithAnalyticsAndRealtime />;
      }
      return <NewDashboardWithAnalytics />;
    }
    return <NewDashboard />;
  }

  if (FEATURES.OLD_DASHBOARD_REFRESH) {
    return <RefreshedOldDashboard />;
  }

  return <OldDashboard />;
}

// 조합 가능한 경우의 수: 2^4 = 16가지! 💥
```

**해결: 전략 패턴**

```javascript
// ✅ 전략 패턴으로 개선
const DASHBOARD_STRATEGIES = {
  "new-with-analytics": {
    condition: () => FEATURES.NEW_DASHBOARD && FEATURES.DASHBOARD_ANALYTICS,
    component: NewDashboardWithAnalytics,
  },
  new: {
    condition: () => FEATURES.NEW_DASHBOARD,
    component: NewDashboard,
  },
  old: {
    condition: () => true, // 기본값
    component: OldDashboard,
  },
};

function Dashboard() {
  const strategy = Object.values(DASHBOARD_STRATEGIES).find((s) =>
    s.condition()
  );

  const Component = strategy.component;
  return <Component />;
}
```

**2. 문제점: 문서 부족**

```javascript
// ❌ 나쁜 예
const FEATURES = {
  NEW_DASHBOARD: true,
  FEATURE_X: false,
  BETA_MODE: true,
};

// ✅ 좋은 예: 메타데이터 추가
const FLAG_LIFECYCLE = {
  NEW_DASHBOARD: {
    status: "active",
    createdAt: "2024-01-15",
    targetRemoval: "2024-04-15", // ⚠️ 삭제 예정일
    rolloutPercentage: 100,
    owner: "team-a@company.com",
    jira: "PROJ-1234",
    description: "새로운 대시보드 UI",
    affectedComponents: ["Dashboard", "Analytics"],
  },
};
```

**3. 해결: Flag 생명주기 관리**

```javascript
// 자동 경고 시스템
function checkExpiredFlags() {
  Object.entries(FLAG_LIFECYCLE).forEach(([flag, config]) => {
    const daysUntilRemoval = getDaysDiff(config.targetRemoval, new Date());

    if (daysUntilRemoval < 0) {
      console.error(`🚨 Flag ${flag} should have been removed!`);
      notifyTeam(config.owner);
    } else if (daysUntilRemoval < 7) {
      console.warn(`⚠️ Flag ${flag} expires in ${daysUntilRemoval} days`);
    }
  });
}

// CI/CD에서 실행
// npm run check-flags
```

**4. ESLint로 오래된 Flag 탐지:**

```javascript
// .eslintrc.js
module.exports = {
  rules: {
    "no-outdated-feature-flags": [
      "error",
      {
        flags: {
          OLD_DASHBOARD_REFRESH: {
            deprecated: true,
            deadline: "2024-02-01",
            replacement: "Use NEW_DASHBOARD instead",
          },
        },
      },
    ],
  },
};

// 코드에서 사용하면 에러!
if (FEATURES.OLD_DASHBOARD_REFRESH) {
  // ❌ ESLint Error!
  // ...
}
```

**5. Flag 제거 체크리스트:**

```markdown
# Feature Flag 제거 프로세스

## 1. 사전 확인 (출시 후 2주)

- [ ] 에러율 정상 (< 0.1%)
- [ ] 성능 메트릭 정상
- [ ] 사용자 피드백 긍정적
- [ ] 롤백 필요 없음

## 2. 코드 정리

- [ ] 조건문 제거
- [ ] 구 코드 삭제
- [ ] 테스트 업데이트
- [ ] Flag 정의 제거

## 3. 배포

- [ ] Staging 배포 & 테스트
- [ ] Production 배포
- [ ] 모니터링 (24시간)

## 4. 문서화

- [ ] CHANGELOG 업데이트
- [ ] Flag 삭제 기록
- [ ] 팀 공지
```

**6. 점진적 롤아웃:**

```javascript
// LaunchDarkly / Unleash 같은 도구 사용
function useFeatureFlag(flagName) {
  const userId = useUserId();

  return useLDClient().variation(flagName, false, {
    userId,
    // 사용자 속성 기반 타겟팅
    email: user.email,
    country: user.country,
  });
}

// 사용
function Dashboard() {
  const showNewDashboard = useFeatureFlag("new-dashboard");

  if (showNewDashboard) {
    return <NewDashboard />;
  }
  return <OldDashboard />;
}

// 관리자 대시보드에서:
// - 0% → 10% → 50% → 100% 점진적 롤아웃
// - 국가별, 사용자별 타겟팅
// - 자동 만료 설정
```

---

## 코드 리뷰 자동화

### Q. PR 단위로 코드 품질을 자동 검사하는 방법은?

**답변:**

**1. Husky + lint-staged:**

```json
// package.json
{
  "scripts": {
    "prepare": "husky install"
  },
  "lint-staged": {
    "*.{js,jsx,ts,tsx}": ["eslint --fix", "prettier --write"]
  }
}
```

```bash
# .husky/pre-commit
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx lint-staged
```

**2. PR 라인 수 제한:**

```yaml
# .github/workflows/pr-check.yml
name: PR Check

on:
  pull_request:
    types: [opened, synchronize]

jobs:
  check-size:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Check PR size
        run: |
          ADDITIONS=$(gh pr view ${{ github.event.pull_request.number }} --json additions -q .additions)
          DELETIONS=$(gh pr view ${{ github.event.pull_request.number }} --json deletions -q .deletions)
          TOTAL=$((ADDITIONS + DELETIONS))

          if [ $TOTAL -gt 500 ]; then
            echo "❌ PR too large: $TOTAL lines"
            echo "Please split into smaller PRs"
            exit 1
          fi
```

**3. 자동 코드 리뷰 (Danger.js):**

```javascript
// dangerfile.js
import { danger, warn, fail } from "danger";

// PR 크기 체크
const additions = danger.github.pr.additions;
const deletions = danger.github.pr.deletions;

if (additions + deletions > 500) {
  warn("📦 PR이 너무 큽니다. 작은 단위로 나눠주세요.");
}

// 테스트 파일 체크
const hasChanges = danger.git.modified_files.length > 0;
const hasTests = danger.git.modified_files.some((f) => f.includes(".test."));

if (hasChanges && !hasTests) {
  warn("🧪 테스트 파일이 포함되지 않았습니다.");
}

// 설명 체크
if (danger.github.pr.body.length < 10) {
  fail("📝 PR 설명을 작성해주세요.");
}
```

**4. 자동 포맷팅 체크:**

```yaml
# .github/workflows/format-check.yml
name: Format Check

on: [pull_request]

jobs:
  format:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2

      - name: Check formatting
        run: |
          npm run format:check

      - name: Auto-fix and commit
        if: failure()
        run: |
          npm run format
          git config user.name github-actions
          git config user.email github-actions@github.com
          git add .
          git commit -m "chore: auto-format code"
          git push
```

**5. 커밋 메시지 검증:**

```bash
# .husky/commit-msg
#!/bin/sh
. "$(dirname "$0")/_/husky.sh"

npx --no -- commitlint --edit "$1"
```

```javascript
// commitlint.config.js
module.exports = {
  extends: ["@commitlint/config-conventional"],
  rules: {
    "type-enum": [
      2,
      "always",
      [
        "feat", // 새 기능
        "fix", // 버그 수정
        "docs", // 문서
        "style", // 포맷팅
        "refactor", // 리팩토링
        "test", // 테스트
        "chore", // 기타
      ],
    ],
  },
};

// 올바른 커밋 메시지:
// feat: add new dashboard
// fix: resolve login issue
// docs: update README

// 잘못된 커밋 메시지:
// updated code
// fix bug
```

---

[← 목차로 돌아가기](../README.md)
