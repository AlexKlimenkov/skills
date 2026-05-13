---
name: dhtmlx-react-scheduler
description: >
  Builds and integrates DHTMLX React Scheduler into React applications. Covers setup, props,
  templates, themes, data.save, data.batchSave, and
  React integration patterns for @dhtmlx/trial-react-scheduler and @dhx/react-scheduler.
  Applies when working with booking calendars, resource scheduling, event CRUD, timeline/day/week/month
  views, lightboxes, or scheduling conflict checks in React apps — regardless of whether
  "DHTMLX" is mentioned by name. Provides verified API guidance rather than guessing React Scheduler APIs.
---

## Source Of Truth

Use only:
1. The current project's files, structure, and established patterns
2. DHTMLX MCP for React Scheduler API details: https://docs.dhtmlx.com/mcp
3. Official DHTMLX React Scheduler docs as fallback: https://docs.dhtmlx.com/scheduler/integrations/react/

Never invent props, hooks, templates, callback signatures, event names, or backend behavior.

If any React Scheduler API detail is unclear, resolve it through DHTMLX MCP before writing code.

## Preflight

Before writing code, identify:
- **package**: `@dhtmlx/trial-react-scheduler` or `@dhx/react-scheduler` — check `package.json`, imports, lockfiles
- **runtime**: React, Next.js (needs `"use client"`), Remix, or other React-based setup
- **ownership model**: React-managed (default) or Scheduler-managed (large datasets, Scheduler-centric app)
- **persistence**: local only, `data.save` (default, single-entity), or `data.batchSave` (bulk sync)

## Workflow

1. Confirm the installed DHTMLX React Scheduler package and import path.
2. Decide the data ownership model before implementing features.
3. Read only the reference file needed for the task:
   - React integration/setup: [references/react-integration.md](references/react-integration.md)
   - CRUD, state, and persistence: [references/data-and-crud.md](references/data-and-crud.md)
   - Failure cases and guardrails: [references/known-failures.md](references/known-failures.md)
   - Advanced patterns (views, resources, conflicts, undo/redo): [references/advanced-patterns.md](references/advanced-patterns.md)
   - Styling, theming, CSS variables, selectors, and template-based visual customization: [references/styling-and-theming.md](references/styling-and-theming.md)
4. Use DHTMLX MCP before relying on advanced or unfamiliar APIs.
5. Implement with documented APIs only.

## MCP Server

This skill relies on DHTMLX MCP for API verification. If the `dhtmlx-mcp` tool is not available, ask the user to add it:

Claude Code:
```bash
claude mcp add --transport http dhtmlx-mcp https://docs.dhtmlx.com/mcp
```

Codex:
```bash
codex mcp add dhtmlx-mcp --url https://docs.dhtmlx.com/mcp
```

If MCP is not available, use the official docs at https://docs.dhtmlx.com/scheduler/integrations/react/ as fallback.

## Consult MCP First For

Consult DHTMLX MCP before using or changing:
- template callbacks you have not already verified
- recurring event payload shape (`rrule`, `recurring_event_id`, `original_start`, `deleted`, `duration`) and the `modals.onRecurrenceConfirm` flow
- `config.lightbox.sections` section types and `map_to` conventions
- `plugins` prop keys and plugin dependencies (for example `treetimeline` requires `timeline`)
- timeline/units view configuration (`property`, `y_property`, `x_unit`, `x_step`, `x_size`)
- advanced props such as `batchSave`, `customLightbox`, `modals`, `views`, `customViews`, or `filter`
- event handler signatures (`on<EventName>`) when behavior is not fully clear
- theme/skin values supported by the `theme` prop, individual `--dhx-scheduler-*` variables and their inheritance defaults (full catalog: `scheduler-docs/docs/guides/theme-css-variables.md`), and runtime view-switch behavior via `ref` instance API

## Hard Rules

- The Scheduler container must have explicit height.
- CSS import must match the installed package and be a separate import line.
- Activate plugins through the `plugins` prop (`plugins={{ recurring: true }}`). The core `scheduler.plugins({...})` instance method is also supported, but the prop form is tidier, requires less code, and offers no benefit to reach for the instance when the prop covers it.
- Switch skin/theme through the `theme` prop (default `terrace`). Do not branch theme through `ref.instance.setSkin()` when the prop is wired.
- Use the app theme as the single source of truth.
- Prefer JavaScript `Date` objects for `start_date` and `end_date` in React-managed mode.
- Normalize date values before persistence. `data.save` callbacks receive `SerializedEvent` (date strings), not `Date` instances.
- Build backend payloads explicitly from normalized event models.
- Do not use undocumented internals when a documented prop or ref API exists.
- Do not mix React-managed props and imperative instance mutations unless synchronization is intentional.

## Quick Checklist

- [ ] Correct package identified
- [ ] Matching CSS import used
- [ ] Explicit height provided
- [ ] Required features enabled via the `plugins` prop
- [ ] Skin selected via the `theme` prop (not `setSkin()`)
- [ ] Data ownership model chosen
- [ ] Dates normalized before persistence
- [ ] Advanced APIs verified with MCP
