# Extension Performance Rules

> **Source**: [VS Code Docs — Bundling Extensions](https://code.visualstudio.com/api/working-with-extensions/bundling-extension) | [Extension Anatomy](https://code.visualstudio.com/api/get-started/extension-anatomy)
> **Archive Date**: 2026-02-10

Performance rules for VS Code extension development. Covers activation strategy, bundling, resource management, and runtime optimization.

## Rules

You are an expert in VS Code extension performance optimization with deep knowledge of the Extension Host architecture.

**Activation Strategy**
- Never use `"activationEvents": ["*"]`. Always use the most specific activation event possible (`onCommand`, `onLanguage`, `onView`, etc.).
- Prefer implicit activation (VS Code 1.74+) — let contribution points auto-generate activation events instead of declaring them manually.
- Use `onStartupFinished` instead of `*` for extensions that need early but non-blocking activation.
- Defer expensive initialization. Do not run file I/O, network calls, or heavy computation synchronously in `activate()`.

**Bundling**
- Always bundle extensions with esbuild or webpack before publishing. Unbundled extensions with hundreds of node_modules files activate significantly slower.
- Set `vscode` as an external dependency in the bundler config — it is provided by the runtime.
- Use production mode (`minify: true` / `mode: 'production'`) for published builds.
- Generate source maps only for development builds. Use `hidden-source-map` for production if error reporting is needed.

**Resource Management**
- Always push disposables to `context.subscriptions`. Leaked disposables cause memory leaks and stale event handlers.
- Dispose of file watchers, decorations, status bar items, and providers when no longer needed.
- Call `.dispose()` explicitly when removing resources before extension deactivation.

```typescript
// GOOD — disposables tracked
const watcher = vscode.workspace.createFileSystemWatcher('**/*.ts');
context.subscriptions.push(watcher);

// BAD — disposable leaked
vscode.workspace.createFileSystemWatcher('**/*.ts');
```

**Event Handling**
- Debounce high-frequency events (`onDidChangeTextDocument`, `onDidChangeSelection`) to avoid excessive processing. Use a 100-300ms debounce interval.
- Check `event.document` against the active editor before processing document change events — avoid processing changes to irrelevant documents.

```typescript
let debounceTimer: NodeJS.Timeout;
vscode.workspace.onDidChangeTextDocument(event => {
  clearTimeout(debounceTimer);
  debounceTimer = setTimeout(() => {
    processDocument(event.document);
  }, 200);
});
```

**Document Selector**
- Use narrow `DocumentSelector` definitions. Prefer `{ language: 'python', scheme: 'file' }` over just `'python'` — this avoids activating for untitled, virtual, or git documents.

**Package Size**
- Use `.vscodeignore` to exclude source files, test files, documentation, and build configs from the .vsix package.
- Never include `node_modules` in the published extension when using a bundler. Only include it for native dependencies.
- Keep the .vsix under 5MB whenever possible. Large extensions slow down installation and updates.

**Caching**
- Cache expensive computations (file parsing, symbol resolution) and invalidate on relevant file change events.
- Use `workspaceState` or `globalState` for persistent caches that survive extension deactivation.

**Async Operations**
- Use `async/await` consistently. Never block the Extension Host with synchronous file I/O or long-running computation.
- Use `vscode.window.withProgress` for long operations to keep the UI responsive and let users cancel.
