# Vercel React Best Practices

> **Source**: [GitHub - vercel-labs/agent-skills/react-best-practices](https://github.com/vercel-labs/agent-skills/tree/main/skills/react-best-practices)
> **Author**: Vercel
> **License**: MIT
> **Archive Date**: 2026-02-26

React and Next.js performance optimization guidelines: 57 rules across 8 categories, prioritized by impact for automated refactoring and code generation.

---

## How to Use

Install the skill via the Vercel agent-skills CLI or copy the `AGENTS.md` into your project root. Rules are referenced by prefix (e.g., `async-defer-await`, `bundle-barrel-imports`). Apply during code reviews, new component creation, or refactoring sessions.

## When to Apply

- Writing new React components or Next.js pages
- Implementing data fetching (server or client)
- Reviewing pull requests for performance issues
- Refactoring existing code for speed
- Optimizing bundle size and load times

## Rule Categories by Priority

| Priority | Category | Impact | Prefix | Rules |
|----------|----------|--------|--------|-------|
| 1 | Eliminating Waterfalls | CRITICAL | `async-` | 5 |
| 2 | Bundle Size Optimization | CRITICAL | `bundle-` | 5 |
| 3 | Server-Side Performance | HIGH | `server-` | 7 |
| 4 | Client-Side Data Fetching | MEDIUM-HIGH | `client-` | 4 |
| 5 | Re-render Optimization | MEDIUM | `rerender-` | 12 |
| 6 | Rendering Performance | MEDIUM | `rendering-` | 9 |
| 7 | JavaScript Performance | LOW-MEDIUM | `js-` | 12 |
| 8 | Advanced Patterns | LOW | `advanced-` | 3 |

## Critical Rules (Summary)

### 1. Eliminating Waterfalls (CRITICAL)

Waterfalls are the #1 performance killer. Each sequential `await` adds full network latency. Parallelizing can yield 2-10x improvements.

**async-defer-await** -- Move `await` after early-return guards:

```typescript
// Bad: awaits before checking skip condition
async function handleRequest(userId: string, skip: boolean) {
  const data = await fetchUserData(userId)
  if (skip) return { skipped: true }
  return processUserData(data)
}

// Good: check first, await only when needed
async function handleRequest(userId: string, skip: boolean) {
  if (skip) return { skipped: true }
  const data = await fetchUserData(userId)
  return processUserData(data)
}
```

**async-parallel** -- Use `Promise.all()` for independent operations:

```typescript
// Bad: sequential
const user = await fetchUser()
const posts = await fetchPosts()

// Good: parallel
const [user, posts] = await Promise.all([fetchUser(), fetchPosts()])
```

**async-suspense-boundaries** -- Wrap async components in `<Suspense>` so the shell renders immediately while data loads:

```tsx
function Page() {
  return (
    <div>
      <Header />
      <Suspense fallback={<Skeleton />}>
        <DataDisplay />
      </Suspense>
      <Footer />
    </div>
  )
}
```

**async-dependencies** -- For dependent fetches, start independent requests early and use `.then()` chaining or `better-all` for dependency-based parallelization.

**async-api-routes** -- Start independent promises immediately in API routes; `await` only when needed.

### 2. Bundle Size Optimization (CRITICAL)

Directly impacts Time to Interactive and Largest Contentful Paint.

**bundle-barrel-imports** -- Avoid barrel file imports; use direct paths or `optimizePackageImports`:

```typescript
// Bad: pulls entire library
import { Check } from 'lucide-react'

// Good: direct import
import Check from 'lucide-react/dist/esm/icons/check'

// Good: Next.js config
// next.config.js
module.exports = {
  experimental: { optimizePackageImports: ['lucide-react'] }
}
```

**bundle-dynamic-imports** -- Use `next/dynamic` for heavy components (editors, charts):

```tsx
const MonacoEditor = dynamic(
  () => import('./monaco-editor').then(m => m.MonacoEditor),
  { ssr: false }
)
```

**bundle-defer-third-party** -- Wrap non-critical third-party components (analytics, chat widgets) in `dynamic(() => ..., { ssr: false })`.

**bundle-preload** -- Preload modules on hover/focus for perceived performance.

**bundle-conditional** -- Conditionally load modules only when a feature is active.

## Key Rules (Summary)

### 3. Server-Side Performance (HIGH)

- **server-auth-actions**: Always authenticate server actions like API routes; validate inputs with Zod.
- **server-cache-react**: Use `React.cache()` for per-request deduplication of expensive queries.
- **server-cache-lru**: Use LRU cache for cross-request caching with TTL.
- **server-dedup-props**: Avoid duplicate serialization; compute derived data on the client with `useMemo`.
- **server-serialization**: Pass only needed fields across RSC boundaries, not entire objects.
- **server-parallel-fetching**: Use component composition so sibling async components fetch in parallel.
- **server-after-nonblocking**: Use `after()` from `next/server` for logging/analytics that should not block the response.

### 4. Client-Side Data Fetching (MEDIUM-HIGH)

- **client-swr-dedup**: Use SWR for automatic request deduplication, caching, and revalidation.
- **client-event-listeners**: Deduplicate global event listeners with a shared registry.
- **client-passive-event-listeners**: Use `{ passive: true }` for touch/wheel handlers.
- **client-localstorage-schema**: Version localStorage keys and store minimal data with migration logic.

### 5. Re-render Optimization (MEDIUM)

- **rerender-derived-state-no-effect**: Compute derived values inline instead of syncing with `useEffect`.
- **rerender-functional-setstate**: Use `setItems(curr => [...curr, item])` to remove state from callback dependencies.
- **rerender-lazy-state-init**: Pass initializer functions to `useState(() => expensiveCalc())`.
- **rerender-transitions**: Wrap non-urgent updates in `startTransition`.
- **rerender-use-ref-transient-values**: Use refs for high-frequency values (mouse position) that drive DOM mutations.
- **rerender-defer-reads**: Read state at usage point (event handler) instead of render time.
- **rerender-memo**: Extract to `memo()` components instead of wrapping JSX in `useMemo`.
- **rerender-memo-with-default-value**: Hoist default non-primitive props to constants for memoized components.
- **rerender-dependencies**: Narrow effect dependencies to primitive fields (e.g., `user.id` not `user`).
- **rerender-move-effect-to-event**: Put interaction logic in event handlers, not effects.
- **rerender-simple-expression-in-memo**: Do not wrap simple boolean expressions in `useMemo`.
- **rerender-derived-state**: Subscribe to derived state (e.g., `useMediaQuery` vs raw width).

### 6. Rendering Performance (MEDIUM)

- **rendering-hydration-no-flicker**: Use inline `<script>` to set theme before hydration.
- **rendering-hydration-suppress-warning**: Use `suppressHydrationWarning` for timestamps.
- **rendering-content-visibility**: Apply `content-visibility: auto` to long lists.
- **rendering-hoist-jsx**: Hoist static JSX to module-level constants.
- **rendering-conditional-render**: Use explicit `count > 0 ?` instead of `count &&` to avoid rendering `0`.
- **rendering-usetransition-loading**: Prefer `useTransition` over manual loading state booleans.
- **rendering-activity**: Use React `<Activity>` component for show/hide without unmounting.
- **rendering-animate-svg-wrapper**: Animate a wrapper `<div>` instead of the `<svg>` element.
- **rendering-svg-precision**: Reduce SVG path decimal precision.

### 7. JavaScript Performance (LOW-MEDIUM)

- **js-index-maps**: Build `Map` from arrays for O(1) lookups instead of `.find()`.
- **js-set-map-lookups**: Use `Set.has()` instead of `Array.includes()`.
- **js-combine-iterations**: Combine multiple `.filter()` calls into a single loop.
- **js-early-exit**: Return early from functions on first failure.
- **js-batch-dom-css**: Batch DOM writes; avoid interleaving reads and writes.
- **js-cache-function-results**: Cache repeated pure function calls with a Map.
- **js-cache-storage**: Cache `localStorage`/cookie reads in memory.
- **js-cache-property-access**: Hoist property access out of loops.
- **js-hoist-regexp**: Hoist `RegExp` creation to module scope or `useMemo`.
- **js-min-max-loop**: Use a loop for min/max instead of sorting the array.
- **js-length-check-first**: Check array lengths before element-by-element comparison.
- **js-tosorted-immutable**: Use `toSorted()` or spread+sort instead of mutating `sort()`.

### 8. Advanced Patterns (LOW)

- **advanced-init-once**: Use a module-level flag to run initialization only once.
- **advanced-event-handler-refs**: Use `useEffectEvent` to keep event handlers stable without listing them in dependencies.
- **advanced-use-latest**: Use `useEffectEvent` for stable callback refs in debounced effects.

## Problem to Rule Mapping

| Problem | Rule(s) |
|---------|---------|
| Slow page load / TTFB | `async-parallel`, `async-api-routes`, `server-parallel-fetching` |
| Large JS bundle | `bundle-barrel-imports`, `bundle-dynamic-imports`, `bundle-defer-third-party` |
| Waterfall requests | `async-defer-await`, `async-dependencies`, `async-suspense-boundaries` |
| Unnecessary re-renders | `rerender-derived-state-no-effect`, `rerender-functional-setstate`, `rerender-memo` |
| Hydration mismatch | `rendering-hydration-no-flicker`, `rendering-hydration-suppress-warning` |
| Jank / layout thrashing | `js-batch-dom-css`, `rerender-use-ref-transient-values` |
| Stale or duplicate data | `client-swr-dedup`, `server-cache-react`, `server-cache-lru` |
| Unresponsive UI | `rerender-transitions`, `rendering-usetransition-loading` |
| Security gaps in server actions | `server-auth-actions` |
| Large RSC payloads | `server-serialization`, `server-dedup-props` |
| Slow list rendering | `rendering-content-visibility`, `js-combine-iterations`, `js-index-maps` |
| Excessive effect runs | `rerender-dependencies`, `rerender-move-effect-to-event`, `advanced-event-handler-refs` |

---

## References

- [vercel-labs/agent-skills GitHub](https://github.com/vercel-labs/agent-skills/tree/main/skills/react-best-practices)
- [AGENTS.md (full rules)](https://github.com/vercel-labs/agent-skills/blob/main/skills/react-best-practices/AGENTS.md)
- [SKILL.md (metadata)](https://github.com/vercel-labs/agent-skills/blob/main/skills/react-best-practices/SKILL.md)
- [README.md (repo structure)](https://github.com/vercel-labs/agent-skills/blob/main/skills/react-best-practices/README.md)
