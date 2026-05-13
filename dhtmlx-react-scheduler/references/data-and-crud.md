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

For normal CRUD, prefer `data.save`. The canonical signature (from the wrapper's `RouterFunction` type) is:

```ts
save: (entity, action, data, id) => {
  if (entity !== "event") return;
  if (action === "create") {
    // persist `data`; return { id: realId } if backend assigns a real ID
  }
  if (action === "update") {
    // persist `data` for event `id`
  }
  if (action === "delete") {
    // delete event `id`
  }
}
```

- `entity` is `"event"` for Scheduler event CRUD. Other entity values may be emitted by extensions — verify with MCP before adding branches.
- `action` is `"create" | "update" | "delete"`.
- `data` is the affected event in **serialized form** (`SerializedEvent` — date fields are ISO strings).
- Return `{ id: realId }` from `"create"` when the backend issues a real ID so the wrapper can replace the temporary client ID.

Strict rules:
- update state from the latest current snapshot, not stale closure state
- for `update`, persist only normalized values
- for `delete`, remove from local state and persistence layer
- if using temporary IDs, replace them deterministically after persistence
- build backend payloads explicitly from normalized event models, not from raw callback objects

## Date Formatting And Normalization

Two contracts apply:
- Props (`events`) — events passed in use JavaScript `Date` for `start_date` and `end_date` (the `Event` interface).
- Callbacks (`data.save`) — the `data` parameter is a `SerializedEvent`; date fields arrive as ISO strings, not `Date` instances.

When mapping callback payloads back into local state that uses `Date`, normalize at the boundary:

```ts
function toDate(value: Date | string): Date {
  const date = value instanceof Date ? value : new Date(value);
  if (Number.isNaN(date.getTime())) throw new Error("Invalid date");
  return date;
}
```

On backend read, map persisted date strings back into `Date` objects before passing events to Scheduler as props.

## Recurring Events

Enable recurring support via the `plugins` prop:

```ts
const plugins: SchedulerPlugins = { recurring: true };
```

Recurring event shape (extends the base `Event` interface):

```ts
interface RecurringEvent extends Event {
  id: string | number;
  text: string;
  start_date: Date;            // series start (or occurrence start on exception rows)
  end_date: Date;              // series end (or occurrence end on exception rows)

  // Series fields
  rrule: string | null;        // RFC-5545 rule on the series row; null on exception rows
  duration: number | null;     // series duration in SECONDS (not ms, not minutes)

  // Exception fields (set only on modified / deleted occurrences)
  recurring_event_id: string | number | null;  // points at the parent series id
  original_start: Date | null;                 // original occurrence date being replaced
  deleted?: boolean | null;                    // true on deleted-occurrence rows
}
```

Three concrete row shapes:

```ts
// 1. Series row — repeats every Monday from 2025-06-03 through 2025-12-02
const seriesRow: RecurringEvent = {
  id: 1,
  text: "Weekly Team Meeting",
  start_date: new Date("2025-06-03T09:00:00"),
  end_date:   new Date("2025-12-02T10:00:00"),
  duration: 3600,
  rrule: "FREQ=WEEKLY;INTERVAL=1;BYDAY=MO",
  recurring_event_id: null,
  original_start: null,
};

// 2. Modified occurrence — the 2025-08-04 instance was moved to 2025-08-06
const modifiedOccurrence: RecurringEvent = {
  id: 2,
  text: "Special Team Meeting",
  start_date: new Date("2025-08-06T12:00:00"),
  end_date:   new Date("2025-08-06T13:00:00"),
  rrule: null,
  duration: null,
  recurring_event_id: 1,
  original_start: new Date("2025-08-04T09:00:00"),
};

// 3. Deleted occurrence — the 2025-06-17 instance was removed
const deletedOccurrence: RecurringEvent = {
  id: 3,
  text: "Deleted Team Meeting",
  start_date: new Date("2025-06-17T09:00:00"),
  end_date:   new Date("2025-06-17T10:00:00"),
  rrule: null,
  duration: null,
  recurring_event_id: 1,
  original_start: new Date("2025-06-17T09:00:00"),
  deleted: true,
};
```

Field rules:
- `rrule` is set **only** on the series row; `null` on exception rows and plain events
- `recurring_event_id` is set **only** on exception rows; `null` on series and plain events
- `original_start` is the date of the occurrence being replaced — required on every exception row, including deleted ones
- `deleted: true` marks a tombstone; the row stays in storage so the series knows to skip that date
- `duration` is the series duration in **seconds** (3600 = 1 hour). Not milliseconds, not minutes.

Persistence model:
- one row for the series
- one row per modified occurrence
- one row per deleted occurrence
- do **not** persist a row per expanded instance — only the series plus its exceptions

User intent on edit/drag of a recurring event is signalled via `modals.onRecurrenceConfirm`. The callback returns (or resolves to) `"occurrence" | "following" | "series" | null`. The `null` return cancels the action. Context fields: `origin` (`"lightbox" | "dnd"`), `occurrence`, `series`, `labels`, `options`.

DHTMLX rrule deviation from iCalendar: `DTSTART`/`DTEND` are **not** embedded in the rrule string; they live in `start_date` and `end_date` on the series row.

## Lightbox Sections

The built-in event editor is configured via `config.lightbox.sections`. Each section is `{ name, map_to, type, ... }`. Built-in section types: `textarea`, `time`, `select`, `radio`, `checkbox`, `template`, `combo`, `multiselect`, `recurring`.

- `map_to` aligns the section to an event field; `map_to: "auto"` lets the section manage its own fields (used by the built-in `time` section).
- The `recurring` section requires the `recurring` plugin and the `rrule`/`recurring_event_id`/`original_start` fields on events.

Pick **one** editor path per screen: built-in `config.lightbox.sections` **or** `customLightbox`. Mixing both produces ambiguous edit behavior. Verify exact section options with MCP before relying on undocumented fields.

## When To Use batchSave

`data.save` is the right default for Scheduler. A single user action in the Scheduler
typically emits one (sometimes two) events, so per-event persistence is adequate.

Reach for `data.batchSave` only when:
- the same project also uses **DHTMLX React Gantt**, which routinely emits batched
  changes, and you want one persistence handler that works for both wrappers
- bulk backend sync is genuinely more appropriate than single-event persistence
- grouped change handling is a hard requirement (e.g. transactional save semantics)

Do not switch to `batchSave` casually. Verify exact behavior and payload shape with MCP first.

## Persistence Guardrails

- Do not persist raw Scheduler callback objects directly.
- Keep app-level custom fields explicit in mapping (for example `location_id`, `resource_id`, `service_id`).
- Reject or sanitize writes that violate application constraints before hitting the backend.
- Keep create/update/delete paths idempotent and deterministic.
- If the app introduces undo/redo, persistence rules must remain correct after history actions.
