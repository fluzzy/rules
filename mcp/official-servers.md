# MCP 공식 서버 모음 (Official Reference Servers)

> **Source**: https://github.com/modelcontextprotocol/servers
> **Registry**: https://registry.modelcontextprotocol.io
> **Archive Date**: 2026-01-19

Model Context Protocol (MCP)의 레퍼런스 구현 및 커뮤니티 서버 모음.

## MCP SDK 목록

| 언어       | SDK                                                                      |
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

## 🌟 Reference Servers

| Server                                                                                                  | 설명                            | NPX 명령어                                                |
| ------------------------------------------------------------------------------------------------------- | ------------------------------- | --------------------------------------------------------- |
| [Everything](https://github.com/modelcontextprotocol/servers/blob/main/src/everything)                  | 테스트용 프롬프트, 리소스, 도구 | `npx -y @modelcontextprotocol/server-everything`          |
| [Fetch](https://github.com/modelcontextprotocol/servers/blob/main/src/fetch)                            | 웹 콘텐츠 가져오기 및 변환      | `npx -y @modelcontextprotocol/server-fetch`               |
| [Filesystem](https://github.com/modelcontextprotocol/servers/blob/main/src/filesystem)                  | 파일 작업 및 접근 제어          | `npx -y @modelcontextprotocol/server-filesystem`          |
| [Git](https://github.com/modelcontextprotocol/servers/blob/main/src/git)                                | Git 저장소 조작                 | `uvx mcp-server-git`                                      |
| [Memory](https://github.com/modelcontextprotocol/servers/blob/main/src/memory)                          | 지식 그래프 기반 메모리         | `npx -y @modelcontextprotocol/server-memory`              |
| [Sequential Thinking](https://github.com/modelcontextprotocol/servers/blob/main/src/sequentialthinking) | 순차적 문제 해결                | `npx -y @modelcontextprotocol/server-sequential-thinking` |
| [Time](https://github.com/modelcontextprotocol/servers/blob/main/src/time)                              | 시간 및 타임존 변환             | `npx -y @modelcontextprotocol/server-time`                |

## 사용 예시 (Claude Desktop)

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

- [MCP 공식 문서](https://modelcontextprotocol.io/)
- [MCP Registry](https://registry.modelcontextprotocol.io/)
- [MCP Specification](https://spec.modelcontextprotocol.io/)

## License

MIT License
