# Design

## Context

See `proposal.md` for motivation and `specs/log-file-input/spec.md` for the observable contract. Today `index.html` stores a single `currentFileHandle`; **Open File** assigns that handle, `loadFileFromHandle()` reads it, and Reload/Auto-Reload assume the handle is reusable. The viewer has no drag/drop surface or empty-state interaction beyond editor placeholder text.

The implementation is a single-page browser application with Monaco already embedded. The change should remain inside the existing page and documentation, add no dependency, and preserve the current full-file `File.text()` loading model.

## Goals / Non-Goals

**Goals:**

- Normalize picker and drag/drop inputs before they reach rendering/filtering state.
- Make reload capability explicit instead of inferring that every opened file is reusable.
- Keep drag feedback independent from Monaco so the entire viewer can act as one stable drop target.
- Keep the existing toolbar workflow intact.

**Non-Goals:**

- Do not introduce a second renderer, parser, filter path, or drag-specific content pipeline.
- Do not add large-file streaming, directory traversal, multiple simultaneous files, persistence of file handles, or new browser dependencies.

## Decisions

### 1. Represent the current logfile as a normalized source

Replace the handle-only state with one conceptual current source that carries the displayed filename, a way to obtain/read the file contents, and whether that source supports rereading. The picker path produces a reusable handle-backed source. Drag/drop prefers a reusable filesystem handle when the browser exposes one and otherwise produces a one-shot `File` source.

This keeps parsing and rendering unaware of how the file entered the viewer. The alternative—maintaining separate `loadPickedFile` and `loadDroppedFile` paths—would duplicate state updates and make Reload/Auto-Reload behavior diverge.

### 2. Funnel source adoption through one load transition

Both entry paths should call one transition that reads the candidate source, makes it current on success, updates the filename and reload-control capability, and refreshes the existing views. Replacing a source must also reconcile any active auto-reload timer so a timer cannot keep acting on stale source state.

For a one-shot source, Auto-Reload is stopped and unchecked and both reload controls are disabled. For a reusable source, Reload is enabled and Auto-Reload can use the existing interval mechanism.

The alternative—letting event handlers mutate `currentFileHandle`, controls, and editor state independently—preserves the current coupling and makes replacement order error-prone.

### 3. Use viewer-level drag events with a presentation overlay

Register drag/drop handling at the viewer/document level rather than on Monaco editor DOM nodes. During a file drag, show a lightweight full-view overlay with a concise “Drop to open log” cue. Use drag-enter/leave bookkeeping so movement across nested elements does not flicker the overlay. Remove the cue on drop or when the drag leaves the viewer.

This avoids depending on Monaco's internal DOM and keeps the whole application as the target. The overlay is presentation only; the actual drop event still feeds the shared source transition.

### 4. Add a real empty state above the main editor

When no logfile is loaded, show a centered empty-state layer in the main view with the drop instruction and an **Open File** action. Hide it once a logfile has loaded. The toolbar **Open File** button remains permanently available.

This is preferable to encoding instructions as Monaco document text because the empty state is application UX, not log content, and can expose a real button without contaminating editor state.

### 5. Preserve permissive text-file compatibility

Do not add an extension or MIME-type allowlist. The current viewer reads the chosen file as text, so drag/drop should keep the same compatibility contract for a single file. Multi-file and directory behavior remain outside this change.

## Risks / Trade-offs

- [Browser drag APIs vary] → Prefer a filesystem handle when the drag item API provides one, then fall back to a normal `File`; capability state makes the difference visible instead of failing the drop.
- [Nested drag events can cause overlay flicker] → Track active drag-entry depth or equivalent state and clear it deterministically on leave/drop.
- [Auto-Reload can act on a replaced source] → Reconcile/clear the timer as part of the single source-adoption transition before enabling behavior for the new source.
- [Full-file reads still scale poorly for very large logs] → Keep the existing `File.text()` model unchanged; large-file optimization remains a separately documented future concern.
- [Overlay could interfere with editor interaction] → Keep it hidden outside active drag/empty states and use document-level drop handling rather than relying on overlay pointer interaction.

## Migration Plan

1. Add the normalized source state and route existing **Open File** loading through it without changing rendering behavior.
2. Add viewer-level drag/drop input and fallback source creation.
3. Add empty-state and drag-over presentation tied to current/drag state.
4. Update README behavior documentation and validate picker, reusable-drop, one-shot-drop, replacement, reload, auto-reload, and existing filtering/synchronization paths.

No data migration is required. Rollback is limited to reverting the page/documentation change and restoring the existing handle-only loading path.
