# AGENTS – Frontend Code Quality

"Good frontend code" in this project means **code that is easy to change**.
Changeability is evaluated by four criteria: **Readability, Predictability, Cohesion, and Coupling**.

These guidelines apply to both humans and AI agents when writing or refactoring code.

---

## 1. Readability

**Goal**: Reduce the amount of context a reader must hold in their head at once.

### 1-1. Separate Code That Doesn't Execute Together

- When a component/function contains **branching logic for different cases** (roles, states, modes):
  - Split into **role-specific components/functions** (e.g., `Viewer` / `Admin`, `LoggedIn` / `Guest`).
  - Handle branching at the parent level: `isViewer ? <Viewer /> : <Admin />`.
- When conditions are **interleaved** (`if (cond) return ...; else ...`):
  - Extract into smaller components/functions that each handle **one responsibility**.

### 1-2. Abstract Implementation Details

- Higher-level components/pages should only show **"what" is happening**, not "how".
  - Hide logic like "check login state and redirect" or "verify permissions" inside:
    - Wrapper components (`AuthGuard`) or HOCs (`withAuthGuard`).
- Keep the number of concepts to understand at once to **6-7 or fewer**.
  - Extract long logic into **meaningful units** (functions/components).

### 1-3. Split Mixed Logic Types

- Don't cram query params, state, API calls, and formatting into **one Hook/function**.
- Avoid monolithic hooks like `usePageState` that handle "all page state/queries".
  - Prefer **single-responsibility hooks**: `useCardIdQueryParam`, `useDateRangeQueryParam`.

### 1-4. Name Things to Reveal Intent

- Don't leave complex conditions as **nested ternaries or logical expressions**.
  - Extract to named variables/functions: `const isSameCategory = ...`, `const isPriceInRange = ...`.
- **No magic numbers** (300, 86400, 404).
  - Use named constants: `ANIMATION_DELAY_MS`, `ONE_DAY_SECONDS`, `HTTP_NOT_FOUND`.

### 1-5. Read Top-to-Bottom, Minimize Context Switching

- Conditions for button disabled states, permission-based UI, etc. should be:
  - Understandable by reading **sequentially from top to bottom** within one function/component.
- Prefer **explicit code over abstract policy objects** (`POLICY_SET`) when the situation is simple.
  - A `switch (role)` that directly reflects requirements is often clearer.

---

## 2. Predictability

**Goal**: Make behavior **predictable from the function/component's name, parameters, and return value alone**.

### 2-1. Avoid Name Collisions

- Don't create wrapper functions/modules with **the same name as existing libraries**.
  - Example: A wrapper around `http` library should be named `httpService`, not `http`.
  - Use intention-revealing names: `getWithAuth`, `fetchWithAuth`.

### 2-2. Unify Return Types for Similar Functions

- React Query hooks calling server APIs should **always return `UseQueryResult`**.
  - Don't have `useUser` return `query` while `useServerTime` returns `query.data`.
- Validation functions should use **consistent return type patterns**.
  - Example: `type ValidationResult = { ok: true } | { ok: false; reason: string }`.
  - Both `checkIsNameValid` and `checkIsAgeValid` should return this type.

### 2-3. Make Hidden Side Effects Visible

- Don't hide **logging, tracking, or notifications** inside functions where the signature doesn't reveal them.
  - Don't silently call `logging.log` inside `fetchBalance()`.
- Place side effects **at the call site** so they're visible:
  ```ts
  const balance = await fetchBalance();
  logging.log("balance_fetched");
  ```

---

## 3. Cohesion

**Goal**: Keep **code that always changes together** in the same place to reduce modification errors.

### 3-1. Co-locate Files That Change Together

- Instead of "hooks/components/utils all in separate folders":
  - Use **domain-based folder structure** as the default.

Example:
```text
src/
  components/         # Global shared
  hooks/
  utils/
  domains/
    User/
      components/
      hooks/
      utils/
    Payment/
      components/
      hooks/
      utils/
```

- If you're importing from `../../../Domain2/...` deeply:
  - Suspect a **wrong dependency direction**.
- Design so that when deleting a feature, you can **delete the entire directory**.

### 3-2. Elevate Magic Numbers to Cohesive Constants

- When numbers (animation, timer, polling) are **likely to change with related code**:
  - Define constants near the related logic.

```ts
const LIKE_ANIMATION_DELAY_MS = 300;

async function onLikeClick() {
  await postLike(url);
  await delay(LIKE_ANIMATION_DELAY_MS);
  await refetchPostLike();
}
```

### 3-3. Design Form Cohesion Intentionally

- When designing form validation, think about the **"unit of change"** first.

**Choose field-level cohesion when:**
- Each field has **independent validation/async logic** and is reusable.
  - Example: Email format check, phone format, username duplication check.
- If field reuse is important:
  - Separate into field component + field-specific validation utils.

**Choose form-level cohesion when:**
- The form represents **one business action** and fields are tightly coupled.
  - Example: Payment info, shipping info, multi-step signup, password confirmation, total calculation.
- In this case:
  - Define a **whole-form schema** with Zod/Schema and manage validation in one place.

---

## 4. Coupling

**Goal**: Keep the **scope of impact from one change predictable and reasonably narrow**.

### 4-1. Manage One Responsibility at a Time

- Avoid patterns like "one Hook handles all query params/state this page needs".
  - Don't create giant hooks like `usePageState`.
- Instead, split into **small hooks with separated responsibilities**:
  - `useCardIdQueryParam`
  - `useDateRangeQueryParam`
  - `useStatusListQueryParam`
- This way, modifying a specific hook has **limited impact**, reducing coupling.

### 4-2. Allow Some Duplication

- Don't do **premature abstraction/generalization** just because of "duplication".
- Only extract to shared Hook/component when:
  - Multiple places currently use the same logic, AND
  - It will **likely stay the same** in the future, AND
  - Calling sites won't diverge significantly in requirements.
- If behavior/logging/text might differ slightly per page:
  - **Allow reasonable duplication** instead of extracting.
  - Prioritize narrowing the scope of impact when changes occur.

### 4-3. Reduce Props Drilling

- When **the same props pass through parent → child → grandchild...**:
  - First question whether "intermediate components are meaningful abstractions".

**Order of solutions:**

1. **Remove unnecessary intermediate abstractions**
   - Vague intermediate components like `ItemEditBody` should be restructured into **clearly-roled components** like `ItemEditSearch`, `ItemEditList`.

2. **Use Composition pattern**
   - Parent passes list/content directly via `children`.
   - Intermediate components handle only layout/shared UI.

3. Only when depth is severe and multiple levels need the same data:
   - Then introduce **Context API** as a last resort.

- Global state libraries (Redux, Jotai, etc.):
  - Use only when needed for **page-to-page state sharing**.
  - Don't abuse for simple Props Drilling elimination.

---

## 5. Trade-offs Between the Four Criteria

- The four criteria (Readability / Predictability / Cohesion / Coupling) **cannot always be maximized simultaneously**.
- Decision priority is typically:

  1. **Readability** – If no one can read it, other criteria are meaningless.
  2. **Predictability** – Confusion leads to bugs and maintenance costs.
  3. **Cohesion/Coupling** – Balance based on change units and impact scope.

- When refactoring:
  - Explicitly think about **"which criterion to prioritize for this change"** before modifying.

---

## References

- [Frontend Fundamentals](https://frontend-fundamentals.com/) by Toss
