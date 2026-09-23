---
type: impl-plan
work_item: linear_TRA
---

_Historical record of work on linear_TRA, as of 2026-09-22. The codebase and git history are the source of truth for current behavior._

# Technical Plan: Issue TRA-11 — Add optional notes field to budget items

**Status:** Draft
**Branch:** feature/tra-11-add-optional-notes-field-to-budget-items
**Created:** 2026-09-22
**Issue:** TRA-11

## Context

Budget items capture only `name`, `category`, `price`, an optional `link`, and the `taxable`/`included` flags (`Item` in `src/App.tsx`). Users have nowhere to record freeform context about a purchase. This adds an optional `notes` field, editable in the add/edit modal only and deliberately absent from the item grid.

## Implementation Approach

Mirror the existing optional `link` field end to end. `notes` becomes an optional property on the shared `Item` interface, and `AddItemForm` gains a matching form control below the Link field. No new persistence, export, import, or state wiring is needed: `App.addItem` spreads `...item`, `App.updateItem` spreads `...updates`, `handleExport` serializes the whole `items` array, and `handleFileChange` restores it — all field-agnostic, so `notes` rides through automatically. Because the field is multi-line, the control is a `<textarea>` rather than an `<Input>`. Consistent with the form's existing raw `<select>`, it is a plain styled `<textarea>` inline in `AddItemForm` — not a new shared shadcn primitive — keeping the change to two files and avoiding a component the app doesn't otherwise need. The grid (`EquipmentItem`) is not touched, which satisfies "never shown in the grid" by construction.

## Phases

### Phase 1: Add the optional notes field

**Goal:** Users can enter and edit multi-line notes on an item in the add/edit modal; notes persist and round-trip through export/import; the grid is unchanged.

- [ ] Task 1.1: Add `notes?: string` to the `Item` interface.
  - Files: `src/App.tsx`
  - Notes: Optional, so pre-existing persisted items (no `notes`) remain valid (AC #6). No other App changes — `addItem`/`updateItem`/export/import are field-agnostic.
- [ ] Task 1.2: Add `notes` state to `AddItemForm`, initialized from `editItem?.notes ?? ""`.
  - Files: `src/components/AddItemForm.tsx`
  - Notes: Pre-fills existing notes when editing (AC #2).
- [ ] Task 1.3: Render a **Notes (optional)** `<textarea>` below the Link field, above the taxable checkbox.
  - Files: `src/components/AddItemForm.tsx`
  - Notes: Use a `<Label htmlFor="notes">` and a plain `<textarea id="notes" rows={3}>` styled to match the existing raw `<select>` className (border-input, rounded-md, focus ring). Not `required`.
- [ ] Task 1.4: Include `notes` in the submitted `itemData` and reset it after submit.
  - Files: `src/components/AddItemForm.tsx`
  - Notes: `notes: notes.trim() || undefined` (mirrors `link`), and add `setNotes("")` alongside the other resets. Blank stays valid (AC #1).
- [ ] Task 1.5: Demonstrate notes in Storybook — give one `mockItems` entry a `notes` value so the `WithEditItem` story renders a populated field.
  - Files: `src/components/__mocks__/data.ts`
  - Notes: Keeps the co-located-story convention meaningful; no new story file needed.

**Checkpoint criteria:**
- [ ] `npm run lint` passes
- [ ] `npm run build` passes (`tsc -b && vite build` — type-checks the new field)
- [ ] Manual: add an item with notes, reload, reopen edit → notes persist and pre-fill
- [ ] Manual: export to JSON → notes present; import that file → notes restored
- [ ] Manual: item grid shows no notes; an item saved before this change still loads and edits

## Files to Modify

| File | Changes | Phase |
|------|---------|-------|
| `src/App.tsx` | Add `notes?: string` to `Item` | 1 |
| `src/components/AddItemForm.tsx` | Add notes state, textarea control, submit + reset | 1 |
| `src/components/__mocks__/data.ts` | Add `notes` to one mock item for the edit story | 1 |

## New Files to Create

None.

## Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|------|------------|--------|------------|
| Old persisted items lack `notes` | Certain | Low | Field is optional; reads as `undefined` and pre-fills as `""` (AC #6) |
| Plain `<textarea>` drifts visually from shadcn inputs | Low | Low | Reuse the exact className already applied to the form's raw `<select>` |

## Test Strategy

- Automated: `npm run lint` and `npm run build` gate the type change and JSX.
- Storybook: the `WithEditItem` story visually confirms the notes field renders and pre-fills (`addon-a11y` covers the label/control association).
- Manual (no unit-test harness exists): the five checkpoint checks above cover all six acceptance criteria — add/blank (AC #1), edit/clear (AC #2), reload persistence (AC #3), export/import round-trip (AC #4), grid absence (AC #5), and backward compatibility (AC #6).

## Out of Scope

- Showing notes anywhere in the item grid (`EquipmentItem`).
- A reusable shadcn `Textarea` primitive.
- Search, filtering, or sorting by notes.
- Rich text / markdown in notes.

## Surfaced candidates

None.
