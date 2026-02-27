# Floating WebView: Popup Decoupling in Hybrid Web-Native Apps

> **Source**: [Woowahan Bros Tech Blog](https://techblog.woowahan.com/24165/)
> **Author**: Park Jaeyong (박재용)
> **Archive Date**: 2026-02-27

How Baemin's commerce team introduced a transparent overlay WebView ("Floating WebView") to solve popup-layer conflicts between web and native UI, and the architecture they built to decouple popup components across pages.

---

## Background: The Web-Native Collision

Baemin's commerce services (B-Mart, Grocery/Shopping, Nationwide Deals) evolved from purely web or native pages to hybrid layouts where native elements (navigation tab bars, floating bars) overlap web content.

**Key triggers:**
- Native navigation tab bars replaced web-based ones for smoother UX and parallel navigation history
- iOS 26's *Liquid Glass* design made native bottom bars float over web content
- Popups (dialogs, bottom sheets) rendered in web could no longer appear above native UI layers

**The problem:** Web popups (dialogs, bottom sheets) get clipped or hidden behind native UI elements since web content cannot render above native layers.

---

## Solution: Floating WebView

A **Floating WebView** is a transparent, full-screen WebView rendered at the topmost native layer. Popups are displayed inside this overlay WebView instead of in the original page's web context.

**Trade-offs acknowledged:**
- **Performance overhead**: A new page must be downloaded and initialized for each popup
- **UX degradation**: Opening/closing animation flow may break
- **Complex decoupling**: Popup components must be separated from their calling pages and communicate across page boundaries

Despite these costs, the team adopted Floating WebView because it provides a structural solution that remains flexible as native UI evolves.

---

## Scoping the Work

To minimize effort under tight deadlines:
- **Excluded** pages where native overlaps were minor (e.g., header-only native areas)
- **Excluded** small components (snackbars, tooltips) that could avoid conflicts using `safe-area-inset`
- Focused only on pages where popups directly conflicted with native UI

---

## Performance Optimization Strategy

Approaches considered (only the first was implemented immediately; others deferred):

| Strategy | Description |
| --- | --- |
| **Minimize loading blockers** | Pass API response data directly as popup props to avoid re-fetching |
| Prefetch/preload | Prefetch popup content or preload the Floating WebView page hidden |
| Native popups | Use native popup containers with web content inside (fragmentation risk) |

---

## Cross-Page Communication

Since the calling page and the Floating WebView popup page are separate web pages, a communication mechanism is needed.

| Approach | Verdict | Reason |
| --- | --- | --- |
| LocalStorage / SessionStorage | Runner-up | Same-origin only |
| IndexedDB | Rejected | High implementation complexity |
| Broadcast Channel | Rejected | iOS 15.4+ only — not supported |
| postMessage | Rejected | Cannot obtain Floating WebView reference |
| **App internal storage (JSON)** | **Chosen** | Simple, cross-origin capable, app-managed lifecycle |

**Closing event**: Uses the app-provided callback when Floating WebView closes.

---

## URL Routing Convention

Each popup gets its own dedicated Floating WebView page at a top-level path:

```
~/floating-webview/{popup-name}
```

**Benefits:**
- Easy to distinguish Floating WebView pages from regular pages by URL
- Better monitoring and error tracking per popup
- Consistent structure accelerates decoupling work

---

## Decoupling Architecture: Caller & Callee

To reduce duplication and ensure consistency across multiple services, the team introduced **Caller** and **Callee** mediator components:

- **Caller**: Sits in the calling page. Handles Floating WebView invocation, data transmission, and close-event handling.
- **Callee**: Sits in the Floating WebView page. Receives data, renders the popup, and manages lifecycle events.

**Backward compatibility**: A single popup component is shared across both old (no Floating WebView) and new app versions. The mediator components control whether the popup renders inline or in a Floating WebView.

### Edge Cases Handled

- **Error boundary**: Callee wraps popup with an error screen that auto-closes the Floating WebView
- **Back button**: Callee intercepts device back button to close the Floating WebView naturally
- **Missing close events**: If the user closes the Floating WebView before popup loads, Caller detects page focus and triggers the close callback

---

## Decoupling Step-by-Step (Example: Sort Bottom Sheet)

1. **Separate UI from logic**: Extract bottom sheet UI into component `F`; keep logic/state in component `A`
2. **Event-based closing**: `F` emits a close event with the selected value; `A` handles it via `onClose`
3. **Add mediators**: Create `F_Caller` (in calling page under `A`) and `F_Callee` (in `~/floating-webview/sort-bottom-sheet`)
4. **Define props**: Pass only the minimum data needed (e.g., current sort state). Avoid circular references or functions (must be JSON-serializable)
5. **Isolate context**: Place only required contexts (e.g., design system provider) in the Floating WebView page. Minimize context dependencies

---

## Library Extraction

Common patterns across multiple services were extracted into a shared library that:
- Generates Caller/Callee mediator components based on service configuration
- Provides a consistent API for applying Floating WebView decoupling to any popup
- Reduced per-popup decoupling effort and enforced architectural consistency across B-Mart, Grocery/Shopping, and Nationwide Deals

---

## Key Takeaways

- **Floating WebView** provides a structural solution for web-native popup layer conflicts that remains resilient as native UI evolves
- **Caller/Callee mediator pattern** enables clean decoupling while maintaining backward compatibility with a single popup codebase
- **Scope aggressively**: Not every popup needs decoupling — focus on high-impact conflicts
- **Standardize early**: Consistent routing conventions and decoupling steps enabled efficient scaling and even AI-assisted development
- **App storage for cross-page communication** is a pragmatic choice in hybrid apps where web pages run within a native container

---

## References

- [Woowahan Bros Tech Blog — Floating WebView](https://techblog.woowahan.com/24165/)
