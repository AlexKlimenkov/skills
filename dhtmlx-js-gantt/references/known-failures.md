# Known Failures

Use this file when debugging incorrect or flaky DHTMLX JavaScript Gantt integrations.

## 1. Missing Height

Problem:
- Gantt is mounted into a container without real height
- grid or timeline appears blank, collapsed, or partially rendered

Fix:
- give the Gantt container explicit height
- verify parent containers also provide height

## 2. Missing CSS Import

Problem:
- Gantt logic runs, but UI is broken or unstyled

Fix:
- import the CSS that matches the installed package

## 3. Duplicate Initialization

Problem:
- Gantt is initialized multiple times on the same container without cleanup
- duplicate events or DataProcessor handlers fire

Fix:
- initialize Gantt in a single lifecycle boundary
- destroy or clear the previous instance before re-initializing

## 4. Wrong Data Shape

Problem:
- Gantt receives raw backend rows or mismatched field names
- hierarchy, dates, or links do not render correctly

Fix:
- map backend rows explicitly to Gantt task/link fields
- use `id`, `text`, `start_date`, `duration`, `parent`, `source`, `target`, and `type` consistently

## 5. Date Serialization Bugs

Problem:
- persistence assumes every date is a `Date`
- date values are sent in inconsistent formats

Fix:
- normalize dates before persistence
- handle both `Date` and string inputs

## 6. Temporary ID Bugs

Problem:
- backend assigns a real id, but Gantt keeps a temporary id
- links are created or persisted using temporary task ids

Fix:
- return `{ id }` or `{ tid }` after create so Gantt replaces the temporary id
- persist links only after both endpoints have real ids
- use the documented id-change method if automatic replacement does not occur

## 7. Stale State In Persistence

Problem:
- persistence handlers use outdated task or link data
- updates overwrite newer changes

Fix:
- always read the latest data when building persistence payloads
- avoid using stale snapshots captured at initialization

## 8. Sortorder Drift

Problem:
- task updates overwrite row order
- reload restores a different order

Fix:
- treat reorder as a dedicated flow
- persist `sortorder`
- load tasks ordered by `sortorder`

## 9. Private Field Persistence

Problem:
- backend payloads or exports include Gantt runtime fields (`$...`, `_...`)

Fix:
- build payloads from normalized data models
- exclude private and runtime fields explicitly

## 10. Template Signature Guessing

Problem:
- a template callback uses guessed parameter order or return type

Fix:
- verify template signatures with DHTMLX MCP or official docs before implementing

## 11. Resource Field Drift

Problem:
- different fields or structures are used for resource binding across UI, Gantt config, and persistence

Fix:
- use one consistent resource field or assignment structure across the feature

## 12. Calendar Drift

Problem:
- Gantt and the application use different working time rules
- durations and non-working time are calculated inconsistently

Fix:
- keep working time rules in one source of truth
- apply them to Gantt configuration
- call `gantt.render()` after changes
