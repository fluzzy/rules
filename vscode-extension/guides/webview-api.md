# Webview API

> **Source**: [VS Code Docs — Webview API](https://code.visualstudio.com/api/extension-guides/webview)
> **Archive Date**: 2026-02-10

Guide to creating webviews in VS Code extensions. Covers creating webview panels, loading local resources, message passing between extension and webview, state management, and content security policy.

## 1. Creating a Webview Panel

```typescript
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      const panel = vscode.window.createWebviewPanel(
        'catCoding',        // viewType — unique identifier
        'Cat Coding',       // title shown in tab
        vscode.ViewColumn.One, // editor column
        {
          enableScripts: true,  // allow JavaScript in webview
          retainContextWhenHidden: true  // keep state when tab is hidden
        }
      );

      panel.webview.html = getWebviewContent();
    })
  );
}
```

### createWebviewPanel Parameters

| Parameter | Type | Description |
|---|---|---|
| `viewType` | `string` | Unique identifier for the webview type |
| `title` | `string` | Panel title |
| `showOptions` | `ViewColumn` | Which editor column to show in |
| `options` | `WebviewPanelOptions & WebviewOptions` | Configuration options |

### Options

| Option | Default | Description |
|---|---|---|
| `enableScripts` | `false` | Allow JavaScript execution |
| `enableForms` | `enableScripts` | Allow form submissions |
| `retainContextWhenHidden` | `false` | Keep webview state when hidden |
| `localResourceRoots` | Extension root + workspace | Restrict local resource loading |
| `enableCommandUris` | `false` | Allow command URIs |
| `portMapping` | — | Map localhost ports |

## 2. Loading Local Resources

Use `asWebviewUri` to convert local file paths to webview-safe URIs:

```typescript
function getWebviewContent(webview: vscode.Webview, extensionUri: vscode.Uri) {
  // Get URIs for local resources
  const styleUri = webview.asWebviewUri(
    vscode.Uri.joinPath(extensionUri, 'media', 'style.css')
  );
  const scriptUri = webview.asWebviewUri(
    vscode.Uri.joinPath(extensionUri, 'media', 'main.js')
  );

  return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="Content-Security-Policy"
          content="default-src 'none'; style-src ${webview.cspSource}; script-src ${webview.cspSource};">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <link href="${styleUri}" rel="stylesheet">
    <title>My Webview</title>
</head>
<body>
    <div id="app"></div>
    <script src="${scriptUri}"></script>
</body>
</html>`;
}
```

### Restrict Local Resource Loading

```typescript
const panel = vscode.window.createWebviewPanel('myView', 'My View', vscode.ViewColumn.One, {
  enableScripts: true,
  localResourceRoots: [
    vscode.Uri.joinPath(extensionUri, 'media'),
    vscode.Uri.joinPath(extensionUri, 'dist')
  ]
});
```

## 3. Message Passing

### Extension → Webview

```typescript
// In extension code
currentPanel.webview.postMessage({ command: 'refactor', data: { count: 42 } });
```

```html
<!-- In webview HTML -->
<script>
  window.addEventListener('message', event => {
    const message = event.data;
    switch (message.command) {
      case 'refactor':
        handleRefactor(message.data);
        break;
    }
  });
</script>
```

### Webview → Extension

```html
<!-- In webview HTML -->
<script>
  const vscode = acquireVsCodeApi();

  document.getElementById('btn').addEventListener('click', () => {
    vscode.postMessage({
      command: 'alert',
      text: 'Button clicked!'
    });
  });
</script>
```

```typescript
// In extension code
panel.webview.onDidReceiveMessage(
  message => {
    switch (message.command) {
      case 'alert':
        vscode.window.showErrorMessage(message.text);
        return;
    }
  },
  undefined,
  context.subscriptions
);
```

## 4. State Management

### setState / getState

Webview state persists across visibility changes (but not across VS Code restarts):

```html
<script>
  const vscode = acquireVsCodeApi();

  // Restore previous state
  const previousState = vscode.getState();
  let count = previousState ? previousState.count : 0;

  // Update state
  function updateCount(newCount) {
    count = newCount;
    vscode.setState({ count });
    document.getElementById('counter').textContent = count;
  }
</script>
```

### Serialization (Persistence Across Restarts)

Register a `WebviewPanelSerializer` to restore webviews after VS Code restart:

```typescript
export function activate(context: vscode.ExtensionContext) {
  // Register serializer to restore webviews
  vscode.window.registerWebviewPanelSerializer('catCoding', {
    async deserializeWebviewPanel(webviewPanel: vscode.WebviewPanel, state: any) {
      // Restore the webview content
      webviewPanel.webview.options = { enableScripts: true };
      webviewPanel.webview.html = getWebviewContent(webviewPanel.webview, context.extensionUri);
    }
  });
}
```

## 5. Content Security Policy

Always set a CSP to restrict what content can be loaded:

```html
<meta http-equiv="Content-Security-Policy"
      content="default-src 'none';
               img-src ${webview.cspSource} https:;
               script-src ${webview.cspSource};
               style-src ${webview.cspSource} 'unsafe-inline';
               font-src ${webview.cspSource};">
```

| Directive | Recommended Value | Description |
|---|---|---|
| `default-src` | `'none'` | Block everything by default |
| `img-src` | `${webview.cspSource} https:` | Local + HTTPS images |
| `script-src` | `${webview.cspSource}` | Only local scripts |
| `style-src` | `${webview.cspSource}` | Only local styles |
| `font-src` | `${webview.cspSource}` | Only local fonts |

## 6. Webview Lifecycle Events

```typescript
// Panel becomes visible/hidden
panel.onDidChangeViewState(e => {
  const isVisible = e.webviewPanel.visible;
  const isActive = e.webviewPanel.active;
});

// Panel is disposed (closed)
panel.onDidDispose(() => {
  // Clean up resources
}, null, context.subscriptions);
```

## 7. Complete Example

```typescript
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  let currentPanel: vscode.WebviewPanel | undefined = undefined;

  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.start', () => {
      if (currentPanel) {
        currentPanel.reveal(vscode.ViewColumn.One);
      } else {
        currentPanel = vscode.window.createWebviewPanel(
          'catCoding',
          'Cat Coding',
          vscode.ViewColumn.One,
          { enableScripts: true }
        );

        currentPanel.webview.html = getWebviewContent(currentPanel.webview, context.extensionUri);

        currentPanel.webview.onDidReceiveMessage(
          message => {
            switch (message.command) {
              case 'alert':
                vscode.window.showErrorMessage(message.text);
                return;
            }
          },
          undefined,
          context.subscriptions
        );

        currentPanel.onDidDispose(() => {
          currentPanel = undefined;
        }, null, context.subscriptions);
      }
    })
  );

  // Send message to webview
  context.subscriptions.push(
    vscode.commands.registerCommand('catCoding.doRefactor', () => {
      if (currentPanel) {
        currentPanel.webview.postMessage({ command: 'refactor' });
      }
    })
  );
}
```
