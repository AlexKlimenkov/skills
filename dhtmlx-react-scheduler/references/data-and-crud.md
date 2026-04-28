# Data And CRUD

Use this file when events, state ownership, or persistence are involved.

## Data Model Essentials

Prefer JavaScript `Date` objects for event date fields in React-managed mode.

Minimum event shape:
```ts
interface Event {
  id: string | number;
  text: string;
  start_date: Date;
  end_date: Date;
}
```

When loading from a backend, map rows explicitly into Scheduler event objects. Do not pass raw DB rows straight through.

## Choose The Ownership Model First

For most React apps, use React-managed data:
- keep `events` in React state or a state manager
- pass them into `<ReactScheduler />`
- handle updates with `data.save` or `data.batchSave`

Use Scheduler-managed data only when:
- datasets are very large
- mass updates are frequent
- React does not need to react to every edit
- the page is primarily Scheduler-centric

## CRUD Default

For normal CRUD, prefer `data.save`.

Pattern:
```tsx
<ReactScheduler
  events={events}
  data={{
    save: async (entity, action, item, id) => {
      // update local state
      // sync backend
      // return { id } after creates when backend assigns a real ID
    },
  }}
/>
```

Strict rules:
- update state from the latest current snapshot, not stale closure state
- for `create`, if the backend assigns a real ID, return `{ id: realId }`
- for `update`, persist only normalized values
- for `delete`, remove from local state and persistence layer
- normalize date values before serialization
- if using temporary IDs, replace them deterministically after persistence
- build backend payloads explicitly from normalized event models, not from raw callback objects

## Date Formatting And Normalization

Treat `start_date` and `end_date` in persistence callbacks as `Date | string`, and normalize them before backend writes:

```ts
function toIso(value: Date | string): string {
  const date = value instanceof Date ? value : new Date(value);
  if (Number.isNaN(date.getTime())) throw new Error("Invalid date");
  return date.toISOString();
}
```

On backend read, map persisted date strings back into `Date` objects before passing to Scheduler.

## When To Use batchSave

Use `data.batchSave` when:
- one user action can emit many related changes
- bulk backend sync is more appropriate than single-event persistence
- grouped change handling is required

Do not switch to `batchSave` casually. Verify exact behavior and payload shape with MCP first.

## Persistence Guardrails

- Do not persist raw Scheduler callback objects directly.
- Keep app-level custom fields explicit in mapping (for example `location_id`, `resource_id`, `service_id`).
- Reject or sanitize writes that violate application constraints before hitting the backend.
- Keep create/update/delete paths idempotent and deterministic.
- If the app introduces undo/redo, persistence rules must remain correct after history actions.
