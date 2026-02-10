# Webview Security Rules

> **Source**: [VS Code Docs — Webview API (Security)](https://code.visualstudio.com/api/extension-guides/webview#security)
> **Archive Date**: 2026-02-10

Security rules for VS Code extension webviews. Covers Content Security Policy, resource loading, message handling, and input sanitization.

## Rules

You are an expert in VS Code extension development with deep knowledge of webview security best practices.

**Content Security Policy**
- Always set a Content Security Policy (CSP) meta tag in every webview HTML. Start with `default-src 'none'` and whitelist only what is needed.
- Use `${webview.cspSource}` for script-src, style-src, img-src, and font-src. Never use `'unsafe-inline'` for script-src.
- Never use `'unsafe-eval'` in any CSP directive. This blocks `eval()`, `new Function()`, and inline event handlers.
- Add a nonce to each inline script if inline scripts are absolutely necessary: `script-src 'nonce-${nonce}'`.

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'none';
               img-src ${webview.cspSource} https:;
               script-src ${webview.cspSource};
               style-src ${webview.cspSource};
               font-src ${webview.cspSource};">
```

**Resource Loading**
- Always use `webview.asWebviewUri()` to convert local file URIs. Never construct webview URIs manually.
- Set `localResourceRoots` to restrict which directories the webview can load resources from. Default is the extension root and workspace folders — narrow it to only the `media/` or `dist/` directory.
- Never load scripts or stylesheets from external CDNs unless absolutely necessary. Bundle all dependencies locally.

```typescript
const panel = vscode.window.createWebviewPanel('myView', 'Title', vscode.ViewColumn.One, {
  enableScripts: true,
  localResourceRoots: [vscode.Uri.joinPath(extensionUri, 'media')]
});
```

**JavaScript Execution**
- Only set `enableScripts: true` when the webview genuinely needs JavaScript. Static content webviews should leave it `false`.
- Never use `innerHTML` with unsanitized data. Use `textContent` for plain text or create elements programmatically.
- Never use `eval()`, `new Function()`, or `document.write()` in webview scripts.
- Never use inline event handlers (`onclick`, `onload`) in HTML. Use `addEventListener` in scripts.

**Message Passing**
- Always validate and type-check incoming messages in both the extension and webview. Use a `command` or `type` field as a discriminator.
- Never trust data from webview messages without validation. Treat webview messages like external user input.
- Use a well-defined message protocol with TypeScript interfaces for all extension ↔ webview communication.

```typescript
// Define message types
interface WebviewMessage {
  command: 'save' | 'delete' | 'refresh';
  payload: unknown;
}

// Validate in handler
panel.webview.onDidReceiveMessage((message: WebviewMessage) => {
  switch (message.command) {
    case 'save':
      // validate payload before use
      break;
    default:
      console.warn('Unknown command:', message.command);
  }
});
```

**State and Storage**
- Never store secrets, tokens, or passwords in webview state (`setState`/`getState`). Use `context.secrets` (SecretStorage) in the extension host.
- Sanitize all user-provided data before rendering in the webview DOM.
