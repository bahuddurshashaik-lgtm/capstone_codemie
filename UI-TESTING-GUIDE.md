# UI Testing Guide - Mark Task as Completed

## Test Case: EPMCDMETST-71468 - Mark Task as Completed

---

## 🚀 Setup

### 1. Start the Application
```bash
node src/server.js
```

### 2. Open Browser
Navigate to: **http://localhost:3000**

---

## ✅ TEST SCENARIO 1: Mark Open Task as Completed

### Initial State:
1. You should see the "Add a task" form at the top
2. Below that is the task list section

### Steps to Test:

#### Step 1: Create a New Task
1. **Enter Task Details:**
   - Title: "Buy groceries"
   - Description: "Milk, bread, eggs"
   - Priority: "High"
   - Due Date: Tomorrow's date (e.g., 2026-11-12)

2. **Click "Add task" button**

3. **Verify Task Created:**
   - ✅ Task appears at the top of the list
   - ✅ Shows "Status: Open"
   - ✅ Shows "Priority: High"
   - ✅ No completion timestamp visible
   - ✅ No checkmark badge
   - ✅ Title is NOT strikethrough

#### Step 2: Mark Task as Completed
1. **Locate the task** you just created ("Buy groceries")

2. **Look for three buttons on the task:**
   - "Edit" button
   - **"Mark complete" button** ⭐ (this is what we're testing)
   - "Delete" button

3. **Click the "Mark complete" button**

#### Step 3: Verify UI Changes
After clicking "Mark complete", verify these changes:

✅ **Visual Changes:**
- Task background color changes (light green tint)
- Checkmark badge (✓) appears next to the title
- Title text has **strikethrough** decoration
- Badge shows "✓ Completed"

✅ **Status Changes:**
- "Status: Completed" is displayed
- Completion timestamp appears: "Completed on Nov 11, 2026, 3:45 PM" (current time)

✅ **Button Changes:**
- "Mark complete" button changes to **"Reopen"** button

✅ **Success Message:**
- A green success message appears: "Task marked complete."

---

## ✅ TEST SCENARIO 2: Reopen Completed Task

### Steps:

#### Step 1: Click "Reopen" Button
1. Find the completed task (green background, checkmark)
2. Click the **"Reopen"** button

#### Step 2: Verify Changes Back to Open
After clicking "Reopen", verify:

✅ **Visual Changes:**
- Background color returns to normal (white/light)
- Checkmark badge disappears
- Strikethrough removed from title
- Task looks like a regular open task

✅ **Status Changes:**
- "Status: Open" is displayed
- Completion timestamp **disappears** (no longer shown)

✅ **Button Changes:**
- "Reopen" button changes back to **"Mark complete"**

✅ **Success Message:**
- Message appears: "Task reopened."

---

## ✅ TEST SCENARIO 3: Verify Persistence (AC2)

This tests that the completion status survives page refresh and server restart.

### Test 3A: Page Refresh

#### Steps:
1. Mark a task as Completed (follow Scenario 1)
2. **Press F5** or click browser refresh button
3. Wait for page to reload

#### Verify:
✅ Task is still marked as Completed:
- Green background still there
- Checkmark badge still visible
- Completion timestamp still shows same time
- Button still shows "Reopen"

### Test 3B: Server Restart

#### Steps:
1. Mark a task as Completed
2. Note the completion timestamp (e.g., "Nov 11, 2026, 3:45 PM")
3. **Stop the server:**
   - Go to terminal
   - Press `Ctrl+C` to stop server
4. **Restart the server:**
   ```bash
   node src/server.js
   ```
5. **Refresh browser** at http://localhost:3000

#### Verify:
✅ Task is still marked as Completed:
- All visual indicators remain (green, checkmark, strikethrough)
- **Completion timestamp is EXACTLY the same** (proves it was saved)
- Status: Completed
- Button shows "Reopen"

---

## ✅ TEST SCENARIO 4: Toggle Multiple Times

This tests that you can complete and reopen the same task multiple times.

#### Steps:
1. Create a task (Status: Open)
2. Click "Mark complete" → Task becomes Completed, note timestamp (e.g., 3:45 PM)
3. Click "Reopen" → Task becomes Open, timestamp disappears
4. Wait 2 minutes
5. Click "Mark complete" again → Task becomes Completed with NEW timestamp (e.g., 3:47 PM)

#### Verify:
✅ **First completion:** Timestamp shows 3:45 PM
✅ **After reopen:** No timestamp
✅ **Second completion:** NEW timestamp shows 3:47 PM (different from first time)

This proves the timestamp updates each time you complete the task.

---

## ✅ TEST SCENARIO 5: Filter Completed Tasks

#### Steps:
1. Create 3 tasks
2. Mark 1 task as Completed
3. Leave 2 tasks as Open
4. Go to **Filters** section
5. Select "Status" dropdown → Choose **"Completed"**

#### Verify:
✅ Only the completed task is shown
✅ Count shows "Showing 1 of 3 tasks"
✅ Task has green background and checkmark

#### Step 2: Filter Open Tasks
1. Change "Status" dropdown to **"Open"**

#### Verify:
✅ Only the 2 open tasks are shown
✅ Count shows "Showing 2 of 3 tasks"
✅ No green backgrounds or checkmarks

---

## 🎨 VISUAL INDICATORS REFERENCE

### Open Task Appearance:
```
┌─────────────────────────────────────────────┐
│ [Normal Title - No Strikethrough]          │
│                                             │
│ Optional description text...                │
│                                             │
│ Priority: High | Due 2026-11-12 | Status: Open │
│                                             │
│ [Edit] [Mark complete] [Delete]            │
└─────────────────────────────────────────────┘
```

### Completed Task Appearance:
```
┌─────────────────────────────────────────────┐ Green background
│ S̶t̶r̶i̶k̶e̶t̶h̶r̶o̶u̶g̶h̶ ̶T̶i̶t̶l̶e̶  ✓ Completed          │
│                                             │
│ Optional description text...                │
│                                             │
│ Priority: High | Due 2026-11-12 | Status: Completed │
│ Completed on Nov 11, 2026, 3:45 PM         │ ← Timestamp
│                                             │
│ [Edit] [Reopen] [Delete]                   │ ← Button changed
└─────────────────────────────────────────────┘
```

---

## 🐛 EDGE CASES TO TEST

### Edge Case 1: Rapid Clicking
1. Click "Mark complete" button
2. Immediately click it again before page updates

**Expected:** No error, task stays Completed (idempotent behavior)

### Edge Case 2: Completing Multiple Tasks
1. Create 5 tasks
2. Mark all 5 as Completed one by one

**Expected:** Each gets its own completion timestamp

### Edge Case 3: Edit Completed Task
1. Mark task as Completed
2. Click "Edit" button
3. Change the title
4. Click "Save changes"

**Expected:** 
- Task stays Completed (status doesn't change)
- Completion timestamp stays the same
- Only edited fields update

---

## 📊 WHAT TO LOOK FOR (ACCEPTANCE CRITERIA)

### AC1: Visual Distinction ✅
- [ ] Completed tasks have different background color (green tint)
- [ ] Checkmark badge (✓) visible
- [ ] Title has strikethrough decoration
- [ ] Status label shows "Completed"

### AC2: Persistence ✅
- [ ] Status survives page refresh (F5)
- [ ] Status survives server restart
- [ ] Completion timestamp preserved exactly

### AC3: Timestamp Rules ✅
- [ ] Completed task shows timestamp in format: "Completed on [date], [time]"
- [ ] Open task has NO timestamp
- [ ] Reopening removes timestamp
- [ ] Completing again sets NEW timestamp

---

## 🧪 AUTOMATED TEST VERIFICATION

After manual testing, run automated tests to confirm:

```bash
npm test
```

**Look for these specific tests:**
- ✅ `completing an Open task sets status Completed and completedAt from the clock`
- ✅ `reopening a Completed task clears completedAt`
- ✅ `the new status is still there after a new server starts on the same file`

**All 480 tests should pass.**

---

## 🚨 COMMON ISSUES TO CHECK

### Issue 1: Button not responding
**Check:** Console for JavaScript errors (F12 → Console tab)

### Issue 2: Timestamp not showing
**Check:** Task status is actually "Completed" and completedAt field exists

### Issue 3: Changes don't persist
**Check:** 
- Server is writing to `data/tasks.json`
- File permissions are correct
- No errors in server console

### Issue 4: Visual styling missing
**Check:** `styles.css` is loading (Network tab in F12)

---

## ✅ TESTING CHECKLIST

Use this checklist when testing:

- [ ] Created a new task successfully
- [ ] Clicked "Mark complete" button
- [ ] Task shows green background
- [ ] Checkmark badge appears
- [ ] Title has strikethrough
- [ ] Completion timestamp displays
- [ ] Button changed to "Reopen"
- [ ] Success message appeared
- [ ] Clicked "Reopen" button
- [ ] All visual changes reverted
- [ ] Timestamp disappeared
- [ ] Button changed back to "Mark complete"
- [ ] Refreshed page (F5) - status persisted
- [ ] Restarted server - status persisted
- [ ] Filtered by "Completed" status - works
- [ ] Completed multiple tasks - each has timestamp
- [ ] Edited completed task - stays completed

---

## 📸 SCREENSHOT COMPARISON

### Before (Open Task):
- White/light background
- No badge
- Normal text (no strikethrough)
- Button: "Mark complete"
- No timestamp

### After (Completed Task):
- Green background
- "✓ Completed" badge
- Strikethrough text
- Button: "Reopen"
- Shows: "Completed on Nov 11, 2026, 3:45 PM"

---

## 🎯 SUCCESS CRITERIA

**Test is SUCCESSFUL if:**
1. ✅ All visual changes appear correctly
2. ✅ Completion timestamp shows current date/time
3. ✅ Reopening reverses all changes
4. ✅ Status persists after refresh and restart
5. ✅ No console errors
6. ✅ All automated tests pass (480/480)

**Test is FAILED if:**
- ❌ Button doesn't respond
- ❌ Visual changes don't appear
- ❌ Timestamp is missing or wrong format
- ❌ Status doesn't persist after refresh
- ❌ JavaScript errors in console
- ❌ Automated tests fail
