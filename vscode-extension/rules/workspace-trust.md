# Workspace Trust Rules

> **Source**: [VS Code Docs — Workspace Trust](https://code.visualstudio.com/api/extension-guides/workspace-trust)
> **Archive Date**: 2026-02-10

Rules for VS Code extension workspace trust integration. Covers trust declarations, restricted mode behavior, and security considerations for untrusted workspaces.

## Rules

You are an expert in VS Code extension security with deep knowledge of the Workspace Trust API and security boundaries.

**Trust Declaration**
- Always declare `capabilities.untrustedWorkspaces` in `package.json`. Explicitly state whether the extension supports untrusted workspaces (`true`, `false`, or `"limited"`).
- Use `"limited"` when the extension can provide partial functionality without trust. List sensitive settings in `restrictedConfigurations`.

```json
{
  "capabilities": {
    "untrustedWorkspaces": {
      "supported": "limited",
      "description": "Code execution features are disabled in untrusted workspaces",
      "restrictedConfigurations": [
        "myExtension.executablePath",
        "myExtension.scriptPath",
        "myExtension.args"
      ]
    }
  }
}
```

**Runtime Trust Checks**
- Check `vscode.workspace.isTrusted` before executing any code from the workspace, spawning processes, or reading executable paths from settings.
- Listen to `vscode.workspace.onDidGrantWorkspaceTrust` to upgrade features when trust is granted.

```typescript
export function activate(context: vscode.ExtensionContext) {
  if (vscode.workspace.isTrusted) {
    enableFullFeatures();
  } else {
    enableRestrictedFeatures();
  }

  context.subscriptions.push(
    vscode.workspace.onDidGrantWorkspaceTrust(() => {
      enableFullFeatures();
    })
  );
}
```

**Restricted Configurations**
- List all settings that accept file paths, executable paths, command-line arguments, or script content in `restrictedConfigurations`. VS Code ignores workspace-level values for these settings in untrusted workspaces.
- Never read workspace settings that specify executable paths or scripts without checking trust first.

**Code Execution**
- Never execute, compile, or evaluate code from workspace files in untrusted workspaces. This includes running linters, formatters, build tools, or task runners.
- Never spawn child processes using paths derived from workspace settings in untrusted workspaces.
- Disable auto-run features (file watchers that trigger execution, on-save hooks) in untrusted workspaces.

**Secret Storage**
- Use `context.secrets` (SecretStorage API) for storing tokens, API keys, and passwords. Never store secrets in workspace settings or workspace state.
- Never log or display secret values in Output Channel, notifications, or webviews.
