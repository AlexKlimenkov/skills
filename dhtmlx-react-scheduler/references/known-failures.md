# Known Failures

Use this file when debugging incorrect or flaky DHTMLX React Scheduler integrations.

## 1. Theme Drift

Problem:
- app shell and Scheduler follow different theme sources, or Scheduler skin is switched imperatively via `ref.instance.setSkin()` while the `theme` prop is also wired

Fix:
- use one shared app theme source
- drive Scheduler skin from that source via the `theme` prop (not `setSkin()`)
- do not introduce separate Scheduler-only theme state

## 2. Date Serialization Bugs

Problem:
- write handlers receive mixed date shapes (`Date` and/or string)
- backend writes become inconsistent or invalid

Fix:
- treat `start_date` and `end_date` as `Date | string`
- normalize before persistence
- map backend date strings back to `Date` before passing events to Scheduler

## 3. Stale Callback State

Problem:
- `data.save` handlers capture old event arrays or old store snapshots

Fix:
- use refs, functional updates, or current-store snapshots
- do not build persistence logic from stale render-time closures

## 4. ID Replacement Gaps

Problem:
- create flow keeps temporary client IDs after backend persistence
- later update/delete targets the wrong event

Fix:
- make temp-id -> persisted-id replacement deterministic
- synchronize replacement in both local state and Scheduler instance data

## 5. Mixed Ownership Bugs

Problem:
- React props and imperative Scheduler instance mutations both modify events without synchronization

Fix:
- choose ownership model first
- if imperative mutations are required, keep React-managed event props synchronized deliberately

## 6. Wrong Template Or Handler Signature

Problem:
- callback wired with wrong parameter order or wrong payload assumptions

Fix:
- verify exact documented signature with MCP
- do not infer callback arguments from memory

## 7. Implicit Height Containers

Problem:
- Scheduler is mounted into a container without real height
- rendering is broken or collapsed

Fix:
- provide explicit container height

## 8. Plugin Not Activated

Problem:
- a feature that requires a plugin (recurring lightbox, timeline view, units view, collision, tooltip, quick_info, ...) is used without enabling it
- symptoms: feature is silently inert, lightbox stays non-recurring, view fails to render, prop or config is ignored

Fix:
- enable the plugin through the `plugins` prop: `plugins={{ recurring: true, collision: true }}`
- the core `scheduler.plugins({...})` instance method also works, but the prop form is preferred from the React wrapper
- check plugin dependencies (e.g. `treetimeline` requires `timeline`) with MCP

## 9. Recurring Series Mishandling

Problem:
- backend persists a row for every expanded occurrence of a recurring series
- modified or deleted occurrences are written without `recurring_event_id` / `original_start`
- result: duplicate rows, broken series edits, occurrences resurrect after reload

Fix:
- persist exactly one row per series (with `rrule`) plus one row per modified or deleted occurrence (`rrule: null`, `recurring_event_id` set, `original_start` set, `deleted: true` if removed)
- route edits through `modals.onRecurrenceConfirm` and translate the returned decision (`"occurrence" | "following" | "series"`) into the right persistence shape
- treat `duration` as seconds

## 10. Mixed Lightbox Paths

Problem:
- `customLightbox` and `config.lightbox.sections` are both configured on the same screen
- edit behavior becomes ambiguous, modal opening duplicates or interleaves

Fix:
- pick one editor path per screen
- if `customLightbox` is set, remove `lightbox.sections` configuration (and vice versa)
