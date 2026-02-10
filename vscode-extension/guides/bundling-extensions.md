# Bundling Extensions

> **Source**: [VS Code Docs — Bundling Extensions](https://code.visualstudio.com/api/working-with-extensions/bundling-extension)
> **Archive Date**: 2026-02-10

Guide to bundling VS Code extensions with esbuild or webpack. Bundling reduces extension size, speeds up activation, and packages all dependencies into a single file.

## 1. Why Bundle?

| Without Bundling | With Bundling |
|---|---|
| Hundreds of files in node_modules | Single output file |
| Slow activation (many file reads) | Fast activation |
| Large .vsix package | Small .vsix package |
| All devDependencies included | Only runtime code included |

## 2. esbuild Configuration (Recommended)

### Install

```bash
npm install --save-dev esbuild
```

### Build Script

Create `esbuild.js`:

```javascript
const esbuild = require('esbuild');

const production = process.argv.includes('--production');
const watch = process.argv.includes('--watch');

async function main() {
  const ctx = await esbuild.context({
    entryPoints: ['src/extension.ts'],
    bundle: true,
    format: 'cjs',
    minify: production,
    sourcemap: !production,
    sourcesContent: false,
    platform: 'node',
    outfile: 'dist/extension.js',
    external: ['vscode'],  // IMPORTANT: vscode is provided by VS Code runtime
    logLevel: 'silent',
    plugins: [
      esbuildProblemMatcherPlugin
    ]
  });

  if (watch) {
    await ctx.watch();
  } else {
    await ctx.rebuild();
    await ctx.dispose();
  }
}

/**
 * Plugin to report build errors in VS Code problem matcher format
 */
const esbuildProblemMatcherPlugin = {
  name: 'esbuild-problem-matcher',
  setup(build) {
    build.onStart(() => {
      console.log('[watch] build started');
    });
    build.onEnd(result => {
      result.errors.forEach(({ text, location }) => {
        console.error(`✘ [ERROR] ${text}`);
        console.error(`    ${location.file}:${location.line}:${location.column}:`);
      });
      console.log('[watch] build finished');
    });
  }
};

main().catch(e => {
  console.error(e);
  process.exit(1);
});
```

### Package.json Scripts

```json
{
  "main": "./dist/extension.js",
  "scripts": {
    "vscode:prepublish": "npm run package",
    "compile": "node esbuild.js",
    "watch": "node esbuild.js --watch",
    "package": "node esbuild.js --production"
  }
}
```

## 3. webpack Configuration

### Install

```bash
npm install --save-dev webpack webpack-cli ts-loader
```

### webpack.config.js

```javascript
//@ts-check
'use strict';

const path = require('path');

/** @type {import('webpack').Configuration} */
const extensionConfig = {
  target: 'node',
  mode: 'none',
  entry: './src/extension.ts',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'extension.js',
    libraryTarget: 'commonjs2'
  },
  externals: {
    vscode: 'commonjs vscode'  // vscode module is provided at runtime
  },
  resolve: {
    extensions: ['.ts', '.js']
  },
  module: {
    rules: [
      {
        test: /\.ts$/,
        exclude: /node_modules/,
        use: [{ loader: 'ts-loader' }]
      }
    ]
  },
  devtool: 'nosources-source-map',
  infrastructureLogging: {
    level: 'log'
  }
};

module.exports = [extensionConfig];
```

### Package.json Scripts (webpack)

```json
{
  "main": "./dist/extension.js",
  "scripts": {
    "vscode:prepublish": "npm run package",
    "compile": "webpack",
    "watch": "webpack --watch",
    "package": "webpack --mode production --devtool hidden-source-map"
  }
}
```

## 4. Key Configuration Points

### External: vscode

The `vscode` module must always be externalized — it is provided by the VS Code runtime:

```javascript
// esbuild
external: ['vscode']

// webpack
externals: { vscode: 'commonjs vscode' }
```

### Platform: node

For standard extensions running in Node.js:

```javascript
// esbuild
platform: 'node'

// webpack
target: 'node'
```

### For Web Extensions

```javascript
// esbuild
platform: 'browser'

// webpack
target: 'webworker'
```

## 5. Source Maps

| Environment | Source Map Type |
|---|---|
| Development | `sourcemap: true` / `devtool: 'source-map'` |
| Production | `sourcemap: false` / `devtool: 'hidden-source-map'` |

Hidden source maps generate the map file but don't reference it in the output — useful for error reporting services.

## 6. .vscodeignore

After bundling, exclude source files from the .vsix package:

```
.vscode/**
src/**
node_modules/**
!node_modules/some-native-dep/**
.gitignore
**/tsconfig.json
**/*.ts
**/*.map
webpack.config.js
esbuild.js
```

## 7. Watch Mode

### VS Code Tasks

Add to `.vscode/tasks.json` for esbuild watch:

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "watch",
      "type": "shell",
      "command": "node esbuild.js --watch",
      "isBackground": true,
      "problemMatcher": [
        {
          "base": "$esbuild-watch",
          "background": {
            "activeOnStart": true,
            "beginsPattern": "\\[watch\\] build started",
            "endsPattern": "\\[watch\\] build finished"
          }
        }
      ],
      "group": {
        "kind": "build",
        "isDefault": true
      }
    }
  ]
}
```

## 8. Native Dependencies

If your extension uses native Node.js addons (`.node` files), they cannot be bundled:

```javascript
// esbuild
external: ['vscode', 'some-native-module']

// webpack
externals: {
  vscode: 'commonjs vscode',
  'some-native-module': 'commonjs some-native-module'
}
```

Keep native dependencies in `node_modules` and include them in the .vsix via `.vscodeignore` exceptions.
