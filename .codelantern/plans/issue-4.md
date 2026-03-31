# Technical Plan: Issue #4 - Add select all / deselect all checkbox to item grid

**Status:** Approved
**Branch:** feature/tra-4-add-select-all-deselect-all-checkbox-to-item-grid
**Created:** 2026-03-30T23:30:00Z
**Issue:** #4

## Context

Users must individually toggle each item's `included` checkbox in the budget planner grid. This is tedious with many items. The fix is a header-level checkbox with three-state behavior (checked / indeterminate / unchecked) that bulk-toggles all items, updating budget totals immediately. Desktop only — mobile card layout has no shared header.

## Implementation Approach

Extend the existing Radix UI Checkbox component to render a `MinusIcon` for the indeterminate state, then wire a new `toggleAllIncluded` callback into the desktop grid header. This follows the existing `toggleIncluded` pattern and leverages Radix UI's native `checked="indeterminate"` API.

**Why this approach:**
- Minimal surface area — two files modified, zero new files
- Radix UI already supports `checked: boolean | "indeterminate"` on the primitive; we just need the visual indicator
- The existing `useCallback` + `setItems` pattern applies directly for the bulk toggle
- Computed state (`allIncluded` / `noneIncluded`) is cheap to derive inline

**Trade-offs considered:**
- Filtering scope: toggle affects ALL items regardless of active category filter (per spec). A category-scoped toggle is explicitly out of scope.
- No memoization for derived booleans — the items array is small enough that computing on every render is negligible.

## Phases

### Phase 1: Enhance Checkbox with indeterminate state

**Goal:** The Checkbox component visually renders all three states (checked, indeterminate, unchecked).

- [ ] Task 1.1: Add `MinusIcon` import from `lucide-react` to `src/components/ui/checkbox.tsx`
- [ ] Task 1.2: Add `data-[state=indeterminate]` Tailwind classes to the Root element (mirror the `data-[state=checked]` styles so the indeterminate state has the same filled background/border)
- [ ] Task 1.3: Update the Indicator's children to conditionally render `MinusIcon` for indeterminate or `CheckIcon` for checked. Use Radix's render function pattern or read the `checked` prop to decide which icon to show. Approach:
  - Accept a `checked` prop (already passed through via `...props`)
  - Inside `CheckboxPrimitive.Indicator`, render both icons but use CSS `group-data-[state=indeterminate]` / `group-data-[state=checked]` to show/hide the correct one. Alternatively, use the Radix `forceMount` on Indicator and conditionally render based on prop value.
  - Simplest: render a wrapper that checks `props.checked === "indeterminate"` to pick the icon.
  - Files: `src/components/ui/checkbox.tsx`
- [ ] Task 1.4: Add an `Indeterminate` story to `src/components/ui/checkbox.stories.tsx`
  - Args: `checked: "indeterminate"` (controlled)
  - Verifies the MinusIcon renders correctly

**Checkpoint criteria:**
- [ ] Checkbox renders CheckIcon when checked, MinusIcon when indeterminate, nothing when unchecked
- [ ] Existing checkbox usage is unaffected (no prop changes needed)
- [ ] Storybook story shows all three states

### Phase 2: Wire select-all into the desktop grid header

**Goal:** Header checkbox replaces "Include" text and bulk-toggles all items.

- [ ] Task 2.1: Add computed state derivation after existing `toggleIncluded` (around line 102 of `src/App.tsx`):
  ```typescript
  const allIncluded = items.length > 0 && items.every((item) => item.included)
  const noneIncluded = items.every((item) => !item.included)
  ```
- [ ] Task 2.2: Add `toggleAllIncluded` callback:
  ```typescript
  const toggleAllIncluded = useCallback(() => {
    setItems((prev) => {
      const shouldInclude = !prev.every((item) => item.included)
      return prev.map((item) => ({ ...item, included: shouldInclude }))
    })
  }, [setItems])
  ```
  Logic: if all included → uncheck all; otherwise (some or none) → check all. Matches spec behavior.
- [ ] Task 2.3: Replace the "Include" header `<div>` at line 277 with:
  ```tsx
  <div className="col-span-1">
    <Checkbox
      checked={allIncluded ? true : noneIncluded ? false : "indeterminate"}
      onCheckedChange={toggleAllIncluded}
      aria-label="Toggle all items"
    />
  </div>
  ```
  - Files: `src/App.tsx`
  - Notes: Add `Checkbox` to imports from `@/components/ui/checkbox`
- [ ] Task 2.4: Verify budget total recalculates — no changes needed to `totalCost` or `categoryBreakdown` (lines 183-194) since they already filter by `item.included`

**Checkpoint criteria:**
- [ ] Header checkbox shows checked when all items included
- [ ] Header checkbox shows indeterminate when some items included
- [ ] Header checkbox shows unchecked when no items included
- [ ] Clicking header checkbox toggles all items
- [ ] Budget total updates immediately after bulk toggle
- [ ] Individual item checkboxes still work and update header state
- [ ] Mobile layout unchanged (header only in `hidden md:grid` container)

## Files to Modify

| File | Changes | Phase |
|------|---------|-------|
| `src/components/ui/checkbox.tsx` | Add indeterminate state: `MinusIcon` import, `data-[state=indeterminate]` styles, conditional icon rendering | 1 |
| `src/components/ui/checkbox.stories.tsx` | Add `Indeterminate` story | 1 |
| `src/App.tsx` | Add `allIncluded`/`noneIncluded` derived state, `toggleAllIncluded` callback, replace "Include" header with `<Checkbox>` | 2 |

## New Files to Create

None.

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Checkbox prop change breaks existing usage | Low | Medium | The `checked` prop type already accepts `boolean \| "indeterminate"` from Radix — existing boolean usage is unaffected. Indicator children change is internal to the component. |
| Indeterminate icon not visible at small size | Low | Low | Use same `size-3.5` as CheckIcon for consistency; verify in Storybook |
| Category filter confusion (user expects select-all to respect filter) | Low | Low | Per spec, toggle affects all items regardless of filter. Out of scope note in issue clarifies this. |

## Test Strategy

### Manual Testing
- [ ] With all items included: header shows checked; click → all unchecked, total drops to $0
- [ ] With no items included: header shows unchecked; click → all checked, total shows full sum
- [ ] With some items included: header shows indeterminate (minus icon); click → all checked
- [ ] Toggle individual item checkbox → header state updates accordingly
- [ ] Mobile viewport: no header checkbox visible, individual checkboxes work normally
- [ ] Storybook: Indeterminate story renders MinusIcon correctly

### Storybook Verification
- [ ] `Checkbox > Indeterminate` story renders with minus icon and filled background
- [ ] Existing stories (`Default`, `Checked`, `Disabled`, etc.) are unaffected

## Dependencies

None — `lucide-react` (already installed) provides `MinusIcon`.

## Out of Scope

- Mobile select-all UI
- Category-scoped select all
- Keyboard shortcut for select all
- Unit/integration tests (no existing test suite to extend; can be added later)
