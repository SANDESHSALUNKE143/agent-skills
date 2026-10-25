---
name: react-typescript
description: >-
  Builds React and TypeScript screens from the shared UI package and the
  organization design system, with TDD, Vitest, and Cypress. Use when creating
  or editing pages, components, fonts, theme, controls, or frontend tests.
---

# React TypeScript

Visual rules come from the organization UI design skill. Before a screen is written, open the nearest page in the same feature and match its structure, spacing, and components. Do not invent a second design system.

The console takes inputs from `ui/packages/controls` (`@agentic/controls`). Screens import `TextField`, `Select`, and the other controls from that package. `ui/admin-dashboard/tests/controls-usage.test.ts` fails if a screen writes its own `input`, `select`, or `textarea`.

## Package

Until the organization library exists:

- Shared chrome, cards, dialogs, buttons, inputs, icons, and data grids live in the shared UI package and are named exports from its entry. A new standard control is added to `@agentic/controls` and exported from `ui/packages/controls/src/index.ts`. It is not copied into the app. New shared files use kebab-case. Existing files keep their names.
- Another UI project creates `ui/packages/<name>` with the same shape: `package.json` exports, one public `index.ts`, and a CSS entry. App code depends on that package name. Those packages are what later move into one organization library. A control that already exists is reused.
- An app page may import `react`, `react-dom`, and the shared package. It does not import `@mui/icons-material`, an icon pack, a font package, or a second button or dialog library. Icons are named exports from the shared icon module, sized to `1em`.
- Open-source UI libraries are MIT, Apache-2.0, or BSD. `@mui/material`, `@mui/system`, `@mui/lab`, and the MUI X community packages are allowed only inside the shared package, and only when that project already uses them. This console does not use MUI, so a new screen here does not add it. `@mui/x-data-grid-pro`, `@mui/x-data-grid-premium`, `@mui/x-date-pickers-pro`, and other MUI X Pro or Premium packages are not added. If a project already depends on a paid package, that code stays. New tables, date fields, and charts use an MIT option already in the app, or TanStack Table, React Day Picker, or MUI X community, wrapped by the package.

## Theme and fonts

Color, font, and radius are theme data. Feature code does not paste a hex, a typeface, or a radius.

- Read color from the theme palette: primary, secondary, tertiary, text, success, error, warning, info, divider, background. Spacing is theme units. Cards and panels use the theme radius. Support light and dark through the palette. Destructive actions use the error color. One primary action per view.
- Default tokens when a project has no brand yet: primary `#1A3A65`, secondary `#C4A880`, tertiary `#616161`, info `#0288d1`, success `#2e7d32`, warning `#ed6c02`, error `#d32f2f`, background `#ffffff` / `#121212`, radius `4`, light mode. These live in the theme module, not in a screen.
- The package is the only module that loads the font. The default face is Roboto. Allowed alternates are Inter, DM Sans, IBM Plex Sans, Dosis, and TT Chocolates. A paid font is not added unless the company approval includes that license. This console still loads Inter and IBM Plex Mono from `ui/admin-dashboard/index.html` and names them in `ui/admin-dashboard/src/styles.css`. Inter may stay. IBM Plex Mono is not on the allowed list, so the package replaces it. After apps read the theme font, a font change is a package version bump. Screens are not edited again for that change.

## Screens

- A feature screen is a column: shared header, optional shared tabs, then content. The title comes from the route. A page header is `h5`. A nested header is `h6`. The subtitle is sentence case. Back is a short text label. Loading in the header is a skeleton, not a spinner in the title.
- Use the shared card, outlined dashboard panel, dialog (`sm` 600, `md` 800, `lg` 1000, `xl` 1200), drawer, and data grid. Do not build an HTML table for an admin list. Do not add a page stylesheet or an inline layout style. This console's existing `styles.css` stays. New layout does not add another file.
- A heavy child (document, embed, chart) sits in its own error boundary. API results are `{ data, error }`. On error, toast the message and leave the screen in place. Empty is one sentence in secondary text, with the primary action when the user can fix it.
- Confirm a destructive action. Do not show secrets, full national IDs, or tokens. Enums go through the shared select and a shared status label. Never show a raw enum.
- Copy is US English. Nav, tabs, page titles, and primary buttons are Title Case. Subtitles, helpers, empty states, and body are sentence case and end with a period. One term per concept. `Create` and `Add` stay different verbs.
- Surfaces stay quiet. Links underline on hover only. Tertiary is for chrome, not for primary actions. A gradient button is rare emphasis only.
- Button labels are authored in the correct case. Tabs and back links force `textTransform: 'none'`. One primary button per view. Pass `loading` while a submit is in flight. Text fields that need a mask use the shared mask (`phone`, `ssn`, `ein`, `tin`, `crd`, `zip`, `currency`, `percent`, `number`). Sensitive values stay hidden until the user reveals them.
- Placeholder chrome is `Coming Soon`. Placeholder body is `Coming soon.`

## Imports and files

Imports are React, then external packages, then the shared package, then relative imports. No `any`. No `@ts-ignore` on new code. Put a stable `data-cy` or `data-testid` on primary actions, dialogs, and regions Cypress must find.

```tsx
import { useState } from "react";
import { PageHeader, PrimaryButton, useNotify } from "<shared-ui>";
```

- One component per file. A new control is its own file under the package, not another function appended to `controls.tsx`. An app screen that is hard to scan is split into components before more JSX is added.
- Reuse a component that already does the job. The same markup is not copied into a second screen.
- Segregate by role. Shared controls stay in `ui/packages/controls`. App-only composition stays in the app as internal components. Custom hooks live in a `hooks` folder. Callbacks that are not the component's own event handler live in a `callbacks` folder. A file does not mix a control, a hook, and a screen.
- Every component, hook, and callback file opens with this note. `external` means a shared package control. `internal` means an app component that composes those controls.

```tsx
/**
 * Kind: external | internal | hook | callback
 * Purpose: what this does.
 * Used by: the importer.
 */
```

## TDD

For a behavior change:

1. Red: write the failing test first and run it so it fails for the right reason.
2. Green: write the smallest change that passes.
3. Refactor only while tests stay green.

## Tests and the pipeline

Vitest and Testing Library stay the runners in `ui/admin-dashboard/package.json`.

- Unit tests cover a component, hook, or pure function on its own. Integration tests render the screen with its data client and assert the control from the package. Both are required for a behavior change.
- Unit coverage of the UI sources must be greater than 90 percent, and integration coverage must be greater than 90 percent. The pipeline fails if either line is missed. Do not exclude a file to pass the gate.
- Most end-to-end functional journeys are Cypress specs against the running app: visit, act, assert. Cypress does not replace the unit or integration coverage. A user-visible flow without a Cypress spec is not done. Visible-string changes update the matching unit test or Cypress assertion.
- TypeScript stays strict. Read nearby tests before adding new ones. Do not ship UI behavior with only production code. The typecheck for each project you edited is clean.

## Delivery

Delivery follows the same pull requests as the API and the infra repo: feature into `dev`, `dev` into `uat`, `uat` into `main`. The UI workflow deploys only after that merge. Merge into `main` deploys prod.
