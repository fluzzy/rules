# Callstack Ultimate Guide to React Native Optimization

> **Source**: [Callstack](https://www.callstack.com/ebooks/the-ultimate-guide-to-react-native-optimization)
> **Code Examples**: [GitHub](https://github.com/callstack/optimization-best-practices)
> **Edition**: 2025
> **Archive Date**: 2026-02-10

Free ebook covering React Native performance optimization across JavaScript, Native, and Bundling. Updated for 2025 with New Architecture, React Compiler, and Hermes optimizations.

## Part 1: JavaScript Optimization

### Profiling & Measurement

- **Profile JS/React code**: React DevTools Profiler, Performance panel
- **Measure FPS**: Performance Monitor, custom FPS tracking
- **Hunt JS memory leaks**: Heap snapshots, allocation tracking

### React Patterns

- **Uncontrolled components**: Avoid re-renders on TextInput by using refs
- **Higher-order specialized components**: Dedicated components for specific layouts
- **Atomic state management**: Jotai/Zustand for granular updates, avoid global re-renders
- **Concurrent React**: `useDeferredValue`, `useTransition` for expensive computations
- **React Compiler**: Automatic memoization (React 19+)

### Animations

- **High-performance animations**: Reanimated worklets on UI thread, avoid JS thread

## Part 2: Native Optimization

### Platform-Specific

- **iOS/Android profiling**: Xcode Instruments (Time Profiler), Android Studio (CPU Profiler)
- **TTI measurement**: `react-native-performance` markers, cold start only
- **Native memory leaks**: Platform-specific detection and prevention

### Architecture

- **View flattening**: Reduce view hierarchy depth for rendering performance
- **TurboModules**: Type-safe native module communication, lazy loading, JSI bindings
- **Threading model**: Background threads for heavy native work, async over sync methods
- **C++ for cross-platform**: Performance-critical code shared across platforms
- **Native SDKs over polyfills**: Prefer dedicated RN SDKs over web library polyfills

### Android-Specific

- **16KB page alignment**: Required for Google Play compliance with third-party libraries

## Part 3: Bundling Optimization

### Analysis

- **Bundle analysis**: `source-map-explorer` for JS bundle visualization
- **App size analysis**: Full binary inspection (not just JS bundle)
- **Library size evaluation**: Check dependency impact before adding

### Optimization

- **Barrel exports**: Avoid index re-exports, import directly from source modules
- **Tree shaking**: Dead code elimination (Expo SDK 52+ or Re.Pack)
- **R8 code shrinking**: Android `minifyEnabled` + `shrinkResources` in release builds
- **Hermes memory mapping**: Disable JS bundle compression on Android for faster startup
- **Native asset management**: Use asset catalogs for optimized delivery
- **Code splitting**: Re.Pack for lazy loading in very large apps

## 2025 Edition Updates

- React 19 hooks (`use`, `useOptimistic`)
- React Compiler for automatic memoization
- Hermes configuration for ~12% TTI improvement
- Native view flattening techniques
- New Architecture (Fabric, TurboModules, JSI) optimization strategies
- Advanced profiling with React Native DevTools

## Quick Wins Checklist

1. Replace ScrollView with FlatList/FlashList for lists
2. Enable React Compiler for automatic memoization
3. Avoid barrel imports (import from source)
4. Enable R8 for Android release builds
5. Disable JS bundle compression on Android (Hermes mmap)
6. Use `useDeferredValue` for expensive computations
7. Use Reanimated for animations (not Animated API)
8. Profile before optimizing (measure, don't guess)
