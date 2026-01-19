# Context7 MCP - Up-to-date Code Docs For Any Prompt

> **Source**: https://github.com/upstash/context7
> **Website**: https://context7.com
> **Archive Date**: 2026-01-19

LLM과 AI 코드 에디터를 위한 최신 코드 문서를 제공하는 MCP 서버.

## 문제점 (Without Context7)

LLM은 사용하는 라이브러리에 대해 오래된/일반적인 정보에 의존:

- ❌ 오래된 학습 데이터를 기반으로 한 코드 예제
- ❌ 존재하지 않는 환상의 API
- ❌ 이전 패키지 버전에 대한 일반적인 답변

## 해결책 (With Context7)

Context7 MCP는 소스에서 직접 최신 버전별 문서와 코드 예제를 가져와 프롬프트에 직접 배치.

프롬프트에 `use context7` 추가:

```
Create a Next.js middleware that checks for a valid JWT in cookies
and redirects unauthenticated users to `/login`. use context7
```

## Installation

### Cursor Remote Server

```json
{
  "mcpServers": {
    "context7": {
      "url": "https://mcp.context7.com/mcp",
      "headers": {
        "CONTEXT7_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

### Cursor Local Server

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp", "--api-key", "YOUR_API_KEY"]
    }
  }
}
```

### Claude Code

```bash
# Local
claude mcp add context7 -- npx -y @upstash/context7-mcp --api-key YOUR_API_KEY

# Remote
claude mcp add --header "CONTEXT7_API_KEY: YOUR_API_KEY" --transport http context7 https://mcp.context7.com/mcp
```

## Tips

### Add a Rule

`use context7`를 매번 입력하지 않으려면 규칙 추가:

- **Cursor**: Cursor Settings > Rules
- **Claude Code**: CLAUDE.md

예시 규칙:

```
Always use Context7 MCP when I need library/API documentation,
code generation, setup or configuration steps without me having to explicitly ask.
```

### Use Library Id

특정 라이브러리 ID 직접 지정:

```
Implement basic authentication with Supabase. use library /supabase/supabase for API and docs.
```

### Specify a Version

버전 지정:

```
How do I set up Next.js 14 middleware? use context7
```

## Available Tools

1. **resolve-library-id**: 일반 라이브러리 이름을 Context7 호환 ID로 변환
   - `query`: 사용자 질문/작업 (필수)
   - `libraryName`: 검색할 라이브러리 이름 (필수)

2. **query-docs**: Context7 호환 ID를 사용하여 라이브러리 문서 검색
   - `libraryId`: Context7 호환 라이브러리 ID (예: /mongodb/docs) (필수)
   - `query`: 관련 문서를 가져올 질문/작업 (필수)

## More Documentation

- [More MCP Clients](https://context7.com/docs/resources/all-clients)
- [Adding Libraries](https://context7.com/docs/adding-libraries)
- [Troubleshooting](https://context7.com/docs/resources/troubleshooting)
- [API Reference](https://context7.com/docs/api-guide)
- [Developer Guide](https://context7.com/docs/resources/developer)

## License

Apache 2.0
