# Known Failures

Use this file when debugging incorrect or flaky DHTMLX React Scheduler integrations.

## 1. Theme Drift

Problem:
- app shell and Scheduler follow different theme sources

Fix:
- use one shared app theme source
- apply Scheduler theme/skin from that source
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
