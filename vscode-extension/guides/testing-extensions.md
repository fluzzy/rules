# Testing Extensions

> **Source**: [VS Code Docs — Testing Extensions](https://code.visualstudio.com/api/working-with-extensions/testing-extension)
> **Archive Date**: 2026-02-10

Guide to testing VS Code extensions. Covers the `@vscode/test-electron` runner, integration test setup, launch.json configuration for debugging tests, and CI considerations.

## 1. Overview

VS Code extensions run inside an Extension Host process, so tests need a running VS Code instance. The `@vscode/test-electron` package handles downloading and launching VS Code for tests.

## 2. Setup

Install the test runner:

```bash
npm install --save-dev @vscode/test-electron
```

## 3. Test Runner Script

Create `src/test/runTest.ts`:

```typescript
import * as path from 'path';
import { runTests } from '@vscode/test-electron';

async function main() {
  try {
    // The folder containing the extension manifest (package.json)
    const extensionDevelopmentPath = path.resolve(__dirname, '../../');

    // The path to the test runner script
    const extensionTestsPath = path.resolve(__dirname, './suite/index');

    // Download VS Code, unzip it, and run the integration tests
    await runTests({
      extensionDevelopmentPath,
      extensionTestsPath,
      launchArgs: ['--disable-extensions']  // Disable other extensions
    });
  } catch (err) {
    console.error('Failed to run tests');
    process.exit(1);
  }
}

main();
```

### runTests Options

| Option | Description |
|---|---|
| `extensionDevelopmentPath` | Path to extension root (with package.json) |
| `extensionTestsPath` | Path to test suite entry point |
| `launchArgs` | Additional VS Code launch arguments |
| `version` | Specific VS Code version to test against |
| `platform` | Target platform (`win32-x64-archive`, `darwin`, `linux-x64`) |

## 4. Test Suite Entry Point

Create `src/test/suite/index.ts`:

```typescript
import * as path from 'path';
import Mocha from 'mocha';
import { glob } from 'glob';

export function run(): Promise<void> {
  const mocha = new Mocha({
    ui: 'tdd',
    color: true,
    timeout: 10000
  });

  const testsRoot = path.resolve(__dirname, '..');

  return new Promise((resolve, reject) => {
    glob('**/**.test.js', { cwd: testsRoot })
      .then(files => {
        files.forEach(f => mocha.addFile(path.resolve(testsRoot, f)));

        try {
          mocha.run(failures => {
            if (failures > 0) {
              reject(new Error(`${failures} tests failed.`));
            } else {
              resolve();
            }
          });
        } catch (err) {
          reject(err);
        }
      })
      .catch(reject);
  });
}
```

## 5. Writing Tests

Create test files like `src/test/suite/extension.test.ts`:

```typescript
import * as assert from 'assert';
import * as vscode from 'vscode';

suite('Extension Test Suite', () => {
  vscode.window.showInformationMessage('Start all tests.');

  test('Extension should be present', () => {
    assert.ok(vscode.extensions.getExtension('publisher.extensionName'));
  });

  test('Extension should activate', async () => {
    const ext = vscode.extensions.getExtension('publisher.extensionName')!;
    await ext.activate();
    assert.ok(ext.isActive);
  });

  test('Command should be registered', async () => {
    const commands = await vscode.commands.getCommands(true);
    assert.ok(commands.includes('myExtension.sayHello'));
  });

  test('Command should show message', async () => {
    await vscode.commands.executeCommand('myExtension.sayHello');
    // Verify side effects
  });
});
```

## 6. Launch Configuration for Debugging Tests

Add to `.vscode/launch.json`:

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Extension Tests",
      "type": "extensionHost",
      "request": "launch",
      "runtimeExecutable": "${execPath}",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}",
        "--extensionTestsPath=${workspaceFolder}/out/test/suite/index"
      ],
      "outFiles": ["${workspaceFolder}/out/test/**/*.js"],
      "preLaunchTask": "npm: compile"
    }
  ]
}
```

## 7. Testing with Workspace

Open a specific workspace during tests:

```typescript
await runTests({
  extensionDevelopmentPath,
  extensionTestsPath,
  launchArgs: [
    path.resolve(__dirname, '../../test-fixtures'),  // Workspace to open
    '--disable-extensions'
  ]
});
```

## 8. @vscode/test-cli (Alternative)

The newer `@vscode/test-cli` provides a simpler approach:

```bash
npm install --save-dev @vscode/test-cli @vscode/test-electron
```

Create `.vscode-test.mjs`:

```javascript
import { defineConfig } from '@vscode/test-cli';

export default defineConfig({
  files: 'out/test/**/*.test.js',
  mocha: {
    ui: 'tdd',
    timeout: 20000
  }
});
```

Run tests:

```bash
npx vscode-test
```

## 9. Package.json Scripts

```json
{
  "scripts": {
    "compile": "tsc -p ./",
    "watch": "tsc -watch -p ./",
    "pretest": "npm run compile",
    "test": "node ./out/test/runTest.js"
  }
}
```

## 10. CI Considerations

- Linux CI requires `xvfb-run` to provide a virtual display
- Set `--disable-extensions` to avoid interference from other extensions
- Use `--disable-gpu` for headless environments
- Pin VS Code version for reproducible tests

```bash
# Linux CI
xvfb-run -a npm test
```
