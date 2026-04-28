---
name: dhtmlx-react-scheduler
description: >
  Builds and integrates DHTMLX React Scheduler into React applications. Covers setup, props,
  templates, themes, data.save, data.batchSave, and
  React integration patterns for @dhtmlx/trial-react-scheduler and @dhx/react-scheduler.
  Applies when working with booking calendars, resource scheduling, event CRUD, timeline/day/week/month
  views, lightboxes, or scheduling conflict checks in React apps — regardless of whether
  "DHTMLX" is mentioned by name. Provides verified API guidance rather than guessing React Scheduler APIs.
paths: "**/*.tsx,**/*.jsx,**/*.ts,**/*.js,package.json"
allowed-tools: Read Grep
metadata:
  tags: dhtmlx, scheduler, booking, calendar, resources, events
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
- advanced props such as `batchSave`, `customLightbox`, `modals`, `view`, or `filter`
- timeline configuration details and related callback signatures
- event handler signatures (`on<EventName>`) when behavior is not fully clear
- theme/skin methods and runtime view-switch behavior via `ref` instance API
- collision/limit/recurring plugin behavior in combined setups

## Hard Rules

- The Scheduler container must have explicit height.
- CSS import must match the installed package and be a separate import line.
- Use the app theme as the single source of truth.
- Prefer JavaScript `Date` objects for `start_date` and `end_date` in React-managed mode.
- Normalize date values before persistence.
- Build backend payloads explicitly from normalized event models.
- Do not use undocumented internals when a documented prop or ref API exists.
- Do not mix React-managed props and imperative instance mutations unless synchronization is intentional.

## Quick Checklist

- [ ] Correct package identified
- [ ] Matching CSS import used
- [ ] Explicit height provided
- [ ] Data ownership model chosen
- [ ] Dates normalized before persistence
- [ ] Advanced APIs verified with MCP
