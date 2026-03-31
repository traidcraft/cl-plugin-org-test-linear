# Branch: feature/tra-4-add-select-all-deselect-all-checkbox-to-item-grid

## TL;DR

Added a select-all / deselect-all checkbox to the desktop equipment grid header. The existing Radix UI Checkbox component was enhanced with indeterminate state support (MinusIcon), then wired into the grid header to bulk-toggle all items' `included` state with immediate budget total recalculation.

## Changes Made

- Enhanced `Checkbox` component with three-state support (checked / indeterminate / unchecked)
- Added `MinusIcon` rendering for indeterminate state with matching `data-[state=indeterminate]` Tailwind styles
- Added `Indeterminate` Storybook story
- Added `allIncluded` / `noneIncluded` derived state and `toggleAllIncluded` callback in `App.tsx`
- Replaced "Include" header text with interactive `<Checkbox>` bound to computed state

## DO (Patterns to Follow)

- **Destructure `checked` prop when conditional rendering depends on it:** The Radix Checkbox passes `checked` through `...props`, but destructuring it explicitly allows conditional icon rendering inside the Indicator.

## DON'T (Mistakes to Avoid)

No anti-patterns encountered during this implementation.

## Key Decisions

- **Conditional rendering over CSS show/hide for icon switching**
  - **Context:** Plan suggested both CSS `group-data-[state=*]` and prop-based approaches
  - **Options:** CSS visibility toggling vs. conditional render based on `checked` prop
  - **Rationale:** Prop-based conditional render is simpler, more readable, and avoids rendering unused DOM nodes

## MCP Servers Used

- **CodeLantern MCP:**
  - Fetched issue #4 details
  - Listed PRs to verify draft PR #2 exists
  - Added `cl-implementing` label

## Files Changed

**Modified:**
- `src/components/ui/checkbox.tsx` — Added MinusIcon import, indeterminate Tailwind styles, conditional icon rendering
- `src/components/ui/checkbox.stories.tsx` — Added Indeterminate story
- `src/App.tsx` — Added Checkbox import, allIncluded/noneIncluded state, toggleAllIncluded callback, replaced header text with Checkbox

## Related

- **PR:** #2
- **Issue:** #4
