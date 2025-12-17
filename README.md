# rules

프론트엔드 코드 품질 규칙 모음

## 구조

```
toss-frontend-fundamentals/                      # Frontend Fundamentals 기반
├── README.md                                    # README.md
├── AGENTS.md                                    # 코드 품질 철학 문서
├── ACCESSIBILITY.md                             # 접근성 철학 문서
└── .cursor/
    └── rules/
        ├── frontend-code-quality.mdc            # 코드 품질 규칙
        └── accessibility.mdc                    # 접근성 규칙

playwright-e2e/                                  # Playwright 공식 문서 기반
├── README.md                                    # README.md
├── AGENTS.md                                    # E2E 테스트 철학 문서
└── .cursor/
    └── rules/
        └── e2e-testing.mdc                      # E2E 테스트 규칙

meta-agents-rule/                                # Meta Agents Rule of Two 기반
├── README.md                                    # README.md
├── AGENTS.md                                    # Agents Rule of Two 철학 문서
└── .cursor/
    └── rules/
        └── agents-rule-of-two.mdc               # Agents Rule of Two 규칙

openai-agents-guide/                             # OpenAI "A practical guide to building agents" 기반
├── README.md                                    # README.md
├── AGENTS.md                                    # 빌딩 에이전트 철학 문서
└── .cursor/
    └── rules/
        └── building-agents.mdc                  # 빌딩 에이전트 규칙
```

## Rules & Documentation

각 규칙 세트의 Cursor Rules와 Agents 기반 철학 문서:

- **TOSS Frontend Fundamentals**
  - [AGENTS.md](./toss-frontend-fundamentals/AGENTS.md) - 코드 품질 철학 문서
  - [.cursor/rules/](./toss-frontend-fundamentals/.cursor/rules/) - Cursor 규칙 파일
- **Playwright E2E Testing**
  - [AGENTS.md](./playwright-e2e/AGENTS.md) - E2E 테스트 철학 문서
  - [.cursor/rules/](./playwright-e2e/.cursor/rules/) - Cursor 규칙 파일
- **Meta Agents Rule of Two**
  - [AGENTS.md](./meta-agents-rule/AGENTS.md) - Agents Rule of Two 철학 문서
  - [.cursor/rules/](./meta-agents-rule/.cursor/rules/) - Cursor 규칙 파일
- **OpenAI Practical Guide to Building Agents**
  - [AGENTS.md](./openai-agents-guide/AGENTS.md) - 빌딩 에이전트 철학 문서
  - [.cursor/rules/](./openai-agents-guide/.cursor/rules/) - Cursor 규칙 파일

## 참고 자료

- [Cursor Rules 문서](https://cursor.com/docs/context/rules)
- [AGENTS.md](https://agents.md/)
