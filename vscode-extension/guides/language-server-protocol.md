# Language Server Protocol

> **Source**: [VS Code Docs — Language Server Extension Guide](https://code.visualstudio.com/api/language-extensions/language-server-extension-guide) | [LSP Sample](https://github.com/microsoft/vscode-extension-samples/tree/main/lsp-sample)
> **Archive Date**: 2026-02-10

Guide to implementing a Language Server Protocol (LSP) client in VS Code. Covers LSP architecture, LanguageClient setup, server options, transport, and the client-server communication pattern.

## 1. LSP Architecture

The Language Server Protocol separates language intelligence from the editor:

```
┌──────────────────────┐         ┌──────────────────────┐
│     VS Code          │         │   Language Server     │
│  (LSP Client)        │◄───────►│   (Separate Process)  │
│                      │  JSON   │                      │
│  - Editor UI         │  RPC    │  - Parsing           │
│  - File management   │         │  - Completion        │
│  - User interaction  │         │  - Diagnostics       │
│                      │         │  - Hover info        │
└──────────────────────┘         └──────────────────────┘
```

### Benefits of LSP

| Benefit | Description |
|---|---|
| Separation of concerns | Server handles language logic, client handles UI |
| Reusability | One server works with multiple editors |
| Process isolation | Server crashes don't crash the editor |
| Language flexibility | Server can be written in any language |

## 2. Project Structure

An LSP extension typically has two parts:

```text
my-lsp-extension/
├── client/
│   ├── src/
│   │   └── extension.ts    // LSP client code
│   └── package.json
├── server/
│   ├── src/
│   │   └── server.ts       // LSP server code
│   └── package.json
├── package.json             // Extension manifest
└── tsconfig.json
```

## 3. Client Setup

The client connects to the language server and bridges VS Code with it:

```typescript
import * as path from 'path';
import { workspace, ExtensionContext } from 'vscode';
import {
  LanguageClient,
  LanguageClientOptions,
  ServerOptions,
  TransportKind
} from 'vscode-languageclient/node';

let client: LanguageClient;

export function activate(context: ExtensionContext) {
  // Path to the server module
  const serverModule = context.asAbsolutePath(
    path.join('server', 'out', 'server.js')
  );

  // Server options — how to start/connect to the server
  const serverOptions: ServerOptions = {
    run: {
      module: serverModule,
      transport: TransportKind.ipc  // IPC for production
    },
    debug: {
      module: serverModule,
      transport: TransportKind.ipc,
      options: {
        execArgv: ['--nolazy', '--inspect=6009']  // Debug port
      }
    }
  };

  // Client options — what the client manages
  const clientOptions: LanguageClientOptions = {
    // Register the server for specific documents
    documentSelector: [{ scheme: 'file', language: 'plaintext' }],
    synchronize: {
      // Notify the server about file changes to config files
      fileEvents: workspace.createFileSystemWatcher('**/.clientrc')
    }
  };

  // Create and start the client
  client = new LanguageClient(
    'languageServerExample',    // ID
    'Language Server Example',   // Display name
    serverOptions,
    clientOptions
  );

  client.start();
}

export function deactivate(): Thenable<void> | undefined {
  if (!client) return undefined;
  return client.stop();
}
```

## 4. Server Options

| Option | Description |
|---|---|
| `module` | Path to the server JS file |
| `transport` | Communication transport (`ipc`, `stdio`, `pipe`, `socket`) |
| `options.execArgv` | Node.js arguments (e.g., debug flags) |
| `args` | Arguments passed to the server |

### Transport Types

| Transport | Use Case |
|---|---|
| `TransportKind.ipc` | Recommended for Node.js servers (fastest) |
| `TransportKind.stdio` | Standard I/O — works with any language |
| `TransportKind.pipe` | Named pipe |
| `TransportKind.socket` | TCP socket (specify port) |

## 5. Client Options

```typescript
const clientOptions: LanguageClientOptions = {
  documentSelector: [
    { scheme: 'file', language: 'python' },
    { scheme: 'untitled', language: 'python' }
  ],
  synchronize: {
    configurationSection: 'myLanguageServer',
    fileEvents: workspace.createFileSystemWatcher('**/*.py')
  },
  outputChannelName: 'My Language Server',
  revealOutputChannelOn: RevealOutputChannelOn.Error,
  middleware: {
    // Intercept and modify requests/responses
    provideCompletionItem: (document, position, context, token, next) => {
      return next(document, position, context, token);
    }
  }
};
```

## 6. Basic Server Implementation

```typescript
import {
  createConnection,
  TextDocuments,
  ProposedFeatures,
  InitializeParams,
  CompletionItem,
  CompletionItemKind,
  TextDocumentPositionParams,
  TextDocumentSyncKind,
  InitializeResult,
  Diagnostic,
  DiagnosticSeverity
} from 'vscode-languageserver/node';
import { TextDocument } from 'vscode-languageserver-textdocument';

const connection = createConnection(ProposedFeatures.all);
const documents = new TextDocuments(TextDocument);

connection.onInitialize((params: InitializeParams) => {
  const result: InitializeResult = {
    capabilities: {
      textDocumentSync: TextDocumentSyncKind.Incremental,
      completionProvider: { resolveProvider: true },
      hoverProvider: true,
      definitionProvider: true
    }
  };
  return result;
});

// Validate documents on change
documents.onDidChangeContent(change => {
  validateTextDocument(change.document);
});

async function validateTextDocument(textDocument: TextDocument): Promise<void> {
  const text = textDocument.getText();
  const diagnostics: Diagnostic[] = [];

  const pattern = /\b[A-Z]{2,}\b/g;
  let m: RegExpExecArray | null;
  while ((m = pattern.exec(text))) {
    const diagnostic: Diagnostic = {
      severity: DiagnosticSeverity.Warning,
      range: {
        start: textDocument.positionAt(m.index),
        end: textDocument.positionAt(m.index + m[0].length)
      },
      message: `${m[0]} is all uppercase.`,
      source: 'my-linter'
    };
    diagnostics.push(diagnostic);
  }

  connection.sendDiagnostics({ uri: textDocument.uri, diagnostics });
}

// Provide completions
connection.onCompletion((_textDocumentPosition: TextDocumentPositionParams): CompletionItem[] => {
  return [
    { label: 'TypeScript', kind: CompletionItemKind.Text, data: 1 },
    { label: 'JavaScript', kind: CompletionItemKind.Text, data: 2 }
  ];
});

connection.onCompletionResolve((item: CompletionItem): CompletionItem => {
  if (item.data === 1) {
    item.detail = 'TypeScript details';
    item.documentation = 'TypeScript documentation';
  }
  return item;
});

documents.listen(connection);
connection.listen();
```

## 7. Dependencies

```json
{
  "dependencies": {
    "vscode-languageclient": "^9.0.0"
  }
}
```

Server-side:
```json
{
  "dependencies": {
    "vscode-languageserver": "^9.0.0",
    "vscode-languageserver-textdocument": "^1.0.0"
  }
}
```

## 8. Server Capabilities

The server declares which features it supports during initialization:

| Capability | Feature |
|---|---|
| `textDocumentSync` | Document synchronization mode |
| `completionProvider` | Auto-completion |
| `hoverProvider` | Hover information |
| `definitionProvider` | Go to Definition |
| `referencesProvider` | Find All References |
| `documentSymbolProvider` | Document Symbols |
| `codeActionProvider` | Code Actions / Quick Fix |
| `codeLensProvider` | CodeLens |
| `documentFormattingProvider` | Document Formatting |
| `renameProvider` | Rename Symbol |
| `diagnosticProvider` | Pull-model diagnostics |
| `foldingRangeProvider` | Folding Ranges |
| `signatureHelpProvider` | Signature Help |
| `workspaceSymbolProvider` | Workspace Symbols |
