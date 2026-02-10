# Callstack React Native Best Practices

> **Source**: [GitHub](https://github.com/callstackincubator/agent-skills)
> **Author**: Callstack
> **License**: MIT
> **Archive Date**: 2026-02-10

Performance optimization skills for React Native apps. 27+ structured skills across JavaScript, Native, and Bundling categories. Based on Callstack's "The Ultimate Guide to React Native Optimization."

## Installation

### Claude Code

```bash
/plugin marketplace add callstackincubator/agent-skills
/plugin install react-native-best-practices@callstack-agent-skills
```

### Cursor

Settings → Rules → Add Rule → Remote Rule (GitHub):

```
https://github.com/callstackincubator/agent-skills.git
```

### Gemini CLI

```bash
gemini skills install https://github.com/callstackincubator/agent-skills.git
```

### npx

```bash
npx add-skill callstackincubator/agent-skills
```

## When to Apply

- Debugging slow/janky UI or animations
- Investigating memory leaks (JS or native)
- Optimizing app startup time (TTI)
- Reducing bundle or app size
- Writing native modules (Turbo Modules)
- Profiling React Native performance

## Priority Order

| Priority | Category | Impact |
| --- | --- | --- |
| 1 | FPS & Re-renders | CRITICAL |
| 2 | Bundle Size | CRITICAL |
| 3 | TTI Optimization | HIGH |
| 4 | Native Performance | HIGH |
| 5 | Memory Management | MEDIUM-HIGH |
| 6 | Animations | MEDIUM |

## Skill List

### JavaScript/React (`js-*`) — 9 skills

| File | Impact | Description |
| --- | --- | --- |
| js-lists-flatlist-flashlist.md | CRITICAL | Replace ScrollView with virtualized lists (FlatList/FlashList) |
| js-profile-react.md | MEDIUM | React DevTools profiling |
| js-measure-fps.md | HIGH | FPS monitoring and measurement |
| js-memory-leaks.md | MEDIUM | JS memory leak hunting |
| js-atomic-state.md | HIGH | Jotai/Zustand patterns for atomic state |
| js-concurrent-react.md | HIGH | useDeferredValue, useTransition |
| js-react-compiler.md | HIGH | Automatic memoization via React Compiler |
| js-animations-reanimated.md | MEDIUM | Reanimated worklets for smooth animations |
| js-uncontrolled-components.md | HIGH | TextInput optimization |

### Native (`native-*`) — 10 skills

| File | Impact | Description |
| --- | --- | --- |
| native-turbo-modules.md | HIGH | Building fast native modules |
| native-sdks-over-polyfills.md | HIGH | Prefer native SDKs over web polyfills |
| native-measure-tti.md | HIGH | TTI measurement setup |
| native-threading-model.md | HIGH | Turbo Module threading patterns |
| native-profiling.md | MEDIUM | Xcode/Android Studio profiling |
| native-platform-setup.md | MEDIUM | iOS/Android tooling guide |
| native-view-flattening.md | MEDIUM | View hierarchy debugging |
| native-memory-patterns.md | MEDIUM | C++/Swift/Kotlin memory management |
| native-memory-leaks.md | MEDIUM | Native memory leak hunting |
| native-android-16kb-alignment.md | CRITICAL | 16KB page alignment for Google Play |

### Bundling (`bundle-*`) — 9 skills

| File | Impact | Description |
| --- | --- | --- |
| bundle-barrel-exports.md | CRITICAL | Avoid barrel imports |
| bundle-analyze-js.md | CRITICAL | JS bundle visualization |
| bundle-tree-shaking.md | HIGH | Dead code elimination |
| bundle-analyze-app.md | HIGH | App size analysis |
| bundle-r8-android.md | HIGH | Android R8 code shrinking |
| bundle-hermes-mmap.md | HIGH | Disable bundle compression for Hermes mmap |
| bundle-native-assets.md | HIGH | Asset catalog setup |
| bundle-library-size.md | MEDIUM | Evaluate dependency size impact |
| bundle-code-splitting.md | MEDIUM | Re.Pack code splitting |

## Problem → Skill Mapping

| Problem | Start With |
| --- | --- |
| App feels slow/janky | js-measure-fps → js-profile-react |
| Too many re-renders | js-profile-react → js-react-compiler |
| Slow startup (TTI) | native-measure-tti → bundle-analyze-js |
| Large app size | bundle-analyze-app → bundle-r8-android |
| Memory growing | js-memory-leaks or native-memory-leaks |
| Animation drops frames | js-animations-reanimated |
| List scroll jank | js-lists-flatlist-flashlist |
| TextInput lag | js-uncontrolled-components |
| Native module slow | native-turbo-modules → native-threading-model |

## Quick Reference

### FPS & Re-renders

```bash
# Open React Native DevTools
# Press 'j' in Metro, or shake device → "Open DevTools"
```

- Replace ScrollView with FlatList/FlashList for lists
- Use React Compiler for automatic memoization
- Use atomic state (Jotai/Zustand) to reduce re-renders
- Use `useDeferredValue` for expensive computations

### Bundle Size

```bash
npx react-native bundle \
  --entry-file index.js \
  --bundle-output output.js \
  --platform ios \
  --sourcemap-output output.js.map \
  --dev false --minify true

npx source-map-explorer output.js --no-border-checks
```

- Avoid barrel imports (import directly from source)
- Remove unnecessary Intl polyfills (Hermes has native support)
- Enable tree shaking (Expo SDK 52+ or Re.Pack)
- Enable R8 for Android native code shrinking

### TTI

- Use `react-native-performance` for markers
- Only measure cold starts (exclude warm/hot/prewarm)
- Disable JS bundle compression on Android (enables Hermes mmap)
- Use native navigation (react-native-screens)
