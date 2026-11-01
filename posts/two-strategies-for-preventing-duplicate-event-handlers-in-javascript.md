---
title: "Two strategies for preventing duplicate event handlers in JavaScript"
date: 2024-12-02T20:18:27Z
updated: 2025-09-24T22:12:59Z
slug: two-strategies-for-preventing-duplicate-event-handlers-in-javascript
published: true
---

# Two strategies for preventing duplicate event handlers in JavaScript

It turns out that (as of November, 2024), there are three available methods for managing events in JavaScript:

1. `addEventListener()`
2. `dispatchEvent()`
3. `removeEventListener()`

There isn't a built-in way to check if a particular event listener has already been added. Here are two ways to keep them in check:

## 1. Use DOM events instead of addEventListener

Instead of:

```javascript
function attach_drag_handlers(target) {
  target.addEventListener("dragstart", dragStartHandler)
  target.addEventListener("dragenter", dragEnterHandler)
  target.addEventListener("dragover", dragOverHandler)
  target.addEventListener("dragleave", dragLeaveHandler)
  target.addEventListener("drop", dropHandler)
  target.addEventListener("dragend", dragEndHandler)
}
```

Use:

```javascript
function attach_drag_handlers(target) {
  target.ondragstart = dragStartHandler
  target.ondragenter = dragEnterHandler
  target.ondragover = dragOverHandler
  target.ondragleave = dragLeaveHandler
  target.ondragend = dragEndHandler
}
```

Assignment is atomic, while `addEventListener` just adds to a list with no option to de-dupe.

If you must `addEventListener()` then, try:

## 2. Use a flag to keep track of whether listeners are attached

```javascript
if (document.has_checkbox_handler) {
  // Do nothing
} else {
  document.addEventListener("task_checkbox_clicked", checkbox_clicked_handler)
  document.has_checkbox_handler = true
}
```
