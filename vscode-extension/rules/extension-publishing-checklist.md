# Extension Publishing Checklist

> **Source**: [VS Code Docs — Publishing Extensions](https://code.visualstudio.com/api/working-with-extensions/publishing-extension) | [VSCE](https://github.com/microsoft/vscode-vsce) | [CI/CD](https://code.visualstudio.com/api/working-with-extensions/continuous-integration)
> **Archive Date**: 2026-02-10

Pre-publish checklist rules for VS Code extensions. Covers documentation, packaging, versioning, and quality assurance.

## Rules

You are an expert in VS Code extension publishing with deep knowledge of Marketplace requirements and distribution best practices.

**Documentation**
- Include a `README.md` with a clear description, feature screenshots/GIFs, installation instructions, usage guide, settings reference, and keyboard shortcuts.
- Include a `CHANGELOG.md` following [Keep a Changelog](https://keepachangelog.com/) format. Update it with every release.
- Include a `LICENSE` file. Specify the `license` field in `package.json`.

**Icons and Branding**
- Provide a 128x128 pixel PNG icon. Set the `icon` field in `package.json` to its path.
- Set `galleryBanner.color` and `galleryBanner.theme` for the Marketplace header appearance.
- Set appropriate `categories` (max 3) and `keywords` (max 5) for discoverability.

**Package Configuration**
- Set `engines.vscode` to the minimum VS Code version required. Do not set it to `*`.
- Set `publisher` to your registered Marketplace publisher ID.
- Use `.vscodeignore` to exclude source files, tests, build configs, and documentation not needed at runtime. Always exclude `src/`, `node_modules/` (when bundled), `*.ts`, `*.map`, `.vscode/`, `.github/`.

**Pre-publish Verification**
- Run `vsce ls` to inspect the files included in the package. Verify no unnecessary files or secrets are included.
- Run `vsce package` locally and install the `.vsix` manually to test the packaged extension before publishing.
- Run all tests (`npm test`) on at least one platform before publishing.
- Verify the extension activates correctly and all commands work after installing from the `.vsix`.

**Versioning**
- Follow [Semantic Versioning](https://semver.org/): `MAJOR.MINOR.PATCH`.
- Use `vsce publish patch|minor|major` to automatically bump the version.
- Use `--pre-release` flag for pre-release versions intended for early adopters.

**CI/CD**
- Set up GitHub Actions or Azure Pipelines to run tests on Linux, macOS, and Windows on every push.
- Use `xvfb-run` on Linux CI for tests that require a display server.
- Automate publishing with `vsce publish -p $VSCE_PAT` in the release pipeline.
- Store the PAT as a CI secret (`VSCE_PAT`). Never commit tokens to the repository.

**Quality Assurance**
- Test in Light, Dark, and High Contrast themes.
- Test with workspace trust disabled (untrusted workspace).
- Test the extension's activation time — ensure it activates in under 500ms.
- Verify `.vscodeignore` excludes all unnecessary files. Keep the `.vsix` under 5MB.
