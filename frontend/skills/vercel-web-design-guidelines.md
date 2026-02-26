# Vercel Web Design Guidelines

> **Source**: [GitHub](https://github.com/vercel-labs/agent-skills/tree/main/skills/web-design-guidelines)
> **Author**: Vercel
> **License**: MIT
> **Archive Date**: 2026-02-26

A CLI-based audit skill that reviews UI code against Vercel's Web Interface Guidelines. It fetches the latest ruleset, analyzes specified files, and reports compliance findings in a concise `file:line` format.

---

## How to Use

The skill works as a code review tool inside an AI coding agent:

1. Provide file paths or glob patterns to review
2. The agent fetches the latest guidelines from the [source rules](https://raw.githubusercontent.com/vercel-labs/web-interface-guidelines/main/command.md)
3. Each file is checked against all rule categories
4. Findings are reported per file in `file:line` format (VS Code clickable)

**Output format example:**

```text
## src/Button.tsx
src/Button.tsx:42 - icon button missing aria-label
src/Button.tsx:55 - animation missing prefers-reduced-motion

## src/Card.tsx
✓ pass
```

## When to Apply

- Reviewing UI components for accessibility compliance
- Auditing forms, modals, or interactive elements
- Checking animation and performance best practices
- Validating dark mode, i18n, and hydration safety
- Pre-merge code review for frontend pull requests

---

## Rule Categories

| Category | Focus Area |
| --- | --- |
| Accessibility | ARIA labels, semantic HTML, keyboard handlers, skip links |
| Focus States | Visible focus rings, `:focus-visible`, `:focus-within` |
| Forms | Autocomplete, input types, paste handling, error placement |
| Animation | `prefers-reduced-motion`, compositor-friendly transforms |
| Typography | Curly quotes, ellipsis, `tabular-nums`, `text-wrap: balance` |
| Content Handling | Truncation, empty states, long content overflow |
| Images | Dimensions, lazy loading, `fetchpriority` |
| Performance | Virtualization, no layout reads in render, preconnect/preload |
| Navigation & State | URL-synced state, deep-linking, destructive action guards |
| Touch & Interaction | `touch-action`, `overscroll-behavior`, drag behavior |
| Safe Areas & Layout | `env(safe-area-inset-*)`, flex/grid over JS measurement |
| Dark Mode & Theming | `color-scheme`, `theme-color`, native select styling |
| Locale & i18n | `Intl.DateTimeFormat`, `Intl.NumberFormat`, language detection |
| Hydration Safety | Controlled vs uncontrolled inputs, date rendering guards |
| Hover & States | Hover feedback, progressive contrast for interactive states |
| Content & Copy | Active voice, Title Case, specific labels, error messages |

---

## Critical Rules (Summary)

### Accessibility

- Icon-only buttons need `aria-label`
- Form controls need `<label>` or `aria-label`
- Interactive elements need keyboard handlers (`onKeyDown`/`onKeyUp`)
- Use `<button>` for actions, `<a>`/`<Link>` for navigation -- never `<div onClick>`
- Use semantic HTML before ARIA; headings must be hierarchical `<h1>`-`<h6>`

### Focus States

- Never `outline-none` without a focus replacement
- Use `:focus-visible` over `:focus` (avoid focus ring on click)

```css
/* Good */
button:focus-visible { ring-2 ring-blue-500; }
/* Bad */
button { outline: none; }
```

### Forms

- Inputs need `autocomplete` and correct `type` (`email`, `tel`, `url`)
- Never block paste (`onPaste` + `preventDefault`)
- Submit button stays enabled until request starts; show spinner during request
- Errors inline next to fields; focus first error on submit

### Performance

- Large lists (>50 items): virtualize (`virtua`, `content-visibility: auto`)
- No layout reads in render (`getBoundingClientRect`, `offsetHeight`)
- `<img>` needs explicit `width` and `height` (prevents CLS)

---

## Key Rules (Summary)

### Animation & Motion

- Honor `prefers-reduced-motion` -- provide reduced variant or disable
- Animate only `transform`/`opacity` (compositor-friendly)
- Never `transition: all` -- list properties explicitly
- Animations must be interruptible by user input

### Typography & Content

| Pattern | Correct | Incorrect |
| --- | --- | --- |
| Ellipsis | `…` | `...` |
| Quotes | `"` `"` | `"` `"` |
| Non-breaking space | `10&nbsp;MB` | `10 MB` |
| Loading states | `Loading…` | `Loading...` |
| Number columns | `font-variant-numeric: tabular-nums` | default |
| Headings | `text-wrap: balance` | default |

### Navigation & State

- URL reflects state (filters, tabs, pagination in query params)
- Deep-link all stateful UI -- if using `useState`, consider URL sync via `nuqs`
- Destructive actions need confirmation modal or undo window

### Touch & Interaction

- `touch-action: manipulation` (prevents double-tap zoom delay)
- `overscroll-behavior: contain` in modals/drawers/sheets
- `autoFocus` sparingly -- desktop only, single primary input

### Dark Mode

- Set `color-scheme: dark` on `<html>` for dark themes
- Set `<meta name="theme-color">` to match page background
- Native `<select>`: explicit `background-color` and `color` for Windows

### i18n

- Dates/times: use `Intl.DateTimeFormat` not hardcoded formats
- Numbers/currency: use `Intl.NumberFormat` not hardcoded formats
- Detect language via `Accept-Language` / `navigator.languages`, not IP

### Hydration Safety

- Inputs with `value` need `onChange` (or use `defaultValue` for uncontrolled)
- Guard date/time rendering against hydration mismatch
- `suppressHydrationWarning` only where truly needed

### Content & Copy

- Active voice: "Install the CLI" not "The CLI will be installed"
- Title Case for headings/buttons (Chicago style)
- Specific button labels: "Save API Key" not "Continue"
- Error messages include fix/next step, not just the problem

---

## Anti-Patterns (Always Flag)

| Anti-Pattern | Why |
| --- | --- |
| `user-scalable=no` / `maximum-scale=1` | Disables zoom accessibility |
| `onPaste` + `preventDefault` | Blocks user paste |
| `transition: all` | Unintended property animations |
| `outline-none` without `:focus-visible` | Removes focus indicator |
| `<div onClick>` for navigation | Not a real link, breaks Cmd+click |
| `<div>` / `<span>` with click handlers | Should be `<button>` |
| Images without `width`/`height` | Causes CLS |
| Large `.map()` without virtualization | Performance issue |
| Form inputs without labels | Accessibility violation |
| Icon buttons without `aria-label` | Screen reader invisible |
| Hardcoded date/number formats | Use `Intl.*` APIs |
| `autoFocus` without justification | Disrupts mobile UX |

---

## References

- [Web Design Guidelines Skill](https://github.com/vercel-labs/agent-skills/tree/main/skills/web-design-guidelines) -- SKILL.md source
- [Web Interface Guidelines](https://github.com/vercel-labs/web-interface-guidelines) -- Full ruleset source
- [Vercel Agent Skills](https://github.com/vercel-labs/agent-skills) -- Parent repository
