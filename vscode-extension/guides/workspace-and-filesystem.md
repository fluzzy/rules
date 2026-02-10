# Workspace and FileSystem

> **Source**: [VS Code Docs — Workspace API](https://code.visualstudio.com/api/references/vscode-api#workspace) | [Workspace Trust](https://code.visualstudio.com/api/extension-guides/workspace-trust)
> **Archive Date**: 2026-02-10

Guide to working with the VS Code workspace, file system, file watchers, virtual documents, and workspace trust API.

## 1. Workspace Folders

```typescript
import * as vscode from 'vscode';

// Get all workspace folders
const folders = vscode.workspace.workspaceFolders;
if (folders) {
  for (const folder of folders) {
    console.log(`${folder.name}: ${folder.uri.fsPath}`);
  }
}

// Get workspace folder for a file
const fileUri = vscode.Uri.file('/path/to/file.ts');
const folder = vscode.workspace.getWorkspaceFolder(fileUri);

// Watch for workspace folder changes
vscode.workspace.onDidChangeWorkspaceFolders(event => {
  for (const added of event.added) {
    console.log(`Added: ${added.name}`);
  }
  for (const removed of event.removed) {
    console.log(`Removed: ${removed.name}`);
  }
});
```

## 2. FileSystem API (workspace.fs)

`workspace.fs` provides a virtual file system API that works with all file system providers:

```typescript
const uri = vscode.Uri.file('/path/to/file.txt');

// Read file
const content = await vscode.workspace.fs.readFile(uri);
const text = Buffer.from(content).toString('utf-8');

// Write file
const data = Buffer.from('Hello, World!', 'utf-8');
await vscode.workspace.fs.writeFile(uri, data);

// Delete file
await vscode.workspace.fs.delete(uri);

// Create directory
await vscode.workspace.fs.createDirectory(vscode.Uri.file('/path/to/newdir'));

// Read directory
const entries = await vscode.workspace.fs.readDirectory(vscode.Uri.file('/path/to/dir'));
for (const [name, type] of entries) {
  console.log(`${name}: ${type === vscode.FileType.File ? 'file' : 'directory'}`);
}

// Stat (file info)
const stat = await vscode.workspace.fs.stat(uri);
console.log(`Size: ${stat.size}, Modified: ${stat.mtime}`);

// Copy
await vscode.workspace.fs.copy(sourceUri, targetUri, { overwrite: true });

// Rename
await vscode.workspace.fs.rename(oldUri, newUri, { overwrite: false });
```

### FileType Enum

| Value | Description |
|---|---|
| `FileType.Unknown` | Unknown |
| `FileType.File` | Regular file |
| `FileType.Directory` | Directory |
| `FileType.SymbolicLink` | Symbolic link |

## 3. FileSystemWatcher

Watch for file changes in the workspace:

```typescript
// Watch for all TypeScript file changes
const watcher = vscode.workspace.createFileSystemWatcher('**/*.ts');

watcher.onDidCreate(uri => {
  console.log(`Created: ${uri.fsPath}`);
});

watcher.onDidChange(uri => {
  console.log(`Changed: ${uri.fsPath}`);
});

watcher.onDidDelete(uri => {
  console.log(`Deleted: ${uri.fsPath}`);
});

// Don't forget to dispose
context.subscriptions.push(watcher);
```

### Glob Patterns

```typescript
// All files in workspace
vscode.workspace.createFileSystemWatcher('**/*');

// Only JSON files
vscode.workspace.createFileSystemWatcher('**/*.json');

// Files in specific directory
vscode.workspace.createFileSystemWatcher('**/src/**/*.ts');

// Exclude patterns (second/third/fourth parameters)
vscode.workspace.createFileSystemWatcher(
  '**/*.ts',
  false,  // ignoreCreateEvents
  false,  // ignoreChangeEvents
  true    // ignoreDeleteEvents
);
```

## 4. Text Documents

```typescript
// Open a text document
const doc = await vscode.workspace.openTextDocument(uri);
console.log(doc.getText());
console.log(doc.languageId);
console.log(doc.lineCount);

// Open from untitled
const untitled = await vscode.workspace.openTextDocument({
  content: 'Hello',
  language: 'markdown'
});

// Show in editor
await vscode.window.showTextDocument(doc);

// Listen for document events
vscode.workspace.onDidOpenTextDocument(doc => { /* ... */ });
vscode.workspace.onDidCloseTextDocument(doc => { /* ... */ });
vscode.workspace.onDidChangeTextDocument(event => {
  for (const change of event.contentChanges) {
    console.log(`Changed: ${change.text} at ${change.range}`);
  }
});
vscode.workspace.onDidSaveTextDocument(doc => { /* ... */ });
```

## 5. WorkspaceEdit

Apply edits across multiple files:

```typescript
const edit = new vscode.WorkspaceEdit();

// Text edits
edit.insert(uri, new vscode.Position(0, 0), '// Header\n');
edit.replace(uri, range, 'new text');
edit.delete(uri, range);

// File operations
edit.createFile(vscode.Uri.file('/path/to/new.ts'), { overwrite: false });
edit.deleteFile(uri);
edit.renameFile(oldUri, newUri);

// Apply all edits atomically
const success = await vscode.workspace.applyEdit(edit);
```

## 6. Virtual Documents

Provide content for virtual document schemes:

```typescript
// Register a content provider for a custom scheme
const provider = vscode.workspace.registerTextDocumentContentProvider('myscheme', {
  provideTextDocumentContent(uri: vscode.Uri): string {
    // Generate content based on the URI
    return `Content for ${uri.path}`;
  }
});

// Open a virtual document
const uri = vscode.Uri.parse('myscheme:report/summary');
const doc = await vscode.workspace.openTextDocument(uri);
await vscode.window.showTextDocument(doc);
```

## 7. Find Files

```typescript
// Find all TypeScript files
const files = await vscode.workspace.findFiles(
  '**/*.ts',           // include pattern
  '**/node_modules/**'  // exclude pattern
);

// With max results
const limited = await vscode.workspace.findFiles('**/*.json', undefined, 10);
```

## 8. Workspace Trust

```typescript
// Check if workspace is trusted
if (vscode.workspace.isTrusted) {
  enableFullFeatures();
} else {
  enableRestrictedMode();
}

// Listen for trust changes
vscode.workspace.onDidGrantWorkspaceTrust(() => {
  enableFullFeatures();
});
```

### Declare Trust Requirements in package.json

```json
{
  "capabilities": {
    "untrustedWorkspaces": {
      "supported": "limited",
      "description": "Only basic features are available in untrusted workspaces",
      "restrictedConfigurations": [
        "myExtension.executablePath",
        "myExtension.customScript"
      ]
    }
  }
}
```

| Value | Description |
|---|---|
| `true` | Full functionality in untrusted workspaces |
| `false` | Extension disabled in untrusted workspaces |
| `"limited"` | Partial features with restricted configurations |

## 9. Workspace State and Storage

```typescript
// Workspace-scoped state (per workspace)
context.workspaceState.get('key', 'default');
await context.workspaceState.update('key', 'value');

// Global state (across workspaces)
context.globalState.get('key', 'default');
await context.globalState.update('key', 'value');

// Secret storage (encrypted)
await context.secrets.store('apiKey', 'secret-value');
const apiKey = await context.secrets.get('apiKey');
await context.secrets.delete('apiKey');

// Watch for secret changes
context.secrets.onDidChange(e => {
  if (e.key === 'apiKey') {
    refreshAuth();
  }
});
```
