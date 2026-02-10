# Web Extensions and Advanced Topics

> **Source**: [VS Code Docs — Web Extensions](https://code.visualstudio.com/api/extension-guides/web-extensions)
> **Archive Date**: 2026-02-10

Guide to building VS Code web extensions, understanding the Extension Host architecture, Remote Development, and proposed APIs.

## 1. Web Extensions Overview

Web extensions run in a browser environment (vscode.dev, github.dev) without access to Node.js APIs. They use a `browser` entry point instead of `main`.

```json
{
  "main": "./dist/extension.js",
  "browser": "./dist/web/extension.js"
}
```

## 2. Package.json Configuration

```json
{
  "name": "my-web-extension",
  "main": "./dist/node/extension.js",
  "browser": "./dist/web/extension.js",
  "engines": {
    "vscode": "^1.74.0"
  },
  "extensionKind": ["ui", "workspace"]
}
```

### extensionKind

| Value | Description |
|---|---|
| `["ui"]` | Runs in UI (local) Extension Host only |
| `["workspace"]` | Runs in workspace (remote/web) Extension Host |
| `["ui", "workspace"]` | Runs in UI first, falls back to workspace |
| `["workspace", "ui"]` | Runs in workspace first, falls back to UI |

## 3. Node.js API Restrictions

Web extensions cannot use:

| Unavailable | Alternative |
|---|---|
| `fs` module | `vscode.workspace.fs` |
| `path` module | `vscode.Uri.joinPath` or URI utilities |
| `child_process` | Not available — use web workers |
| `os` module | Not available |
| `net` / `http` | `fetch` API |
| `crypto` (Node) | `globalThis.crypto` |
| Native addons | Not available |

## 4. Bundling for Web

Use esbuild or webpack with browser target:

### esbuild

```javascript
await esbuild.build({
  entryPoints: ['src/web/extension.ts'],
  bundle: true,
  format: 'cjs',
  platform: 'browser',
  outfile: 'dist/web/extension.js',
  external: ['vscode']
});
```

### webpack

```javascript
/** @type {import('webpack').Configuration} */
const webExtensionConfig = {
  target: 'webworker',
  entry: './src/web/extension.ts',
  output: {
    filename: 'extension.js',
    path: path.resolve(__dirname, 'dist', 'web'),
    libraryTarget: 'commonjs'
  },
  externals: {
    vscode: 'commonjs vscode'
  },
  resolve: {
    extensions: ['.ts', '.js'],
    fallback: {
      path: require.resolve('path-browserify'),
      os: false,
      fs: false
    }
  },
  module: {
    rules: [{ test: /\.ts$/, use: 'ts-loader', exclude: /node_modules/ }]
  }
};
```

## 5. Testing Web Extensions

### Launch Configuration

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Web Extension",
      "type": "extensionHost",
      "debugWebWorkerHost": true,
      "request": "launch",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}",
        "--extensionDevelopmentKind=web"
      ]
    },
    {
      "name": "Test Web Extension",
      "type": "extensionHost",
      "debugWebWorkerHost": true,
      "request": "launch",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}",
        "--extensionTestsPath=${workspaceFolder}/dist/web/test/suite/index",
        "--extensionDevelopmentKind=web"
      ]
    }
  ]
}
```

### Test with @vscode/test-web

```bash
npm install --save-dev @vscode/test-web
npx @vscode/test-web --extensionDevelopmentPath=. --extensionTestsPath=dist/web/test/suite/index.js
```

## 6. Extension Host Architecture

VS Code runs extensions in separate processes called Extension Hosts:

| Host | Environment | Extensions |
|---|---|---|
| Local Extension Host | Node.js | `extensionKind: ["ui"]` |
| Remote Extension Host | Node.js (remote) | `extensionKind: ["workspace"]` |
| Web Extension Host | Web Worker | `browser` entry point |

### Remote Development

When connecting to a remote target (SSH, Container, WSL):
- UI extensions run locally
- Workspace extensions run on the remote
- Extensions must declare `extensionKind` to control placement

## 7. Proposed APIs

Proposed APIs are experimental and not yet stable:

### Enable in Development

```json
{
  "enabledApiProposals": ["fileSearchProvider", "textSearchProvider"]
}
```

### Launch Configuration

```json
{
  "args": [
    "--enable-proposed-api=publisher.extensionName"
  ]
}
```

**Important:** Extensions using proposed APIs cannot be published to the Marketplace.

## 8. Virtual Workspaces

Support for virtual file systems (e.g., GitHub Repositories):

```json
{
  "capabilities": {
    "virtualWorkspaces": {
      "supported": "limited",
      "description": "Only syntax highlighting works in virtual workspaces"
    }
  }
}
```

| Value | Description |
|---|---|
| `true` | Full support for virtual workspaces |
| `false` | Extension disabled in virtual workspaces |
| `"limited"` | Partial support with description |

## 9. Extension Bisect

VS Code includes a built-in extension bisect tool for debugging extension conflicts:

1. Open Command Palette → "Start Extension Bisect"
2. VS Code systematically enables/disables extensions
3. Identifies the problematic extension

## 10. Extension API Compatibility

Check API version availability:

```typescript
// Check if an API exists before using it
if (typeof vscode.window.createWebviewPanel === 'function') {
  // API is available
}

// Use when clause to check engine version
// "when": "vscode.version >= 1.80.0"
```
