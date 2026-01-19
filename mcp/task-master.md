# Taskmaster AI - AI 기반 작업 관리 시스템

> **Source**: https://github.com/eyaltoledano/claude-task-master
> **Website**: https://www.task-master.dev
> **Docs**: https://docs.task-master.dev
> **Archive Date**: 2026-01-19

Claude와 AI 기반 개발을 위한 작업 관리 시스템. Cursor AI와 원활하게 작동하도록 설계됨.

## Overview

복잡한 프로젝트를 AI 에이전트가 쉽게 one-shot 할 수 있는 관리 가능한 작업으로 분해. AI 에이전트를 트랙에 유지하고, 컨텍스트 오버로드를 제거하고, 좋은 코드가 손상되는 것을 방지. 더 나은, 더 야심찬 프로젝트를 배포.

**완전 무료, 오픈 소스, 자체 API 키 사용.**

## Requirements

최소 하나의 API 키 필요 (Claude Code/Codex CLI OAuth 사용 시 제외):

- Anthropic API key (Claude API)
- OpenAI API key
- Google Gemini API key
- Perplexity API key (연구 모델용)
- xAI API Key
- OpenRouter API Key
- Azure OpenAI API Key
- Ollama API Key

3가지 모델 유형 정의 가능: **main model**, **research model**, **fallback model**

## Quick Start

### Option 1: MCP (권장)

#### 1. MCP 설정 파일 위치

| Editor   | Mac/Linux                             | Windows                                           |
| -------- | ------------------------------------- | ------------------------------------------------- |
| Cursor   | `~/.cursor/mcp.json`                  | `%USERPROFILE%\.cursor\mcp.json`                  |
| Windsurf | `~/.codeium/windsurf/mcp_config.json` | `%USERPROFILE%\.codeium\windsurf\mcp_config.json` |
| VS Code  | `.vscode/mcp.json`                    | `.vscode\mcp.json`                                |

#### 2. 설정 예시 (Cursor/Windsurf)

```json
{
  "mcpServers": {
    "task-master-ai": {
      "command": "npx",
      "args": ["-y", "task-master-ai"],
      "env": {
        "ANTHROPIC_API_KEY": "YOUR_ANTHROPIC_API_KEY_HERE",
        "PERPLEXITY_API_KEY": "YOUR_PERPLEXITY_API_KEY_HERE",
        "OPENAI_API_KEY": "YOUR_OPENAI_KEY_HERE",
        "GOOGLE_API_KEY": "YOUR_GOOGLE_KEY_HERE"
      }
    }
  }
}
```

#### 3. Claude Code Quick Install

```bash
claude mcp add taskmaster-ai -- npx -y task-master-ai
```

#### 4. Task Master 초기화

AI 채팅창에서:

```
Initialize taskmaster-ai in my project
```

#### 5. PRD 준비 (권장)

- 새 프로젝트: `.taskmaster/docs/prd.txt`에 PRD 생성
- 기존 프로젝트: `task-master migrate` 사용

예시 PRD 템플릿: `.taskmaster/templates/example_prd.txt`

### Option 2: Command Line

```bash
# 설치
npm install -g task-master-ai

# 초기화
task-master init

# PRD 파싱
task-master parse-prd scripts/prd.txt
```

## Common Commands

AI 어시스턴트를 통해:

- **PRD 파싱**: `Can you parse my PRD at scripts/prd.txt?`
- **다음 단계 계획**: `What's the next task I should work on?`
- **작업 구현**: `Can you help me implement task 3?`
- **여러 작업 보기**: `Can you show me tasks 1, 3, and 5?`
- **작업 확장**: `Can you help me expand task 4?`

## Documentation

- [Configuration Guide](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/configuration.md)
- [Tutorial](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/tutorial.md)
- [Command Reference](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/command-reference.md)
- [Task Structure](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/task-structure.md)
- [Example Interactions](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/examples.md)
- [Migration Guide](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/migration-guide.md)
- [Available Models](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/models.md)

## Tool Loading Configuration

`TASK_MASTER_TOOLS` 환경 변수로 로드할 도구 설정:

- `all`: 모든 도구
- `standard`: 표준 도구 세트
- `core`: 핵심 도구만
- 또는 쉼표로 구분된 도구 목록

## License

MIT License (with commercial use restrictions for enterprise)

## Authors

- [@eyaltoledano](https://x.com/eyaltoledano)
- [@RalphEcom](https://x.com/RalphEcom)
