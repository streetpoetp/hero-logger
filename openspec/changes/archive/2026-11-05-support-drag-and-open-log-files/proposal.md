# Proposal

## Why

Hero.LogViewer currently requires the toolbar file picker even though the viewer itself is the natural target for a logfile. Supporting drag-and-drop alongside **Open File** makes repeated debugging faster while keeping the existing explicit file-selection workflow available.

## What Changes

- Add drag-and-drop as a first-class way to open a logfile, with the whole viewer acting as the drop target.
- Show a clear empty-state drop affordance before any logfile is loaded and a temporary full-view drag-over cue while a file is being dragged over the viewer.
- Route both **Open File** and drag/drop through one shared current-log-source/load path so replacing the current logfile has consistent state and rendering behavior.
- Preserve **Reload** and **Auto-Reload** when the selected source exposes a reusable filesystem handle.
- Allow one-shot browser `File` drops to load normally while disabling reload capabilities that cannot be supported by that source.
- Document the dual file-entry behavior and fallback contract.
- Keep existing filtering, rendering, split-view synchronization, and log-format behavior unchanged.

Intentionally out of scope: large-file streaming/chunking, multi-file sessions, directory drops, log history, parser redesign, or a separate drag-specific viewer path.

## Capabilities

### New Capabilities

- `log-file-input`: Defines how users open, replace, and reload the current logfile through explicit file selection or drag-and-drop, including capability fallback for one-shot file sources.

### Modified Capabilities

None.

## Impact

- Primary code ownership: `index.html`, specifically current file-source state, load/reload actions, and viewer-level input UX.
- Documentation: `README.md` to record the user-visible input and reload behavior.
- Dependencies: no new runtime dependency; existing browser File System Access support remains optional.
- Architecture/ownership: no module or ownership boundary change; the implementation should centralize the existing file-loading behavior rather than add parallel infrastructure.
- Public API/protocol: none.
- Compatibility: existing **Open File** behavior remains supported; browsers that cannot provide a reusable filesystem handle may still open a dropped `File` but must not expose unsupported reload behavior.
- Repository/project structure: no production structure change.
- Unresolved decisions: none within the approved scope.
