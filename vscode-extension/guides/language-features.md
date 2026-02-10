# Programmatic Language Features

> **Source**: [VS Code Docs — Programmatic Language Features](https://code.visualstudio.com/api/language-extensions/programmatic-language-features)
> **Archive Date**: 2026-02-10

Guide to implementing language features programmatically using VS Code's `vscode.languages` API. Covers all major providers: completion, hover, definition, references, diagnostics, CodeLens, formatting, rename, and more.

## 1. Provider Registration Pattern

All language features follow the same registration pattern:

```typescript
const provider = vscode.languages.registerXxxProvider(
  selector,    // DocumentSelector — which files to apply to
  providerObj  // Object implementing the provider interface
);
context.subscriptions.push(provider);
```

### Document Selector

```typescript
// Single language
const selector: vscode.DocumentSelector = 'javascript';

// Multiple languages
const selector: vscode.DocumentSelector = ['javascript', 'typescript'];

// With scheme
const selector: vscode.DocumentSelector = { language: 'python', scheme: 'file' };

// Pattern-based
const selector: vscode.DocumentSelector = { pattern: '**/*.custom' };
```

## 2. Provider Reference Table

| Provider | Registration Method | Trigger |
|---|---|---|
| Completion | `registerCompletionItemProvider` | Typing / Ctrl+Space |
| Hover | `registerHoverProvider` | Mouse hover |
| Definition | `registerDefinitionProvider` | Go to Definition (F12) |
| Type Definition | `registerTypeDefinitionProvider` | Go to Type Definition |
| Implementation | `registerImplementationProvider` | Go to Implementation |
| Declaration | `registerDeclarationProvider` | Go to Declaration |
| References | `registerReferenceProvider` | Find All References |
| Document Symbols | `registerDocumentSymbolProvider` | Go to Symbol (Ctrl+Shift+O) |
| Workspace Symbols | `registerWorkspaceSymbolProvider` | Open Symbol (Ctrl+T) |
| Code Actions | `registerCodeActionsProvider` | Light bulb / Quick Fix |
| CodeLens | `registerCodeLensProvider` | Inline actionable info |
| Document Formatting | `registerDocumentFormattingEditProvider` | Format Document |
| Range Formatting | `registerDocumentRangeFormattingEditProvider` | Format Selection |
| On Type Formatting | `registerOnTypeFormattingEditProvider` | Format as you type |
| Rename | `registerRenameProvider` | Rename Symbol (F2) |
| Document Links | `registerDocumentLinkProvider` | Clickable links |
| Color | `registerColorProvider` | Color picker |
| Folding Range | `registerFoldingRangeProvider` | Code folding |
| Selection Range | `registerSelectionRangeProvider` | Expand/Shrink Selection |
| Call Hierarchy | `registerCallHierarchyProvider` | Call Hierarchy |
| Type Hierarchy | `registerTypeHierarchyProvider` | Type Hierarchy |
| Semantic Tokens | `registerDocumentSemanticTokensProvider` | Semantic highlighting |
| Inlay Hints | `registerInlayHintsProvider` | Inline parameter hints |
| Signature Help | `registerSignatureHelpProvider` | Function parameter info |
| Diagnostics | `createDiagnosticCollection` | Error/warning squiggles |

## 3. Completion Provider

```typescript
const completionProvider = vscode.languages.registerCompletionItemProvider(
  'javascript',
  {
    provideCompletionItems(
      document: vscode.TextDocument,
      position: vscode.Position,
      token: vscode.CancellationToken,
      context: vscode.CompletionContext
    ): vscode.CompletionItem[] {
      const items: vscode.CompletionItem[] = [];

      const snippet = new vscode.CompletionItem('log');
      snippet.insertText = new vscode.SnippetString('console.log(${1:message});');
      snippet.documentation = new vscode.MarkdownString('Log to console');
      snippet.kind = vscode.CompletionItemKind.Snippet;
      items.push(snippet);

      const keyword = new vscode.CompletionItem('forEach');
      keyword.insertText = new vscode.SnippetString('.forEach((${1:item}) => {\n\t$0\n});');
      keyword.kind = vscode.CompletionItemKind.Method;
      items.push(keyword);

      return items;
    }
  },
  '.'  // Trigger character
);
```

## 4. Hover Provider

```typescript
const hoverProvider = vscode.languages.registerHoverProvider('javascript', {
  provideHover(document, position, token) {
    const range = document.getWordRangeAtPosition(position);
    const word = document.getText(range);

    if (word === 'console') {
      return new vscode.Hover(
        new vscode.MarkdownString('**console** — The Console object provides access to the browser debugging console.'),
        range
      );
    }
  }
});
```

## 5. Definition Provider

```typescript
const definitionProvider = vscode.languages.registerDefinitionProvider('javascript', {
  provideDefinition(document, position, token) {
    const word = document.getText(document.getWordRangeAtPosition(position));
    // Find the definition location
    const targetUri = vscode.Uri.file('/path/to/definition.js');
    const targetPosition = new vscode.Position(10, 0);
    return new vscode.Location(targetUri, targetPosition);
  }
});
```

## 6. Diagnostics

```typescript
export function activate(context: vscode.ExtensionContext) {
  const diagnosticCollection = vscode.languages.createDiagnosticCollection('myLinter');
  context.subscriptions.push(diagnosticCollection);

  // Update diagnostics when document changes
  context.subscriptions.push(
    vscode.workspace.onDidChangeTextDocument(e => updateDiagnostics(e.document, diagnosticCollection)),
    vscode.workspace.onDidOpenTextDocument(doc => updateDiagnostics(doc, diagnosticCollection))
  );
}

function updateDiagnostics(document: vscode.TextDocument, collection: vscode.DiagnosticCollection) {
  if (document.languageId !== 'javascript') return;

  const diagnostics: vscode.Diagnostic[] = [];
  const text = document.getText();

  // Example: flag console.log statements
  const regex = /console\.log/g;
  let match;
  while ((match = regex.exec(text)) !== null) {
    const startPos = document.positionAt(match.index);
    const endPos = document.positionAt(match.index + match[0].length);
    const range = new vscode.Range(startPos, endPos);

    const diagnostic = new vscode.Diagnostic(
      range,
      'Avoid console.log in production code',
      vscode.DiagnosticSeverity.Warning
    );
    diagnostic.code = 'no-console';
    diagnostic.source = 'myLinter';
    diagnostics.push(diagnostic);
  }

  collection.set(document.uri, diagnostics);
}
```

## 7. CodeLens Provider

```typescript
class MyCodeLensProvider implements vscode.CodeLensProvider {
  private _onDidChangeCodeLenses = new vscode.EventEmitter<void>();
  readonly onDidChangeCodeLenses = this._onDidChangeCodeLenses.event;

  provideCodeLenses(document: vscode.TextDocument): vscode.CodeLens[] {
    const lenses: vscode.CodeLens[] = [];
    const regex = /function\s+(\w+)/g;
    const text = document.getText();
    let match;

    while ((match = regex.exec(text)) !== null) {
      const pos = document.positionAt(match.index);
      const range = new vscode.Range(pos, pos);
      lenses.push(new vscode.CodeLens(range));
    }
    return lenses;
  }

  resolveCodeLens(codeLens: vscode.CodeLens): vscode.CodeLens {
    codeLens.command = {
      title: 'Run Function',
      command: 'myExtension.runFunction',
      arguments: [codeLens.range]
    };
    return codeLens;
  }
}
```

## 8. Code Actions Provider

```typescript
class MyCodeActionProvider implements vscode.CodeActionProvider {
  static readonly providedCodeActionKinds = [
    vscode.CodeActionKind.QuickFix
  ];

  provideCodeActions(
    document: vscode.TextDocument,
    range: vscode.Range | vscode.Selection,
    context: vscode.CodeActionContext
  ): vscode.CodeAction[] {
    return context.diagnostics
      .filter(diag => diag.code === 'no-console')
      .map(diag => {
        const fix = new vscode.CodeAction(
          'Remove console.log',
          vscode.CodeActionKind.QuickFix
        );
        fix.edit = new vscode.WorkspaceEdit();
        fix.edit.delete(document.uri, diag.range);
        fix.isPreferred = true;
        fix.diagnostics = [diag];
        return fix;
      });
  }
}
```

## 9. Document Formatting Provider

```typescript
const formatProvider = vscode.languages.registerDocumentFormattingEditProvider('javascript', {
  provideDocumentFormattingEdits(document: vscode.TextDocument): vscode.TextEdit[] {
    const edits: vscode.TextEdit[] = [];
    // Example: trim trailing whitespace
    for (let i = 0; i < document.lineCount; i++) {
      const line = document.lineAt(i);
      const trimmed = line.text.trimEnd();
      if (trimmed !== line.text) {
        edits.push(vscode.TextEdit.replace(line.range, trimmed));
      }
    }
    return edits;
  }
});
```

## 10. Rename Provider

```typescript
const renameProvider = vscode.languages.registerRenameProvider('javascript', {
  provideRenameEdits(document, position, newName, token) {
    const word = document.getText(document.getWordRangeAtPosition(position));
    const edit = new vscode.WorkspaceEdit();

    // Find all occurrences and replace
    const text = document.getText();
    const regex = new RegExp(`\\b${word}\\b`, 'g');
    let match;
    while ((match = regex.exec(text)) !== null) {
      const start = document.positionAt(match.index);
      const end = document.positionAt(match.index + word.length);
      edit.replace(document.uri, new vscode.Range(start, end), newName);
    }

    return edit;
  },

  prepareRename(document, position) {
    const range = document.getWordRangeAtPosition(position);
    if (!range) {
      throw new Error('Cannot rename this element');
    }
    return { range, placeholder: document.getText(range) };
  }
});
```
