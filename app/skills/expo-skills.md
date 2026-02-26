# Expo Skills

> **Source**: [GitHub](https://github.com/expo/skills) | [Landing Page](https://expo.dev/expo-skills)
> **Author**: Expo
> **License**: MIT
> **Archive Date**: 2026-02-26

Official AI agent skills by the Expo team. 10 skills across 3 plugins covering app design, deployment, and SDK upgrades. Each skill provides context-rich instructions for AI coding agents working with Expo projects.

---

## How to Use

This document is a summary. To apply in a real project, **install the original plugins**:

> **Original repo**: [expo/skills](https://github.com/expo/skills)

### Claude Code

```bash
/plugin marketplace add expo/skills
/plugin install expo-app-design@expo-skills
/plugin install expo-deployment@expo-skills
/plugin install upgrading-expo@expo-skills
```

### Cursor

Settings → Rules → Add Rule → Remote Rule (GitHub):

```
https://github.com/expo/skills.git
```

---

## Overview

| Plugin | Skill | Description |
| --- | --- | --- |
| expo-app-design | building-native-ui | Native UI guidelines — routing, styling, navigation, library preferences |
| expo-app-design | native-data-fetching | Networking — fetch, React Query, auth tokens, offline, env vars |
| expo-app-design | expo-api-routes | Server-side API routes with `+api.ts` and EAS Hosting |
| expo-app-design | expo-dev-client | Build and distribute development clients via EAS Build |
| expo-app-design | expo-tailwind-setup | Tailwind CSS v4 + NativeWind v5 setup and component wrappers |
| expo-app-design | expo-ui-swift-ui | SwiftUI views and modifiers via `@expo/ui/swift-ui` |
| expo-app-design | use-dom | DOM components — run web code in native webviews |
| expo-deployment | expo-cicd-workflows | EAS Workflows — CI/CD pipeline YAML configuration |
| expo-deployment | expo-deployment | Deploy to App Store, Play Store, web, and EAS Hosting |
| upgrading-expo | upgrading-expo | Expo SDK version upgrades and migration |

---

## Skill Summaries

### building-native-ui

Native UI guidelines for Expo apps. Covers routing conventions, styling rules, navigation patterns, and library preferences.

**Key rules:**

- Always try Expo Go first before custom builds
- Use kebab-case filenames, import aliases over relative paths
- Routes in `app/` directory only — never co-locate components there
- `expo-image` for images and SF Symbols, `expo-audio`/`expo-video` over `expo-av`
- `process.env.EXPO_OS` over `Platform.OS`, `React.use` over `React.useContext`
- Inline styles over `StyleSheet.create`, flex gap over margin
- `boxShadow` CSS prop for shadows (never legacy RN shadow props)
- `borderCurve: 'continuous'` for rounded corners
- `contentInsetAdjustmentBehavior="automatic"` on ScrollView/FlatList
- `useWindowDimensions` over `Dimensions.get()`
- Use `Link.Preview` and `Link.Menu` for context menus frequently
- Prefer `presentation: "formSheet"` for sheets/modals
- Use `NativeTabs` from `expo-router/unstable-native-tabs`

**Reference files:** animations, controls, form-sheet, gradients, icons, media, route-structure, search, storage, tabs, toolbar-and-headers, visual-effects, webgpu-three, zoom-transitions.

### native-data-fetching

Networking and data fetching patterns for Expo apps.

**Key patterns:**

- Prefer `expo/fetch` over axios
- Use React Query (TanStack Query) for complex apps, SWR for simpler needs
- Store auth tokens in `expo-secure-store` (never AsyncStorage)
- Implement token refresh with deduplication
- Use `EXPO_PUBLIC_` prefix for client-side env vars, non-prefixed for server secrets
- Use `AbortController` for request cancellation
- Offline support via `@react-native-community/netinfo` + React Query `onlineManager`
- Always check `response.ok` before parsing JSON
- Exponential backoff for retry logic

### expo-api-routes

Server-side API routes using the `+api.ts` file convention in Expo Router with EAS Hosting.

**Key points:**

- Files follow `app/api/[name]+api.ts` pattern with dynamic segments
- Named exports handle HTTP verbs (`GET`, `POST`, `PUT`, `DELETE`)
- Use `process.env` for secrets (never `EXPO_PUBLIC_` for server-side)
- Runs on Cloudflare Workers — no filesystem, no native Node modules, 30s timeout
- Use Web APIs (`crypto.subtle`) and cloud databases (Turso, Supabase)
- Local dev: `npx expo serve`, production: `eas deploy`

### expo-dev-client

Build and distribute development clients for testing custom native code.

**Key points:**

- Only needed for custom native modules, Apple targets, or third-party native modules not in Expo Go
- Configure `eas.json` with `developmentClient: true` in development profile
- Build for TestFlight, local `.ipa`/`.apk`, or both platforms
- Install via `xcrun simctl install` (simulator), `adb install` (Android), or Xcode (device)
- Connect to local Metro bundler via URL or QR code

### expo-tailwind-setup

Tailwind CSS v4 + NativeWind v5 + react-native-css setup for universal styling.

**Key points:**

- No `babel.config.js` needed (CSS-first configuration)
- Metro config with `withNativewind`, PostCSS with `@tailwindcss/postcss`
- Components must be wrapped with `useCssElement` for `className` support
- Custom CSS wrappers for View, Text, ScrollView, Pressable, Image, etc.
- Platform-specific fonts via `@media ios` / `@media android`
- Apple system colors via `platformColor()` in CSS with Tailwind theme integration
- Use `@theme` in CSS instead of `tailwind.config.js`

### expo-ui-swift-ui

Use SwiftUI views and modifiers in Expo apps via `@expo/ui/swift-ui` (SDK 55+).

**Key points:**

- Install `@expo/ui`, requires native rebuild
- Components from `@expo/ui/swift-ui`, modifiers from `@expo/ui/swift-ui/modifiers`
- Every SwiftUI tree must be wrapped in `Host`
- Use `RNHostView` to embed React Native components inside SwiftUI trees
- Missing modifiers/views can be extended via local Expo modules

### use-dom

DOM components run web code in native webviews while rendering as-is on web.

**Key points:**

- `'use dom'` directive at top of file, single default export per file
- Props must be serializable (strings, numbers, booleans, arrays, plain objects)
- Async functions can be passed as props for native-to-web communication
- Use for web-only libraries (recharts, syntax highlighters, rich text editors)
- `dom` prop controls webview config (scrollEnabled, contentInsetAdjustmentBehavior)
- Expo Router `<Link />` and `useRouter` work inside DOM components
- Route params must be passed as props from native parent
- Layout files (`_layout`) cannot be DOM components

### expo-cicd-workflows

EAS Workflows for CI/CD pipeline automation.

**Key points:**

- Workflow files live in `.eas/workflows/*.yml`
- Top-level keys: `name`, `on` (triggers), `jobs`, `defaults`, `concurrency`
- Expression syntax: `${{ }}` with contexts (github, inputs, needs, jobs, steps, workflow)
- Always fetch the JSON schema from `api.expo.dev/v2/workflows/schema` for validation
- Supports pre-packaged job types for common operations
- Validate generated YAML against schema before use

### expo-deployment

Deploy Expo apps to App Store, Play Store, web, and EAS Hosting.

**Key points:**

- Install `eas-cli` globally, authenticate with `eas login`
- iOS: `eas build -p ios --profile production`, submit with `--submit` flag
- Android: `eas build -p android --profile production`, track progression (internal → production)
- Web: EAS Hosting with preview URLs for PRs
- TestFlight shorthand: `npx testflight`
- Auto version management with `appVersionSource: "remote"`
- CI/CD via `.eas/workflows/` YAML files

### upgrading-expo

Expo SDK version upgrade guide.

**Key steps:**

1. `npx expo install expo@latest` then `npx expo install --fix`
2. `npx expo-doctor` to diagnose issues
3. Clear caches (node_modules, .expo, platform-specific)

**Key migrations:**

- `expo-av` → `expo-audio` + `expo-video`
- `expo-permissions` → individual package permission APIs
- `@expo/vector-icons` → `expo-symbols`
- SDK 54+: add `"experiments": { "reactCompiler": true }` to app.json
- Native changes require `npx expo prebuild --clean`

---

## References

- [expo/skills (GitHub)](https://github.com/expo/skills)
- [Expo Skills (Landing Page)](https://expo.dev/expo-skills)
- [Expo Documentation](https://docs.expo.dev)
- [Expo Router](https://docs.expo.dev/router/introduction/)
- [EAS Build](https://docs.expo.dev/build/introduction/)
- [EAS Workflows](https://docs.expo.dev/eas/workflows/)
