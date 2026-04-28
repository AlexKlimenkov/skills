# Advanced Patterns

Use this file when implementing custom views, plugins, advanced filtering, undo/redo, or scheduling safeguards.

## Contents
- Views And Plugins As Optional Features
- Resource/Section Mapping
- Undo/Redo With State Management
- Conflict And Availability Guards
- Custom Fields And Backend Mapping

## Views And Plugins As Optional Features

When configuring non-trivial custom views:
- verify required plugin set and view configuration with MCP
- verify callback signatures and payload shapes with MCP

## Resource/Section Mapping

- keep one canonical event field for section/resource assignment (for example `resource_id`)
- ensure section keys and event assignment values use the same ID type/format
- map section labels from persisted data, not hardcoded display-only arrays
- if unassigned events are supported, keep their behavior explicit in UI and persistence
- keep row/section mapping deterministic across reloads and refetches

Guardrails:
- do not silently remap unknown section IDs
- verify timeline/section configuration details with MCP before implementation

## Undo/Redo With State Management

- use refs to access the latest state snapshot inside `data.save` callbacks — do not rely on render-time closures
- if using Redux or another external store for Scheduler history, normalize date values before writing to the store (state must be serializable)
- undo and redo actions must also update persistence — client-only history creates data gaps on reload
- persist history snapshots when rehydration across sessions is required
- save or refetch operations must not wipe the undo/redo history stack
- keep history refs and store state in sync after every mutation

## Conflict And Availability Guards

- run conflict/availability checks before persistence when the app requires scheduling rules
- typical checks: overlap on the same assignment field (for example `resource_id`), location scope, and business-hours window
- keep the check path explicit: `candidate event -> checks -> decision -> persist`
- if conflicts need confirmation, require explicit user confirmation before save
- if conflicts are blocked, stop the write and keep UI state consistent with persisted state

## Custom Fields And Backend Mapping

- treat custom fields (for example `location_id`, `resource_id`, `service_id`, `status`, `notes`) as app-level schema and keep them explicit in all CRUD paths
- map backend rows to Scheduler events explicitly (including `start_date` / `end_date` conversion)
- map Scheduler write payloads to backend payloads explicitly; do not persist raw callback objects
- keep server-assigned IDs and client temporary IDs synchronized with a deterministic replacement flow
- if the app has scope constraints (for example location-scoped resources/services), validate them before persistence
