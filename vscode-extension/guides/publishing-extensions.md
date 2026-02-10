# Publishing Extensions

> **Source**: [VS Code Docs — Publishing Extensions](https://code.visualstudio.com/api/working-with-extensions/publishing-extension) | [VSCE](https://github.com/microsoft/vscode-vsce)
> **Archive Date**: 2026-02-10

Guide to packaging and publishing VS Code extensions to the Visual Studio Marketplace using `vsce`. Covers publisher setup, PAT tokens, packaging, publishing, pre-release, and package.json configuration.

## 1. Prerequisites

### Install vsce

```bash
npm install -g @vscode/vsce
```

### Create a Publisher

1. Go to [Visual Studio Marketplace](https://marketplace.visualstudio.com/manage)
2. Sign in with your Microsoft account
3. Create a publisher with a unique ID

### Generate Personal Access Token (PAT)

1. Go to [Azure DevOps](https://dev.azure.com)
2. User Settings → Personal Access Tokens → New Token
3. Set Organization to "All accessible organizations"
4. Set Scopes → Marketplace → Manage
5. Copy the token

### Login

```bash
vsce login <publisher-id>
# Paste your PAT when prompted
```

## 2. Package.json Requirements

Ensure these fields are set in `package.json`:

```json
{
  "name": "my-extension",
  "displayName": "My Extension",
  "description": "A useful VS Code extension",
  "version": "1.0.0",
  "publisher": "your-publisher-id",
  "engines": {
    "vscode": "^1.80.0"
  },
  "categories": ["Other"],
  "repository": {
    "type": "git",
    "url": "https://github.com/user/repo"
  },
  "icon": "images/icon.png",
  "license": "MIT",
  "keywords": ["keyword1", "keyword2"],
  "galleryBanner": {
    "color": "#1e1e1e",
    "theme": "dark"
  },
  "pricing": "Free"
}
```

### Required Fields

| Field | Description |
|---|---|
| `name` | Extension identifier |
| `displayName` | Marketplace display name |
| `description` | Short description (max 200 chars) |
| `version` | SemVer version |
| `publisher` | Publisher ID |
| `engines.vscode` | Minimum VS Code version |

### Recommended Fields

| Field | Description |
|---|---|
| `icon` | 128x128 PNG icon |
| `categories` | One of: Programming Languages, Snippets, Linters, Themes, Debuggers, Formatters, Keymaps, SCM Providers, Other, Extension Packs, Language Packs, Data Science, Machine Learning, Visualization, Notebooks, Education, Testing |
| `repository` | Source repository URL |
| `license` | License identifier or file |
| `keywords` | Search keywords (max 5) |

## 3. Packaging

Create a `.vsix` file for distribution:

```bash
# Package the extension
vsce package

# Package with specific version
vsce package 1.2.3

# Package with pre-release flag
vsce package --pre-release
```

### Verify Package Contents

```bash
# List files in the package
vsce ls
```

## 4. Publishing

```bash
# Publish to marketplace
vsce publish

# Publish with version bump
vsce publish minor    # 1.0.0 → 1.1.0
vsce publish major    # 1.0.0 → 2.0.0
vsce publish patch    # 1.0.0 → 1.0.1

# Publish specific version
vsce publish 1.2.3

# Publish pre-release
vsce publish --pre-release
```

## 5. .vscodeignore

Control which files are included in the .vsix:

```
.vscode/**
.vscode-test/**
src/**
node_modules/**
.gitignore
.eslintrc.json
tsconfig.json
**/*.ts
**/*.map
vsc-extension-quickstart.md
CONTRIBUTING.md
esbuild.js
webpack.config.js
```

Files always included: `package.json`, `README.md`, `CHANGELOG.md`, `LICENSE`.

## 6. Pre-release Extensions

Pre-release versions allow early testing:

```bash
# Publish as pre-release
vsce publish --pre-release
```

Users opt-in via the extension page in VS Code. Pre-release and stable versions are tracked separately.

## 7. Platform-Specific Extensions

For extensions with native dependencies:

```bash
# Package for specific platform
vsce package --target win32-x64
vsce package --target linux-x64
vsce package --target darwin-arm64

# Publish all platforms
vsce publish --target win32-x64 win32-arm64 linux-x64 linux-arm64 darwin-x64 darwin-arm64
```

## 8. Unpublishing

```bash
# Unpublish specific version
vsce unpublish <publisher>.<extension> <version>

# Unpublish entire extension (CAUTION)
vsce unpublish <publisher>.<extension>
```

## 9. Extension Manifest (vsce) Options

Additional `package.json` fields for vsce:

```json
{
  "vsce": {
    "dependencies": true,
    "yarn": false,
    "githubBranch": "main"
  }
}
```

## 10. README Best Practices

The README is the main landing page on the marketplace:

- Start with a clear description and screenshot/GIF
- Include feature list with visuals
- Add installation and usage instructions
- Describe all settings and commands
- Include keyboard shortcuts
- Add a changelog link

## 11. Versioning

Follow [Semantic Versioning](https://semver.org/):

| Type | When | Example |
|---|---|---|
| `patch` | Bug fixes | 1.0.0 → 1.0.1 |
| `minor` | New features (backward-compatible) | 1.0.0 → 1.1.0 |
| `major` | Breaking changes | 1.0.0 → 2.0.0 |
