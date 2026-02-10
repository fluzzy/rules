# Activation Events

> **Source**: [VS Code Docs — Activation Events](https://code.visualstudio.com/api/references/activation-events)
> **Archive Date**: 2026-02-10

Complete reference for all VS Code extension activation events. Activation events control when your extension is loaded into memory — use the most specific event possible to minimize startup impact.

## 1. Activation Events Overview

Extensions are lazy-loaded. VS Code activates an extension only when a declared activation event occurs. Declare events in `package.json` under `activationEvents`.

```json
{
  "activationEvents": [
    "onCommand:extension.sayHello"
  ]
}
```

## 2. Event Reference

| Event | Trigger | Example |
|---|---|---|
| `onCommand:commandId` | When the command is invoked | `onCommand:extension.sayHello` |
| `onLanguage:languageId` | When a file of that language is opened | `onLanguage:python` |
| `onLanguage` | When any file is opened (generic) | `onLanguage` |
| `onView:viewId` | When a specific view is expanded in sidebar | `onView:nodeDependencies` |
| `onUri` | When a system URI for this extension is opened | `onUri` |
| `workspaceContains:pattern` | When a workspace folder matching the glob is opened | `workspaceContains:**/.editorconfig` |
| `onFileSystem:scheme` | When a file/folder from a specific scheme is opened | `onFileSystem:ftp` |
| `onDebug` | Before a debug session starts | `onDebug` |
| `onDebugResolve:type` | Before a debug session of a specific type starts | `onDebugResolve:node` |
| `onDebugInitialConfigurations` | Before `provideDebugConfigurations` is called | `onDebugInitialConfigurations` |
| `onDebugDynamicConfigurations:type` | Before `provideDynamicDebugConfigurations` is called | `onDebugDynamicConfigurations:node` |
| `onStartupFinished` | After VS Code startup completes (delayed) | `onStartupFinished` |
| `onTaskType:type` | When a task of a specific type is run | `onTaskType:npm` |
| `onWebviewPanel:viewType` | When a webview with the specified viewType is restored | `onWebviewPanel:catCoding` |
| `onCustomEditor:viewType` | When a custom editor of the specified viewType is opened | `onCustomEditor:myEditor.imagePreview` |
| `onAuthenticationRequest:providerId` | When an authentication request is made for a provider | `onAuthenticationRequest:github` |
| `onWalkthrough:walkthroughId` | When a Getting Started walkthrough is opened | `onWalkthrough:myExtension.welcome` |
| `*` | On VS Code startup (use sparingly!) | `*` |

## 3. Usage Guidelines

### Prefer Specific Events

```json
// GOOD — activates only when the command is invoked
{
  "activationEvents": ["onCommand:myExtension.format"]
}

// BAD — activates on every startup, slows VS Code
{
  "activationEvents": ["*"]
}
```

### Implicit Activation

Many contribution points trigger activation automatically without an explicit `activationEvents` entry:

- `commands` → generates `onCommand:...` automatically
- `languages` → generates `onLanguage:...` automatically
- `customEditors` → generates `onCustomEditor:...` automatically
- `views` → generates `onView:...` automatically

Since VS Code 1.74+, you can omit `activationEvents` for most contributions — they are inferred.

### Multiple Events

An extension can declare multiple activation events:

```json
{
  "activationEvents": [
    "onLanguage:json",
    "onLanguage:markdown",
    "onCommand:myExtension.format"
  ]
}
```

The extension activates when **any** of these events fires.

## 4. onStartupFinished vs *

| Aspect | `onStartupFinished` | `*` |
|---|---|---|
| Timing | After startup completes | During startup |
| Impact | Minimal — deferred | Blocks startup |
| Use case | Background tasks, indexing | Almost never needed |

Always prefer `onStartupFinished` over `*` for extensions that need early activation.

## 5. Workspace Contains Pattern

`workspaceContains` uses glob patterns relative to workspace folders:

```json
{
  "activationEvents": [
    "workspaceContains:**/package.json",
    "workspaceContains:.eslintrc*"
  ]
}
```

The search is limited to prevent excessive file system scanning. Place patterns that match common files near the workspace root for faster detection.
