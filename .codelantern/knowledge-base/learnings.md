---
type: learnings
---

_Distilled from session experience by `/consolidate`. See `conventions.md` for rules established at project discovery._

## DO

- **Add a new item field in two files.** To add an optional field to a budget `Item`, extend the `Item` interface (`src/App.tsx`) and add the control to `AddItemForm` — nothing else. `addItem`/`updateItem` spread the whole item and `handleExport`/`handleFileChange` serialize the whole `items` array, so persistence, export, and import carry the field with no extra wiring. _from linear_TRA_

## DON'T

- **Don't read a red `npm run lint` as your change failing.** This repo has 3 pre-existing lint errors on a clean checkout — `react-hooks/set-state-in-effect` in `BudgetDialog.tsx` and `react-refresh/only-export-components` in the shadcn `ui/badge.tsx` and `ui/button.tsx`. Judge a change by whether it adds errors to the files it touches: stash the change and re-lint to compare. _from linear_TRA_
