# Taskmaster AI - AI-Powered Task Management System

> **Source**: https://github.com/eyaltoledano/claude-task-master
> **Website**: https://www.task-master.dev
> **Docs**: https://docs.task-master.dev
> **Archive Date**: 2026-01-19

A task management system for Claude and AI-powered development. Designed to work seamlessly with Cursor AI.

## Overview

Break down complex projects into manageable tasks that AI agents can easily one-shot. Keep AI agents on track, eliminate context overload, and prevent good code from being damaged. Ship better, more ambitious projects.

**Completely free, open source, use your own API keys.**

## Requirements

At least one API key required (except when using Claude Code/Codex CLI OAuth):

- Anthropic API key (Claude API)
- OpenAI API key
- Google Gemini API key
- Perplexity API key (for research models)
- xAI API Key
- OpenRouter API Key
- Azure OpenAI API Key
- Ollama API Key

Three model types can be defined: **main model**, **research model**, **fallback model**

## Quick Start

### Option 1: MCP (Recommended)

#### 1. MCP Config File Location

| Editor   | Mac/Linux                             | Windows                                           |
| -------- | ------------------------------------- | ------------------------------------------------- |
| Cursor   | `~/.cursor/mcp.json`                  | `%USERPROFILE%\.cursor\mcp.json`                  |
| Windsurf | `~/.codeium/windsurf/mcp_config.json` | `%USERPROFILE%\.codeium\windsurf\mcp_config.json` |
| VS Code  | `.vscode/mcp.json`                    | `.vscode\mcp.json`                                |

#### 2. Configuration Example (Cursor/Windsurf)

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

#### 4. Initialize Task Master

In AI chat:

```
Initialize taskmaster-ai in my project
```

#### 5. Prepare PRD (Recommended)

- New projects: Create PRD at `.taskmaster/docs/prd.txt`
- Existing projects: Use `task-master migrate`

Example PRD template: `.taskmaster/templates/example_prd.txt`

### Option 2: Command Line

```bash
# Install
npm install -g task-master-ai

# Initialize
task-master init

# Parse PRD
task-master parse-prd scripts/prd.txt
```

## Common Commands

Through AI assistant:

- **Parse PRD**: `Can you parse my PRD at scripts/prd.txt?`
- **Plan next steps**: `What's the next task I should work on?`
- **Implement task**: `Can you help me implement task 3?`
- **View multiple tasks**: `Can you show me tasks 1, 3, and 5?`
- **Expand task**: `Can you help me expand task 4?`

## Documentation

- [Configuration Guide](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/configuration.md)
- [Tutorial](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/tutorial.md)
- [Command Reference](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/command-reference.md)
- [Task Structure](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/task-structure.md)
- [Example Interactions](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/examples.md)
- [Migration Guide](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/migration-guide.md)
- [Available Models](https://github.com/eyaltoledano/claude-task-master/blob/main/docs/models.md)

## Tool Loading Configuration

Configure tools to load with `TASK_MASTER_TOOLS` environment variable:

- `all`: All tools
- `standard`: Standard tool set
- `core`: Core tools only
- Or comma-separated tool list

## License

MIT License (with commercial use restrictions for enterprise)

## Authors

- [@eyaltoledano](https://x.com/eyaltoledano)
- [@RalphEcom](https://x.com/RalphEcom)
