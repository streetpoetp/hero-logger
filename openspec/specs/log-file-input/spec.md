# Log File Input Specification

## Purpose

Defines the user-visible contract for opening, replacing, and reloading the current logfile through explicit selection or drag-and-drop while preserving clear capability feedback.

## Requirements

### Requirement: Users can open a logfile through either supported entry path
The viewer SHALL allow a user to make a single file the current logfile either by using **Open File** or by dropping that file onto the viewer, and both paths SHALL render the selected file through the same normal log-view behavior.

#### Scenario: Open a logfile with Open File
- **WHEN** the user chooses a file through the **Open File** action
- **THEN** that file becomes the current logfile and its contents are shown in the main log view

#### Scenario: Open a logfile by dropping it
- **WHEN** the user drops a single file onto the viewer
- **THEN** that file becomes the current logfile and its contents are shown in the main log view

#### Scenario: Replace the current logfile by dropping another file
- **GIVEN** a logfile is already open
- **WHEN** the user drops a different single file onto the viewer
- **THEN** the dropped file replaces the previous current logfile and the viewer refreshes to show the new contents

### Requirement: The viewer communicates drag-and-drop availability
The viewer SHALL provide an obvious drop affordance when no logfile is loaded and SHALL provide temporary visual feedback while a file is being dragged over the viewer.

#### Scenario: Empty viewer advertises file drop
- **GIVEN** no logfile is loaded
- **WHEN** the viewer is idle
- **THEN** the main viewing area indicates that a logfile can be dropped there and still offers **Open File**

#### Scenario: Dragging a file over the viewer shows drop feedback
- **WHEN** a file is dragged over the viewer
- **THEN** the viewer displays a clear drop-target cue over the viewing surface

#### Scenario: Drag feedback ends without a drop
- **GIVEN** the drop-target cue is visible
- **WHEN** the dragged file leaves the viewer without being dropped
- **THEN** the temporary drop-target cue is removed

### Requirement: Reload controls reflect the capabilities of the current file source
The viewer SHALL enable manual and automatic reload only when the current logfile source can be read again without another user selection, and SHALL disable unsupported reload behavior for one-shot file sources.

#### Scenario: Reusable file source supports reload
- **GIVEN** the current logfile was opened with a reusable filesystem handle
- **WHEN** the file is loaded successfully
- **THEN** **Reload** is available and **Auto-Reload** can be enabled

#### Scenario: One-shot dropped file does not pretend to support reload
- **GIVEN** a dropped file is available only as a one-shot browser file object
- **WHEN** that file becomes the current logfile
- **THEN** **Reload** and **Auto-Reload** are unavailable for that logfile

#### Scenario: Replacing a reloadable file with a one-shot file stops automatic reload
- **GIVEN** **Auto-Reload** is active for the current logfile
- **WHEN** a one-shot dropped file replaces it
- **THEN** automatic reload stops and reload controls become unavailable for the new current logfile

### Requirement: Existing viewer behavior remains consistent across file entry paths
After a logfile becomes current, the viewer SHALL apply the same filename display, parsing, filtering, formatting controls, and split-view synchronization regardless of whether the file was selected or dropped.

#### Scenario: Dropped file uses normal viewer features
- **WHEN** a dropped file becomes the current logfile
- **THEN** its displayed filename and all existing log viewing and filtering behavior work the same way as for a file opened with **Open File**
