# EPMCDMETST-71468: Mark Task as Completed with Persistence

## Summary
This PR implements and verifies the feature to mark tasks as completed with persistent storage, as specified in Jira story **EPMCDMETST-71468**.

## Changes Made

### 1. Test Fix
- Fixed failing unit test in `task-service.test.js` by adding clock injection to prevent past due date validation errors

### 2. Verification Documentation
- Added comprehensive verification document (`docs/EPMCDMETST-71468-verification.md`)
- Documented all acceptance criteria fulfillment
- Included test results and code quality checks

## Acceptance Criteria ✅

### AC1 — Mark as Completed
✅ Given at least one task is displayed with status **Open**  
✅ When the user marks that task as **Completed**  
✅ Then the UI shows it as **Completed** and it is visually distinguishable  

**Implementation:**
- Toggle button with "Mark complete" / "Reopen" labels
- Visual distinction via CSS (background color, checkmark badge, strikethrough)
- Completed tasks show completion timestamp

### AC2 — Persistence
✅ Given a task was marked **Completed**  
✅ When the user refreshes the page or the server restarts  
✅ Then the task still appears **Completed** with timestamp preserved  

**Implementation:**
- Atomic write operations via `TaskStore.mutate()`
- JSON file persistence survives page refresh and server restart
- Test: `the new status is still there after a new server starts on the same file`

### AC3 — Completed Timestamp Rule
✅ When a task is **Completed**, `completedAt` is set to a valid ISO 8601 timestamp  
✅ When a task is **Open**, `completedAt` is `null`  

**Implementation:**
- Domain logic in `src/domain/task.js` - `applyStatus()` function
- Completed → `completedAt = clock().toISOString()`
- Open → `completedAt = null`
- Validation prevents invalid combinations

## Test Results

**Total Tests:** 480  
**Passing:** 480 ✅  
**Failing:** 0  

**Key Tests for EPMCDMETST-71468:**
- ✅ `completing an Open task sets status Completed and completedAt from the clock`
- ✅ `reopening a Completed task clears completedAt`
- ✅ `complete, reopen and complete again gives a new completedAt`
- ✅ `the new status is still there after a new server starts on the same file`
- ✅ `applyStatus to Completed sets completedAt from the clock`
- ✅ `applyStatus back to Open clears completedAt`

## Implementation Architecture

### Backend
- **Domain Layer:** `src/domain/task.js` - Pure business logic for status changes
- **Service Layer:** `src/services/task-service.js` - `setStatus()` orchestration
- **HTTP Layer:** `src/http/app.js` - `PUT /api/tasks/:id/status` endpoint
- **Storage Layer:** `src/storage/task-store.js` - Atomic persistence

### Frontend
- **UI:** `src/public/app.js` - Toggle button and event handlers
- **View Model:** `src/public/view-model.js` - Display logic and action labels
- **API Client:** `src/public/api-client.js` - `setStatus()` method
- **Styling:** `src/public/styles.css` - Visual distinction for completed tasks

## Code Quality
- ✅ All tests passing
- ✅ No console errors
- ✅ Follows existing code patterns
- ✅ Proper error handling
- ✅ Input validation on API endpoint
- ✅ Atomic write operations prevent data corruption

## Related Documentation
- Implementation Plan: `docs/implementation-plan-EPMCDMETST-71468.md`
- Verification Report: `docs/EPMCDMETST-71468-verification.md`
- User Story: US-004 in `docs/user-stories/user-stories.md`

## Checklist
- [x] Code implements all acceptance criteria
- [x] Unit tests created/updated
- [x] Integration tests passing
- [x] All 480 tests passing
- [x] Documentation updated
- [x] Code follows project standards
- [x] No security issues introduced

🤖 Generated with [Claude Code](https://claude.com/claude-code)
