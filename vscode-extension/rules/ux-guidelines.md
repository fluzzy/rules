# Extension UX Guidelines

> **Source**: [VS Code Docs — Extension Capabilities Overview](https://code.visualstudio.com/api/extension-capabilities/overview) | [UX Guidelines](https://code.visualstudio.com/api/ux-guidelines/overview)
> **Archive Date**: 2026-02-10

UX rules for VS Code extension development. Covers UI patterns, notifications, theming, accessibility, and user interaction best practices.

## Rules

You are an expert in VS Code extension UX design with deep knowledge of VS Code's design language and accessibility standards.

**Icons**
- Use Codicons (`$(icon-name)`) for all status bar items, tree view items, and quick pick items. Do not use custom icon fonts.
- Use the [Codicon library](https://microsoft.github.io/vscode-codicons/) — search for semantically appropriate icons rather than generic ones.
- Provide both light and dark theme variants for custom SVG icons in tree views and commands.

**Notifications**
- Use notifications sparingly. Do not show a notification on extension activation or for routine operations.
- Use `showInformationMessage` for actionable messages with buttons, not for status updates.
- Use `showWarningMessage` only for situations requiring user attention. Use `showErrorMessage` only for actual errors.
- Prefer Status Bar items, Output Channel, or Progress indicators over notifications for ongoing status.

**Progress Indicators**
- Use `vscode.window.withProgress` for any operation longer than 1 second. Specify `cancellable: true` when the operation can be interrupted.
- Choose the right `ProgressLocation`:
  - `Notification` — for user-initiated actions
  - `Window` — for background tasks (shows in status bar)
  - `SourceControl` — for SCM operations

```typescript
vscode.window.withProgress({
  location: vscode.ProgressLocation.Notification,
  title: 'Indexing workspace',
  cancellable: true
}, async (progress, token) => {
  token.onCancellationRequested(() => cleanup());
  progress.report({ increment: 0, message: 'Starting...' });
  // ...
});
```

**Theme Compatibility**
- Never hardcode colors. Use VS Code theme colors (`new vscode.ThemeColor('editor.foreground')`) for all colored UI elements.
- Test extensions in Light, Dark, and High Contrast themes before publishing.
- Use `statusBarItem.warningBackground` and `statusBarItem.errorBackground` for status bar background colors — these are the only allowed values.

**Accessibility**
- Set `accessibilityInformation` on status bar items and tree items for screen reader users.
- Provide keyboard shortcuts for all primary commands. Declare them in `contributes.keybindings`.
- Use `aria-label` attributes in webview HTML elements.
- Ensure all interactive elements are focusable and operable with keyboard.

**Standard UI Patterns**
- Use VS Code's built-in UI elements before creating custom ones: Quick Pick for selection, Input Box for text input, Tree View for hierarchical data.
- Do not create custom modal dialogs. Use `showInformationMessage` with action buttons or Quick Pick instead.
- Use the Settings UI (`contributes.configuration`) for extension settings. Do not create custom settings pages.

**Tree Views**
- Use `contextValue` on tree items to control context menu visibility with `when` clauses.
- Provide `viewsWelcome` content for empty views with a call-to-action link.
- Add a refresh button in `view/title` with `group: "navigation"` for data that can become stale.

**Commands**
- Use `category` in command contributions to group related commands in the Command Palette (e.g., `"category": "Git"`).
- Provide `enablement` when clauses to disable commands that are not applicable in the current context.
