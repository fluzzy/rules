# Vercel Agent Skills

> **Source**: https://github.com/vercel-labs/agent-skills
> **Archive Date**: 2026-01-19

AI 코딩 에이전트의 기능을 확장하는 스킬 모음. Agent Skills 형식을 따르며, 설치 후 에이전트가 관련 작업 감지 시 자동 활용.

## 설치

```bash
npx add-skill vercel-labs/agent-skills
```

## 스킬 목록

### 1. react-best-practices

React와 Next.js 성능 최적화 가이드라인.

**특징:**

- 40개 이상의 규칙, 8개 범주로 분류
- 영향도 기준 우선순위화

**사용 시점:**

- React 컴포넌트나 Next.js 페이지 작성
- 클라이언트/서버 측 데이터 페칭 구현
- 성능 문제 코드 검토
- 번들 크기/로딩 시간 최적화

**범주:**

- 워터폴 제거
- 번들 최적화
- 서버 성능
- 클라이언트 데이터 페칭
- 리렌더링 최적화

### 2. web-design-guidelines

UI 코드가 웹 인터페이스 모범 사례를 준수하는지 검토.

**특징:**

- 100개 이상의 규칙 감사
- 접근성, 성능, UX 포함

**사용 시점:**

- "UI 검토해줘"
- "접근성 확인"
- "모범 사례 검사"

**범주:**

- 접근성
- 포커스 상태
- 폼
- 애니메이션
- 타이포그래피
- 이미지
- 다크 모드
- 터치 상호작용

### 3. vercel-deploy-claimable

애플리케이션을 Vercel에 즉시 배포.

**특징:**

- 40개 이상 프레임워크 자동 감지
- 프리뷰 URL 및 클레임 URL 반환
- 정적 HTML 자동 처리

**사용 시점:**

- "앱 배포해줘"
- "프로덕션에 배포"

**작동 방식:**

1. 프로젝트 타르볼 패키징
2. 프레임워크 감지
3. 업로드
4. URL 반환

## License

MIT License
