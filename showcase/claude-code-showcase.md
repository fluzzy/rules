# Claude Code Showcase

> **Source**: https://github.com/ChrisWiles/claude-code-showcase
> **Archive Date**: 2026-01-19

Claude Code 에이전트를 팀의 슈퍼파워 동료로 만들기 위한 종합 설정 예제. 재사용 가능한 스킬, 에이전트, 훅을 통해 일관되고 체계적인 개발 워크플로우 구축.

## 디렉토리 구조

```
.claude/
├── settings.json        # 훅, 환경변수, 권한 설정
├── settings.md          # 설정 문서
├── agents/              # 커스텀 AI 에이전트
├── commands/            # 슬래시 커맨드 (/명령어)
├── hooks/               # 자동화 스크립트
├── skills/              # 도메인 지식 문서
└── rules/               # 모듈식 지침

.mcp.json               # MCP 서버 설정 (JIRA, GitHub 등)
CLAUDE.md               # 프로젝트 메모리
```

## 핵심 구성 요소

### 1. CLAUDE.md - 프로젝트 메모리

세션 시작 시 자동 로드되는 영구 컨텍스트.

**포함 내용:**

- 프로젝트 스택 및 아키텍처
- 테스트, 빌드, 린트 명령어
- 코드 스타일 가이드라인
- 중요 디렉토리 설명

**위치 우선순위:**

1. `.claude/CLAUDE.md`
2. `./CLAUDE.md` (프로젝트 루트)
3. `~/.claude/CLAUDE.md` (사용자 레벨)

### 2. settings.json - 훅과 환경

자동화된 품질 관리를 위한 중앙 설정.

**주요 훅 이벤트:**

| 이벤트             | 발동 시점       | 활용                     |
| ------------------ | --------------- | ------------------------ |
| `PreToolUse`       | 도구 실행 전    | 메인 브랜치 편집 차단    |
| `PostToolUse`      | 도구 완료 후    | 자동 포매팅, 테스트 실행 |
| `UserPromptSubmit` | 프롬프트 제출 시 | 컨텍스트 추가, 스킬 제안 |
| `Stop`             | 에이전트 종료 시 | 계속 실행 여부 결정      |

**훅 응답 형식:**

```json
{
  "block": true,
  "message": "이유",
  "feedback": "정보",
  "continue": false
}
```

### 3. MCP 서버 - 외부 통합

Model Context Protocol을 통해 JIRA, GitHub, Slack 등과 연결.

**.mcp.json 구조:**

```json
{
  "mcpServers": {
    "server-name": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "패키지명"],
      "env": {
        "API_KEY": "${API_KEY}"
      }
    }
  }
}
```

**주요 통합:**

| 카테고리   | 서버             | 기능                   |
| ---------- | ---------------- | ---------------------- |
| 이슈 추적  | JIRA, Linear     | 티켓 읽기/수정         |
| 코드/DevOps | GitHub, Sentry   | PR 관리, 에러 추적     |
| 커뮤니케이션 | Slack            | 메시지 전송            |
| 데이터베이스 | PostgreSQL       | 쿼리 실행, 스키마 확인 |

### 4. 스킬 - 도메인 지식

프로젝트 특화 가이드. YAML 헤더 + 마크다운 콘텐츠 형식.

```yaml
---
name: testing-patterns
description: Jest 테스팅 패턴. 테스트 작성/모킹 시 사용.
---

# 테스팅 패턴

## 테스트 구조
- describe 블록으로 그룹화
- AAA 패턴: Arrange, Act, Assert
```

### 5. 에이전트 - 전문화된 어시스턴트

자동 실행 또는 사용자 호출 가능한 특정 작업 담당자.

- **코드 리뷰 에이전트**: TypeScript 엄격 모드, 에러 처리 체크
- **문서 동기화**: 월간 커밋 읽기 → 문서 정렬 확인
- **품질 검토**: 주간 무작위 디렉토리 검토 및 자동 수정
- **의존성 감사**: 격주 안전한 업데이트

### 6. 커맨드 - 슬래시 커맨드

사용자가 명시적으로 호출하는 워크플로우.

- `/ticket PROJ-123`: 티켓 읽기 → 코드 구현 → 상태 업데이트 → PR 생성
- `/onboard`: 깊은 작업 탐색
- `/pr-review`: PR 검토 워크플로우

## GitHub Actions 워크플로우

| 워크플로우      | 빈도 | 기능               |
| --------------- | ---- | ------------------ |
| PR Claude 리뷰  | 자동 | 전체 PR 검토       |
| 문서 동기화     | 월간 | 문서-코드 동기화   |
| 코드 품질       | 주간 | 무작위 리뷰 및 수정 |
| 의존성 감사     | 격주 | 안전한 업데이트    |

## 빠른 시작

```bash
# 1. 디렉토리 생성
mkdir -p .claude/{agents,commands,hooks,skills}

# 2. CLAUDE.md 작성 (프로젝트 루트)
# 3. settings.json 설정 (훅 추가)
# 4. 첫 스킬 작성: .claude/skills/testing-patterns/SKILL.md
# 5. MCP 구성 (선택): .mcp.json
```

## License

MIT License
