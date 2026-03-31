# Branch: feature/tra-8-add-select-deselect-all-checkbox-to-item-grid

## TL;DR

Added a "select all" checkbox to the item grid header that bulk-toggles the included/excluded state of all visible (filtered) items. Updated the Checkbox component to support indeterminate visual state, then wired up derived state and a bulk toggle callback in App.tsx.

## Changes Made

- Extended `Checkbox` component with indeterminate state support (MinusIcon + data-state styles)
- Added `toggleAllIncluded` callback that operates only on filtered items
- Added `headerChecked` derived state via `useMemo` (true/false/"indeterminate")
- Replaced static "Include" header text with interactive Checkbox

## Key Decisions

- **Derived state over stored state**
  - **Context:** The header checkbox state needs to reflect the current included status of filtered items
  - **Options:** Store a separate "selectAll" boolean vs. derive from item state
  - **Rationale:** Derived state stays in sync automatically — no risk of stale state when individual items are toggled or filters change

- **Radix Indicator with group-data selectors for conditional icons**
  - **Context:** Radix Checkbox Indicator renders its children for both checked and indeterminate states
  - **Options:** Conditional rendering with JS vs. CSS-only via Tailwind group-data selectors
  - **Rationale:** CSS-only approach avoids needing to access checkbox state inside the Indicator; uses Tailwind's `group-data-[state=*]` pattern for clean conditional display

## MCP Servers Used

- **CodeLantern MCP:**
  - Created issue TRA-8, set size/priority, managed labels throughout workflow
  - Created draft PR #3, posted issue comments

## Files Changed

**Modified:**
- `src/components/ui/checkbox.tsx` — Added MinusIcon import, indeterminate CSS classes, group-data conditional icon rendering
- `src/App.tsx` — Added Checkbox import, toggleAllIncluded callback, headerChecked useMemo, replaced header text with Checkbox

## Related

- **PR:** #3
- **Issue:** TRA-8
