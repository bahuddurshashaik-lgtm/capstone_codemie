# EPMCDMETST-71468 Code Changes Summary

## Jira Story: Mark Task as Completed and Persist Completion Status

This document details all code changes made to implement EPMCDMETST-71468.

---

## Implementation Commits

The feature was implemented across 4 commits:

1. **T-5** (1d928c4) - Domain validation for status
2. **T-11** (653320a) - Service layer setStatus method
3. **T-20** (8c55e30) - HTTP API route for status updates
4. **T-29** (04342b9) - UI toggle button and handlers

---

## 1. Domain Validation Layer (T-5: 1d928c4)

### File: `src/domain/validation.js`

**Lines Added: ~62**

```javascript
// Added status validation constants
export const STATUSES = Object.freeze(['Open', 'Completed']);

// Added validation messages
status: 'Choose Open or Completed.',

// Added parseStatus function
export function parseStatus(value) {
  if (!STATUSES.includes(value)) return invalid(MESSAGES.status);
  return valid(value);
}

// Added validateStatus function
export function validateStatus(input) {
  requirePlainObject(input);
  const errors = newErrorMap();
  flagUnknownKeys(input, ['status'], errors);
  const result = parseStatus(readField(input, 'status'));
  if ('error' in result) errors.status = result.error;
  throwIfAny(errors);
  return result.value;
}
```

**Purpose:** 
- Validates status input (must be 'Open' or 'Completed')
- Rejects invalid status values
- Ensures only 'status' field is accepted in request body

---

## 2. Service Layer (T-11: 653320a)

### File: `src/services/task-service.js`

**Lines Added: ~5**

```javascript
function setStatus(id, input) {
  return changeTask(id, (existing) => applyStatus(existing, validateStatus(input), clock));
}

// Added to return statement
return { createTask, listTasks, getTask, updateTask, setStatus, deleteTask };
```

**Purpose:**
- Orchestrates status change operation
- Validates input using `validateStatus()`
- Applies status change using domain `applyStatus()` function
- Uses `changeTask()` helper for atomic persistence
- Injects clock for timestamp generation

**Key Behavior:**
- Uses `TaskStore.mutate()` for atomic writes
- Sets `completedAt` timestamp when status becomes 'Completed'
- Clears `completedAt` when status becomes 'Open'

---

## 3. HTTP API Layer (T-20: 8c55e30)

### File: `src/http/app.js`

**Lines Added: ~3**

```javascript
// Added handler
const setStatus = async ({ params, request }) =>
  ok(await service.setStatus(params.id, await readJsonBody(request)));

// Added route
{ segments: ['api', 'tasks', ':id', 'status'], methods: { PUT: setStatus } },
```

**API Endpoint:** `PUT /api/tasks/:id/status`

**Request:**
```json
{
  "status": "Completed"  // or "Open"
}
```

**Response (200):**
```json
{
  "id": "task-id",
  "title": "Task title",
  "description": "...",
  "priority": "Medium",
  "dueDate": "2026-11-10",
  "status": "Completed",
  "createdAt": "2026-11-05T12:00:00.000Z",
  "completedAt": "2026-11-10T15:30:00.000Z",
  "overdue": false
}
```

**Error Responses:**
- `400` - Invalid status value or unknown field
- `404` - Task not found
- `500` - Storage write failure

---

## 4. UI Layer (T-29: 04342b9)

### File: `src/public/app.js`

**Lines Added: ~25**

```javascript
// Added focus helper
function focusListHeadingIfFocusLost() {
  const active = document.activeElement;
  if (!active || active === document.body) ui.listHeading.focus();
}

// Added toggle handler
async function toggleTask(id) {
  const task = state.tasksById.get(id);
  if (!task) return;
  const result = await api.setStatus(id, toggleAction(task).nextStatus);
  if (!result.ok) {
    await handleTaskError(result.error, id);
    return;
  }
  showMessage(result.data.status === 'Completed' ? 'Task marked complete.' : 'Task reopened.');
  await refreshList();
  focusListHeadingIfFocusLost();
}

// Added to action handlers
const TASK_ACTIONS = Object.freeze({
  edit: startEdit,
  toggle: toggleTask,  // NEW
  delete: askDelete,
});
```

**UI Behavior:**
- Clicking "Mark complete" button calls `toggleTask()`
- Sends opposite status to API (toggle behavior)
- Shows success message ("Task marked complete." or "Task reopened.")
- Refreshes task list to show updated state
- Manages focus for accessibility

**Visual Changes:**
- Completed tasks show checkmark badge (✓)
- Completed tasks have different background color
- Title text has strikethrough
- Button label changes: "Mark complete" ↔ "Reopen"
- Completion timestamp displayed below task details

---

## Supporting Code (Already Existed)

These components were already in place and did not need changes:

### `src/domain/task.js` - applyStatus() function
```javascript
export function applyStatus(task, status, clock = systemClock) {
  if (task.status === status) return task;
  const completedAt = status === STATUS_COMPLETED ? clock().toISOString() : null;
  return { ...task, status, completedAt };
}
```
- Sets `completedAt` to ISO timestamp when Completed
- Sets `completedAt` to `null` when Open
- Returns unchanged task if status is the same

### `src/public/view-model.js` - toggleAction() function
```javascript
export function toggleAction(task) {
  return task.status === COMPLETED
    ? { label: 'Reopen', nextStatus: OPEN }
    : { label: 'Mark complete', nextStatus: COMPLETED };
}
```
- Determines button label based on current status
- Returns opposite status for toggle behavior

### `src/public/api-client.js` - setStatus() method
```javascript
setStatus: (id, status) => send('PUT', `${taskPath(id)}/status`, { status }),
```
- API client method for status updates

### `src/storage/task-store.js` - mutate() method
- Already handles atomic persistence
- No changes needed

---

## Test Files Added/Modified

### `tests/integration/api-status.test.js`
**Lines Added: ~238**
- Tests for `PUT /api/tasks/:id/status` endpoint
- Completion and reopening scenarios
- Persistence after server restart
- Validation errors
- Concurrency handling

### `tests/unit/validation.test.js`
**Lines Added: ~30**
- Tests for `validateStatus()` function
- Tests for status parsing

---

## Summary of Changes

| Layer | File | Lines Added | Purpose |
|-------|------|-------------|---------|
| Domain | `src/domain/validation.js` | ~62 | Status validation |
| Service | `src/services/task-service.js` | ~5 | setStatus orchestration |
| HTTP | `src/http/app.js` | ~3 | API route |
| UI | `src/public/app.js` | ~25 | Toggle button handler |
| Tests | `tests/integration/api-status.test.js` | ~238 | Integration tests |
| Tests | `tests/unit/validation.test.js` | ~30 | Validation tests |

**Total Lines Added: ~363**

---

## Acceptance Criteria Coverage

### AC1 - Mark as Completed ✅
**Code:** UI toggle button (`src/public/app.js:387-407`)
- Button shows "Mark complete" for Open tasks
- Clicking updates status to Completed
- Visual distinction via CSS classes

### AC2 - Persistence ✅
**Code:** Service layer uses `TaskStore.mutate()` (`src/services/task-service.js:64-66`)
- Atomic write to JSON file
- Survives page refresh and server restart
- Test: `tests/integration/api-status.test.js:191-211`

### AC3 - Timestamp Rules ✅
**Code:** Domain logic `applyStatus()` (`src/domain/task.js:28-32`)
- Completed → `completedAt = clock().toISOString()`
- Open → `completedAt = null`
- Tests: `tests/unit/task.test.js:87-120`

---

## No Changes Required

These files were reviewed but did NOT need changes:
- ✅ `src/domain/task.js` - applyStatus already existed
- ✅ `src/storage/task-store.js` - persistence already worked
- ✅ `src/public/view-model.js` - toggleAction already existed
- ✅ `src/public/api-client.js` - only needed setStatus method (already there)
- ✅ `src/public/styles.css` - styling already existed
- ✅ `src/public/index.html` - no markup changes needed
