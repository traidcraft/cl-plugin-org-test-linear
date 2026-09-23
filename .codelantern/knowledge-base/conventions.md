---
type: conventions
---

# Conventions

## Route PM and SCM operations through the CodeLantern MCP

- Date: 2026-09-22
- Source skill: /discover

All issue, pull-request, and project-board operations go through the CodeLantern MCP server (`get_issue`, `search_issues`, `update_issue`, `create_pull_request`, `add_pull_request_comment`, `get_execution_context`, …); never fall back to another MCP for PM/SCM work. Use the Context7 MCP for library documentation. Keeping one server authoritative for project state avoids split-brain issue/PR data. (`AGENTS.md`.)

## Follow the CodeLantern skill workflow for feature work

- Date: 2026-09-22
- Source skill: /discover

Feature work follows the fixed pipeline: spec → claim-issue → [design-solution → review-design / reduce-complexity] → create-impl-plan → approve-plan → implement → consolidate → finalize → review-code → merge. The design-check steps are for larger or riskier work; small issues may go straight from claim-issue to create-impl-plan. Each skill ends by naming the next step, so the workflow is self-guiding. (`AGENTS.md`.)

## Write app source with double quotes, no semicolons, 2-space indent

- Date: 2026-09-22
- Source skill: /discover

Application source uses double quotes, no semicolons, and 2-space indentation (e.g. `src/components/AddItemForm.tsx`, `src/lib/utils.ts`, `src/hooks/useLocalStorage.ts`). Storybook `.stories.tsx` files and `src/components/__mocks__/data.ts` are the exception — they use double quotes *with* semicolons. There is no Prettier config, so this split is not tool-enforced: match the style of the file type you are editing.

## Treat lint and strict types as the quality gate

- Date: 2026-09-22
- Source skill: /discover

Quality is enforced by ESLint and the TypeScript compiler, not a formatter. `npm run lint` runs `eslint .` over `**/*.{ts,tsx}` with a flat config extending `js.recommended`, `typescript-eslint.recommended`, `react-hooks`, `react-refresh/vite`, and `storybook/flat/recommended` (`dist` ignored). TypeScript runs in strict mode with `noUnusedLocals`, `noUnusedParameters`, `noFallthroughCasesInSwitch`, and `erasableSyntaxOnly` (`tsconfig.app.json`) — leave no unused locals or params. The build gate is `tsc -b && vite build`, so a type error fails the build. Only the non-type-checked ESLint tier is currently on. (`eslint.config.js`, `tsconfig.app.json`, `package.json`.)

## Declare components as PascalCase named `export function`

- Date: 2026-09-22
- Source skill: /discover

Feature components are PascalCase function declarations exported as named exports (`export function AddItemForm(...)`, `export function EquipmentItem(...)`); only the root `App` uses `export default`. Prefer `export function X()` over `const X = () => …` for components. This keeps a single default export at the app root and named exports everywhere else.

## Build shadcn/ui primitives with the registry pattern

- Date: 2026-09-22
- Source skill: /discover

Primitives in `src/components/ui/` follow the shadcn "new-york" registry convention: a lowercase `function Button(...)` declaration, a `data-slot` attribute on the root element (e.g. `data-slot="card-header"`), and a single bottom `export { ... }` block. Variants use `class-variance-authority` (`cva`) with a `defaultVariants` block, and the variants object is exported alongside the component (`button.tsx`, `card.tsx`, `badge.tsx`). Class names in this layer are composed with the `cn()` helper (`src/lib/utils.ts`, `clsx` + `tailwind-merge`); feature components instead use raw template-literal class strings, so `cn()` is reserved for the `ui/` layer.

## Name hooks `useX`, one per file in `src/hooks/`

- Date: 2026-09-22
- Source skill: /discover

Custom hooks are named `useX`, live one-per-file in `src/hooks/`, are generic-typed, and are named exports. `useLocalStorage<T>(key, defaultValue)` returns a `useState`-style `[value, setValue]` tuple (`src/hooks/useLocalStorage.ts`), so it drops in wherever `useState` would go.

## Type props with an `interface <Component>Props`

- Date: 2026-09-22
- Source skill: /discover

Feature components declare an `interface <Component>Props` above the component and destructure it in the signature (`AddItemForm.tsx`, `EquipmentItem.tsx`). Use `interface` for object shapes and `type` aliases only for unions (e.g. `type SortColumn = "name" | …` in `App.tsx`). shadcn primitives instead type props inline via `React.ComponentProps<"button"> & VariantProps<…>`.

## Separate type imports with `import type`

- Date: 2026-09-22
- Source skill: /discover

`verbatimModuleSyntax` is on (`tsconfig.app.json`), so value and type imports must be separated: use `import type { Item } from "@/App"`, the inline `type` specifier (`import { cva, type VariantProps }`), or `import type { Meta, StoryObj }`. A mixed value/type import will fail the build.

## Import intra-src modules via the `@/` alias

- Date: 2026-09-22
- Source skill: /discover

All imports within `src/` use the `@/` path alias (`@/*` → `./src/*`, defined in `tsconfig` and mirrored in `components.json` and `vite.config.ts`); never use deep relative paths. The one exception is sibling imports inside Storybook stories, which use a relative `./` (e.g. `./AddItemForm`).

## Order imports external, then internal, then types

- Date: 2026-09-22
- Source skill: /discover

Observed (not linted) import order: external packages first (`react`, `lucide-react`), then `@/` internal modules, then type imports last (`AddItemForm.tsx`, `App.tsx`). Keep new imports in this order.

## Fail silently on storage errors and guard with early returns

- Date: 2026-09-22
- Source skill: /discover

Browser-storage access is wrapped in try/catch and falls back to a default rather than throwing (`useLocalStorage.ts` returns `defaultValue` on error; a full or unavailable store fails silently). Validation guards inline and early-returns rather than throwing (e.g. `if (!editedName.trim() || isNaN(price) || price < 0) return` in `EquipmentItem.tsx`). Errors from best-effort operations should not surface to the user.

## Persist top-level state via `useLocalStorage` with `budget-*` keys

- Date: 2026-09-22
- Source skill: /discover

Anything that should survive a reload uses `useLocalStorage` rather than bare `useState`, with a `budget-` key prefix (`App.tsx` uses it for items, budget, project name, categories, tax rate, and selected category). Transient UI state (modal flags, editing id, sort order) stays in plain `useState`.

## Test via co-located Storybook CSF stories

- Date: 2026-09-22
- Source skill: /discover

There are no Jest/Vitest unit-test files; tests are Storybook CSF stories co-located next to each component as `<Component>.stories.tsx` (every feature component and ui primitive has one). A story has a typed `meta` with `satisfies Meta<typeof X>`, a grouped `title` (e.g. `"Features/AddItemForm"`), `tags: ["autodocs"]`, a default-exported meta, then `type Story = StoryObj<typeof meta>` with each variant named-exported. Callbacks are mocked with `fn()` from `storybook/test`. Vitest is installed but wired to Storybook via `@storybook/addon-vitest`, not standalone.

## Share Storybook fixtures from `__mocks__/data.ts`

- Date: 2026-09-22
- Source skill: /discover

Shared mock/fixture data lives in `src/components/__mocks__/data.ts` (`mockItems`, `mockCategories`, `mockCategoryBreakdown`) and is imported by stories; reuse these rather than inlining fixtures per story.

## Import lucide-react icons individually

- Date: 2026-09-22
- Source skill: /discover

Icons come from `lucide-react` (the configured `iconLibrary` in `components.json`), imported individually (e.g. `import { DollarSign, Plus } from "lucide-react"`) and sized with Tailwind utilities (`w-4 h-4`).

## Render separate mobile and desktop layout blocks

- Date: 2026-09-22
- Source skill: /discover

Feature components render distinct mobile and desktop layouts rather than one fluid block: a `md:hidden` mobile block and a `hidden md:grid md:grid-cols-12` desktop block, each labeled with a `{/* Mobile Layout */}` / `{/* Desktop Layout */}` comment (`EquipmentItem.tsx`). Keep the two blocks in sync when changing a row's fields.
