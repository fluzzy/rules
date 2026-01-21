# Context7 MCP - Up-to-date Code Docs For Any Prompt

> **Source**: https://github.com/upstash/context7
> **Website**: https://context7.com
> **Archive Date**: 2026-01-19

An MCP server that provides up-to-date code documentation for LLMs and AI code editors.

## The Problem (Without Context7)

LLMs rely on outdated/generic information about libraries you use:

- Outdated code examples based on old training data
- Hallucinated APIs that don't exist
- Generic answers for old package versions

## The Solution (With Context7)

Context7 MCP fetches up-to-date, version-specific documentation and code examples directly from the source and places them in your prompt.

Add `use context7` to your prompt:

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

To avoid typing `use context7` every time, add a rule:

- **Cursor**: Cursor Settings > Rules
- **Claude Code**: CLAUDE.md

Example rule:

```
Always use Context7 MCP when I need library/API documentation,
code generation, setup or configuration steps without me having to explicitly ask.
```

### Use Library Id

Specify a specific library ID directly:

```
Implement basic authentication with Supabase. use library /supabase/supabase for API and docs.
```

### Specify a Version

Specify version:

```
How do I set up Next.js 14 middleware? use context7
```

## Available Tools

1. **resolve-library-id**: Convert general library name to Context7-compatible ID
   - `query`: User question/task (required)
   - `libraryName`: Library name to search (required)

2. **query-docs**: Query library documentation using Context7-compatible ID
   - `libraryId`: Context7-compatible library ID (e.g., /mongodb/docs) (required)
   - `query`: Question/task to fetch relevant docs (required)

## More Documentation

- [More MCP Clients](https://context7.com/docs/resources/all-clients)
- [Adding Libraries](https://context7.com/docs/adding-libraries)
- [Troubleshooting](https://context7.com/docs/resources/troubleshooting)
- [API Reference](https://context7.com/docs/api-guide)
- [Developer Guide](https://context7.com/docs/resources/developer)

## License

Apache 2.0
