# TreeView API

> **Source**: [VS Code Docs — Tree View API](https://code.visualstudio.com/api/extension-guides/tree-view) | [Tree View Sample](https://github.com/microsoft/vscode-extension-samples/tree/main/tree-view-sample)
> **Archive Date**: 2026-02-10

Guide to creating custom tree views in the VS Code sidebar. Covers TreeDataProvider, refresh events, view containers, drag and drop, and contextValue for conditional menus.

## 1. Overview

Tree views display hierarchical data in the Side Bar or Panel. They require:
1. A `TreeDataProvider` that supplies the data
2. A `views` contribution in `package.json`
3. Registration via `vscode.window.registerTreeDataProvider` or `createTreeView`

## 2. TreeDataProvider Interface

```typescript
import * as vscode from 'vscode';

class DependencyItem extends vscode.TreeItem {
  constructor(
    public readonly label: string,
    public readonly version: string,
    public readonly collapsibleState: vscode.TreeItemCollapsibleState
  ) {
    super(label, collapsibleState);
    this.tooltip = `${this.label} v${this.version}`;
    this.description = this.version;
  }
}

class DependencyProvider implements vscode.TreeDataProvider<DependencyItem> {
  // Event for refreshing the tree
  private _onDidChangeTreeData = new vscode.EventEmitter<DependencyItem | undefined | null | void>();
  readonly onDidChangeTreeData = this._onDidChangeTreeData.event;

  getTreeItem(element: DependencyItem): vscode.TreeItem {
    return element;
  }

  getChildren(element?: DependencyItem): Thenable<DependencyItem[]> {
    if (element) {
      // Return children of this node
      return Promise.resolve(this.getSubDeps(element));
    } else {
      // Return root nodes
      return Promise.resolve(this.getRootDeps());
    }
  }

  refresh(): void {
    this._onDidChangeTreeData.fire();
  }

  private getRootDeps(): DependencyItem[] {
    return [
      new DependencyItem('express', '4.18.2', vscode.TreeItemCollapsibleState.Collapsed),
      new DependencyItem('typescript', '5.3.2', vscode.TreeItemCollapsibleState.None)
    ];
  }

  private getSubDeps(item: DependencyItem): DependencyItem[] {
    if (item.label === 'express') {
      return [
        new DependencyItem('body-parser', '1.20.0', vscode.TreeItemCollapsibleState.None),
        new DependencyItem('cookie', '0.5.0', vscode.TreeItemCollapsibleState.None)
      ];
    }
    return [];
  }
}
```

## 3. Registration

### Option A: registerTreeDataProvider

```typescript
export function activate(context: vscode.ExtensionContext) {
  const provider = new DependencyProvider();
  vscode.window.registerTreeDataProvider('nodeDependencies', provider);

  // Register refresh command
  vscode.commands.registerCommand('nodeDependencies.refreshEntry', () => provider.refresh());
}
```

### Option B: createTreeView (more control)

```typescript
export function activate(context: vscode.ExtensionContext) {
  const provider = new DependencyProvider();
  const treeView = vscode.window.createTreeView('nodeDependencies', {
    treeDataProvider: provider,
    showCollapseAll: true,
    canSelectMany: true
  });

  context.subscriptions.push(treeView);

  // Access treeView.selection, treeView.visible, etc.
  treeView.onDidChangeSelection(e => {
    console.log('Selected:', e.selection);
  });
}
```

## 4. Package.json Configuration

### View Container (Activity Bar)

```json
{
  "contributes": {
    "viewsContainers": {
      "activitybar": [
        {
          "id": "package-explorer",
          "title": "Package Explorer",
          "icon": "resources/package-explorer.svg"
        }
      ]
    },
    "views": {
      "package-explorer": [
        {
          "id": "nodeDependencies",
          "name": "Dependencies",
          "icon": "resources/dep.svg",
          "contextualTitle": "Package Explorer"
        }
      ]
    }
  }
}
```

### View in Existing Container

```json
{
  "contributes": {
    "views": {
      "explorer": [
        {
          "id": "nodeDependencies",
          "name": "Node Dependencies"
        }
      ]
    }
  }
}
```

Built-in containers: `explorer`, `scm`, `debug`, `test`.

## 5. Menus and Context Actions

```json
{
  "contributes": {
    "commands": [
      {
        "command": "nodeDependencies.refreshEntry",
        "title": "Refresh",
        "icon": "$(refresh)"
      },
      {
        "command": "nodeDependencies.editEntry",
        "title": "Edit",
        "icon": "$(edit)"
      },
      {
        "command": "nodeDependencies.deleteEntry",
        "title": "Delete"
      }
    ],
    "menus": {
      "view/title": [
        {
          "command": "nodeDependencies.refreshEntry",
          "when": "view == nodeDependencies",
          "group": "navigation"
        }
      ],
      "view/item/context": [
        {
          "command": "nodeDependencies.editEntry",
          "when": "view == nodeDependencies && viewItem == dependency",
          "group": "inline"
        },
        {
          "command": "nodeDependencies.deleteEntry",
          "when": "view == nodeDependencies && viewItem == dependency"
        }
      ]
    }
  }
}
```

## 6. contextValue

Use `contextValue` on TreeItem to control which menu items appear:

```typescript
class DependencyItem extends vscode.TreeItem {
  constructor(label: string, version: string, collapsibleState: vscode.TreeItemCollapsibleState) {
    super(label, collapsibleState);
    // Controls which context menu items appear
    this.contextValue = 'dependency';
  }
}
```

Then in `when` clauses: `"when": "viewItem == dependency"`

## 7. Welcome Content

Show content when the view is empty:

```json
{
  "contributes": {
    "viewsWelcome": [
      {
        "view": "nodeDependencies",
        "contents": "No dependencies found.\n[Add Dependency](command:nodeDependencies.addEntry)"
      }
    ]
  }
}
```

## 8. TreeItem Properties

| Property | Type | Description |
|---|---|---|
| `label` | `string \| TreeItemLabel` | Display text |
| `description` | `string` | Secondary text (dimmed) |
| `tooltip` | `string \| MarkdownString` | Hover tooltip |
| `iconPath` | `Uri \| ThemeIcon` | Item icon |
| `collapsibleState` | `TreeItemCollapsibleState` | None, Collapsed, or Expanded |
| `contextValue` | `string` | Controls context menu visibility |
| `command` | `Command` | Command to run on click |
| `resourceUri` | `Uri` | URI for file-based decorations |
| `checkboxState` | `TreeItemCheckboxState` | Checkbox state |
