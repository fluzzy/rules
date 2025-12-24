# ACCESSIBILITY – A11y Guide

The goal of accessibility (a11y) in this project is to create **UI where Role, Label, and State are always clear**.

These guidelines apply to both humans and AI agents when building or refactoring components.

---

## 1. Core Accessibility Principles

### 1-1. Role

- Use **semantic tags** first whenever possible.
  - `<button>`, `<a>`, `<input>`, `<form>`, `<fieldset>`, `<legend>`, `<dialog>`, `<details>`, `<summary>`, etc.

- If semantic tags can't express it, specify with `role`.
  - Examples: `role="tablist" | "tab" | "tabpanel" | "dialog" | "checkbox" | "radio" | "switch" | "button" | "region"`, etc.

### 1-2. Label

- All **interactive elements** (buttons, links, inputs, select components, etc.) must always have a name that explains "what this is".
- Priority: `aria-labelledby` → `aria-label` → `<label>`/element text.
- If visual text sufficiently explains the meaning, ARIA can be omitted.
- **Icon-only buttons / switches / toggles** require `aria-label` or `aria-labelledby`.

### 1-3. State

- Use both **HTML native attributes** and **ARIA state attributes**, always keeping them in sync.
- Commonly used attributes:
  - `checked` / `aria-checked`
  - `selected` / `aria-selected`
  - `open` / `aria-expanded`
  - `disabled` / `aria-disabled`
  - `aria-current` (current page/date, etc.)

### 1-4. Live Regions

Use live regions to announce dynamically changing content to screen readers:

- **`aria-live`**: Screen reader announces when content changes
  - `aria-live="polite"`: Announces after current content is finished (general updates)
  - `aria-live="assertive"`: Announces immediately (urgent errors, important alerts)

- **`role="alert"`**: Equivalent to `aria-live="assertive"` + `aria-atomic="true"`. For error messages, urgent alerts.

- **`role="status"`**: Equivalent to `aria-live="polite"` + `aria-atomic="true"`. For status updates, success messages.

- **`aria-busy`**: Indicates content is loading. When `true`, screen reader holds off on change announcements.

- **`aria-atomic`**: When `true`, reads the entire region, not just changed parts.

Examples:

```tsx
// Form submission result notification
<div role="status" aria-live="polite">
  {submitSuccess && "Saved successfully."}
</div>

// Error message
<div role="alert">
  {errorMessage}
</div>

// Loading state
<div aria-busy={isLoading} aria-live="polite">
  {isLoading ? "Loading..." : content}
</div>
```

### 1-5. Keyboard Navigation

- Make UI operable with Tab/Shift+Tab, Enter, Space, and arrow keys.
- When creating **custom interactive elements**:
  - Add `tabIndex={0}`
  - Set `role`
  - Handle Enter / Space in `onKeyDown`

### 1-6. Don't Rely Solely on Visual Information

- Information must not be conveyed only through color/icons/images.
- Always supplement with **text or ARIA labels**.
- Mark decorative images with `alt=""` to prevent screen reader announcement.

---

## 2. UI Component Rules

### 2-1. Tabs

**Goal:** Make screen readers accurately understand the "tab list / current tab / corresponding panel" structure.

- Structure:
  - Tab list container: `role="tablist"` + `aria-label` or `aria-labelledby`
  - Tab button: `role="tab"`
    - Current tab: `aria-selected="true"`
    - Other tabs: `aria-selected="false"`
  - Tab panel: `role="tabpanel"`
    - `id` required
    - Tab button has `aria-controls="<tabpanel-id>"`
    - Tab panel has `aria-labelledby="<tab-id>"`

- Show/hide:
  - **Only active panel** is actually visible; hide others with `hidden` or `aria-hidden`/CSS.
  - Always sync `aria-selected` value with **panel visibility**.

- Keyboard:
  - Implement arrow key (left/right) navigation between tabs by default.
  - Tab/Shift+Tab for entering/exiting the entire tab group.

### 2-2. Accordion

**Goal:** Make it clear which item is expanded.

- **Prefer HTML standard elements first**:
  - `<details>` + `<summary>`
    - `open` attribute represents state
    - `onToggle` for state management

- Custom implementation rules:
  - Header (toggle button):
    - Use `<button>`
    - `aria-expanded={isOpen}`
    - `aria-controls="<panel-id>"`
  - Panel:
    - `id="<panel-id>"`
    - `role="region"`
    - `aria-labelledby="<button-id>"`
    - `hidden={!isOpen}`

- State synchronization:
  - `aria-expanded="true"` ↔ panel visible
  - `aria-expanded="false"` ↔ panel hidden
  - These must never be out of sync.

### 2-3. Modal/Dialog

**Goal:** When modal is open, **user focus and screen reader attention stay only within the modal**.

- When `<dialog>` is available (preferred):
  - Trigger button: `aria-haspopup="dialog"`
  - Open with `dialog.showModal()`
  - Close with `dialog.close()`
  - Provide title via `aria-labelledby` or `aria-label`
  - Browser automatically handles:
    - Focus moves inside modal
    - Focus trap
    - ESC to close
    - Focus returns to original element on close

- When `<dialog>` can't be used (custom modal):
  - Modal container:
    - `role="dialog"`
    - `aria-modal="true"`
    - `aria-labelledby="<title-id>"` or `aria-label="..."`
  - Focus management:
    - On modal open: move to first focusable element inside
    - On modal close: return focus to trigger
  - ESC handling:
    - Close when `e.key === "Escape"` in `keydown` event
  - Background content:
    - Add `inert` to main area when modal is open
    - Remove `inert` when closed
  - Focus trap:
    - Tab/Shift+Tab cycles only within the modal

### 2-4. Radio Buttons

**Goal:** Make the "select 1 answer from multiple options for one question" structure clear.

- Basic structure (recommended):
  - Container: `<fieldset>`
  - Group title: `<legend>`
  - Each option:
    - `<input type="radio" name="...">`
    - `<label htmlFor="...">Option text</label>`

- Alternative structure:
  - If `<fieldset>` can't be used:
    - Container: `role="radiogroup"` + `aria-labelledby="<heading-id>"`
    - Still use `<input type="radio" name=".."> + <label>` inside

- Required rules:
  - **Same group has same name** (browser automatically ensures "select only 1").
  - Each option **always connected to label** (`id` + `htmlFor`).

- Custom radio (without input):
  - Each option:
    - `role="radio"`
    - `aria-checked={true|false}`
    - `tabIndex={0}` (at least one)
    - Space key to toggle selection (`onKeyDown` handling)
  - Group:
    - `role="radiogroup"` + `aria-labelledby`

### 2-5. Checkbox

**Goal:** Make it clear that multiple options can be selected simultaneously.

- When there's a group:
  - Container: `<fieldset>`
  - Title: `<legend>`
  - Each item:
    - `<input type="checkbox" id="..." />`
    - `<label htmlFor="...">Text</label>`

- Custom checkbox:
  - `role="checkbox"`
  - `aria-checked={true|false}`
  - `tabIndex={0}`
  - Space key to check/uncheck (`onKeyDown`)

### 2-6. Switch

**Goal:** Clearly convey on/off state.

- HTML-based switch:
  - `<input type="checkbox" role="switch" checked={isOn} />`
  - Label:
    - Wrap with `<label>`, or
    - Provide via `aria-label` / `aria-labelledby`

- Custom switch:
  - `role="switch"`
  - `aria-checked={isOn}`
  - `tabIndex={0}`
  - Space key to change state
  - Label required
    - Explain "what is being turned on/off" (e.g., `"Dark mode"`, `"Notification settings"`)

---

## 3. Practical Pattern Rules

### 3-1. Don't Nest Buttons/Links Inside Buttons

- Prohibited:
  - `<button>` inside `<button>`
  - `<button>` inside `<a>`
  - `<a>` inside `<button>`

- Solutions:
  - **Polymorphic button** pattern:
    - `<Button as="a" href="/...">Text</Button>`
  - "Entire card click + individual buttons inside" pattern:
    - Outer is `div` + invisible button for full-area click
    - Individual buttons are separate actual `<button>` elements
    - Focus style with `.wrapper:focus-within { … }`

### 3-2. Don't Make Table Rows Directly Clickable

- Prohibited:
  - `<tr onClick={...}>`

- Solution:
  - Put **actual `<a>` link** inside the row, expand area with CSS:
    - Link has `position: relative` + `::after { position: absolute; inset: 0; content: "" }`
  - Benefits:
    - Preserves keyboard focus, context menu (open in new tab, copy link), etc.
    - Screen reader recognizes it as "link"

### 3-3. Label Interactive Elements

- Basic rule:
  - All inputs/buttons/select elements **always have a name**.

- Input fields:
  - Prefer `<label for="id">` + `<input id="id">`
  - Use `aria-label` if visual label can't be placed
  - Use `aria-labelledby` when reusing nearby text

- Buttons/Icons:
  - Text button: inner text is the label
  - Icon button: `aria-label` required

- Select:
  - `<label>` + `<select id="...">` structure by default

- Placeholder:
  - **Not a label substitute**, only for hints
  - Always use with a label

### 3-4. Multiple Buttons with Same Name

- Example: "Select" button repeated in each list item
- Rule:
  - Create "context text" for each item and connect to button
  - Pattern example:
    - List item: `<li aria-labelledby="title-id">`
    - Title text: `<div id="title-id">When using paper</div>`
    - Button: `id="paper-button" aria-labelledby="title-id paper-button"`
  - Or make button `aria-label="Select when using paper"` format

### 3-5. Match Button Role and Behavior

- Prohibited:
  - Using `<div onClick>` with just `cursor: pointer` as a button

- Default:
  - Use `<button>` whenever possible.

- If you must use `<div>` etc.:
  - `role="button"`
  - `tabIndex={0}`
  - Handle Enter / Space click in `onKeyDown`

### 3-6. Always Wrap Input Elements in `<form>`

- Default:
  - Put related input groups inside `<form>`.
  - Submit button is `<button type="submit">`.
  - Simple click buttons **must** specify `type="button"`.

- Behavior:
  - When handling in JS:
    - Call `event.preventDefault()` in `onSubmit`, then execute logic.
  - Screen reader:
    - Provide form name via `aria-label` or `<form aria-labelledby="...">`.

- Submit button outside form:
  - Connect with `form="form-id"` attribute on button.

- Auxiliary buttons (clear input, etc.):
  - Exclude from focus order with `tabIndex="-1"` if focus isn't needed, describe role with `aria-label`.

### 3-7. Image / Icon Alt Text

- Principle:
  - Write alt based on "What **information** or **function** is this image conveying to the user?"

- Cases where alt is required:
  - Image is **the only content of a link/button** → describe function in alt
  - Contains **core information** like product/graph/chart → summarize content

- Cases where alt should be empty (`alt=""`):
  - Decorative images (simple dividers, backgrounds, etc.)
  - Nearby text/caption already explains the same content
  - Icon with text (delete icon + "Delete" text, etc.)

- Writing style:
  - Remove unnecessary words (`icon`, `button`)
    - Screen reader already adds "button"
  - Write based on context
    - `"Go to previous page"`, `"Go to next page"` etc. based on function/intent

---

## 4. eslint-plugin-jsx-a11y Configuration

Essential rules to auto-catch accessibility violations in JSX/React code.

### 4-1. Basic Setup

```ts
// eslint.config.js (flat config example)
import jsxA11y from "eslint-plugin-jsx-a11y";

export default [
  jsxA11y.flatConfigs.recommended,
  {
    rules: {
      "jsx-a11y/control-has-associated-label": "error",
      // Add/adjust other rules as needed
    },
  },
];
```

### 4-2. Key Rules Summary

- `alt-text`
  - `<img>` must always have `alt`.
  - Decorative images with no info: `alt=""`.
  - Icons with text: `alt=""` recommended.

- `control-has-associated-label`
  - All controls (inputs, buttons, selects, etc.) must have an associated label.
  - Form-related markup must have at least one of:
    - `<label>` + `<input>`
    - `aria-label`
    - `aria-labelledby`

- `no-noninteractive-element-interactions`
  - Warns when `onClick` etc. used on non-interactive elements like `<div>`, `<span>`.
  - Must add `role`/`tabIndex` to indicate it's interactive, or change to `<button>`, `<a>`.

- `no-noninteractive-element-to-interactive-role`
  - Don't assign interactive roles like `role="button"` to semantic containers like `<main>`, `<ul>`, `<li>`, `<img>`.
  - Use `<button>`, `<a>` if real interaction is needed.

- `no-noninteractive-tabindex`
  - Don't give `tabIndex` to non-interactive elements.
  - If focus/interaction is really needed, upgrade to interactive element with `role`.

- `tabindex-no-positive`
  - `tabIndex > 0` prohibited.
  - Allowed values: only `0` or `-1`.

### 4-3. Integration with Design Systems / Polymorphic Components

#### Component Mapping

```ts
export default [
  jsxA11y.flatConfigs.recommended,
  {
    rules: {
      "jsx-a11y/control-has-associated-label": "error",
    },
    settings: {
      "jsx-a11y": {
        components: {
          MyButton: "button",
          MyTxt: "span",
        },
      },
    },
  },
];
```

- This applies the same a11y rules to `<MyButton>`/`<MyTxt>` as `<button>`/`<span>`.

#### Polymorphic as prop

```ts
export default [
  jsxA11y.flatConfigs.recommended,
  {
    rules: {
      "jsx-a11y/control-has-associated-label": "error",
    },
    settings: {
      "jsx-a11y": {
        polymorphicPropName: "as",
        components: {
          MyButton: "button",
          MyTxt: "span",
        },
      },
    },
  },
];
```

- Ensures a11y rules work correctly for components that render as different tags like `<MyButton as="a" href="/home">`.

#### Custom Label Prop Support

```ts
export default [
  jsxA11y.flatConfigs.recommended,
  {
    rules: {
      "jsx-a11y/control-has-associated-label": [
        "error",
        {
          labelAttributes: ["contents"], // e.g., <MyCard contents="Card content" />
        },
      ],
    },
    settings: {
      "jsx-a11y": {
        polymorphicPropName: "as",
        components: {
          MyButton: "button",
          MyTxt: "span",
          MyCard: "button",
        },
      },
    },
  },
];
```

---

## 5. New Component Checklist

Agents/developers must verify the following when creating new components:

1. **Role**: Did you use semantic tags? Did you specify `role` when needed?
2. **Label**: Do all interactive elements have names?
3. **State**: Are state attributes synchronized between HTML/ARIA?
4. **Keyboard Navigation**: Is it operable with Tab, Enter, Space, arrow keys?
5. **Visual Dependency**: Is information not conveyed only through color/icons?
6. **eslint-plugin-jsx-a11y**: Does it not violate configured rules?

---

## References

- [A11y Fundamentals](https://frontend-fundamentals.com/a11y/) by Toss
