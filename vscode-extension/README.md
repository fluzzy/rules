# VS Code Extension Development

Resources for building VS Code extensions. Guides cover the full extension lifecycle — from anatomy and API to testing, bundling, and publishing. Rules enforce security, performance, and UX best practices.

## Guides

| Guide | Description |
| --- | --- |
| [Extension Anatomy](./guides/extension-anatomy.md) | package.json manifest, extension.ts lifecycle, ExtensionContext, Disposable pattern |
| [Activation Events](./guides/activation-events.md) | onCommand, onLanguage, onView, onUri, workspaceContains, onStartupFinished |
| [Contribution Points](./guides/contribution-points.md) | commands, menus, keybindings, configuration, views, viewsContainers, languages, grammars |
| [VS Code API Overview](./guides/vscode-api-overview.md) | Declarative vs Programmatic features, API namespaces, common capabilities |
| [Commands and Configuration](./guides/commands-and-configuration.md) | registerCommand, executeCommand, configuration schema, getConfiguration |
| [Webview API](./guides/webview-api.md) | createWebviewPanel, asWebviewUri, message passing, state management, CSP |
| [TreeView API](./guides/treeview-api.md) | TreeDataProvider, EventEmitter refresh, viewsContainers, contextValue |
| [Custom Editor API](./guides/custom-editor-api.md) | CustomEditorProvider, CustomDocument, undo/redo, save/revert/backup |
| [Status Bar API](./guides/status-bar-api.md) | createStatusBarItem, alignment/priority, command binding, dynamic updates |
| [Language Features](./guides/language-features.md) | Completion, Hover, Definition, Diagnostics, CodeLens, Formatting, Rename providers |
| [Language Server Protocol](./guides/language-server-protocol.md) | LSP architecture, LanguageClient setup, ServerOptions, TransportKind |
| [Workspace and FileSystem](./guides/workspace-and-filesystem.md) | workspace.fs, FileSystemWatcher, WorkspaceEdit, Virtual Documents, Trust API |
| [Testing Extensions](./guides/testing-extensions.md) | @vscode/test-electron, integration tests, launch.json extensionHost |
| [Bundling Extensions](./guides/bundling-extensions.md) | esbuild config, webpack config, externals, watch mode |
| [Publishing Extensions](./guides/publishing-extensions.md) | vsce package/publish, Azure DevOps PAT, pre-release, .vscodeignore |
| [CI/CD](./guides/ci-cd.md) | GitHub Actions, Azure Pipelines, xvfb, matrix testing, automated publishing |
| [Web Extensions and Advanced](./guides/web-extensions-and-advanced.md) | browser entry point, Node.js restrictions, Extension Host architecture, Remote Development |

## Rules

| Rule | Description |
| --- | --- |
| [Webview Security](./rules/webview-security.md) | CSP, asWebviewUri, innerHTML/eval prohibition, message protocol, localResourceRoots |
| [Performance](./rules/performance.md) | Bundling, activation strategy, Disposable management, debounce, DocumentSelector |
| [UX Guidelines](./rules/ux-guidelines.md) | Codicons, withProgress, notification restraint, theme compatibility, accessibility |
| [Workspace Trust](./rules/workspace-trust.md) | untrustedWorkspaces declaration, isTrusted checks, code execution restrictions |
| [Extension Publishing Checklist](./rules/extension-publishing-checklist.md) | README/CHANGELOG/LICENSE, icon, .vscodeignore, semantic versioning, CI/CD |
