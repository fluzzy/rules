# Contribution Points

> **Source**: [VS Code Docs — Contribution Points](https://code.visualstudio.com/api/references/contribution-points)
> **Archive Date**: 2026-02-10

Complete reference for VS Code extension contribution points. Contribution points are static declarations in `package.json` that extend VS Code's UI and behavior.

## 1. Overview

Contribution points are declared under the `contributes` field in `package.json`. They describe what your extension adds to VS Code — commands, settings, views, languages, and more.

## 2. Contribution Points Reference

| Contribution Point | Description |
|---|---|
| `commands` | Commands shown in Command Palette |
| `menus` | Menu items in editor, explorer, etc. |
| `keybindings` | Default keyboard shortcuts |
| `configuration` | Settings exposed in Settings UI |
| `configurationDefaults` | Default configuration values |
| `views` | Tree views in sidebar/panel |
| `viewsContainers` | Custom view containers in Activity Bar/Panel |
| `viewsWelcome` | Welcome content for empty views |
| `languages` | Language declarations |
| `grammars` | TextMate grammars for syntax highlighting |
| `themes` | Color themes |
| `iconThemes` | File icon themes |
| `productIconThemes` | Product icon themes |
| `snippets` | Code snippets |
| `jsonValidation` | JSON schema validation |
| `taskDefinitions` | Task types |
| `problemMatchers` | Problem matchers for tasks |
| `problemPatterns` | Problem patterns |
| `debuggers` | Debugger contributions |
| `breakpoints` | Breakpoint support for languages |
| `customEditors` | Custom editor types |
| `notebooks` | Notebook types |
| `notebookRenderer` | Notebook output renderers |
| `terminal` | Terminal profiles |
| `walkthroughs` | Getting Started walkthroughs |

## 3. Commands

Register commands that appear in the Command Palette:

```json
{
  "contributes": {
    "commands": [
      {
        "command": "myExtension.sayHello",
        "title": "Say Hello",
        "category": "My Extension",
        "icon": {
          "light": "resources/light/hello.svg",
          "dark": "resources/dark/hello.svg"
        },
        "enablement": "editorLangId == javascript"
      }
    ]
  }
}
```

| Field | Required | Description |
|---|---|---|
| `command` | Yes | Unique command identifier |
| `title` | Yes | Display title in Command Palette |
| `category` | No | Group prefix (shown as "Category: Title") |
| `icon` | No | Icon for menus/toolbars |
| `enablement` | No | When clause controlling availability |

## 4. Menus

Place commands in specific menus:

```json
{
  "contributes": {
    "menus": {
      "editor/title": [
        {
          "command": "myExtension.format",
          "when": "editorLangId == javascript",
          "group": "navigation"
        }
      ],
      "editor/context": [
        {
          "command": "myExtension.insertSnippet",
          "when": "editorTextFocus",
          "group": "1_modification"
        }
      ],
      "explorer/context": [
        {
          "command": "myExtension.openFile",
          "when": "resourceExtname == .json"
        }
      ]
    }
  }
}
```

Common menu locations: `editor/title`, `editor/context`, `explorer/context`, `view/title`, `view/item/context`, `commandPalette`, `scm/title`, `scm/resourceGroup/context`.

## 5. Keybindings

Define default keyboard shortcuts:

```json
{
  "contributes": {
    "keybindings": [
      {
        "command": "myExtension.format",
        "key": "ctrl+shift+f",
        "mac": "cmd+shift+f",
        "when": "editorTextFocus && editorLangId == javascript"
      }
    ]
  }
}
```

## 6. Configuration

Expose settings in the Settings UI:

```json
{
  "contributes": {
    "configuration": {
      "title": "My Extension",
      "properties": {
        "myExtension.enable": {
          "type": "boolean",
          "default": true,
          "description": "Enable or disable the extension"
        },
        "myExtension.maxItems": {
          "type": "number",
          "default": 100,
          "description": "Maximum number of items to display",
          "minimum": 1,
          "maximum": 1000
        },
        "myExtension.logLevel": {
          "type": "string",
          "default": "info",
          "enum": ["debug", "info", "warn", "error"],
          "enumDescriptions": [
            "Verbose output",
            "Standard output",
            "Warnings only",
            "Errors only"
          ],
          "description": "Set the logging level"
        }
      }
    }
  }
}
```

## 7. Views and View Containers

### Custom View Container

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
          "id": "package-dependencies",
          "name": "Dependencies"
        },
        {
          "id": "package-outline",
          "name": "Outline"
        }
      ]
    }
  }
}
```

Views can be placed in: `explorer`, `scm`, `debug`, `test`, or custom containers.

## 8. Languages

Declare a new language:

```json
{
  "contributes": {
    "languages": [
      {
        "id": "python",
        "extensions": [".py"],
        "aliases": ["Python", "py"],
        "filenames": [],
        "firstLine": "^#!/.*\\bpython[0-9.-]*\\b",
        "configuration": "./language-configuration.json",
        "icon": {
          "light": "./icons/python-light.png",
          "dark": "./icons/python-dark.png"
        }
      }
    ]
  }
}
```

## 9. Grammars

Register TextMate grammars for syntax highlighting:

```json
{
  "contributes": {
    "grammars": [
      {
        "language": "mylang",
        "scopeName": "source.mylang",
        "path": "./syntaxes/mylang.tmLanguage.json"
      }
    ]
  }
}
```

## 10. Snippets

```json
{
  "contributes": {
    "snippets": [
      {
        "language": "javascript",
        "path": "./snippets/javascript.json"
      }
    ]
  }
}
```
