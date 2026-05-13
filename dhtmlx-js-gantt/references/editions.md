# Editions

Use this file to identify which Gantt edition is installed, pick the right install/import block, and decide whether a requested feature is available before scaffolding code.

## Three Packages

DHTMLX Gantt ships in three packages. Detect which one is installed by reading `package.json` dependencies; do not guess.

| Package | Edition | Registry | Notes |
| --- | --- | --- | --- |
| `dhtmlx-gantt` | Standard (GPL) | public npm | Free. Missing many PRO features — see list below. |
| `@dhx/trial-gantt` | Professional Evaluation | public npm | 30-day trial. Full PRO surface. |
| `@dhx/gantt` | Professional | private registry `https://npm.dhtmlx.com` | Requires Client's Area credentials + `npm login`. Full PRO surface. |

The `@dhtmlx/trial-*` namespace on public npm is reserved for framework wrappers (`@dhtmlx/trial-react-gantt`, `@dhtmlx/trial-vue-gantt`). The vanilla JS package is never `@dhtmlx/trial-gantt`.

## Detecting The Edition

Different setups expose the edition through different signals — check in this order:

1. **npm-based setup** — read `package.json` dependencies. `dhtmlx-gantt` → Standard, `@dhx/trial-gantt` → Trial PRO, `@dhx/gantt` → PRO. Stop here when one matches.
2. **Script-tag / CDN / vendored build** — there is no `package.json` entry, so fall back to the JS file or the runtime object:
   - Open the served `dhtmlxgantt.js` (CDN url or local copy) and look for the `@license` banner near the top. The banner names the version and the edition (e.g. `dhtmlxGantt v.9.1.0 Standard`, `... Professional Edition`).
   - If the banner has been stripped (rare — only by deliberate post-processing), the runtime object still carries the metadata:
     - `gantt.license` — one of `"gpl"`, `"individual"`, `"commercial"`, `"enterprise"`, `"ultimate"`
     - `gantt.version` — version string
   - Map `gantt.license`: `"gpl"` → Standard; everything else is a PRO edition.

## Install + Import

### Standard (GPL)

```bash
npm install dhtmlx-gantt
```

```ts
import { gantt } from "dhtmlx-gantt";
import "dhtmlx-gantt/codebase/dhtmlxgantt.css";
```

`Gantt` factory is also exported: `import { gantt, Gantt } from "dhtmlx-gantt"`. Multiple instances on the same page are PRO-only — see below.

### Professional Evaluation (Trial)

```bash
npm install @dhx/trial-gantt
```

```ts
import { gantt } from "@dhx/trial-gantt";
import "@dhx/trial-gantt/codebase/dhtmlxgantt.css";
```

### Professional (Commercial)

```bash
npm config set @dhx:registry=https://npm.dhtmlx.com
npm login --registry=https://npm.dhtmlx.com --scope=@dhx
npm install @dhx/gantt
```

```ts
import { gantt } from "@dhx/gantt";
import "@dhx/gantt/codebase/dhtmlxgantt.css";
```

Credentials are generated in the [Client's Area](https://dhtmlx.com/clients/). The user must run the registry config and login commands before install.

## PRO-Only Features

The following are unavailable in Standard (`dhtmlx-gantt`). Source: <https://docs.dhtmlx.com/gantt/guides/editions-comparison/>.

- Resource management (resource grid, resource timeline, resource histogram, workload, assignments)
- Baselines / deadlines / custom timeline elements
- Critical path calculation
- Auto-scheduling
- Constraint control (constraint_type / constraint_date)
- Split tasks
- Tasks grouping
- Dynamic loading (branch loading)
- Projects and milestones as task types
- Custom task types (`gantt.config.types`)
- Auto type detection (`gantt.config.auto_types`)
- Calendar assignment to project
- Calendar assignment to resource
- Decimal duration units (decimal hours/days)
- Multiple Gantt charts on one page (factory pattern with `Gantt.getGanttInstance()`)
- Custom scale — hiding time units
- Resizing grid columns from the UI
- Hiding/showing grid columns from the UI
- Link formatter for the Predecessor inline editor

Standard supports basic CRUD, lightbox, single working calendar, sorting, multiselection, inline editing, export to PDF/PNG/Excel/MS Project, undo/redo, locale, RTL, accessibility, smart rendering, tooltips, and jQuery integration.

## Detect-And-Warn Behavior

Before scaffolding any feature from the PRO-only list above:

1. Inspect `package.json` to determine the edition.
2. If the package is `dhtmlx-gantt`, surface a warning to the user that the requested feature is PRO-only and points to upgrade options (`@dhx/trial-gantt` for evaluation, `@dhx/gantt` for production).
3. Scaffold the requested code anyway. Do not silently substitute, omit, or rewrite the feature.
4. If the package is `@dhx/trial-gantt` or `@dhx/gantt`, no warning is needed.

This rule lets the user make an informed upgrade decision without losing the requested implementation.
