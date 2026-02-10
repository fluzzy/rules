# Maestro

> **Source**: [GitHub](https://github.com/mobile-dev-inc/Maestro) | [Docs](https://docs.maestro.dev)
> **Archive Date**: 2026-02-10

E2E testing framework for mobile and web. YAML-based test flows with built-in flakiness tolerance, AI-powered assertions, and MCP server for AI agent integration.

## Installation

### Maestro CLI

```bash
curl -Ls "https://get.maestro.mobile.dev" | bash
```

### MCP Server (Claude Code)

```json
{
  "mcpServers": {
    "maestro": {
      "command": "maestro",
      "args": ["mcp"]
    }
  }
}
```

## MCP Tools

### Device Management

| Tool | Description |
| --- | --- |
| `list_devices` | List connected devices and emulators |
| `take_screenshot` | Capture device screenshot (returns JPEG) |

### UI Interaction

| Tool | Description |
| --- | --- |
| `tap_on` | Tap element by text, ID, or selector |
| `long_press_on` | Long press on element |
| `input_text` | Enter text into focused field |
| `erase_text` | Remove characters from text field |
| `swipe` | Perform swipe gesture |
| `scroll` | Scroll within views |
| `back` | Navigate back (Android) |

### Inspection & Assertion

| Tool | Description |
| --- | --- |
| `inspect_ui` | Get full UI element hierarchy |
| `assert_visible` | Assert element is visible |
| `assert_not_visible` | Assert element is not visible |
| `assert_with_ai` | AI-powered visual assertion |

### App & Flow Management

| Tool | Description |
| --- | --- |
| `launch_app` | Launch app by ID |
| `stop_app` | Stop running app |
| `clear_state` | Clear app data |
| `run_flow` | Execute a Maestro flow file |

## YAML Flow Syntax

### Basic Flow

```yaml
appId: com.example.app
---
- launchApp
- tapOn: "Login"
- inputText: "user@example.com"
- tapOn: "Password"
- inputText: "password123"
- tapOn: "Submit"
- assertVisible: "Welcome"
```

### AI-Powered Assertion

```yaml
- assertWithAI:
    assertion: A two-factor authentication prompt, with space for 6 digits, is visible.
```

### AI Defect Detection

```yaml
- assertNoDefectsWithAi
```

### Running Flows

```bash
maestro test flow.yaml          # Run single flow
maestro test flows/             # Run all flows in directory
maestro test --format junit     # JUnit output for CI
```

## Key Features

- **Built-in flakiness tolerance**: Auto-retries taps/waits if element isn't immediately present
- **No sleep() needed**: Automatic waiting for UI elements
- **AI assertions**: LLM-powered visual verification via `assertWithAI`
- **AI defect detection**: Automatic detection of UI defects (clipped text, overlaps)
- **Cross-platform**: iOS simulators, Android emulators, real devices
- **43 commands**: Full set including scroll, swipe, permissions, location, recording, media

## Full Command Reference

| Command | Description |
| --- | --- |
| `launchApp` | Launch app with options |
| `stopApp` | Stop running app |
| `killApp` | Kill app process (Android) |
| `clearState` | Clear app data |
| `tapOn` | Tap by text/ID with wait |
| `doubleTapOn` | Double tap element |
| `longPressOn` | Long press element |
| `inputText` | Enter text (supports random data) |
| `eraseText` | Remove characters |
| `pasteText` | Paste from clipboard |
| `copyTextFrom` | Copy text from element |
| `assertVisible` | Assert element visible |
| `assertNotVisible` | Assert element not visible |
| `assertTrue` | Assert JS expression is true |
| `assertWithAI` | AI visual assertion |
| `assertNoDefectsWithAi` | AI defect detection |
| `extractTextWithAI` | AI text extraction from images |
| `scroll` | Scroll in view |
| `scrollUntilVisible` | Scroll until element appears |
| `swipe` | Swipe gesture |
| `back` | Navigate back |
| `pressKey` | Press device key |
| `hideKeyboard` | Hide virtual keyboard |
| `openLink` | Open URL/deep link |
| `setLocation` | Set device GPS location |
| `setOrientation` | Set screen orientation |
| `setPermissions` | Set app permissions |
| `setAirplaneMode` | Set airplane mode |
| `toggleAirplaneMode` | Toggle airplane mode |
| `setClipboard` | Set clipboard text |
| `addMedia` | Add media to device gallery |
| `takeScreenshot` | Capture screenshot |
| `startRecording` | Begin screen recording |
| `stopRecording` | End screen recording |
| `clearKeychain` | Clear iOS keychain |
| `evalScript` | Run inline JavaScript |
| `runScript` | Run external script |
| `runFlow` | Run sub-flow from file |
| `repeat` | Repeat commands |
| `retry` | Retry failed commands |
| `extendedWaitUntil` | Wait with custom timeout |
| `waitForAnimationToEnd` | Wait for animations |
| `travel` | Simulate device travel |
