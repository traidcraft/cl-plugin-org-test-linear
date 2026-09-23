---
type: decisions
work_item: linear_TRA
---

_Historical record of work on linear_TRA, as of 2026-09-22. The codebase and git history are the source of truth for current behavior._

# Decisions: Issue TRA-11 — Add optional notes field to budget items

## Notes control is a plain styled `<textarea>`, not a new shadcn primitive

- Date: 2026-09-22
- Trigger: Choosing between viable alternatives
- Attribution: Agent-made

The Notes field needs a multi-line control, but the repo has no `Textarea` primitive in `src/components/ui/`. Rather than vendor a new shadcn `Textarea`, the form uses a plain `<textarea>` inline in `AddItemForm`, reusing the same className already applied to the form's raw `<select>`. Rationale: the form already mixes shadcn primitives with raw styled elements (the category `<select>`), so this matches existing local convention; adding a shared primitive the app uses in exactly one place would spread complexity for no reuse benefit. Reversible — a `Textarea` primitive can be extracted later if a second use appears.

## Left three pre-existing lint errors unfixed

- Date: 2026-09-22
- Trigger: Working around a constraint
- Attribution: Agent-made

`npm run lint` reports 3 errors, all confirmed pre-existing on the committed baseline (verified by re-running lint with this change stashed) and all in files untouched by TRA-11: `src/components/BudgetDialog.tsx` (`react-hooks/set-state-in-effect`) and `src/components/ui/{badge,button}.tsx` (`react-refresh/only-export-components`, inherent to shadcn's pattern of exporting a `*Variants` object beside the component). The three files this change edits lint clean and the build passes. Fixing the shadcn primitives would fight the documented registry convention, and the `BudgetDialog` effect is unrelated behavior — both are out of scope for this issue. Flagged here so a reviewer sees they predate this work.
