# MCP Official Reference Servers

> **Source**: https://github.com/modelcontextprotocol/servers
> **Registry**: https://registry.modelcontextprotocol.io
> **Archive Date**: 2026-01-19

Reference implementations and community servers for Model Context Protocol (MCP).

## MCP SDK List

| Language   | SDK                                                                      |
| ---------- | ------------------------------------------------------------------------ |
| TypeScript | [typescript-sdk](https://github.com/modelcontextprotocol/typescript-sdk) |
| Python     | [python-sdk](https://github.com/modelcontextprotocol/python-sdk)         |
| Go         | [go-sdk](https://github.com/modelcontextprotocol/go-sdk)                 |
| Rust       | [rust-sdk](https://github.com/modelcontextprotocol/rust-sdk)             |
| C#         | [csharp-sdk](https://github.com/modelcontextprotocol/csharp-sdk)         |
| Java       | [java-sdk](https://github.com/modelcontextprotocol/java-sdk)             |
| Kotlin     | [kotlin-sdk](https://github.com/modelcontextprotocol/kotlin-sdk)         |
| Swift      | [swift-sdk](https://github.com/modelcontextprotocol/swift-sdk)           |
| Ruby       | [ruby-sdk](https://github.com/modelcontextprotocol/ruby-sdk)             |
| PHP        | [php-sdk](https://github.com/modelcontextprotocol/php-sdk)               |

## Reference Servers

| Server                                                                                                  | Description                              | NPX Command                                               |
| ------------------------------------------------------------------------------------------------------- | ---------------------------------------- | --------------------------------------------------------- |
| [Everything](https://github.com/modelcontextprotocol/servers/blob/main/src/everything)                  | Prompts, resources, tools for testing    | `npx -y @modelcontextprotocol/server-everything`          |
| [Fetch](https://github.com/modelcontextprotocol/servers/blob/main/src/fetch)                            | Fetch and convert web content            | `npx -y @modelcontextprotocol/server-fetch`               |
| [Filesystem](https://github.com/modelcontextprotocol/servers/blob/main/src/filesystem)                  | File operations and access control       | `npx -y @modelcontextprotocol/server-filesystem`          |
| [Git](https://github.com/modelcontextprotocol/servers/blob/main/src/git)                                | Git repository manipulation              | `uvx mcp-server-git`                                      |
| [Memory](https://github.com/modelcontextprotocol/servers/blob/main/src/memory)                          | Knowledge graph-based memory             | `npx -y @modelcontextprotocol/server-memory`              |
| [Sequential Thinking](https://github.com/modelcontextprotocol/servers/blob/main/src/sequentialthinking) | Sequential problem solving               | `npx -y @modelcontextprotocol/server-sequential-thinking` |
| [Time](https://github.com/modelcontextprotocol/servers/blob/main/src/time)                              | Time and timezone conversion             | `npx -y @modelcontextprotocol/server-time`                |

## Usage Example (Claude Desktop)

```json
{
  "mcpServers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    },
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/path/to/allowed/files"
      ]
    },
    "git": {
      "command": "uvx",
      "args": ["mcp-server-git", "--repository", "path/to/git/repo"]
    }
  }
}
```

## Resources

- [MCP Official Documentation](https://modelcontextprotocol.io/)
- [MCP Registry](https://registry.modelcontextprotocol.io/)
- [MCP Specification](https://spec.modelcontextprotocol.io/)

## License

MIT License
