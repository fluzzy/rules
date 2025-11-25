# ACCESSIBILITY – 접근성 가이드

이 프로젝트에서 접근성(a11y)의 목표는 **"역할(Role) · 이름(Label) · 상태(State)"가 항상 명확한 UI**를 만드는 것입니다.

아래 기준은 사람과 AI 모두가 컴포넌트를 작성·리팩토링할 때 따라야 하는 공통 규칙입니다.

---

## 1. 접근성 기본 원칙

### 1-1. 역할(Role)

- 가능한 한 **시맨틱 태그**를 먼저 사용합니다.

  - `<button>`, `<a>`, `<input>`, `<form>`, `<fieldset>`, `<legend>`, `<dialog>`, `<details>`, `<summary>` 등.

- 시맨틱 태그로 표현이 안 되면 `role`로 명시합니다.

  - 예: `role="tablist" | "tab" | "tabpanel" | "dialog" | "checkbox" | "radio" | "switch" | "button" | "region"` 등.

### 1-2. 레이블(Label)

- 모든 **인터랙티브 요소**(버튼, 링크, 인풋, 선택 컴포넌트 등)는 항상 "이게 뭔지"를 설명하는 이름을 가져야 합니다.
- 우선순위: `aria-labelledby` → `aria-label` → `<label>`/요소 텍스트.
- 시각 텍스트가 충분히 의미를 설명하면 ARIA는 생략해도 됩니다.
- **아이콘만 있는 버튼 / 스위치 / 토글**은 `aria-label` 또는 `aria-labelledby`로 필수 레이블을 부여합니다.

### 1-3. 상태(State)

- 상태는 **HTML 기본 속성**과 **ARIA 상태 속성**을 혼용하되, 항상 동기화합니다.
- 자주 쓰는 속성

  - `checked` / `aria-checked`
  - `selected` / `aria-selected`
  - `open` / `aria-expanded`
  - `disabled` / `aria-disabled`
  - `aria-current` (현재 페이지/날짜 등)

### 1-4. 라이브 리전(Live Region)

동적으로 변경되는 콘텐츠를 스크린 리더에 알려주기 위해 라이브 리전을 사용합니다:

- **`aria-live`**: 콘텐츠 변경 시 스크린 리더가 읽어줌

  - `aria-live="polite"`: 현재 읽고 있는 내용이 끝난 후 알림 (일반적인 업데이트)
  - `aria-live="assertive"`: 즉시 알림 (긴급한 오류, 중요 알림)

- **`role="alert"`**: `aria-live="assertive"` + `aria-atomic="true"`와 동일. 오류 메시지, 긴급 알림에 사용.

- **`role="status"`**: `aria-live="polite"` + `aria-atomic="true"`와 동일. 상태 업데이트, 성공 메시지에 사용.

- **`aria-busy`**: 콘텐츠가 로딩 중임을 알림. `true`일 때 스크린 리더가 변경 알림을 보류.

- **`aria-atomic`**: `true`이면 변경된 부분만이 아닌 전체 영역을 읽음.

예시:

```tsx
// 폼 제출 결과 알림
<div role="status" aria-live="polite">
  {submitSuccess && "저장되었습니다."}
</div>

// 오류 메시지
<div role="alert">
  {errorMessage}
</div>

// 로딩 상태
<div aria-busy={isLoading} aria-live="polite">
  {isLoading ? "로딩 중..." : content}
</div>
```

### 1-5. 키보드 탐색

- Tab/Shift+Tab, Enter, Space, 방향키로 UI를 조작할 수 있게 만듭니다.
- **커스텀 인터랙티브 요소**를 만들면:

  - `tabIndex={0}` 부여
  - `role` 설정
  - `onKeyDown`에서 Enter / Space 처리

### 1-6. 시각 정보에만 의존하지 않기

- 색/아이콘/이미지로만 정보가 전달되면 안 됩니다.
- 항상 **텍스트 또는 ARIA 레이블**로 보완합니다.
- 의미 없는 장식 이미지는 `alt=""`로 처리해서 낭독을 막습니다.

---

## 2. UI 컴포넌트별 규칙

### 2-1. 탭(Tab)

**목표:** 스크린 리더가 "탭 목록 / 현재 탭 / 해당 패널" 구조를 정확히 이해하게 만들 것.

- 구조

  - 탭 리스트 컨테이너: `role="tablist"` + `aria-label` 또는 `aria-labelledby`
  - 탭 버튼: `role="tab"`

    - 현재 탭: `aria-selected="true"`
    - 나머지 탭: `aria-selected="false"`

  - 탭 패널: `role="tabpanel"`

    - `id` 필수
    - 탭 버튼에서 `aria-controls="<tabpanel-id>"`
    - 탭 패널에서 `aria-labelledby="<tab-id>"`

- 표시/숨김

  - **활성 패널만** 실제로 보이게 하고, 나머지는 `hidden` 또는 `aria-hidden`/CSS로 숨깁니다.
  - `aria-selected` 값과 **패널의 표시 여부를 항상 동기화**합니다.

- 키보드

  - 방향키(좌/우)로 탭 이동 가능하게 구현하는 것을 기본 방향으로 합니다.
  - Tab/Shift+Tab은 탭 그룹 전체를 빠져나가거나 진입하는 용도로.

### 2-2. 아코디언(Accordion)

**목표:** 어떤 항목이 펼쳐져 있는지 명확히 알 수 있도록.

- 가능하면 **HTML 표준 요소 사용 우선**

  - `<details>` + `<summary>`

    - `open` 속성으로 상태 표현
    - `onToggle`로 상태 관리

- 커스텀 구현 시 규칙

  - 헤더(토글 버튼)

    - `<button>` 사용
    - `aria-expanded={isOpen}`
    - `aria-controls="<panel-id>"`

  - 패널

    - `id="<panel-id>"`
    - `role="region"`
    - `aria-labelledby="<button-id>"`
    - `hidden={!isOpen}`

- 상태 동기화

  - `aria-expanded="true"` ↔ 패널 표시
  - `aria-expanded="false"` ↔ 패널 숨김
    → 이 둘이 다르면 안 됩니다.

### 2-3. 모달(Modal/Dialog)

**목표:** 모달이 열려 있으면 **사용자 포커스와 스크린 리더 관심이 모달 안에만 머무르게** 할 것.

- `<dialog>` 사용 가능한 경우 (우선)

  - 트리거 버튼: `aria-haspopup="dialog"`
  - `dialog.showModal()`로 열기
  - `dialog.close()`로 닫기
  - `aria-labelledby` 또는 `aria-label`로 제목 제공
  - 브라우저가 자동으로:

    - 포커스 모달 내부로 이동
    - 포커스 트랩
    - ESC로 닫기
    - 닫힐 때 원래 포커스로 복귀

- `<dialog>` 못 쓸 때 (커스텀 모달)

  - 모달 컨테이너

    - `role="dialog"`
    - `aria-modal="true"`
    - `aria-labelledby="<title-id>"` 또는 `aria-label="..."`

  - 포커스 관리

    - 모달 열릴 때: 내부 첫 포커스 가능한 요소로 이동
    - 모달 닫힐 때: 열었던 트리거로 포커스 복귀

  - ESC 핸들링

    - `keydown` 이벤트에서 `e.key === "Escape"`이면 닫기

  - 배경 콘텐츠

    - 모달 열려 있을 때 메인 영역에 `inert` 부여
    - 닫히면 `inert` 제거

  - 포커스 트랩

    - Tab/Shift+Tab으로 포커스가 모달 안에서만 순환되게 합니다.

### 2-4. 라디오 버튼(Radio)

**목표:** "하나의 질문에 여러 답변 중 1개 선택" 구조를 명확히.

- 기본 구조 (권장)

  - 컨테이너: `<fieldset>`
  - 그룹 제목: `<legend>`
  - 각 옵션:

    - `<input type="radio" name="...">`
    - `<label htmlFor="...">옵션명</label>`

- 대안 구조

  - `<fieldset>`을 쓸 수 없으면:

    - 컨테이너: `role="radiogroup"` + `aria-labelledby="<heading-id>"`
    - 내부는 여전히 `<input type="radio" name=".."> + <label>` 사용

- 필수 규칙

  - **같은 그룹은 name 동일** (브라우저가 자동으로 "1개만 선택" 보장).
  - 각 옵션은 **항상 레이블과 연결** (`id` + `htmlFor`).

- 커스텀 라디오 (input 미사용)

  - 각 옵션:

    - `role="radio"`
    - `aria-checked={true|false}`
    - `tabIndex={0}` (최소 1개 이상)
    - Space 키로 선택 토글 (`onKeyDown` 처리)

  - 그룹:

    - `role="radiogroup"` + `aria-labelledby`

### 2-5. 체크박스(Checkbox)

**목표:** 여러 개를 동시에 선택할 수 있는 옵션 그룹을 명확히.

- 그룹이 있는 경우

  - 컨테이너: `<fieldset>`
  - 제목: `<legend>`
  - 각 항목:

    - `<input type="checkbox" id="..." />`
    - `<label htmlFor="...">텍스트</label>`

- 커스텀 체크박스

  - `role="checkbox"`
  - `aria-checked={true|false}`
  - `tabIndex={0}`
  - Space 키로 체크/언체크 (`onKeyDown`)

### 2-6. 스위치(Switch)

**목표:** 켜짐/꺼짐 상태를 명확히 전달.

- HTML 기반 스위치

  - `<input type="checkbox" role="switch" checked={isOn} />`
  - 라벨:

    - `<label>`로 감싸거나
    - `aria-label` / `aria-labelledby`로 제공

- 커스텀 스위치

  - `role="switch"`
  - `aria-checked={isOn}`
  - `tabIndex={0}`
  - Space 키로 상태 변경
  - 레이블 필수

    - "무엇을 켜고 끄는지"를 설명 (예: `"다크 모드"`, `"알림 설정"`)

---

## 3. 실전 패턴 규칙

### 3-1. 버튼 안에 버튼/링크 넣지 않기

- 금지

  - `<button>` 안에 `<button>`
  - `<a>` 안에 `<button>`
  - `<button>` 안에 `<a>`

- 해결

  - **폴리모픽 버튼** 패턴 사용

    - `<Button as="a" href="/...">텍스트</Button>`

  - "카드 전체 클릭 + 내부 개별 버튼" 패턴

    - 바깥은 `div` + 가상의 투명 버튼으로 전체 영역 클릭 처리
    - 개별 버튼은 별도의 실제 `<button>`으로 두기
    - 포커스 스타일은 `.wrapper:focus-within { … }`로 처리

### 3-2. 테이블 행을 직접 클릭 가능하게 만들지 않기

- 금지

  - `<tr onClick={...}>`

- 해결

  - 행 안에 **실제 `<a>` 링크**를 넣고, CSS로 영역 확장

    - 링크에 `position: relative` + `::after { position: absolute; inset: 0; content: "" }`

  - 장점

    - 키보드 포커스, context menu(새 탭 열기, 링크 복사) 등 기본 기능 유지
    - 스크린 리더에 "링크"로 인식

### 3-3. 인터랙티브 요소에 이름 붙이기

- 기본 룰

  - 모든 입력/버튼/선택 요소는 **항상 이름을 가집니다**.

- 입력 필드

  - 우선 `<label for="id">` + `<input id="id">`
  - 시각 레이블을 둘 수 없으면 `aria-label`
  - 주변 텍스트를 재사용할 때 `aria-labelledby`

- 버튼/아이콘

  - 텍스트 버튼이면 내부 텍스트가 레이블
  - 아이콘 버튼이면 `aria-label` 필수

- Select

  - `<label>` + `<select id="...">` 구조 기본

- placeholder

  - **레이블 대체용 X**, 힌트 용도로만 사용
  - 항상 레이블과 같이 사용

### 3-4. 같은 이름의 버튼이 여러 개 있을 때

- 예: "선택" 버튼이 리스트마다 반복될 때
- 룰

  - 각 항목에 "맥락 텍스트"를 만들고, 이를 버튼과 연결
  - 패턴 예:

    - 리스트 아이템: `<li aria-labelledby="title-id">`
    - 제목 텍스트: `<div id="title-id">종이를 사용할 경우</div>`
    - 버튼: `id="paper-button" aria-labelledby="title-id paper-button"`

  - 또는 버튼에 `aria-label="종이를 사용할 경우 선택"` 형식으로 구체화

### 3-5. 버튼의 역할과 동작을 일치시키기

- 금지

  - 단순 `<div onClick>`에 `cursor: pointer`만 주고 버튼처럼 쓰는 것

- 기본

  - 가능하면 무조건 `<button>` 사용.

- 꼭 `<div>` 등으로 해야 한다면:

  - `role="button"`
  - `tabIndex={0}`
  - `onKeyDown`에서 Enter / Space 시 클릭 처리

### 3-6. 입력 요소는 항상 `<form>`으로 감싸기

- 기본

  - 관련 입력 그룹은 `<form>` 안에 넣습니다.
  - submit 버튼은 `<button type="submit">`.
  - 단순 클릭용 버튼은 **반드시** `type="button"` 명시.

- 동작

  - JS에서 처리할 때:

    - `onSubmit`에서 `event.preventDefault()` 호출 후 로직 실행.

  - 스크린 리더:

    - `aria-label` 또는 `<form aria-labelledby="...">`로 폼 이름 제공.

- 폼 밖에 있는 submit 버튼

  - 버튼에 `form="form-id"` 속성으로 연결.

- 보조 버튼(입력값 삭제 등)

  - 꼭 포커스가 필요하지 않은 보조 버튼은 `tabIndex="-1"`로 포커스 순서에서 제외하고, `aria-label`로 역할 설명.

### 3-7. 이미지 / 아이콘 대체 텍스트(alt)

- 원칙

  - "이 이미지가 **사용자에게 전달하려는 정보** 또는 **기능**이 무엇인가?"를 기준으로 alt를 씁니다.

- 반드시 alt가 필요한 경우

  - 이미지가 **링크/버튼의 유일한 내용**일 때 → alt로 기능 설명
  - 상품/그래프/차트 등 **핵심 정보**를 담은 경우 → 내용 요약

- alt를 비워야 하는 경우 (`alt=""`)

  - 장식용 이미지 (단순 구분선, 배경 등)
  - 같은 내용을 주변 텍스트/캡션이 이미 설명하는 경우
  - 텍스트와 함께 있는 아이콘 (삭제 아이콘 + "삭제" 텍스트 등)

- 작성 방식

  - 불필요한 단어(`아이콘`, `버튼`) 제거

    - 스크린 리더가 어차피 "버튼"을 붙여 읽습니다.

  - 맥락 기반으로 작성

    - `"이전 페이지로 이동"`, `"다음 페이지로 이동"` 등 기능/의도 기준

---

## 4. eslint-plugin-jsx-a11y 설정

JSX/React 코드에서 접근성 위반을 자동으로 잡기 위해 필수로 켜둘 규칙들.

### 4-1. 기본 설정

```ts
// eslint.config.js (flat config 예시)
import jsxA11y from "eslint-plugin-jsx-a11y";

export default [
  jsxA11y.flatConfigs.recommended,
  {
    rules: {
      "jsx-a11y/control-has-associated-label": "error",
      // 다른 규칙들은 필요 시 여기에 명시적으로 추가/조정
    },
  },
];
```

### 4-2. 주요 규칙 요약

- `alt-text`

  - `<img>`는 항상 `alt`가 있어야 합니다.
  - 정보 없는 장식 이미지: `alt=""`.
  - 텍스트와 함께 있는 아이콘: `alt=""` 권장.

- `control-has-associated-label`

  - 모든 컨트롤(인풋, 버튼, Select 등)에 연관 레이블이 있어야 합니다.
  - 폼 관련 마크업은:

    - `<label>` + `<input>`
    - `aria-label`
    - `aria-labelledby`
    - 중 하나 이상을 갖도록 구현.

- `no-noninteractive-element-interactions`

  - `<div>`, `<span>` 같은 비상호작용 요소에 `onClick` 등을 사용하면 경고.
  - 반드시 `role`/`tabIndex`를 넣어 상호작용 요소임을 명시하거나, 아예 `<button>`, `<a>`로 바꿀 것.

- `no-noninteractive-element-to-interactive-role`

  - `<main>`, `<ul>`, `<li>`, `<img>` 등 의미 있는 컨테이너에 `role="button"` 같은 상호작용 역할을 부여하면 안 됩니다.
  - 진짜 상호작용이 필요하면 `<button>`, `<a>`를 사용.

- `no-noninteractive-tabindex`

  - 비상호작용 요소에 `tabIndex`를 주지 말 것.
  - 정말 포커스/인터랙션이 필요하면 `role`과 함께 상호작용 요소로 승격.

- `tabindex-no-positive`

  - `tabIndex > 0` 금지.
  - 허용 값: `0` 또는 `-1`만 사용.

### 4-3. 디자인 시스템/폴리모픽 컴포넌트와 연동

#### 컴포넌트 매핑

```ts
export default [
  jsxA11y.flatConfigs.recommended,
  {
    rules: {
      "jsx-a11y/control-has-associated-label": "error",
    },
    settings: {
      "jsx-a11y": {
        components: {
          MyButton: "button",
          MyTxt: "span",
        },
      },
    },
  },
];
```

- 이렇게 하면 `<MyButton>`/`<MyTxt>`에도 `<button>`/`<span>`의 a11y 규칙이 동일 적용.

#### 폴리모픽 as prop

```ts
export default [
  jsxA11y.flatConfigs.recommended,
  {
    rules: {
      "jsx-a11y/control-has-associated-label": "error",
    },
    settings: {
      "jsx-a11y": {
        polymorphicPropName: "as",
        components: {
          MyButton: "button",
          MyTxt: "span",
        },
      },
    },
  },
];
```

- `<MyButton as="a" href="/home">`처럼 다양한 태그로 렌더링되는 컴포넌트에서 a11y 규칙이 제대로 동작하도록 설정.

#### 커스텀 레이블 prop 지원

```ts
export default [
  jsxA11y.flatConfigs.recommended,
  {
    rules: {
      "jsx-a11y/control-has-associated-label": [
        "error",
        {
          labelAttributes: ["contents"], // 예: <MyCard contents="카드 내용" />
        },
      ],
    },
    settings: {
      "jsx-a11y": {
        polymorphicPropName: "as",
        components: {
          MyButton: "button",
          MyTxt: "span",
          MyCard: "button",
        },
      },
    },
  },
];
```

---

## 5. 새 컴포넌트 작성 시 체크리스트

에이전트/개발자는 새 컴포넌트를 만들 때 다음을 반드시 확인합니다:

1. **역할(Role)**: 시맨틱 태그를 사용했는가? 필요시 `role`을 명시했는가?
2. **레이블(Label)**: 모든 인터랙티브 요소에 이름이 있는가?
3. **상태(State)**: 상태 속성이 HTML/ARIA와 동기화되어 있는가?
4. **키보드 탐색**: Tab, Enter, Space, 방향키로 조작 가능한가?
5. **시각 의존성**: 색/아이콘만으로 정보를 전달하지 않았는가?
6. **eslint-plugin-jsx-a11y**: 설정된 규칙을 위반하지 않았는가?
