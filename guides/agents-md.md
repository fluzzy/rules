# AGENTS.md 작성 가이드

> **Source**: [HumanLayer - Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md), [AGENTS.md](https://agents.md/)

AI 코딩 에이전트가 프로젝트에서 효과적으로 작업하도록 돕는 표준화된 파일.

---

## 핵심 원칙: LLM은 (대부분) Stateless

LLM은 stateless 함수. 가중치는 추론 시 고정되어 시간이 지나도 학습하지 않음. **모델이 코드베이스에 대해 아는 것은 오직 입력된 토큰뿐.**

**의미:**

1. 코딩 에이전트는 각 세션 시작 시 코드베이스에 대해 아무것도 모름
2. 매 세션 시작마다 중요한 모든 것을 알려줘야 함
3. `AGENTS.md`가 이를 위한 선호 방법

---

## AGENTS.md로 에이전트 온보딩

에이전트가 세션 시작 시 아무것도 모르므로 `AGENTS.md`로 온보딩:

- **WHAT**: 기술 스택, 프로젝트 구조. 코드베이스 지도 제공. 모노레포에서 특히 중요
- **WHY**: 프로젝트의 **목적**. 저장소의 각 부분의 목적과 기능
- **HOW**: 프로젝트 작업 방법. `bun` 대신 `node` 사용? 테스트, 타입 체크, 컴파일 방법

---

## 좋은 AGENTS.md 작성법

### 적은 지시가 더 좋다

연구 결과:

1. 프론티어 사고 LLM은 ~150-200개 지시를 합리적으로 따름
2. 작은 모델은 훨씬 빠르게 성능 저하
3. LLM은 프롬프트 주변부(시작과 끝)의 지시에 편향
4. 지시 수 증가 시 모든 지시를 균일하게 무시하기 시작

**`AGENTS.md`는 가능한 적은 지시를 포함해야 함** — 이상적으로 보편적으로 적용 가능한 것만.

### 파일 길이와 적용성

`AGENTS.md`가 **모든 세션에 포함**되므로 내용은 가능한 보편적으로 적용 가능해야 함.

예: 새 데이터베이스 스키마 구조화 방법을 포함하지 말 것 — 다른 관련 없는 작업 시 모델을 산만하게 함!

길이: **300줄 미만이 최선**, 짧을수록 좋음. HumanLayer의 루트 `AGENTS.md`는 **60줄 미만**.

### Progressive Disclosure

간결한 `AGENTS.md`로 모든 것을 커버하기 어려울 수 있음. **점진적 공개** 원칙 활용:

작업별 지시를 프로젝트 어딘가의 **별도 마크다운 파일**에 보관:

```
agent_docs/
  ├── building_the_project.md
  ├── running_tests.md
  ├── code_conventions.md
  └── service_architecture.md
```

`AGENTS.md`에 이 파일들의 목록과 간단한 설명 포함. 에이전트가 관련 파일을 결정하고 작업 시작 전에 읽도록 지시.

**복사보다 포인터 선호.** 가능하면 코드 스니펫 포함 피함 — 금방 stale 됨. 대신 `file:line` 참조 포함.

### 에이전트는 (비싼) 린터가 아니다

LLM은 전통적 린터/포매터에 비해 상대적으로 비싸고 **훨씬 느림**. **가능하면 결정론적 도구 사용.**

**LLM은 in-context learner!** 코드가 특정 스타일 가이드라인을 따른다면, 코드베이스 검색으로 무장한 에이전트는 명시하지 않아도 기존 패턴을 따르는 경향.

---

## 결론

1. `AGENTS.md`는 에이전트를 코드베이스에 온보딩하는 것. **WHY**, **WHAT**, **HOW** 정의
2. **적은 지시가 더 좋음.** 필요한 지시 생략하지 말되, 가능한 적게 포함
3. 내용을 **간결하고 보편적으로 적용 가능**하게 유지
4. **점진적 공개** — 에이전트에게 필요한 모든 것을 알려주지 말고, **중요한 정보를 찾는 방법** 알려주기
5. 에이전트는 린터가 아님. 린터와 포매터 사용
6. **`AGENTS.md`는 가장 높은 레버리지 포인트** — 자동 생성하지 말고 신중하게 수작업

---

## References

- [HumanLayer: Writing a good CLAUDE.md](https://www.humanlayer.dev/blog/writing-a-good-claude-md)
- [AGENTS.md Official Site](https://agents.md/)
