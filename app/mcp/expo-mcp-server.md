# Expo MCP Server

> **Source**: [Expo Documentation](https://docs.expo.dev/eas/ai/mcp/)
> **Type**: Remote MCP Server (hosted by Expo)
> **Archive Date**: 2026-02-10

Official Expo MCP server that connects AI coding assistants to Expo projects. Provides SDK documentation access, package management, build/workflow automation, and local simulator interaction.

## Setup

### Claude Code

```bash
claude mcp add --transport http expo-mcp https://mcp.expo.dev/mcp
```

### Cursor

Settings → MCP → Add Server:

- **Type**: Streamable HTTP
- **URL**: `https://mcp.expo.dev/mcp`

### VS Code

Add to `.vscode/mcp.json`:

```json
{
  "servers": {
    "expo-mcp": {
      "type": "http",
      "url": "https://mcp.expo.dev/mcp"
    }
  }
}
```

### Local Capabilities (SDK 54+)

For simulator interaction and automation:

```bash
npx expo install expo-mcp --dev
```

Requires a running Expo development server (`npx expo start`).

## Tools

### Documentation & Packages (Server)

| Tool | Description | Example |
| --- | --- | --- |
| `learn` | Learn Expo how-to for a specific topic | "learn how to use expo-router" |
| `search_documentation` | Search Expo documentation | "search documentation for CNG" |
| `add_library` | Install packages with `npx expo install` | "add sqlite and basic CRUD" |
| `generate_claude_md` | Generate CLAUDE.md configuration | "generate a CLAUDE.md for the project" |
| `generate_agents_md` | Generate AGENTS.md files | "generate an AGENTS.md file" |

### Build (Server)

| Tool | Description | Example |
| --- | --- | --- |
| `build_run` | Trigger a new build from git ref | "run a production build for iOS" |
| `build_info` | Get details of a specific build | "get status of my latest iOS build" |
| `build_list` | List builds for a project | "list recent builds" |
| `build_logs` | Fetch logs for a completed build | "show logs for the failed build" |
| `build_cancel` | Cancel a queued/in-progress build | "cancel the current build" |
| `build_submit` | Submit a build to app stores | "submit latest build to App Store" |

### Workflow (Server)

| Tool | Description | Example |
| --- | --- | --- |
| `workflow_create` | Create a new workflow YAML file | "create a CI/CD workflow" |
| `workflow_run` | Trigger a workflow run from git ref | "run the build-and-deploy workflow" |
| `workflow_info` | Get details of a workflow run | "get status of latest workflow run" |
| `workflow_list` | List recent workflow runs | "list recent workflow runs" |
| `workflow_logs` | Fetch logs for a workflow job | "show logs for the build job" |
| `workflow_cancel` | Cancel a running workflow | "cancel the running workflow" |
| `workflow_validate` | Validate workflow YAML syntax | "validate my workflow file" |

### Local Automation (requires dev server + SDK 54+)

| Tool | Description | Example |
| --- | --- | --- |
| `automation_take_screenshot` | Take full device screenshot | "take a screenshot and verify" |
| `automation_tap` | Tap at screen coordinates | "tap at x=12, y=22" |
| `automation_find_view_by_testid` | Find and analyze views by testID | "dump properties for testID 'btn'" |
| `automation_tap_by_testid` | Tap views by testID | "click view with testID 'btn'" |
| `automation_take_screenshot_by_testid` | Screenshot specific views | "screenshot testID 'btn'" |
| `expo_router_sitemap` | Display expo-router sitemap | - |
| `open_devtools` | Open React Native DevTools | - |

## Requirements

- EAS paid plan required
- Authentication via OAuth
- Local capabilities require SDK 54+ and running dev server
