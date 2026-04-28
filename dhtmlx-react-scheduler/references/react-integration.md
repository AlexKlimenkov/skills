# React Integration

Use this file when setting up or modifying the React wrapper integration.

## Installation

Trial:
```bash
npm install @dhtmlx/trial-react-scheduler
```

Commercial (requires private registry setup and login):
```bash
npm config set @dhx:registry=https://npm.dhtmlx.com
npm login --registry=https://npm.dhtmlx.com --scope=@dhx
npm install @dhx/react-scheduler
```
The user must generate credentials in the [Client's Area](https://dhtmlx.com/clients/) and run the registry config and login commands themselves before the package can be installed. See [installation docs](https://docs.dhtmlx.com/scheduler/integrations/react/installation/) for details.

CSS import must match the installed package and must be a separate import line:

```ts
import "@dhtmlx/trial-react-scheduler/dist/react-scheduler.css";
```

or

```ts
import "@dhx/react-scheduler/dist/react-scheduler.css";
```

## Next.js

In Next.js (App Router), add `"use client"` at the top of the Scheduler component file. Next.js uses Server Components by default, but React Scheduler must render as a Client Component.

## Height Rule

The Scheduler container must have explicit height.

Valid:
```tsx
<div style={{ height: "600px" }}>
  <ReactScheduler ... />
</div>
```

## Documented Props

Use documented props only:

- `events`
- `view`
- `date`
- `config`
- `templates`
- `xy`
- `data`
- `customLightbox`
- `modals`
- `filter`
- `on<EventName>`
- `ref`

## Ref Access

Use `ref` with `ReactSchedulerRef` when direct instance access is required:

```tsx
const schedulerRef = useRef<ReactSchedulerRef>(null);
const scheduler = schedulerRef.current?.instance;
```

If you mutate data through the instance while also passing `events` as props, keep them synchronized. Otherwise React can overwrite those changes on re-render.

## Theme Rule

Use the app theme as the single source of truth.

Example:
```tsx
useEffect(() => {
  const scheduler = schedulerRef.current?.instance;
  if (!scheduler) return;
  scheduler.setSkin(appTheme === "dark" ? "dark" : "terrace");
}, [appTheme]);
```

Do not introduce separate Scheduler-only theme state if the app already has a global theme source.

## Config And Templates

Wrap `config` and `templates` in `useMemo` when they contain functions.

Typical:
```tsx
const config: SchedulerConfig = useMemo(() => ({ ... }), [deps]);
const templates: SchedulerTemplates = useMemo(() => ({ ... }), [deps]);
```

Common documented config areas:
- `config.first_hour`
- `config.last_hour`
- `config.hour_size_px`
- `config.time_step`
- `config.readonly`
- `config.drag_move`
- `config.drag_resize`
- `config.drag_create`
- `config.header`

Do not guess template callback signatures. Verify with MCP if there is any doubt.

## Event Handlers

When wiring `on<EventName>` callbacks:
- verify exact callback signatures with MCP
- keep handlers side-effect-safe and avoid stale closure state
- route writes through one persistence path (`data.save` or controlled app action), not both at once
