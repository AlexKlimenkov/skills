# Data And CRUD

Use this file when tasks, links, state ownership, or persistence are involved.

## Data Model Essentials

Use the `tasks` and `links` shape expected by Gantt.

Example:

```ts
const projectData = {
  tasks: [
    {
      id: "1",
      text: "Website Relaunch",
      type: "project",
      start_date: "2026-05-04T00:00:00",
      duration: 20,
      progress: 0.35,
      open: true,
      parent: 0,
    },
    {
      id: "2",
      text: "Discovery",
      start_date: "2026-05-04T00:00:00",
      duration: 4,
      progress: 0.8,
      parent: "1",
    },
  ],
  links: [{ id: "1", source: "2", target: "3", type: "0" }],
};

gantt.parse(projectData);
```

Common task fields:
- `id`
- `text`
- `start_date`
- `duration`
- `end_date`
- `progress`
- `parent`
- `type`
- `open`

Common link fields:
- `id`
- `source`
- `target`
- `type`

When loading from a backend, map rows explicitly into Gantt task/link objects. Do not pass raw database rows straight through.

## Dates

Use ISO date strings or `Date` objects consistently.

## DataProcessor

`gantt.createDataProcessor()` creates a DataProcessor instance and attaches it to Gantt.

It accepts:
- a predefined request config object
- a router function
- a router config object

Router function:

```ts
gantt.createDataProcessor((entity, action, data, id) => {
  // entity: "task" | "link" | "resource" | "assignment"
  // action: "create" | "update" | "delete"
  // data: processed object
  // id: processed object id
});
```

Router object:

```ts
gantt.createDataProcessor({
  task: {
    create: (data) => Promise.resolve({}),
    update: (data, id) => Promise.resolve({}),
    delete: (id) => Promise.resolve({}),
  },
  link: {
    create: (data) => Promise.resolve({}),
    update: (data, id) => Promise.resolve({}),
    delete: (id) => Promise.resolve({}),
  },
});
```

All router handlers should return either a Promise or a data response object.

If the backend assigns a new id during create, return `id` or `tid` so Gantt can apply the database id:

```ts
gantt.createDataProcessor(async (entity, action, data, id) => {
  if (entity === "task" && action === "create") {
    const created = await api.createTask(data);
    return { tid: created.id };
  }

  return {};
});
```

If the installed version does not replace the id reliably from the DataProcessor response alone, call the documented id-change method for the entity as part of the create flow and still return `{ id }` or `{ tid }`.

```ts
if (entity === "task" && action === "create") {
  const created = await store.createTask(payload);
  if (created.id !== id) gantt.changeTaskId(id, created.id);
  return { tid: created.id };
}
```

## Persistence Guardrails

- Build backend payloads explicitly from normalized task/link models.
- Do not persist raw Gantt callback objects.
- Normalize date values before sending them to the backend.
- Keep `duration` aligned with the configured `duration_unit`.
- If temporary ids are used, make id replacement explicit and deterministic.
- Persist links only when both endpoints refer to persisted task ids.
