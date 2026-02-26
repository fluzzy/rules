# React Composition Patterns

> **Source**: [GitHub - vercel-labs/agent-skills/composition-patterns](https://github.com/vercel-labs/agent-skills/tree/main/skills/composition-patterns)
> **Author**: Vercel
> **License**: MIT
> **Archive Date**: 2026-02-26

Composition patterns for building flexible, maintainable React components. Eliminates boolean prop proliferation by using compound components, lifted state, and dependency-injectable context providers.

---

## How to Use

1. Copy the rules into your project's agent instructions (e.g., `AGENTS.md` or `.cursor/rules/`).
2. Reference when building or refactoring component libraries with complex variant logic.
3. Patterns target React 19+ but the composition principles apply to any React version.

## When to Apply

- Refactoring components that have accumulated multiple boolean props (`isThread`, `isEditing`, etc.)
- Building reusable component libraries or design systems
- Designing flexible component APIs that need multiple variants
- Components where sibling/external elements need access to shared state
- Migrating to React 19 APIs (`use()`, ref as prop)

## Pattern Categories

| # | Category | Impact | Key Idea |
|---|----------|--------|----------|
| 1 | Component Architecture | **HIGH** | Eliminate boolean props; use compound components |
| 2 | State Management | **MEDIUM** | Decouple state from UI; lift into providers |
| 3 | Implementation Patterns | **MEDIUM** | Explicit variants; prefer children over render props |
| 4 | React 19 APIs | **MEDIUM** | Drop `forwardRef`; use `use()` instead of `useContext()` |

---

## Critical Patterns (Summary)

### 1. Avoid Boolean Prop Proliferation (CRITICAL)

Each boolean prop doubles possible states. Instead of one component with flags like `isThread`, `isEditing`, `isDMThread`, create explicit composed variants.

**Bad** -- boolean props create exponential complexity:

```tsx
function Composer({ isThread, isEditing, isDMThread, ... }: Props) {
  return (
    <form>
      {isDMThread ? <AlsoSendToDMField /> : isThread ? <AlsoSendToChannelField /> : null}
      {isEditing ? <EditActions /> : <DefaultActions />}
    </form>
  )
}
```

**Good** -- each variant is explicit and self-contained:

```tsx
function ThreadComposer({ channelId }: { channelId: string }) {
  return (
    <Composer.Frame>
      <Composer.Input />
      <AlsoSendToChannelField id={channelId} />
      <Composer.Footer>
        <Composer.Submit />
      </Composer.Footer>
    </Composer.Frame>
  )
}

function EditComposer() {
  return (
    <Composer.Frame>
      <Composer.Input />
      <Composer.Footer>
        <Composer.CancelEdit />
        <Composer.SaveEdit />
      </Composer.Footer>
    </Composer.Frame>
  )
}
```

### 2. Compound Components with Shared Context (HIGH)

Structure complex components as a namespace of subcomponents sharing state via context.

```tsx
const ComposerContext = createContext<ComposerContextValue | null>(null)

function ComposerProvider({ children, state, actions, meta }: ProviderProps) {
  return (
    <ComposerContext value={{ state, actions, meta }}>
      {children}
    </ComposerContext>
  )
}

// Export as compound component
const Composer = {
  Provider: ComposerProvider,
  Frame: ComposerFrame,
  Input: ComposerInput,
  Submit: ComposerSubmit,
  Footer: ComposerFooter,
}
```

### 3. Generic Context Interface for Dependency Injection (HIGH)

Define a three-part interface (`state`, `actions`, `meta`) that any provider can implement. UI components depend on the interface, not the implementation.

```tsx
interface ComposerContextValue {
  state: { input: string; attachments: Attachment[]; isSubmitting: boolean }
  actions: { update: (fn: (s: State) => State) => void; submit: () => void }
  meta: { inputRef: React.RefObject<TextInput> }
}
```

Different providers (local state, global sync, server state) implement the same interface. The same `<Composer.Input />` works with all of them.

---

## Key Patterns (Summary)

### 4. Decouple State from UI

- Provider handles all state management (useState, Zustand, server sync).
- UI components only consume the context interface.
- Swap providers without touching UI code.

### 5. Lift State into Provider Components

- Move state into a dedicated provider so sibling components outside the main UI can access it.
- Avoids `useEffect` syncing, ref-based hacks, and prop callbacks.
- **Key insight**: Components sharing state only need to be within the same provider, not visually nested together.

### 6. Explicit Component Variants

| Instead of | Use |
|------------|-----|
| `<Composer isThread isEditing={false} channelId="abc" />` | `<ThreadComposer channelId="abc" />` |
| `<Composer isEditing messageId="xyz" />` | `<EditMessageComposer messageId="xyz" />` |

Each variant is self-documenting with no impossible states.

### 7. Prefer Children over Render Props

- Use `children` for composing static structure.
- Reserve render props for cases where the parent must pass data back to children (e.g., `renderItem` in lists).

### 8. React 19 API Updates

| Old (React 18) | New (React 19+) |
|----------------|-----------------|
| `forwardRef<T, P>((props, ref) => ...)` | `function Comp({ ref, ...props }: Props & { ref?: Ref<T> })` |
| `useContext(MyContext)` | `use(MyContext)` (can be called conditionally) |

---

## References

- [React Documentation](https://react.dev)
- [Passing Data Deeply with Context](https://react.dev/learn/passing-data-deeply-with-context)
- [React `use()` API Reference](https://react.dev/reference/react/use)
- [Source Repository](https://github.com/vercel-labs/agent-skills/tree/main/skills/composition-patterns)
