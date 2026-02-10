# VS Code API Overview

> **Source**: [VS Code Docs — Extension Capabilities Overview](https://code.visualstudio.com/api/extension-capabilities/overview)
> **Archive Date**: 2026-02-10

High-level overview of the VS Code Extension API. Covers the distinction between declarative and programmatic language features, and the main API namespaces.

## 1. Extension Capabilities

VS Code extensions can:

- **Common capabilities** — Core features available to all extensions (commands, configuration, keybindings, storage, notifications)
- **Theming** — Color themes, file icon themes, product icon themes
- **Declarative Language Features** — Static language support via grammars, snippets, configuration
- **Programmatic Language Features** — Dynamic language support via providers (completion, hover, diagnostics)
- **Workbench Extensions** — Custom views, webviews, status bar items, tree views
- **Debugging** — Debug adapters and protocol

## 2. Declarative vs Programmatic Language Features

| Aspect | Declarative | Programmatic |
|---|---|---|
| Definition | package.json `contributes` | TypeScript API calls |
| Examples | Syntax highlighting, bracket matching, snippets | IntelliSense, diagnostics, refactoring |
| Runtime | No extension code runs | Extension code runs providers |
| Performance | Lightweight, fast | Heavier, needs activation |

### Declarative Features

Defined entirely in `package.json`:
- Syntax highlighting (TextMate grammars)
- Snippet completion
- Bracket matching/autoclosing
- Comment toggling
- Language configuration (indentation, folding)

### Programmatic Features

Registered via `vscode.languages.register*Provider` methods:
- Completion (IntelliSense)
- Hover information
- Go to Definition / References
- Diagnostics (errors/warnings)
- Code Actions (quick fixes, refactoring)
- CodeLens
- Formatting
- Rename
- Document Symbols / Workspace Symbols

## 3. API Namespaces

The `vscode` module exposes these primary namespaces:

| Namespace | Description | Key APIs |
|---|---|---|
| `vscode.commands` | Command registration and execution | `registerCommand`, `executeCommand`, `getCommands` |
| `vscode.window` | UI interactions | `showInformationMessage`, `showQuickPick`, `createWebviewPanel`, `createStatusBarItem`, `createOutputChannel`, `createTerminal`, `showTextDocument` |
| `vscode.workspace` | Workspace and file system | `workspaceFolders`, `openTextDocument`, `applyEdit`, `getConfiguration`, `createFileSystemWatcher`, `fs` |
| `vscode.languages` | Language feature providers | `registerCompletionItemProvider`, `registerHoverProvider`, `registerDefinitionProvider`, `createDiagnosticCollection`, `registerCodeActionsProvider` |
| `vscode.debug` | Debug session management | `startDebugging`, `registerDebugConfigurationProvider`, `activeDebugSession` |
| `vscode.extensions` | Extension management | `getExtension`, `all`, `onDidChange` |
| `vscode.env` | Environment info | `appName`, `language`, `machineId`, `clipboard`, `openExternal` |
| `vscode.tasks` | Task management | `registerTaskProvider`, `executeTask`, `taskExecutions` |
| `vscode.authentication` | Authentication providers | `getSession`, `registerAuthenticationProvider` |
| `vscode.tests` | Test framework | `createTestController`, `createTestRunProfile` |
| `vscode.notebooks` | Notebook support | `createNotebookController`, `registerNotebookSerializer` |
| `vscode.scm` | Source control | `createSourceControl` |
| `vscode.comments` | Comment threads | `createCommentController` |

## 4. Common Capabilities

### Notifications

```typescript
vscode.window.showInformationMessage('Info');
vscode.window.showWarningMessage('Warning');
vscode.window.showErrorMessage('Error');

// With action buttons
const answer = await vscode.window.showInformationMessage('Save changes?', 'Yes', 'No');
```

### Quick Pick

```typescript
const item = await vscode.window.showQuickPick(['Option A', 'Option B', 'Option C'], {
  placeHolder: 'Select an option'
});
```

### Input Box

```typescript
const value = await vscode.window.showInputBox({
  prompt: 'Enter your name',
  placeHolder: 'Name',
  validateInput: (text) => text.length === 0 ? 'Name cannot be empty' : null
});
```

### Output Channel

```typescript
const channel = vscode.window.createOutputChannel('My Extension');
channel.appendLine('Starting process...');
channel.show();
```

### Progress

```typescript
vscode.window.withProgress({
  location: vscode.ProgressLocation.Notification,
  title: 'Processing...',
  cancellable: true
}, async (progress, token) => {
  progress.report({ increment: 0 });
  await doWork();
  progress.report({ increment: 50, message: 'Halfway done' });
  await doMoreWork();
  progress.report({ increment: 100 });
});
```

## 5. Extension Kinds

Extensions run in different contexts:

| Kind | Description | Example |
|---|---|---|
| `ui` | Runs in VS Code UI process | Themes, keymaps |
| `workspace` | Runs in workspace process (or remote) | Language servers, formatters |

Declare in `package.json`:

```json
{
  "extensionKind": ["workspace"]
}
```
