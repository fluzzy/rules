# Custom Editor API

> **Source**: [VS Code Docs — Custom Editors](https://code.visualstudio.com/api/extension-guides/custom-editors) | [Custom Editor Sample](https://github.com/microsoft/vscode-extension-samples/tree/main/custom-editor-sample)
> **Archive Date**: 2026-02-10

Guide to creating custom editors in VS Code. Covers CustomTextEditorProvider for text files and CustomReadonlyEditorProvider/CustomEditorProvider for binary files with full edit/undo/save support.

## 1. Custom Editor Types

| Type | Use Case | Base Class |
|---|---|---|
| `CustomTextEditorProvider` | Text-based files (JSON, XML, etc.) | Uses VS Code's `TextDocument` for edit/save |
| `CustomReadonlyEditorProvider` | Read-only binary viewers | No edit support |
| `CustomEditorProvider` | Editable binary files (images, hex, etc.) | Custom `CustomDocument` for edit tracking |

## 2. Package.json Declaration

```json
{
  "contributes": {
    "customEditors": [
      {
        "viewType": "myExtension.imagePreview",
        "displayName": "Image Preview",
        "selector": [
          { "filenamePattern": "*.{png,jpg,gif,bmp,svg}" }
        ],
        "priority": "default"
      }
    ]
  }
}
```

| Field | Description |
|---|---|
| `viewType` | Unique identifier matching the registration |
| `displayName` | Name shown in editor picker |
| `selector` | File patterns this editor handles |
| `priority` | `default` (opens by default) or `option` (user must choose) |

## 3. CustomTextEditorProvider

For custom editors backed by text files:

```typescript
import * as vscode from 'vscode';

class JsonEditorProvider implements vscode.CustomTextEditorProvider {
  public static readonly viewType = 'myExtension.jsonEditor';

  public static register(context: vscode.ExtensionContext): vscode.Disposable {
    return vscode.window.registerCustomEditorProvider(
      JsonEditorProvider.viewType,
      new JsonEditorProvider(context),
      { supportsMultipleEditorsPerDocument: false }
    );
  }

  constructor(private readonly context: vscode.ExtensionContext) {}

  async resolveCustomTextEditor(
    document: vscode.TextDocument,
    webviewPanel: vscode.WebviewPanel,
    _token: vscode.CancellationToken
  ): Promise<void> {
    webviewPanel.webview.options = { enableScripts: true };
    webviewPanel.webview.html = this.getHtmlForWebview(webviewPanel.webview);

    // Send document content to webview
    function updateWebview() {
      webviewPanel.webview.postMessage({
        type: 'update',
        text: document.getText()
      });
    }

    // Watch for text document changes
    const changeDocumentSubscription = vscode.workspace.onDidChangeTextDocument(e => {
      if (e.document.uri.toString() === document.uri.toString()) {
        updateWebview();
      }
    });

    webviewPanel.onDidDispose(() => changeDocumentSubscription.dispose());

    // Handle messages from webview
    webviewPanel.webview.onDidReceiveMessage(message => {
      switch (message.type) {
        case 'edit':
          this.applyEdit(document, message.data);
          return;
      }
    });

    updateWebview();
  }

  private applyEdit(document: vscode.TextDocument, newContent: string) {
    const edit = new vscode.WorkspaceEdit();
    edit.replace(
      document.uri,
      new vscode.Range(0, 0, document.lineCount, 0),
      newContent
    );
    return vscode.workspace.applyEdit(edit);
  }

  private getHtmlForWebview(webview: vscode.Webview): string {
    return `<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="Content-Security-Policy"
          content="default-src 'none'; script-src ${webview.cspSource}; style-src ${webview.cspSource};">
    <title>JSON Editor</title>
</head>
<body>
    <div id="editor"></div>
    <script>
      const vscode = acquireVsCodeApi();
      window.addEventListener('message', event => {
        const message = event.data;
        if (message.type === 'update') {
          // Update editor with document content
          document.getElementById('editor').textContent = message.text;
        }
      });
    </script>
</body>
</html>`;
  }
}

// In activate()
export function activate(context: vscode.ExtensionContext) {
  context.subscriptions.push(JsonEditorProvider.register(context));
}
```

## 4. CustomEditorProvider (Binary Files)

For binary files with full edit support:

```typescript
class ImageDocument implements vscode.CustomDocument {
  uri: vscode.Uri;
  private _edits: ImageEdit[] = [];
  private _savedEdits: ImageEdit[] = [];

  constructor(uri: vscode.Uri) {
    this.uri = uri;
  }

  dispose(): void {
    // Clean up resources
  }
}

class ImageEditorProvider implements vscode.CustomEditorProvider<ImageDocument> {
  private static readonly viewType = 'myExtension.imageEditor';

  // Fires when edits happen
  private readonly _onDidChangeCustomDocument = new vscode.EventEmitter<
    vscode.CustomDocumentEditEvent<ImageDocument>
  >();
  readonly onDidChangeCustomDocument = this._onDidChangeCustomDocument.event;

  async openCustomDocument(
    uri: vscode.Uri,
    openContext: vscode.CustomDocumentOpenContext,
    token: vscode.CancellationToken
  ): Promise<ImageDocument> {
    const document = new ImageDocument(uri);
    // Load from backup if available
    if (openContext.backupId) {
      // Restore from backup
    }
    return document;
  }

  async resolveCustomEditor(
    document: ImageDocument,
    webviewPanel: vscode.WebviewPanel,
    token: vscode.CancellationToken
  ): Promise<void> {
    webviewPanel.webview.options = { enableScripts: true };
    webviewPanel.webview.html = this.getHtmlForWebview(webviewPanel.webview);

    // Handle edits from webview
    webviewPanel.webview.onDidReceiveMessage(message => {
      if (message.type === 'edit') {
        // Fire edit event — this integrates with VS Code's undo/redo
        this._onDidChangeCustomDocument.fire({
          document,
          undo: async () => { /* reverse the edit */ },
          redo: async () => { /* re-apply the edit */ }
        });
      }
    });
  }

  async saveCustomDocument(document: ImageDocument, cancellation: vscode.CancellationToken): Promise<void> {
    // Save document to disk
  }

  async saveCustomDocumentAs(document: ImageDocument, destination: vscode.Uri, cancellation: vscode.CancellationToken): Promise<void> {
    // Save document to new location
  }

  async revertCustomDocument(document: ImageDocument, cancellation: vscode.CancellationToken): Promise<void> {
    // Revert document to last saved state
  }

  async backupCustomDocument(document: ImageDocument, context: vscode.CustomDocumentBackupContext, cancellation: vscode.CancellationToken): Promise<vscode.CustomDocumentBackup> {
    // Create backup for hot exit
    return {
      id: context.destination.toString(),
      delete: () => { /* delete backup */ }
    };
  }

  private getHtmlForWebview(webview: vscode.Webview): string {
    return '<!DOCTYPE html>...';
  }
}
```

## 5. Undo/Redo Integration

For `CustomEditorProvider`, edits integrate with VS Code's undo/redo stack via `onDidChangeCustomDocument`:

```typescript
this._onDidChangeCustomDocument.fire({
  document,
  label: 'Crop Image',  // shown in Edit menu
  undo: async () => {
    // Reverse the edit
    document.undoLastEdit();
    updateWebview();
  },
  redo: async () => {
    // Re-apply the edit
    document.redoLastEdit();
    updateWebview();
  }
});
```

## 6. Activation Event

Custom editors auto-generate activation events, but you can also declare explicitly:

```json
{
  "activationEvents": [
    "onCustomEditor:myExtension.imagePreview"
  ]
}
```
