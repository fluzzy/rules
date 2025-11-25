# AGENTS – E2E Testing with Playwright

이 문서는 **Next.js(App Router)** 기반 웹 애플리케이션에서  
**Playwright Test + Playwright Test Agents (planner / generator / healer)** 를 사용할 때  
AI 에이전트가 따라야 할 **E2E 테스트 작성 규칙**을 정의합니다.

핵심 목표:

1. 사람이 작성한 **TODO 시나리오 문서**(자연어)를 기반으로 안정적인 E2E 테스트를 만듭니다.
2. **Playwright Test Agents** 의 표준 흐름
   - planner → generator → healer 를 활용해  
     **테스트 플랜 → 테스트 코드 → 실패 시 치유** 루프를 최대한 활용합니다.
3. 테스트는 항상 **사용자 관점(User journey)** 을 우선합니다.  
   구현 디테일(내부 함수, DB 구조 등)에 불필요하게 의존하지 않습니다.

---

## 1. 기본 환경 & 디렉터리 규칙

### 1.1 Tech Stack

- Framework: **Next.js (App Router)**
- Test Runner: `@playwright/test`
- Playwright Test Agents: **planner / generator / healer**
- 언어: TypeScript
- 브라우저: Chromium, Firefox, WebKit 3개 동시 실행
- MCP: `@playwright/mcp` (IDE/에이전트에서 브라우저 제어용)

### 1.2 디렉터리 컨벤션

> 실제 레포에 이미 Playwright Agents 기본 구조가 있다면, 그 구조를 우선합니다.  
> (예: `npx playwright init-agents --loop=claude` 로 생성된 구조)

기본 구조(예시):

```txt
repo/
  .github/
    agents/                  # init-agents 로 생성된 agent 정의들 (planner/generator/healer)
  specs/                     # planner가 생성하는 공식 test plan
    basic-operations.md
  e2e/
    specs/                   # 사람이 작성하는 TODO 시나리오 문서
      posts/
        post-detail.todo.md
        post-like.todo.md
      home/
        home.todo.md
    tests/                   # 실제 Playwright 테스트 코드
      posts/
        post-like-guest.spec.ts
        post-like-member.spec.ts
      home/
        home-guest.spec.ts
    fixtures/
      auth.fixtures.ts
    page-objects/
      post-list.page.ts
      post-detail.page.ts
  tests/                     # (Playwright 기본 tests 디렉토리가 이미 있다면 공존)
    seed.spec.ts             # seed test (planner/generator에서 사용)
  playwright.config.ts
```

- **Playwright Agents 표준**:

  - planner: `specs/*.md` 에 Markdown test plan 생성
  - generator: `tests/*.spec.ts` (또는 구성에 따라 하위 디렉터리) 생성

- **이 프로젝트 추가 규칙**:

  - 사람이 직접 작성하는 TODO 문서는 `e2e/specs/{feature}/**.todo.md`
  - 최종 E2E 테스트 코드는 `e2e/tests/{feature}/**.spec.ts`
  - **specs와 tests는 동일한 폴더 구조**를 유지합니다 (예: `e2e/specs/posts/` ↔ `e2e/tests/posts/`)

---

## 2. TODO 문서 기반 워크플로우

### 2.1 사람이 작성하는 TODO 문서 포맷

TODO 문서는 **페이지 단위** 또는 **기능 단위**로 작성합니다.

예시: `e2e/specs/post-detail.todo.md`

```md
# /posts/[id] 상세 페이지 - 좋아요

[시나리오 1] 비회원이 첫 번째 게시물에 좋아요를 누르는 경우

1. 비회원 상태로 /posts 페이지에 접속한다.
2. 리스트에서 첫 번째 게시물 카드를 클릭해 상세 페이지로 이동한다.
3. 좋아요 버튼(하트 아이콘)을 클릭한다.
4. 로그인 유도 모달이 노출되고, "로그인이 필요한 기능입니다." 문구가 보인다.

[시나리오 2] 회원이 첫 번째 게시물에 좋아요를 누르는 경우

1. 테스트용 계정으로 로그인한 상태에서 /posts 페이지에 접속한다.
2. 리스트에서 첫 번째 게시물 카드를 클릭해 상세 페이지로 이동한다.
3. 좋아요 버튼을 클릭하면, 좋아요 카운트가 +1 증가한다.
4. 새로고침 후에도 좋아요 상태와 카운트가 유지된다.
```

규칙:

- `[시나리오 N]` 단위로 **하나의 user journey** 를 설명합니다.
- 각 단계는 "사용자가 하는 행동"과 "보여야 하는 결과"를 자연스럽게 섞어도 됩니다.
- "첫 번째 게시물" 처럼 데이터 의존성이 있는 경우, 항상 존재하는 고정 데이터 기준으로 작성하는 것을 권장합니다.

### 2.2 Agents 와 TODO 문서의 관계

에이전트는 TODO 문서를 다음과 같이 사용합니다:

- **planner**

  - `e2e/specs/*.todo.md` 와 실제 앱을 함께 참고하여,
    Playwright 표준 포맷의 Markdown test plan (`specs/*.md`) 을 생성/보완합니다.
  - 사람이 쓴 TODO 내용을 **덮어쓰지 말고**,
    시나리오를 더 구조화(섹션/Expected results 분리 등)하는 방향으로 정리합니다.

- **generator**

  - `specs/*.md` (planner output)를 기반으로,
    `e2e/tests/**.spec.ts` 파일을 만듭니다.

- **healer**

  - 실패한 `e2e/tests/**.spec.ts` 를 재실행·분석하고,
    locator/타이밍/데이터 이슈를 수정하여 테스트를 회복시킵니다.

---

## 3. Planner 에이전트 규칙

### 3.1 입력

- 자연어 요청: 예) `"비회원/회원 좋아요 플로우에 대한 플랜을 만들어줘"`
- 필수: **seed test** 파일 (예: `tests/seed.spec.ts`)
- 선택:

  - 관련 TODO 문서(`e2e/specs/post-detail.todo.md`)
  - 관련 PRD / 디자인 문서

### 3.2 동작 규칙

1. **seed test 실행**

   - Playwright가 제공하는 seed test 구조를 그대로 따릅니다.
   - 이 프로젝트에서 seed test는 보통 "기본 Next.js 앱 + 공통 fixture 세팅" 정도만 정의합니다.

   ```ts
   // tests/seed.spec.ts
   import { test, expect } from "./fixtures";

   test("seed", async ({ page }) => {
     // 공통 fixture / 전역 셋업이 정상 동작하는지만 확인
   });
   ```

2. **앱 탐색 + TODO 문서 병합**

   - seed test를 기준으로 앱을 실행하고, **브라우저를 실제로 탐색**해 DOM 구조를 이해합니다.
   - TODO 문서의 시나리오(예: "비회원이 좋아요를 누르면 로그인 모달 노출")를 기준으로,

     - 단계(steps)
     - 기대 결과(expected results)
       를 명시적으로 나눈 test plan 을 만듭니다.

3. **Markdown test plan 작성**

- 출력 위치: `specs/{feature-name}.md`
- 구조 예시 (공식 TodoMVC 예제를 참고한 형태):

```md
# /posts 상세 & 상세 페이지 - 좋아요 관련 Test Plan

## 1. Guest Like Flow

**Seed:** `tests/seed.spec.ts`  
**Source TODO:** `e2e/specs/post-detail.todo.md` 의 [시나리오 1]

### Steps

1. 비회원 상태로 `/posts` 페이지에 접근한다.
2. 첫 번째 게시물 카드를 클릭해 상세 페이지로 이동한다.
3. 좋아요 버튼(하트 아이콘)을 클릭한다.

### Expected Results

- 로그인 유도 모달이 노출된다.
- 모달 내에 `"로그인이 필요한 기능입니다."` 텍스트가 보인다.
- 좋아요 카운트 / 상태는 변경되지 않는다.
```

4. **주의사항**

- planner는 **테스트 코드를 직접 생성하지 않습니다.**
- 앱 구조를 모른다고 가정하고 locator 를 박지 말고,
  "무엇을 검증해야 하는지"만 Markdown 에 표현합니다.
- TODO 문서에 없는 새로운 시나리오를 임의로 추가할 수 있으나,
  그 경우에는 섹션에 `Origin: planner-suggested` 같은 메모를 남깁니다.

---

## 4. Generator 에이전트 규칙

### 4.1 입력

- `specs/*.md` (planner가 만든 Markdown plan)
- seed test (`tests/seed.spec.ts`)
- 이 레포의 `playwright.config.ts`

### 4.2 출력

- Playwright TypeScript 테스트 코드
- 위치: `e2e/tests/**.spec.ts`

예: `specs/basic-operations.md` → `e2e/tests/posts/post-like-guest.spec.ts`

### 4.3 코드 작성 규칙

#### 4.3.1 기본 구조

```ts
// spec: specs/post-like.md
// seed: tests/seed.spec.ts

import { test, expect } from "../../fixtures/auth.fixtures";

test.describe("/posts/[id] - 좋아요", () => {
  test("비회원이 좋아요를 누르면 로그인 모달이 뜬다", async ({ page }) => {
    // ...
  });

  test("회원이 좋아요를 누르면 카운트가 증가한다", async ({ memberPage }) => {
    // ...
  });
});
```

- 파일명은 **기능 + 시나리오** 조합:

  - `/posts/[id]` 좋아요 → `post-like-guest.spec.ts`, `post-like-member.spec.ts` 등

- 테스트 이름은 TODO 문서의 시나리오 설명을 거의 그대로 사용하되, 약간 더 명확하게 다듬습니다.

#### 4.3.2 Next.js & 네비게이션

- 라우트 주소는 **절대 하드코딩하지 않습니다**. 반드시 constants 파일에서 가져옵니다.

```ts
// constants/routes.ts 또는 유사한 파일에서
import { ROUTES } from "@/constants/routes";

// 라우트가 constants에 정의되어 있는지 확인
if (!ROUTES.POSTS) {
  throw new Error(
    "ROUTES.POSTS가 constants에 정의되어 있지 않습니다. constants 파일을 확인하세요."
  );
}

await page.goto(ROUTES.POSTS);
```

- 라우트 주소가 constants에 없으면:

  - 테스트를 실행하지 않고 사용자에게 알립니다.
  - 예: `"ROUTES.POSTS가 constants에 정의되어 있지 않습니다. constants 파일을 확인하세요."` 같은 에러 메시지

- `playwright.config.ts` 의 `use.baseURL` 을 최대한 사용합니다.

- 페이지 전환 후:

  - `await page.waitForLoadState('networkidle')` **또는**
  - 특정 주요 요소 visibility assertion (`await expect(...).toBeVisible()`)
    로 안정화 후 나머지 검증을 진행합니다.

- `waitForTimeout` 사용은 **금지** (flaky 원인).

#### 4.3.3 스크린샷 / 비디오 / Trace

- `playwright.config.ts`에서 실패 시 자동 캡처 설정:

```ts
// playwright.config.ts
export default defineConfig({
  use: {
    screenshot: "only-on-failure", // 실패 시 스크린샷 자동 저장
    video: "retain-on-failure",    // 실패 시 비디오 자동 저장
    trace: "retain-on-failure",    // 실패 시 trace 자동 저장
  },
});
```

- 디버깅 시 Trace Viewer 활용:

```bash
npx playwright test --trace on
npx playwright show-report
```

#### 4.3.4 Locator 전략

> 공식 가이드와 MCP/Agents 글의 best practice 를 따릅니다.

우선순위:

1. `page.getByTestId(...)` — 최우선. 가장 안정적이고 명시적.
2. `page.getByRole(...)` — 접근성 검증 겸용
3. `page.getByLabel(...)` — 폼 컨트롤용
4. `page.getByText(...)` — 비상호작용 요소용
5. CSS/XPath locator (`page.locator`) — 최후 수단

예:

```ts
const firstPostCard = page.getByTestId("post-card").first();
const likeButton = page.getByRole("button", { name: /좋아요/i });
const loginModal = page.getByRole("dialog", {
  name: /로그인이 필요한 기능입니다./i,
});
```

리스트 + 필터:

```ts
const postItem = page
  .getByRole("article")
  .filter({ hasText: "첫 번째 게시물 제목" });
```

#### 4.3.5 Assertions

- web-first assertion(`await expect(...)`) 만 사용합니다.

```ts
await expect(page.getByText("로그인이 필요한 기능입니다.")).toBeVisible();
await expect(likeButton).toBeEnabled();
```

- `.isVisible()` 호출 후 동기 `expect` 쓰는 패턴은 금지.
- 여러 검증이 필요한 경우:

  - 주요 결과는 hard assertion
  - 덜 중요한 UI 메시지 등은 `expect.soft` 사용 가능.

#### 4.3.6 로그인 / 권한

- 이미 존재하는 fixture / helper 를 **최우선 재사용**합니다. (예: `auth.fixtures.ts`)
- 없다면 새로 작성하되, 재사용을 위한 이름으로 만듭니다.

```ts
// e2e/fixtures/auth.fixtures.ts
import { test as base } from "@playwright/test";
import { ROUTES } from "@/constants/routes";

export const test = base.extend<{
  memberPage: (typeof base)["page"];
}>({
  memberPage: async ({ page }, use) => {
    if (!ROUTES.LOGIN) {
      throw new Error(
        "ROUTES.LOGIN이 constants에 정의되어 있지 않습니다. constants 파일을 확인하세요."
      );
    }
    await page.goto(ROUTES.LOGIN);
    await page.getByLabel("이메일").fill(process.env.E2E_USER_EMAIL!);
    await page.getByLabel("비밀번호").fill(process.env.E2E_USER_PASSWORD!);
    await page.getByRole("button", { name: /로그인/ }).click();
    if (!ROUTES.HOME) {
      throw new Error(
        "ROUTES.HOME이 constants에 정의되어 있지 않습니다. constants 파일을 확인하세요."
      );
    }
    await page.waitForURL(ROUTES.HOME);
    await use(page);
  },
});
```

---

## 5. Healer 에이전트 규칙

### 5.1 역할

- 실패한 Playwright 테스트를 디버깅하고, 가능한 경우 자동으로 수정합니다.
- 주로 아래를 수정 대상로 삼습니다:

  - 바뀐 locator (레이블 이름 변경, 버튼 텍스트 변경 등)
  - 타이밍 이슈 (불필요한 `waitForTimeout`, 잘못된 load 조건 등)
  - 경미한 데이터 변경 (텍스트 문구 변경, 카운트 값 정합성 등)

### 5.2 동작 순서

1. 실패한 테스트 식별

   - `mcp__playwright-test__test_list` / `test_run` 등을 통해 실패 케이스를 찾습니다.

2. 재실행 & UI 조사

   - 실패 지점을 재실행하고, `browser_snapshot`, `browser_console_messages`, `browser_network_requests` 등을 활용해 원인을 찾습니다.

3. 패치 제안 & 적용

   - locator 업데이트, 불필요한 wait 제거, assertion 문구 수정 등,
     **테스트 코드만 수정**하는 패치를 제안하고 적용합니다.

4. 재실행 후 결과 보고

   - 수정 후 다시 테스트를 실행해서 통과 여부를 확인합니다.
   - 기능 자체가 깨진 경우, 테스트를 무리하게 "통과" 시키지 말고
     주석/메모와 함께 실패 상태로 남기거나 `test.skip` 으로 전환합니다.

### 5.3 금지사항

- 앱의 실제 구현 코드(Next.js 페이지/컴포넌트/백엔드 코드)를 수정하지 않습니다.
- 의미를 흐리는 "느슨한 assertion"(ex. `toBeTruthy`) 로 바꾸어 억지로 통과시키지 않습니다.
- TODO 문서 / PRD 의 요구사항을 무시하고, 임의의 behavior로 정의를 바꾸지 않습니다.

---

## 6. 공통 테스트 철학

> 여러 자료에서 공통으로 강조하는 E2E / Playwright best practice 를 이 프로젝트 기준으로 요약한 섹션.

1. **사용자 눈에 보이는 행동만 검증합니다.**

   - DOM 깊은 구조, 내부 state, API 형태 등 구현 세부사항에 과도하게 의존하지 않습니다.

2. **테스트는 서로 독립적이어야 합니다 (Test Isolation).**

   - 각 테스트는 필요한 상태(로그인, seed 데이터)를 스스로 준비합니다.
   - 테스트 격리는 **재현성을 높이고, 디버깅을 쉽게 하며, 연쇄 실패를 방지**합니다.
   - `test.beforeEach()` 훅으로 각 테스트 전 반복 작업을 자동화하거나,
     setup project를 활용해 로그인 상태를 재사용합니다.

3. **약간의 중복은 허용하되, 복잡해지면 helper/fixture 로 추상화합니다.**
4. **테스트 명과 파일 명을 보면 "무엇을 검증하는지" 바로 떠오르게 만듭니다.**
5. **플레이크 방지를 최우선으로 고려합니다.**

   - 타이밍 이슈는 web-first assertion/locator 전략으로 해결합니다.

6. **에이전트는 테스트 코드만 자동화합니다.**

   - 비즈니스 로직, 도메인 규칙은 사람이 정의한 TODO 문서 / PRD 를 항상 우선합니다.

---

## 7. 이 문서를 읽는 에이전트에게

1. 사람이 `e2e/specs/*.todo.md` 에 적어둔 시나리오를 **진실의 근원(source of truth)** 으로 삼습니다.
2. Playwright Test Agents 의 흐름(planner → generator → healer)을 따르되,
   각 단계에서 이 문서에 정의된 규칙을 항상 참고합니다.
3. **앱 코드 대신 테스트 코드에만 손을 댑니다.**
4. 모호한 부분이 있다면:

   - (대화 가능한 환경이면) 사용자에게 질문합니다.
   - 그렇지 않다면 테스트 파일 상단에 `// TODO:` 주석을 남기고, 가장 보수적인 가정으로 작성합니다.

이 가이드는, TODO 문서만 써주면
Planner / Generator / Healer / MCP 를 통해 **끝까지 끌고 갈 수 있는** 테스트 환경을 만들기 위한 기준입니다.
