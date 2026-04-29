# Data And CRUD

Use this file when tasks, links, state ownership, or persistence are involved.

## Data Model Essentials

In Angular wrapper integrations, keep a normalized app model and map it explicitly to Gantt task/link payloads.

Minimum task shape:
```ts
interface Task {
  id: string | number;
  text: string;
  start_date: Date;
  duration: number;
  progress: number;
  parent: string | number;
  type?: "task" | "project" | "milestone";
  open?: boolean;
  end_date?: Date;
}
```

Minimum link shape:
```ts
interface Link {
  id: string | number;
  source: string | number;
  target: string | number;
  type: "0" | "1" | "2" | "3";
}
```

When loading from backend, map rows into app task/link models explicitly. Do not pass raw DB rows straight through.

## Choose Ownership Model First

Choose one model per page/feature area and keep it consistent.

Angular state/store as source of truth (recommended for most apps):
- store/component state owns `tasks` and `links`
- wrapper receives arrays through inputs
- edits are captured in `data.save` or `data.batchSave`
- callbacks update state/store and arrays flow back into `<dhx-gantt>`

Gantt as source of truth (specialized/performance-heavy pages):
- chart and backend own most runtime lifecycle
- Angular mirrors only when required
- requires a clear reconciliation strategy

Do not mix both models casually.

## CRUD Default

For regular CRUD, prefer `data.save`.

Pattern:
```ts
dataConfig: AngularGanttDataConfig = {
  save: async (entity, action, item, id) => {
    // call store action
    // store action normalizes payload
    // store action persists backend write
    // store action updates state on confirmed success
  },
};
```

Strict rules:
- route persistence through store/service actions, not inline component mutation spaghetti
- normalize callback payloads before backend writes
- for create with backend-assigned IDs, return remap response (`{ id }` or `{ tid }`) when using per-change save flow
- update state from latest snapshot, not stale captured arrays
- keep links persistence guarded by real persisted task IDs
- for `update`, persist only normalized values
- for `delete`, remove from state and persistence layer
- normalize date values before serialization
- if using temporary IDs, replace them deterministically after persistence
- persist a link only when both `source` and `target` already refer to real persisted task IDs
- if a task still has a temporary client-side ID, defer or reject link persistence until the real task ID replacement flow completes
- do not send links that reference temporary IDs to the backend
- build backend payloads explicitly from normalized task/link models, not from raw callback objects

## When To Use `batchSave`

Use `data.batchSave` when:
- heavy operations can emit many changes
- bulk backend sync is more appropriate than single-entity persistence
- you need grouped change handling rather than one callback per edit

Pattern:
```ts
dataConfig: AngularGanttDataConfig = {
  batchSave: (changes) => {
    // apply grouped task/link/resource changes
    // persist grouped payloads
    // update store in one coherent transition
  },
};
```

Do not switch to `batchSave` casually. Verify the exact expected behavior with MCP first.

## ID Remapping And Temporary IDs

Create flows often start with temporary client-side IDs.

Rules:
- backend must return persistent IDs
- in `save` mode, return `{ id: dbId }` or `{ tid: dbId }` so Gantt can remap
- do not persist links that reference temporary task IDs
- replace temporary IDs deterministically in state after create confirmation

## Row Reorder And Sortorder

Treat row reorder as dedicated flow, not normal update.

Rules:
- detect reorder intent from update payload (`target` when available)
- rebuild full ordered task list in memory
- recompute `sortorder` for all affected tasks
- persist full reordered set
- if hierarchy changes during move, persist `parent_id` together with `sortorder`
- load tasks ordered by `sortorder`

## Persistence Guardrails

- Do not assume task updates always contain real `Date` instances.
- Normalize date-like inputs before serialization and persistence.
- Build backend payloads explicitly from normalized task/link models, not from raw callback objects.
- Keep frontend ordering (`sortorder`), backend payloads, and DB schema aligned.
- If IDs are temporary on create, make ID replacement flow explicit and deterministic.
- If the app introduces undo/redo, persistence rules must remain correct after history actions.
