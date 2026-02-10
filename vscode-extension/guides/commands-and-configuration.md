# Commands and Configuration

> **Source**: [VS Code Docs — Extension Guides](https://code.visualstudio.com/api/extension-guides/command) | [Configuration](https://code.visualstudio.com/api/references/contribution-points#contributes.configuration)
> **Archive Date**: 2026-02-10

Guide to registering commands, executing built-in commands, and working with VS Code's configuration system. Covers registerCommand, executeCommand, configuration schema, getConfiguration read/write/watch patterns.

## 1. Registering Commands

Commands are the primary way users interact with extensions. Register in `activate()`:

```typescript
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  // Simple command
  const disposable = vscode.commands.registerCommand('myExtension.sayHello', () => {
    vscode.window.showInformationMessage('Hello!');
  });

  // Command with arguments
  const withArgs = vscode.commands.registerCommand('myExtension.openFile', (uri: vscode.Uri) => {
    vscode.window.showTextDocument(uri);
  });

  // Command returning a value
  const withReturn = vscode.commands.registerCommand('myExtension.getCount', () => {
    return 42;
  });

  context.subscriptions.push(disposable, withArgs, withReturn);
}
```

Commands must also be declared in `package.json`:

```json
{
  "contributes": {
    "commands": [
      {
        "command": "myExtension.sayHello",
        "title": "Say Hello",
        "category": "My Extension"
      }
    ]
  }
}
```

## 2. Executing Commands

Execute any command (built-in or extension-provided) programmatically:

```typescript
// Execute without arguments
await vscode.commands.executeCommand('editor.action.formatDocument');

// Execute with arguments
await vscode.commands.executeCommand('vscode.openFolder', uri);

// Execute and capture return value
const result = await vscode.commands.executeCommand<number>('myExtension.getCount');
```

### Useful Built-in Commands

| Command | Description |
|---|---|
| `vscode.open` | Open a URI in the editor |
| `vscode.openFolder` | Open a folder in a new window |
| `vscode.diff` | Open a diff editor |
| `editor.action.formatDocument` | Format the active document |
| `workbench.action.openSettings` | Open Settings UI |
| `workbench.action.showCommands` | Open Command Palette |
| `setContext` | Set a custom context key |

### Context Keys

Set context keys to control `when` clauses:

```typescript
// Set a context key
await vscode.commands.executeCommand('setContext', 'myExtension.isActive', true);

// Use in package.json when clauses
// "when": "myExtension.isActive"
```

## 3. TextEditorCommand

For commands that operate on the active text editor:

```typescript
const editorCmd = vscode.commands.registerTextEditorCommand(
  'myExtension.insertDate',
  (textEditor, edit) => {
    const date = new Date().toISOString().split('T')[0];
    edit.insert(textEditor.selection.active, date);
  }
);
context.subscriptions.push(editorCmd);
```

`registerTextEditorCommand` guarantees the callback runs only when a text editor is active, and passes both the editor and an edit builder.

## 4. Configuration Schema

Declare settings in `package.json`:

```json
{
  "contributes": {
    "configuration": {
      "title": "My Extension",
      "properties": {
        "myExtension.enable": {
          "type": "boolean",
          "default": true,
          "description": "Enable the extension",
          "scope": "resource"
        },
        "myExtension.maxItems": {
          "type": "number",
          "default": 100,
          "minimum": 1,
          "maximum": 1000,
          "description": "Maximum items to display"
        },
        "myExtension.exclude": {
          "type": "array",
          "items": { "type": "string" },
          "default": ["node_modules"],
          "description": "Glob patterns to exclude"
        },
        "myExtension.outputFormat": {
          "type": "string",
          "enum": ["json", "csv", "table"],
          "default": "json",
          "enumDescriptions": [
            "JSON format",
            "Comma-separated values",
            "ASCII table"
          ]
        }
      }
    }
  }
}
```

### Configuration Scopes

| Scope | Description |
|---|---|
| `application` | Machine-specific, not synced |
| `machine` | Machine-specific, not synced (same as application) |
| `machine-overridable` | Machine-specific but can be overridden in workspace |
| `window` | Window-specific (default) |
| `resource` | Per-resource (file/folder level) |
| `language-overridable` | Can be overridden per language |

## 5. Reading Configuration

```typescript
// Get the full configuration section
const config = vscode.workspace.getConfiguration('myExtension');

// Read values with defaults
const enabled: boolean = config.get('enable', true);
const maxItems: number = config.get('maxItems', 100);
const exclude: string[] = config.get('exclude', []);

// Inspect a value to see where it comes from
const inspection = config.inspect('enable');
// inspection.defaultValue, inspection.globalValue, inspection.workspaceValue, inspection.workspaceFolderValue
```

## 6. Writing Configuration

```typescript
const config = vscode.workspace.getConfiguration('myExtension');

// Update globally (User settings)
await config.update('enable', false, vscode.ConfigurationTarget.Global);

// Update for workspace
await config.update('maxItems', 200, vscode.ConfigurationTarget.Workspace);

// Update for workspace folder
await config.update('maxItems', 50, vscode.ConfigurationTarget.WorkspaceFolder);

// Remove a setting (reset to default)
await config.update('enable', undefined, vscode.ConfigurationTarget.Global);
```

## 7. Watching Configuration Changes

```typescript
vscode.workspace.onDidChangeConfiguration(event => {
  if (event.affectsConfiguration('myExtension.enable')) {
    const config = vscode.workspace.getConfiguration('myExtension');
    const enabled = config.get<boolean>('enable');
    if (enabled) {
      activateFeature();
    } else {
      deactivateFeature();
    }
  }

  if (event.affectsConfiguration('myExtension.maxItems')) {
    refreshData();
  }
});
```

## 8. Complete Example

```typescript
import * as vscode from 'vscode';

export function activate(context: vscode.ExtensionContext) {
  // Read initial config
  let config = vscode.workspace.getConfiguration('myExtension');
  let maxItems = config.get<number>('maxItems', 100);

  // Register command that uses config
  const cmd = vscode.commands.registerCommand('myExtension.listItems', async () => {
    const items = await fetchItems(maxItems);
    const pick = await vscode.window.showQuickPick(items, {
      placeHolder: `Showing up to ${maxItems} items`
    });
    if (pick) {
      vscode.window.showInformationMessage(`Selected: ${pick}`);
    }
  });

  // Watch for config changes
  const configWatcher = vscode.workspace.onDidChangeConfiguration(e => {
    if (e.affectsConfiguration('myExtension.maxItems')) {
      config = vscode.workspace.getConfiguration('myExtension');
      maxItems = config.get<number>('maxItems', 100);
    }
  });

  context.subscriptions.push(cmd, configWatcher);
}
```
