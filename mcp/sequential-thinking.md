# Sequential Thinking MCP Server

> **Source**: https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking
> **Archive Date**: 2026-01-19

구조화된 사고 프로세스를 통해 동적이고 반성적인 문제 해결을 위한 도구를 제공하는 MCP 서버 구현.

## Features

- 복잡한 문제를 관리 가능한 단계로 분해
- 이해가 깊어지면 생각을 수정하고 다듬기
- 대안적 추론 경로로 분기
- 생각의 총 수를 동적으로 조정
- 솔루션 가설 생성 및 검증

## Tool: sequential_thinking

문제 해결 및 분석을 위한 상세한 단계별 사고 프로세스를 촉진.

**입력값:**

- `thought` (string): 현재 사고 단계
- `nextThoughtNeeded` (boolean): 다른 사고 단계가 필요한지 여부
- `thoughtNumber` (integer): 현재 사고 번호
- `totalThoughts` (integer): 필요한 총 사고 수 추정
- `isRevision` (boolean, optional): 이전 사고를 수정하는지 여부
- `revisesThought` (integer, optional): 재고 중인 사고
- `branchFromThought` (integer, optional): 분기점 사고 번호
- `branchId` (string, optional): 분기 식별자
- `needsMoreThoughts` (boolean, optional): 더 많은 사고가 필요한지 여부

## Usage

Sequential Thinking 도구 사용 사례:

- 복잡한 문제를 단계로 분해
- 수정 여지가 있는 계획 및 설계
- 방향 수정이 필요할 수 있는 분석
- 전체 범위가 처음에 명확하지 않은 문제
- 여러 단계에 걸쳐 컨텍스트를 유지해야 하는 작업
- 관련 없는 정보를 필터링해야 하는 상황

## Configuration

### Claude Desktop

`claude_desktop_config.json`에 추가:

```json
{
  "mcpServers": {
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
```

### Docker

```json
{
  "mcpServers": {
    "sequentialthinking": {
      "command": "docker",
      "args": ["run", "--rm", "-i", "mcp/sequentialthinking"]
    }
  }
}
```

### VS Code

```json
{
  "servers": {
    "sequential-thinking": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-sequential-thinking"]
    }
  }
}
```

### Codex CLI

```bash
codex mcp add sequential-thinking npx -y @modelcontextprotocol/server-sequential-thinking
```

## Building

Docker:

```bash
docker build -t mcp/sequentialthinking -f src/sequentialthinking/Dockerfile .
```

## License

MIT License
