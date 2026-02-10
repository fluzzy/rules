# Extension Anatomy

> **Source**: [VS Code Docs — Extension Anatomy](https://code.visualstudio.com/api/get-started/extension-anatomy)
> **Archive Date**: 2026-02-10

Guide to the structure and lifecycle of a VS Code extension. Covers the extension manifest (package.json), entry point (extension.ts), activation/deactivation, ExtensionContext, and the Disposable pattern.

## 1. Extension File Structure

A generated extension project has this layout:

```text
.
├── .vscode
│   ├── launch.json     // Config for launching and debugging the extension
│   └── tasks.json      // Config for build task that compiles TypeScript
├── .gitignore          // Ignore build output and node_modules
├── README.md           // Readable description of your extension's functionality
├── src
│   └── extension.ts    // Extension source code
├── package.json        // Extension manifest
├── tsconfig.json       // TypeScript configuration
```

## 2. Extension Manifest (package.json)

The `package.json` serves as the extension manifest. Key fields:

| Field | Description |
|---|---|
| `name` | Extension identifier (lowercase, no spaces) |
| `displayName` | Marketplace display name |
| `description` | Short description |
| `version` | SemVer version string |
| `publisher` | Publisher ID from marketplace |
| `engines.vscode` | Minimum VS Code version (e.g. `^1.51.0`) |
| `main` | Entry point path (e.g. `./out/extension.js`) |
| `activationEvents` | Events that trigger activation |
| `contributes` | Declared contribution points |
| `categories` | Marketplace categories |

```json
{
  "name": "helloworld-sample",
  "displayName": "helloworld-sample",
  "description": "HelloWorld example for VS Code",
  "version": "0.0.1",
  "publisher": "vscode-samples",
  "repository": "https://github.com/microsoft/vscode-extension-samples/helloworld-sample",
  "engines": {
    "vscode": "^1.51.0"
  },
  "categories": ["Other"],
  "activationEvents": [],
  "main": "./out/extension.js",
  "contributes": {
    "commands": [
      {
        "command": "helloworld.helloWorld",
        "title": "Hello World"
      }
    ]
  },
  "scripts": {
    "vscode:prepublish": "npm run compile",
    "compile": "tsc -p ./",
    "watch": "tsc -watch -p ./"
  },
  "devDependencies": {
    "@types/node": "^8.10.25",
    "@types/vscode": "^1.51.0",
    "typescript": "^3.4.5"
  }
}
```

## 3. Entry Point (extension.ts)

Every extension exports two functions: `activate` and `deactivate`.

```typescript
import * as vscode from 'vscode';

// Called when the extension is activated
export function activate(context: vscode.ExtensionContext) {
  console.log('Extension "helloworld-sample" is now active!');

  let disposable = vscode.commands.registerCommand('helloworld.helloWorld', () => {
    vscode.window.showInformationMessage('Hello World!');
  });

  context.subscriptions.push(disposable);
}

// Called when the extension is deactivated
export function deactivate() {}
```

### activate(context)

- Called once when an activation event fires
- Receives `ExtensionContext` — provides utilities for the extension lifecycle
- Register commands, providers, and listeners here

### deactivate()

- Called when the extension is unloaded
- Use for cleanup (close connections, release resources)
- Can return a `Promise` for async cleanup

## 4. ExtensionContext

The `ExtensionContext` object provides:

| Property | Description |
|---|---|
| `subscriptions` | Array of `Disposable` — cleaned up on deactivation |
| `workspaceState` | `Memento` for workspace-scoped persistent storage |
| `globalState` | `Memento` for global persistent storage |
| `extensionPath` | Absolute filesystem path of the extension directory |
| `extensionUri` | URI of the extension directory |
| `storagePath` | Path for workspace-specific data |
| `globalStoragePath` | Path for global data |
| `logPath` | Path for log files |
| `secrets` | `SecretStorage` for sensitive data |

## 5. Disposable Pattern

VS Code uses the Disposable pattern extensively. Any resource that needs cleanup implements `Disposable`:

```typescript
export function activate(context: vscode.ExtensionContext) {
  // Each registration returns a Disposable
  const cmd = vscode.commands.registerCommand('ext.cmd', () => { /* ... */ });
  const provider = vscode.languages.registerCompletionItemProvider('javascript', new MyProvider());
  const watcher = vscode.workspace.createFileSystemWatcher('**/*.ts');

  // Push all disposables to subscriptions for automatic cleanup
  context.subscriptions.push(cmd, provider, watcher);
}
```

**Rules:**
- Always push disposables to `context.subscriptions`
- Never let disposables leak — they cause memory leaks and stale listeners
- Call `.dispose()` manually only when removing a resource before deactivation

## 6. TypeScript Configuration

Recommended `tsconfig.json` for extensions:

```json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "ES2020",
    "outDir": "out",
    "lib": ["ES2020"],
    "sourceMap": true,
    "rootDir": "src",
    "strict": true
  },
  "exclude": ["node_modules", ".vscode-test"]
}
```

Key settings:
- `module: "commonjs"` — required for Node.js-based extensions
- `strict: true` — recommended for type safety
- `outDir: "out"` — compiled JS output separate from source
- `exclude` — skip node_modules and test runner files
