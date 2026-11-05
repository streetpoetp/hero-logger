# Tasks

## 1. Unify logfile source state

- [x] 1.1 Refactor `index.html` from handle-only `currentFileHandle` state to one normalized current-log source used by loading and reloading, and verify an existing logfile opened through **Open File** still displays, reloads, filters, and synchronizes between panes as before.
- [x] 1.2 Centralize source adoption and reload-capability updates so replacing a source cannot leave a stale Auto-Reload interval; verify reusable sources enable Reload/Auto-Reload while a one-shot source disables both and stops/unchecks active Auto-Reload.

## 2. Add drag-and-drop file entry UX

- [x] 2.1 Add viewer-level file drag/drop handling that prefers a reusable filesystem handle when available and falls back to a one-shot browser `File`, then verify dropping one file loads it and dropping a second file replaces the current logfile through the shared source path.
- [x] 2.2 Add the no-file empty state with both the drop instruction and an **Open File** action, and verify it is visible only before a logfile is loaded and the toolbar **Open File** action remains permanently available.
- [x] 2.3 Add the temporary full-view drag-over cue with stable enter/leave bookkeeping, and verify it appears during a file drag, does not flicker when crossing nested viewer elements, and disappears on leave or drop.
- [x] 2.4 Update `README.md` to document both file-entry paths and the reusable-handle versus one-shot reload behavior, and verify the documented behavior matches the implemented controls.

## 3. Integration validation

- [x] 3.1 Run a fresh browser acceptance pass covering **Open File**, drag-to-open, drag-to-replace, reusable-source Reload, Auto-Reload, one-shot fallback capability disabling, empty/drag states, search/filtering, formatting toggles, and split-view synchronization; record/fix any regressions within this change before completion.
- [x] 3.2 Run strict OpenSpec validation for `support-drag-and-open-log-files` and inspect the final scoped diff to verify only the planned viewer/documentation behavior and change artifacts are included.
