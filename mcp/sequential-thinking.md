# Sequential Thinking MCP Server

> **Source**: https://github.com/modelcontextprotocol/servers/tree/main/src/sequentialthinking
> **Archive Date**: 2026-01-19

An MCP server implementation that provides a tool for dynamic and reflective problem-solving through structured thinking processes.

## Features

- Break down complex problems into manageable steps
- Revise and refine thoughts as understanding deepens
- Branch into alternative reasoning paths
- Dynamically adjust total number of thoughts
- Generate and verify solution hypotheses

## Tool: sequential_thinking

Facilitates detailed step-by-step thinking processes for problem-solving and analysis.

**Input parameters:**

- `thought` (string): Current thinking step
- `nextThoughtNeeded` (boolean): Whether another thought step is needed
- `thoughtNumber` (integer): Current thought number
- `totalThoughts` (integer): Estimated total thoughts needed
- `isRevision` (boolean, optional): Whether this revises previous thinking
- `revisesThought` (integer, optional): Which thought is being reconsidered
- `branchFromThought` (integer, optional): Branching point thought number
- `branchId` (string, optional): Branch identifier
- `needsMoreThoughts` (boolean, optional): If more thoughts are needed

## Usage

Sequential Thinking tool use cases:

- Breaking down complex problems into steps
- Planning and design with room for revision
- Analysis that might need course correction
- Problems where full scope isn't initially clear
- Tasks that need to maintain context over multiple steps
- Situations where irrelevant information needs filtering

## Configuration

### Claude Desktop

Add to `claude_desktop_config.json`:

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
