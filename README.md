# DHTMLX Skills

This repository contains AI coding skills for working with DHTMLX components.

Official site:

https://dhtmlx.com/

MCP server docs:

https://docs.dhtmlx.com/suite/guides/ai/

MCP server endpoint:

https://docs.dhtmlx.com/mcp

## Skills

### `dhtmlx-react-gantt`

Guides AI agents through building and integrating DHTMLX React Gantt in web applications. Covers:

- Package installation (trial and commercial)
- CSS setup and layout requirements
- React integration patterns
- Task and link state, mapping, and persistence
- Styling, CSS variables, template-based visual customization
- Common pitfalls and constraints

### `dhtmlx-angular-gantt`

Guides AI agents through building and integrating DHTMLX Angular Gantt in web applications. Covers:

- Package installation (trial and commercial)
- CSS setup and layout requirements
- Angular integration patterns
- Task and link state, mapping, and persistence
- Styling, CSS variables, template-based visual customization
- Common pitfalls and constraints

### `dhtmlx-react-scheduler`

Guides AI agents through building and integrating DHTMLX React Scheduler in web applications. Covers:

- Package installation (trial and commercial)
- CSS setup and layout requirements
- React integration patterns
- Event state, mapping, and persistence
- Scheduler views, resources, lightboxes, and conflict checks
- Styling, CSS variables, template-based visual customization
- Common pitfalls and constraints

### `dhtmlx-js-gantt`

Guides AI agents through building and integrating core DHTMLX JavaScript Gantt in JavaScript and TypeScript applications. Covers:

- Package installation and setup for `@dhx/gantt` and `@dhx/trial-gantt`
- CSS setup, initialization, and lifecycle requirements
- `gantt.parse`, `gantt.load`, DataProcessor, task/link CRUD, and persistence
- Templates, resources, calendars, undo/redo, row reorder, and sort order
- Styling, CSS variables, template-based visual customization, and theming
- Common pitfalls and constraints

## Installation

```bash

npx skills add DHTMLX/skills --skill dhtmlx-js-gantt
npx skills add DHTMLX/skills --skill dhtmlx-react-gantt
npx skills add DHTMLX/skills --skill dhtmlx-angular-gantt
npx skills add DHTMLX/skills --skill dhtmlx-react-scheduler

```

Or copy manually:

```bash
git clone https://github.com/DHTMLX/skills
cp -r skills/dhtmlx-js-gantt ~/.claude/skills/
cp -r skills/dhtmlx-react-gantt ~/.claude/skills/
cp -r skills/dhtmlx-angular-gantt ~/.claude/skills/
cp -r skills/dhtmlx-react-scheduler ~/.claude/skills/

```

## Requirements

- An AI coding agent that supports the Agent Skills standard (Claude Code, Cursor, Copilot, Codex, Gemini CLI, and others)
