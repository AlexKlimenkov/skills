---
name: dhtmlx-js-gantt
description: >
  Builds and integrates core DHTMLX JavaScript Gantt into JavaScript or TypeScript
  applications. Covers setup, initialization, lifecycle, gantt.parse/gantt.load,
  templates, DataProcessor, task/link CRUD, resources, calendars, undo/redo, row
  reorder, sortorder, styling, and theming for @dhx/gantt and @dhx/trial-gantt.
  Applies when working with gantt charts, project timelines, task dependencies,
  task CRUD, resource panels, working calendars, lightboxes, workload views, or
  weekend highlighting, regardless of whether "DHTMLX" is mentioned
  by name. Provides verified API guidance rather than guessing core Gantt APIs.
paths: "**/*.ts,**/*.js,**/*.html,**/*.css,package.json,vite.config.*"
allowed-tools: Read Grep
metadata:
  tags: dhtmlx, gantt, dependencies, baselines, milestones, resources
---

## Source Of Truth

Use only:
1. The current project's files, structure, and established patterns
2. DHTMLX MCP for core Gantt API details: https://docs.dhtmlx.com/mcp
3. Official DHTMLX Gantt docs as fallback: https://docs.dhtmlx.com/gantt/

Never invent config names, template callbacks, event names, method signatures, data shapes, or backend behavior.

If any core Gantt API detail is unclear, resolve it through DHTMLX MCP before writing code.

## Preflight

Before writing code, identify:
- **package**: `@dhx/gantt`, `@dhx/trial-gantt` — check `package.json`, imports, lockfiles
- **runtime**: plain JavaScript, TypeScript, Vite, or another app setup
- **ownership model**: app-managed data or Gantt-managed data
- **persistence**: local only, `gantt.createDataProcessor`, or custom backend/API clients

## Workflow

1. Confirm the installed DHTMLX Gantt package and import path.
2. Decide the data ownership model before implementing features.
3. Read only the reference file needed for the task:
   - Setup: [references/setup.md](references/setup.md)
   - CRUD, state, and persistence: [references/data-and-crud.md](references/data-and-crud.md)
   - Failure cases and guardrails: [references/known-failures.md](references/known-failures.md)
   - Advanced patterns (reorder, resources, undo/redo, schema): [references/advanced-patterns.md](references/advanced-patterns.md)
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

If MCP is not available, use the official docs at https://docs.dhtmlx.com/gantt/ as fallback.

## Consult MCP First For

Consult DHTMLX MCP before using or changing:
- template callbacks you have not already verified
- lifecycle patterns around multiple instances or repeated mount/unmount
- advanced DataProcessor modes, delete confirmation, resources, or assignments
- row reorder config, events, and event signatures
- resource panel, resource datastore, resource timeline, or histogram APIs
- working calendar rules and work-time helpers
- lightbox section types and option-list patterns
- CSS variable names beyond the current project usage

## Hard Rules

- The Gantt container must have explicit height.
- Import the matching Gantt CSS.
- Use the app theme as the single source of truth.
- Prefer documented config, templates, events, and methods over undocumented internals.
- Keep DHTMLX init/parse/events/DataProcessor cleanup in one lifecycle boundary.
- Normalize date values before persistence.
- Build backend payloads explicitly from normalized task/link models.
- Return `{ id: databaseId }` or `{ tid: databaseId }` after creates when the backend assigns a real id.
- Do not mix app-managed data and direct Gantt instance mutations unless synchronization is intentional.
- Treat row reorder as a dedicated batch flow, not a normal single-task update.

## Quick Checklist

- [ ] Correct package identified
- [ ] Matching CSS import used
- [ ] Explicit height provided
- [ ] Data ownership model chosen
- [ ] Dates normalized before persistence
- [ ] DataProcessor return values handled
- [ ] Advanced APIs verified with MCP
