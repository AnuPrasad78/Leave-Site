# EMIDS Absence Management - Frontend

A leave management web app for a company. This file covers the **frontend only**. Build the UI to match the reference images in `docs/design/` using a reusable component system and industry-standard code quality.

Backend is out of scope for now. Use typed static/mock data placed in `src/mocks/` and keep it out of components so it can be replaced later.

## Tech Stack
- React 18 + TypeScript (strict) + Vite
- Tailwind CSS with design tokens in `tailwind.config.ts`
- React Router
- React Hook Form + Zod for forms
- date-fns for dates
- Vitest + React Testing Library
- ESLint + Prettier + `tsc --noEmit`
- npm only. Do not add dependencies without asking and explaining why.

## Commands
- `npm run dev` - start dev server
- `npm run build` - production build
- `npm run lint` - ESLint
- `npm run typecheck` - TypeScript check
- `npm run test` - run tests
- `npm run check` - lint + typecheck + test. Run before saying a task is done.

## How to Build From Reference Images
Reference images live in `docs/design/` and are the source of truth for visuals.

For every screen:
1. Look at the image carefully. List the layout regions, repeated patterns and distinct UI elements you see.
2. Identify which parts already exist in `components/ui`. Reuse them.
3. Identify what is new and reusable (e.g. a stat tile, status badge). Build it in `components/ui` first, then use it in the screen.
4. Build the screen by composing components. The screen file should read like a layout, not contain styling details.
5. Compare against the image: spacing, font sizes, weights, colors, alignment, borders, icon sizes. Fix differences.
6. Check mobile, tablet and desktop widths.

Do this at the start, before building any screen:
- Extract colors, font family, font sizes, font weights, spacing scale, border radius and border colors from the images into Tailwind tokens. Do not guess values. If a value is unclear, ask.
- Build the base component kit first (below), then screens.

If an image is ambiguous or a detail is missing (hover state, error state, mobile layout), ask or choose a sensible default consistent with the existing design, and mention the assumption in your summary.

## Component System (reusability is a hard requirement)

### Structure
```
src/
  app/                  # router, providers, layout shell
  components/
    ui/                 # generic building blocks, zero business logic
    layout/             # TopBar, Drawer, PageHeader, PageContainer
  features/<name>/      # screen-specific components, hooks, schemas, types
  hooks/                # shared hooks
  lib/                  # utils, constants, formatters
  mocks/                # typed mock data
  types/                # shared types
docs/design/            # reference images
```

### Rules
- **Two layers.** `components/ui` is generic and reusable (Button, Input, Badge, Card...). `features/*` composes them into domain components (LeaveRequestTable, BalanceSummary...).
- **No duplication.** If the same pattern appears twice, extract it. Before creating a component, search for an existing one that can be extended through props or variants.
- **Variants over copies.** Use a variants approach (e.g. `variant`, `size`, `tone` props, via `class-variance-authority` if approved) instead of separate near-identical components.
- **Props API.** Small, typed, predictable. Accept `className` for layout overrides, forward refs on form controls, spread native props where sensible. Prefer composition (`children`, slots) over many boolean flags.
- **Styling.** Tailwind utilities with tokens only. No inline styles, no hardcoded hex values, no arbitrary values (`w-[137px]`) unless unavoidable and commented. Use a `cn()` helper for class merging.
- **Presentational vs. logic.** UI components receive data and callbacks via props. They never fetch data or hold business rules.
- **Single responsibility.** One component per file. Keep components under ~150 lines; extract subcomponents or hooks when they grow.
- **Consistent API across the kit.** Same prop names for the same ideas (`size`, `variant`, `disabled`, `error`, `label`).
- Each reusable component gets a short usage comment at the top (what it is for) and is exported via `components/ui/index.ts`.

### Base kit to build first (based on the reference images)
Button (primary, secondary, ghost, danger), IconButton, Input, Textarea, Select, DatePicker, FormField (label + control + error), ToggleGroup / Chip group, Badge (status with dot), Card, StatTile, ProgressBar, DonutChart, DataTable, Modal, Drawer, Banner/Callout, Avatar, Tooltip, Toast, Skeleton, EmptyState, ErrorState, PageHeader (eyebrow + title).

Build extra components only when a screen needs them.

## Screens
Screens come from images the user provides in `docs/design/`. Typical screens for this app: Dashboard, Apply Leave form, My Requests table, Holiday Calendar, side drawer navigation. Others may be added.

- Each screen lives in `features/<name>/` with its own components, hooks and schemas.
- Routes are lazy-loaded.
- Page layout (header, container, spacing) comes from shared layout components, not repeated per screen.

## Quality Requirements

### States
Every view that shows data handles **loading** (skeleton), **empty** and **error** states using the shared components.

### Responsiveness
Design mobile-first. Must work from 360px to wide desktop. Tables scroll horizontally or collapse to cards on small screens. Drawer works with touch.

### Accessibility (WCAG 2.1 AA)
- Semantic HTML (`button`, `nav`, `main`, `table`, headings in order).
- Every input has a label. Errors are linked with `aria-describedby`.
- Visible focus styles. Fully keyboard operable.
- Modals and drawers trap focus, close on Esc, restore focus.
- Color is never the only indicator (status badges also show text).
- Sufficient contrast. Icon-only buttons have `aria-label`.

### Forms
- React Hook Form + Zod. Schemas in `schemas.ts`; derive types with `z.infer`.
- Inline field errors, disabled/loading submit state, no double submit.

### Performance
Lazy-load routes, avoid unnecessary re-renders, memoize only when needed, keep bundle lean.

## Coding Standards
- TypeScript strict. No `any` (use `unknown` and narrow). No `@ts-ignore` without a one-line reason.
- Function components and hooks only. Named exports (default only for lazy route pages).
- Naming: components `PascalCase.tsx`, hooks `useThing.ts`, utilities `camelCase.ts`, constants `UPPER_SNAKE_CASE`.
- No magic numbers or strings: use constants, enums or union types.
- Business and formatting logic goes into pure, tested functions in `lib/` or feature hooks, not inside JSX.
- No `console.log`, commented-out code, dead code or unused imports.
- Dates handled with date-fns only; format at the UI edge.
- Comments explain why, not what. Prefer clear code over clever code.
- Do not refactor unrelated code during a task. Do not loosen lint or TypeScript settings to silence errors.

## Testing
- Unit tests for utilities and pure logic.
- Component tests for reusable UI components (rendering, variants, keyboard interaction) and for key forms.
- Query by role and label (`getByRole`, `getByLabelText`), not by implementation details.
- Test files sit next to the code: `Button.test.tsx`.

## Git and Workflow
- Branches: `feature/<name>`, `fix/<name>`. Conventional Commits (`feat:`, `fix:`, `refactor:`, `test:`, `chore:`, `docs:`).
- Small, focused commits. Never commit secrets or `.env.local`.
- For a new screen, a new shared component, or any change touching more than ~3 files: **post a short plan first and wait for approval.**

## Build Order
1. Project setup (Vite, TS strict, Tailwind, ESLint, Prettier, Vitest, folder structure, `npm run check`)
2. Design tokens extracted from the reference images
3. Base component kit (`components/ui`) with tests
4. Layout shell (TopBar, Drawer, PageHeader, routing)
5. Screens one at a time, in the order the images are provided
6. Responsiveness, accessibility and states pass

## Definition of Done (per screen or component)
- Visually matches the reference image at desktop and mobile widths
- Built from reusable components; no copied styling or duplicated markup
- Loading, empty and error states handled
- Keyboard accessible with proper labels and focus handling
- `npm run check` passes
- Short summary of what was built, components reused/added, and any assumptions

## Do Not
- Do not build backend, API or database code.
- Do not hardcode colors, fonts or spacing outside Tailwind tokens.
- Do not build a screen without a reference image or approval.
- Do not create a new component if an existing one can be extended.
