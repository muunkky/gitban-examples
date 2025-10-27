# App Crashes When Clicking Delete Button

## Description

The application crashes with "Cannot read property 'id' of undefined" when users click the delete button on todo items.

**Type**: Bug
**Severity**: High (blocks core functionality)
**Impact**: All users
**Discovered**: 2025-10-27

## Steps to Reproduce

1. Navigate to `/todos` page
2. Create a new todo item
3. Click the delete button (trash icon) next to the todo
4. **Expected**: Todo is deleted and removed from list
5. **Actual**: Browser console shows error and app crashes

## Error Message

```
TypeError: Cannot read property 'id' of undefined
    at deleteTodo (TodoList.tsx:45)
    at onClick (Button.tsx:12)
```

## Environment

- **Browser**: Chrome 120, Firefox 121 (reproduced on both)
- **OS**: Windows 11, macOS Sonoma
- **App Version**: v1.2.3
- **Reproducibility**: 100% (happens every time)

## Root Cause

The `deleteTodo` function expects `todo.id` but receives `undefined` when the delete button is clicked. Investigation shows the `onClick` handler passes the event object instead of the todo item.

**Problematic Code** (src/components/TodoList.tsx:45):
```tsx
const handleDelete = (todo) => {
  deleteTodo(todo.id);  // 'todo' is actually the click event
};

// Button in render:
<Button onClick={handleDelete} />  // Missing todo argument
```

## Proposed Fix

Update the onClick handler to properly pass the todo item:

```tsx
<Button onClick={() => handleDelete(todo)} />
```

## Testing Plan

- [ ] Manual test: Delete todo items in browser
- [ ] Unit test: TodoList component delete handler
- [ ] Regression test: Verify other buttons still work
- [ ] E2E test: Full delete workflow

## Related Issues

- Similar pattern in EditTodoModal (check for same bug)
