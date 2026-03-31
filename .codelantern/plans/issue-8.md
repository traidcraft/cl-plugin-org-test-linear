# Technical Plan: Issue #8 - Add select/deselect all checkbox to item grid

**Status:** Complete
**Branch:** feature/tra-8-add-select-deselect-all-checkbox-to-item-grid
**Created:** 2026-03-31T14:17:00Z
**Issue:** #8

## Context

Users must toggle each item's "included" checkbox one at a time, which is tedious with many items. This enhancement adds a "select all" checkbox to the desktop grid header that bulk-toggles the included state of all visible (filtered) items, with checked/unchecked/indeterminate visual states.

## Implementation Approach

Single-phase, two-file change:

1. **Update the Checkbox component** to visually support the indeterminate state. Radix UI's `Checkbox` primitive accepts `checked="indeterminate"` and sets `data-state="indeterminate"` on the element, but the current component only renders a `CheckIcon` inside the `Indicator`. We need to add a `MinusIcon` for the indeterminate state and matching `data-[state=indeterminate]` styles.

2. **Add bulk toggle logic to App.tsx** — derive the header checkbox state from `filteredItems`, add a `toggleAllIncluded` callback that sets `included` on filtered items only, and replace the static "Include" text in the header with a `<Checkbox>`.

This approach was chosen because:
- It reuses the existing `Checkbox` component and `setItems` pattern
- The indeterminate support is a generic improvement to the Checkbox component, not a one-off hack
- State is derived (not stored), so it stays in sync automatically

## Phases

### Phase 1: Checkbox indeterminate support + header select-all

**Goal:** Add indeterminate visual state to Checkbox, then wire up select-all in the grid header.

- [x] Task 1.1: Update `Checkbox` component to support indeterminate state
  - Files: `src/components/ui/checkbox.tsx`
  - Notes: Import `MinusIcon` from lucide-react. Inside `CheckboxPrimitive.Indicator`, conditionally render `MinusIcon` when `data-state="indeterminate"` and `CheckIcon` when `data-state="checked"`. Add `data-[state=indeterminate]` CSS classes matching the checked state styling (bg-primary, text-primary-foreground, border-primary).

- [x] Task 1.2: Add `toggleAllIncluded` callback to App.tsx
  - Files: `src/App.tsx`
  - Notes: Add a `useCallback` that takes `filteredItems` IDs, determines current all-included state, and calls `setItems` to set `included` on matching items. If all are included → set all to false; otherwise → set all to true.

- [x] Task 1.3: Derive header checkbox state
  - Files: `src/App.tsx`
  - Notes: Add `useMemo` computing: `allIncluded` (every filtered item included), `noneIncluded` (no filtered item included). Header checkbox `checked` value: `allIncluded ? true : noneIncluded ? false : "indeterminate"`.

- [x] Task 1.4: Replace "Include" header text with Checkbox
  - Files: `src/App.tsx`
  - Notes: Import `Checkbox`. Replace `<div className="col-span-1">Include</div>` with `<div className="col-span-1"><Checkbox checked={...} onCheckedChange={toggleAllIncluded} /></div>`.

**Checkpoint criteria:**
- [ ] Lint passes
- [ ] Header checkbox renders with correct state
- [ ] Clicking toggles all visible items
- [ ] Category filter scopes the toggle correctly
- [ ] Budget totals update immediately

## Files to Modify

| File | Changes | Phase |
|------|---------|-------|
| `src/components/ui/checkbox.tsx` | Add indeterminate icon + styles | 1 |
| `src/App.tsx` | Add toggleAllIncluded, derive state, update header | 1 |

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Radix Checkbox Indicator renders same icon for checked and indeterminate | Medium | Low | Use conditional rendering inside Indicator based on parent data-state |
| Toggling all items when filter changes could confuse users | Low | Low | State is derived from current filter, no stored "select all" state |

## Test Strategy

### Manual Testing
- [ ] With all items included: header shows checked; clicking deselects all
- [ ] With no items included: header shows unchecked; clicking selects all
- [ ] With mixed items: header shows indeterminate; clicking selects all
- [ ] With category filter active: only filtered items toggled, others unchanged
- [ ] Budget total updates immediately after toggle
- [ ] Mobile layout unchanged (header hidden)
- [ ] Individual item checkboxes still work and update header state

## Dependencies

- None — Radix UI Checkbox already supports `checked="indeterminate"`

## Out of Scope

- Mobile select-all control
- Keyboard shortcut for select/deselect all
- Persisting select-all state
