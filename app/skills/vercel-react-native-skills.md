# Vercel React Native Skills

> **Source**: [GitHub](https://github.com/vercel-labs/agent-skills/tree/main/skills/react-native-skills)
> **Author**: Vercel
> **License**: MIT
> **Archive Date**: 2026-02-26

Performance optimization skills for React Native & Expo. 13 categories, 35+ rules. Each rule includes incorrect/correct code examples. Designed for AI agents and LLMs to reference when generating or refactoring RN code.

---

## How to Use

This document is a summary. To apply in a real project, **copy the original skills**:

> **Original repo**: [vercel-labs/agent-skills/skills/react-native-skills](https://github.com/vercel-labs/agent-skills/tree/main/skills/react-native-skills)

- **AGENTS.md** — A single compiled document with all 35+ rules. Copy to project root for immediate use
- **rules/** — Individual rule files. Selectively copy only the rules you need
- **SKILL.md** — Agent skill definition. Register as a skill in Claude Code, Cursor, etc.

### Quick Install

```bash
# Clone repo and copy AGENTS.md to your project
curl -o AGENTS.md https://raw.githubusercontent.com/vercel-labs/agent-skills/main/skills/react-native-skills/AGENTS.md
```

### Claude Code

```bash
/plugin marketplace add vercel-labs/agent-skills
/plugin install react-native-skills@vercel-agent-skills
```

### Cursor

Settings → Rules → Add Rule → Remote Rule (GitHub):

```
https://github.com/vercel-labs/agent-skills.git
```

---

## Overview

### Claude Code

```bash
/plugin marketplace add vercel-labs/agent-skills
/plugin install react-native-skills@vercel-agent-skills
```

### Cursor

Settings → Rules → Add Rule → Remote Rule (GitHub):

```
https://github.com/vercel-labs/agent-skills.git
```

### npx

```bash
npx add-skill vercel-labs/agent-skills
```

---

## When to Apply

- React Native or Expo app development
- List and scroll performance optimization
- Reanimated animation implementation
- Image/media handling
- Native module or font configuration
- Monorepo native dependency management

---

## Rule Categories by Priority

| Priority | Category | Impact | Prefix | Rules |
| --- | --- | --- | --- | --- |
| 1 | Core Rendering | CRITICAL | `rendering-` | 2 |
| 2 | List Performance | HIGH | `list-performance-` | 8 |
| 3 | Animation | HIGH | `animation-` | 3 |
| 4 | Scroll Performance | HIGH | `scroll-` | 1 |
| 5 | Navigation | HIGH | `navigation-` | 1 |
| 6 | React State | MEDIUM | `react-state-` | 3 |
| 7 | State Architecture | MEDIUM | `state-` | 1 |
| 8 | React Compiler | MEDIUM | `react-compiler-` | 2 |
| 9 | User Interface | MEDIUM | `ui-` | 9 |
| 10 | Design System | MEDIUM | `design-system-` | 1 |
| 11 | Monorepo | LOW | `monorepo-` | 2 |
| 12 | Third-Party Deps | LOW | `imports-` | 1 |
| 13 | JavaScript | LOW | `js-` | 1 |
| 14 | Fonts | LOW | `fonts-` | 1 |

---

## Critical Rules (Summary)

### 1. Prevent Rendering Crashes

**No falsy values with `&&` operator** — `0` or `""` rendered in JSX causes production crashes.

```tsx
// Bad: crashes when count is 0
{count && <Text>{count} items</Text>}

// Good: use ternary operator
{count ? <Text>{count} items</Text> : null}
```

**Strings must be inside `<Text>`** — Placing a string directly under `<View>` causes a crash.

### 2. List Performance

**Virtualization required** — Use `LegendList` or `FlashList` instead of `ScrollView` + `map`.

**Maintain stable references** — Do not recreate data passed to lists with `map`/`filter`. Virtualization depends on reference stability.

**No inline objects in renderItem** — New object references invalidate memoization. Pass primitive values or the item directly.

**Create callbacks at root** — Do not create new callbacks per item. Create once with `useCallback`.

**Keep items lightweight** — No queries, excessive Context access, or expensive computations in list items. Use Zustand selectors.

### 3. Animation

**Animate only GPU properties** — Use `transform` (scale, translate) + `opacity` instead of `width`/`height`/`margin`.

```tsx
// Bad: layout recalculation every frame
{ height: withTiming(expanded ? 200 : 0) }

// Good: GPU accelerated
{ transform: [{ scaleY: withTiming(expanded ? 1 : 0) }] }
```

**Use GestureDetector for press animations** — Use `Gesture.Tap()` + shared value for UI thread execution instead of `Pressable`'s `onPressIn/Out`.

### 4. Scroll

**Never store scroll position in useState** — Causes re-render every frame. Use Reanimated shared value or `useRef`.

### 5. Navigation

**Use native navigators** — Use `native-stack` instead of `@react-navigation/stack`. Use `react-native-bottom-tabs` instead of `@react-navigation/bottom-tabs`.

---

## Key Rules (Summary)

### React State

- **Use dispatch updater**: `setState(prev => ...)` — prevents stale closures
- **Fallback state pattern**: `useState(undefined)` + `??` — reactive defaults
- **Minimize state**: Do not store derivable values in state; compute during render

### State Architecture

- **State stores ground truth**: Store actual state like `pressed`, `isOpen`. Derive visual values like `scale`, `opacity` via interpolate

### React Compiler

- **Destructure functions early**: Use `const { push } = useRouter()` instead of `router.push()` — stable references
- **Use Reanimated `.get()`/`.set()`**: Instead of `.value` — React Compiler compatible

### UI Patterns

- **Use expo-image**: Instead of RN `Image` — blurhash, caching, progressive loading
- **Use Galeria for image galleries**: Shared element transition, pinch-to-zoom
- **Use Zeego for native menus**: Native dropdown/context menus instead of custom JS menus
- **Use native Modal**: `presentationStyle="formSheet"` instead of JS bottom sheet
- **Use Pressable**: Instead of `TouchableOpacity`
- **`contentInsetAdjustmentBehavior="automatic"`**: Instead of wrapping with `SafeAreaView`
- **`contentInset` for dynamic spacing**: Instead of padding — avoids layout recalculation
- **Modern styling**: `gap`, `borderCurve: 'continuous'`, `boxShadow` CSS syntax, `experimental_backgroundImage`

### Design System

- **Compound Components pattern**: `Button` + `ButtonText` + `ButtonIcon` — explicit composition instead of polymorphic children

### Monorepo

- **Install native dependencies in app directory**: Autolinking only scans the app's `node_modules`
- **Single version management**: Use identical versions across the monorepo, specify exact versions

### Other

- **Hoist Intl formatters**: Create at module scope — prevents recreation on every render
- **Import from design system folder**: Use re-export wrappers instead of direct third-party package imports
- **Fonts via config plugin**: Use `expo-font` config plugin for build-time embedding instead of `useFonts`

---

## Problem → Rule Mapping

| Problem | Start With |
| --- | --- |
| Production crashes | `rendering-no-falsy-and`, `rendering-text-in-text-component` |
| List scroll jank | `list-performance-virtualize` → `list-performance-function-references` |
| Animation frame drops | `animation-gpu-properties` → `scroll-position-no-state` |
| Slow press response | `animation-gesture-detector-press` |
| Complex state management | `react-state-minimize` → `state-ground-truth` |
| React Compiler compatibility | `react-compiler-destructure-functions`, `react-compiler-reanimated-shared-values` |
| Image performance | `ui-expo-image` → `list-performance-images` |
| Slow navigation | `navigation-native-navigators` |
| Monorepo autolinking failure | `monorepo-native-deps-in-app` |

---

## References

- [vercel-labs/agent-skills (Full Source)](https://github.com/vercel-labs/agent-skills/tree/main/skills/react-native-skills)
- [React Native](https://reactnative.dev)
- [Expo](https://docs.expo.dev)
- [Reanimated](https://docs.swmansion.com/react-native-reanimated)
- [Gesture Handler](https://docs.swmansion.com/react-native-gesture-handler)
- [LegendList](https://legendapp.com/open-source/legend-list)
- [Galeria](https://github.com/nandorojo/galeria)
- [Zeego](https://zeego.dev)
