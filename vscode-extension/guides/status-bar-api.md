# Status Bar API

> **Source**: [VS Code Docs — Status Bar](https://code.visualstudio.com/api/extension-guides/status-bar) | [Status Bar Sample](https://github.com/microsoft/vscode-extension-samples/tree/main/statusbar-sample)
> **Archive Date**: 2026-02-10

Guide to creating and managing status bar items in VS Code extensions. Covers creating items, alignment, priority, command binding, and dynamic updates.

## 1. Creating a Status Bar Item

```typescript
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  // Create a status bar item
  const statusBarItem = vscode.window.createStatusBarItem(
    vscode.StatusBarAlignment.Right,  // Left or Right
    100  // Priority — higher numbers appear more to the left
  );

  statusBarItem.text = '$(file-code) Lines: 0';
  statusBarItem.tooltip = 'Number of lines in the active file';
  statusBarItem.command = 'myExtension.showLineCount';
  statusBarItem.show();

  context.subscriptions.push(statusBarItem);
}
```

## 2. StatusBarItem Properties

| Property | Type | Description |
|---|---|---|
| `text` | `string` | Display text (supports Codicon `$(icon-name)`) |
| `tooltip` | `string \| MarkdownString` | Hover tooltip |
| `color` | `string \| ThemeColor` | Text color |
| `backgroundColor` | `ThemeColor` | Background color (`statusBarItem.errorBackground` or `statusBarItem.warningBackground`) |
| `command` | `string \| Command` | Command executed on click |
| `accessibilityInformation` | `AccessibilityInformation` | Screen reader info |
| `name` | `string` | Identifier shown in status bar context menu |

## 3. Alignment and Priority

```typescript
// Left side, high priority (closer to left edge)
const leftItem = vscode.window.createStatusBarItem(vscode.StatusBarAlignment.Left, 1000);

// Right side, low priority (closer to right edge)
const rightItem = vscode.window.createStatusBarItem(vscode.StatusBarAlignment.Right, 1);
```

| Alignment | Priority | Position |
|---|---|---|
| `Left` | Higher number | More to the left |
| `Right` | Higher number | More to the left (closer to center) |

## 4. Dynamic Updates

Update the status bar based on editor events:

```typescript
export function activate(context: vscode.ExtensionContext) {
  const statusBarItem = vscode.window.createStatusBarItem(
    vscode.StatusBarAlignment.Right, 100
  );
  statusBarItem.name = 'Line Counter';

  // Update when active editor changes
  context.subscriptions.push(
    vscode.window.onDidChangeActiveTextEditor(editor => {
      updateStatusBar(statusBarItem, editor);
    })
  );

  // Update when selection changes
  context.subscriptions.push(
    vscode.window.onDidChangeTextEditorSelection(event => {
      updateStatusBar(statusBarItem, event.textEditor);
    })
  );

  // Update when document changes
  context.subscriptions.push(
    vscode.workspace.onDidChangeTextDocument(event => {
      const editor = vscode.window.activeTextEditor;
      if (editor && event.document === editor.document) {
        updateStatusBar(statusBarItem, editor);
      }
    })
  );

  // Initial update
  updateStatusBar(statusBarItem, vscode.window.activeTextEditor);

  context.subscriptions.push(statusBarItem);
}

function updateStatusBar(item: vscode.StatusBarItem, editor: vscode.TextEditor | undefined) {
  if (!editor) {
    item.hide();
    return;
  }

  const lineCount = editor.document.lineCount;
  const selectedLines = editor.selections.reduce(
    (total, sel) => total + (sel.end.line - sel.start.line + 1), 0
  );

  item.text = `$(file-code) ${lineCount} lines`;
  if (selectedLines > 1) {
    item.text += ` (${selectedLines} selected)`;
  }
  item.show();
}
```

## 5. Background Colors

Use special theme colors for error/warning states:

```typescript
// Error state (red background)
statusBarItem.backgroundColor = new vscode.ThemeColor('statusBarItem.errorBackground');
statusBarItem.text = '$(error) Build Failed';

// Warning state (yellow background)
statusBarItem.backgroundColor = new vscode.ThemeColor('statusBarItem.warningBackground');
statusBarItem.text = '$(warning) 3 Issues';

// Reset to normal
statusBarItem.backgroundColor = undefined;
statusBarItem.text = '$(check) All Good';
```

## 6. Codicons

Status bar text supports [Codicons](https://microsoft.github.io/vscode-codicons/dist/codicon.html):

```typescript
statusBarItem.text = '$(sync~spin) Syncing...';   // spinning sync icon
statusBarItem.text = '$(check) Ready';              // checkmark
statusBarItem.text = '$(error) Error';              // error icon
statusBarItem.text = '$(warning) Warning';          // warning icon
statusBarItem.text = '$(gear) Settings';            // gear icon
statusBarItem.text = '$(git-branch) main';          // git branch icon
```

## 7. Complete Example

```typescript
import * as vscode from 'vscode';

let statusBarItem: vscode.StatusBarItem;

export function activate(context: vscode.ExtensionContext) {
  // Register command
  const cmd = vscode.commands.registerCommand('myExtension.wordCount', () => {
    const editor = vscode.window.activeTextEditor;
    if (editor) {
      const text = editor.document.getText();
      const words = text.split(/\s+/).filter(w => w.length > 0).length;
      vscode.window.showInformationMessage(`Word count: ${words}`);
    }
  });

  // Create status bar item
  statusBarItem = vscode.window.createStatusBarItem(vscode.StatusBarAlignment.Left, 0);
  statusBarItem.command = 'myExtension.wordCount';
  statusBarItem.name = 'Word Count';

  // Update on events
  context.subscriptions.push(
    cmd,
    statusBarItem,
    vscode.window.onDidChangeActiveTextEditor(updateWordCount),
    vscode.workspace.onDidChangeTextDocument(updateWordCount)
  );

  updateWordCount();
}

function updateWordCount() {
  const editor = vscode.window.activeTextEditor;
  if (!editor) {
    statusBarItem.hide();
    return;
  }

  const text = editor.document.getText();
  const words = text.split(/\s+/).filter(w => w.length > 0).length;
  statusBarItem.text = `$(book) ${words} words`;
  statusBarItem.show();
}

export function deactivate() {}
```
