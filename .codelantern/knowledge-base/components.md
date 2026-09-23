---
type: components
---

# Components

**Storybook is the source of truth for this component library.** Config lives in `.storybook/main.ts` and `preview.ts` (Storybook 10 on `@storybook/react-vite`, globbing `src/**/*.stories.@(js|jsx|mjs|ts|tsx)`; `preview.ts` imports `src/index.css` so stories render with the real theme). Nearly every component has a colocated `.stories.tsx`, so consult the live stories (`npm run storybook`, port 6006) rather than treating the inventory below as authoritative — it will drift.

## Component inventory by role

**Views / feature components** (`src/components/`) — `BudgetTracker.tsx`, `CategoryManager.tsx`, `EquipmentItem.tsx` (one row per item, rendered by mapping sorted items), `CommandPalette.tsx` (`cmdk` ⌘K palette).

**Modals / dialogs** — `AddItemDialog.tsx` (thin shell around `AddItemForm`), `BudgetDialog.tsx` (set budget + tax rate). `CategoryManager` also renders a dialog internally.

**Forms** — `AddItemForm.tsx` (holds local field state; calls `onAdd` / `onUpdate`).

**App shell / other** — `src/App.tsx` (top-level composition; also imports `Badge`), `src/main.tsx` (entry), `src/components/__mocks__/data.ts` (shared Storybook fixtures).

## Vendored UI primitives

shadcn/ui set in `src/components/ui/` (style "new-york", base color neutral), all built on the unified `radix-ui` package (not `@radix-ui/react-*`):

| Primitive | Underlying | Used by feature code? |
|-----------|-----------|-----------------------|
| `button` | radix Slot + cva | Yes (AddItemForm, BudgetTracker, BudgetDialog, CategoryManager, EquipmentItem) |
| `card` | plain | Yes (BudgetTracker) |
| `dialog` | radix Dialog | Yes (AddItemDialog, BudgetDialog, CategoryManager) |
| `input` | plain | Yes (most feature components) |
| `checkbox` | radix Checkbox | Yes (AddItemForm, EquipmentItem) |
| `label` | radix Label | Yes (AddItemForm, BudgetDialog, CategoryManager) |
| `progress` | radix Progress | Yes (BudgetTracker) |
| `tooltip` | radix Tooltip | Yes (BudgetTracker) |
| `command` | cmdk | Yes (CommandPalette) |
| `badge` | radix Slot + cva | Only by `src/App.tsx` |
| `select` | radix Select | **Unused** — only referenced by its own story (candidate dead code) |

## Storybook coverage

Effectively 100%. Feature stories: `AddItemDialog`, `AddItemForm`, `BudgetDialog`, `BudgetTracker`, `CategoryManager`, `CommandPalette`, `EquipmentItem` (under `src/components/`). Primitive stories: `badge`, `button`, `card`, `checkbox`, `command`, `dialog`, `input`, `label`, `progress`, `select`, `tooltip` (under `src/components/ui/`). Stories pull fixtures from `src/components/__mocks__/data.ts`. Storybook 10 runs `addon-a11y` (accessibility), `addon-vitest` (story-based testing), and Chromatic (`@chromatic-com/storybook`) for visual regression.

## Design tokens

All tokens live in `src/index.css` (Tailwind v4, CSS-first — no JS `tailwind.config`). An `@theme inline` block maps Tailwind theme tokens to CSS vars (`--color-*` for background/foreground/card/popover/primary/secondary/muted/accent/destructive/border/input/ring, a `chart-1..5` palette, and a full `sidebar-*` family) plus a radius scale (`--radius-sm/md/lg/xl/2xl/3xl/4xl` derived from `--radius: 0.625rem`). `:root` holds light-theme values in oklch; `.dark` overrides the same set for dark theme; `@layer base` sets global border-color and body defaults. A `@custom-variant dark` defines the `.dark` class strategy.

## Styling approach

Tailwind CSS v4 utility classes (`@import "tailwindcss"` in `index.css`; also imports `tw-animate-css` and `shadcn/tailwind.css`). Class names are merged with the `cn()` helper (`src/lib/utils.ts` — `twMerge(clsx(inputs))`). Variant APIs use `class-variance-authority` (`cva` + `VariantProps`) in `ui/button.tsx` and `ui/badge.tsx`.

## Icon system

`lucide-react` exclusively (`iconLibrary: "lucide"` in `components.json`), imported individually per component and inside primitives (e.g. `XIcon` in dialog, `CheckIcon` in checkbox, `SearchIcon` in command).

## Notes

- `select.tsx` is vendored but consumed only by its own story — dead code or reserved for future use.
- `Badge` is used only in `App.tsx`, not by the feature components.
- No JS Tailwind config exists; all theming is CSS-variable driven in `src/index.css`.
