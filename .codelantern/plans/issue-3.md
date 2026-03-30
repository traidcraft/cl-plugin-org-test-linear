# Technical Plan: Issue #3 - Add select/deselect all checkbox to item grid

**Status:** Complete
**Branch:** feat/select-all-items-issue-3
**Created:** 2026-03-30T22:09:00Z
**Issue:** #3

## Context

Users currently toggle item inclusion one-by-one. This enhancement adds a "select all" checkbox to the grid header that bulk-toggles the `included` field on all filtered items, with tri-state support (checked / indeterminate / unchecked).

## Implementation Approach

Extend the existing `included` field and `toggleIncluded` pattern. Add a `toggleAllIncluded` callback in `App.tsx` that operates on filtered items only. Replace the static "Include" header text with a `<Checkbox>` that computes its state from filtered items. Update the shadcn `Checkbox` component to render a minus icon for the indeterminate state. Add a mobile "Select all" row above the item cards.

**Why this approach:** No data model changes, no new storage keys, and the existing `useLocalStorage` persistence handles everything automatically.

## Phases

### Phase 1: Update Checkbox primitive for indeterminate support
**Goal:** The shadcn `<Checkbox>` visually distinguishes checked vs. indeterminate states.

- [x] Task 1.1: Update `src/components/ui/checkbox.tsx` to render a `MinusIcon` when `data-state="indeterminate"` and `CheckIcon` when `data-state="checked"`
  - Files: `src/components/ui/checkbox.tsx`
  - Notes: Use Radix's `checked="indeterminate"` support. Conditionally render icon based on checkbox state.

**Checkpoint criteria:**
- [ ] Storybook shows both checked and indeterminate states correctly

### Phase 2: Add select all logic and UI
**Goal:** Header checkbox toggles all filtered items' `included` state.

- [x] Task 2.1: Add `toggleAllIncluded` callback in `App.tsx` that sets `included` on all filtered items based on current state
  - Files: `src/App.tsx`
  - Notes: Compute `allIncluded` and `noneIncluded` from `filteredItems`. If `noneIncluded`, set all to `true`; otherwise set all to `false`.
- [x] Task 2.2: Replace the static "Include" text in the desktop grid header with a `<Checkbox>` that reflects tri-state
  - Files: `src/App.tsx`
  - Notes: `checked={allIncluded ? true : noneIncluded ? false : "indeterminate"}`
- [x] Task 2.3: Add a mobile "Select all" row above the item cards (visible only on `md:hidden`)
  - Files: `src/App.tsx`
  - Notes: Render a `<Checkbox>` with "Select all" label, same tri-state logic

**Checkpoint criteria:**
- [ ] `npm run build` passes
- [ ] `npm run lint` passes
- [ ] Toggling select all updates budget total immediately
- [ ] Category filter scopes the toggle correctly

## Files to Modify

| File | Changes | Phase |
|------|---------|-------|
| `src/components/ui/checkbox.tsx` | Add indeterminate icon rendering | 1 |
| `src/App.tsx` | Add `toggleAllIncluded` callback, header checkbox, mobile select-all row | 2 |

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Radix Checkbox indeterminate rendering differs from expected | Low | Low | Verify with Storybook before integrating |
| Category filter edge case — toggling "All" vs. specific category | Low | Medium | `toggleAllIncluded` uses `filteredItems` IDs to scope updates |

## Test Strategy

### Manual Testing
- [ ] Select all when all items are included — all deselect
- [ ] Select all when some items are included — all deselect
- [ ] Select all when no items are included — all select
- [ ] Filter by category, then select all — only filtered items change
- [ ] Budget total updates immediately
- [ ] Refresh page — state persists

### Storybook
- [ ] Checkbox story shows indeterminate state

## Dependencies

None — all required components and primitives already exist.

## Out of Scope

- Bulk delete or other bulk actions beyond include/exclude
- Keyboard shortcut for select all
