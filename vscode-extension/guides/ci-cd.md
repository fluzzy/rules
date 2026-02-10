# CI/CD for Extensions

> **Source**: [VS Code Docs — Continuous Integration](https://code.visualstudio.com/api/working-with-extensions/continuous-integration)
> **Archive Date**: 2026-02-10

Guide to setting up continuous integration and deployment for VS Code extensions. Covers Azure Pipelines, GitHub Actions, testing on multiple platforms, and automated publishing.

## 1. GitHub Actions

### Test on Push

```yaml
# .github/workflows/test.yml
name: Test Extension

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    runs-on: ${{ matrix.os }}

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Compile
        run: npm run compile

      - name: Run tests (Linux)
        if: runner.os == 'Linux'
        run: xvfb-run -a npm test
        env:
          DISPLAY: ':99.0'

      - name: Run tests (non-Linux)
        if: runner.os != 'Linux'
        run: npm test
```

### Publish on Release

```yaml
# .github/workflows/publish.yml
name: Publish Extension

on:
  release:
    types: [published]

jobs:
  publish:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Compile and Package
        run: npm run package

      - name: Publish to Marketplace
        run: npx @vscode/vsce publish -p ${{ secrets.VSCE_PAT }}

      - name: Publish to Open VSX
        run: npx ovsx publish -p ${{ secrets.OVSX_PAT }}
```

## 2. Azure Pipelines

```yaml
# azure-pipelines.yml
trigger:
  branches:
    include:
      - main

strategy:
  matrix:
    linux:
      imageName: 'ubuntu-latest'
    mac:
      imageName: 'macos-latest'
    windows:
      imageName: 'windows-latest'

pool:
  vmImage: $(imageName)

steps:
  - task: NodeTool@0
    inputs:
      versionSpec: '20.x'
    displayName: 'Install Node.js'

  - bash: npm ci
    displayName: 'Install dependencies'

  - bash: npm run compile
    displayName: 'Compile'

  - bash: |
      /usr/bin/Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
      npm test
    displayName: 'Run tests (Linux)'
    condition: eq(variables['Agent.OS'], 'Linux')
    env:
      DISPLAY: ':99.0'

  - bash: npm test
    displayName: 'Run tests (macOS/Windows)'
    condition: ne(variables['Agent.OS'], 'Linux')
```

## 3. Linux Display Server (xvfb)

VS Code tests need a display server on Linux CI. Use `xvfb-run`:

```bash
# Simple approach
xvfb-run -a npm test

# Manual approach
/usr/bin/Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
export DISPLAY=':99.0'
npm test
```

## 4. Secrets Configuration

| Secret | Purpose | Where to Get |
|---|---|---|
| `VSCE_PAT` | VS Marketplace publishing | Azure DevOps → Personal Access Tokens |
| `OVSX_PAT` | Open VSX publishing | open-vsx.org → Access Tokens |

Add secrets in GitHub: Settings → Secrets and variables → Actions → New repository secret.

## 5. Automated Version Bump

```yaml
# In publish workflow
- name: Get version from tag
  id: version
  run: echo "VERSION=${GITHUB_REF#refs/tags/v}" >> $GITHUB_OUTPUT

- name: Update package version
  run: npm version ${{ steps.version.outputs.VERSION }} --no-git-tag-version
```

## 6. Matrix Testing

Test across Node.js versions and VS Code versions:

```yaml
strategy:
  matrix:
    node-version: [18, 20]
    vscode-version: [stable, insiders]
  fail-fast: false
```

## 7. Caching

Speed up CI with dependency caching:

```yaml
- name: Cache node modules
  uses: actions/cache@v4
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
    restore-keys: |
      ${{ runner.os }}-node-
```

## 8. Pre-release Publishing

```yaml
# Publish pre-release on push to main
- name: Publish pre-release
  if: github.ref == 'refs/heads/main'
  run: npx @vscode/vsce publish --pre-release -p ${{ secrets.VSCE_PAT }}
```

## 9. Complete CI/CD Pipeline

```yaml
name: CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]
  release:
    types: [published]

jobs:
  test:
    strategy:
      matrix:
        os: [ubuntu-latest, macos-latest, windows-latest]
    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run compile
      - run: xvfb-run -a npm test
        if: runner.os == 'Linux'
      - run: npm test
        if: runner.os != 'Linux'

  publish:
    needs: test
    if: github.event_name == 'release'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: 20
      - run: npm ci
      - run: npm run package
      - run: npx @vscode/vsce publish -p ${{ secrets.VSCE_PAT }}
```
