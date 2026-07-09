# Klogg-like NLog Web Log Viewer Design Document

## 1. Overview

This document describes the design of a browser-based log viewer
inspired by klogg, optimized for Unity development and NLog formatted
logs.

Goals:

-   Fast log inspection workflow
-   NLog-aware structured display
-   Synchronized split-view for context preservation
-   Better readability than traditional text viewers
-   Support wrapped long messages and stack traces
-   Dark developer-oriented UI

------------------------------------------------------------------------

# 2. Design Goals

## Primary Goals

-   Support common NLog layouts
-   Display logs in vertical structured format
-   Provide a synchronized dual-pane view (Main vs Filtered)
-   Provide readable Unity runtime debugging experience
-   Support large log files with incremental rendering

## Non Goals

-   Replace production log aggregation systems
-   Provide database indexing
-   Full distributed tracing

------------------------------------------------------------------------

# 3. UI Design

## Layout

    +------------------------------------------------+
    | File | Search | Filter | Follow Tail           |
    +------------------------------------------------+
    | MAIN LOG VIEW                                  |
    | 000001 2026-07-08 14:28:39 [1] TRACE           |
    |        Hero.Core.Streaming.Pipeline            |
    |        Loading voxel model data                |
    |                                                |
    | 000002 2026-07-08 14:28:40 [1] ERROR           |
    |        Hero.Core.Battle.System                 |
    |        NullReferenceException                  |
    |        stack trace...                          |
    |                                                |
    +------------------------------------------------+
    | ================= SPLITTER =================== |
    +------------------------------------------------+
    | FILTER RESULT VIEW                             |
    | 000002 2026-07-08 14:28:40 [1] ERROR           |
    |        Hero.Core.Battle.System                 |
    |        NullReferenceException                  |
    |        stack trace...                          |
    |                                                |
    +------------------------------------------------+

## Default Style

    Font:
    Courier New

    Size:
    12px

    Theme:
    Dark

    Background:
    #1e1e1e

------------------------------------------------------------------------

# 4. Log Entry Model

Each log entry is represented as:

``` javascript
{
    lineNumber: 100,
    timestamp: "2026-07-08 14:28:39",
    thread: "1",
    level: "TRACE",
    logger: "Hero.Core.Streaming.Pipeline",
    message: "Loading voxel model",
    stackTrace: []
}
```

------------------------------------------------------------------------

# 5. NLog Parser

## Supported Layout Example

    ${longdate} [${threadid}] ${level} ${logger} - ${message}

Example:

    2026-07-08 14:28:39.9932 [1] TRACE Hero.Core.Streaming.Pipeline - Loading VoxelModel

Parsed result:

  Field     Value
  --------- ------------------------------
  Date      2026-07-08 14:28:39.9932
  Thread    1
  Level     TRACE
  Logger    Hero.Core.Streaming.Pipeline
  Message   Loading VoxelModel

------------------------------------------------------------------------

# 6. Parser Architecture

    LogParser

        |
        +-- NLogParser
        |
        +-- GenericParser
        |
        +-- CustomSeparatorParser

Parser responsibilities:

-   Detect timestamp
-   Detect log level
-   Extract logger name
-   Extract message body
-   Merge multiline stack traces

------------------------------------------------------------------------

# 7. Rendering Architecture

Both views share the same Virtual Renderer, differing only by their data source.

    LogViewer

     |
     +-- Parser

     |
     +-- Filter Engine
     
     |
     +-- Selection Sync Engine

     |
     +-- View Pane (Main View)
     |    |
     |    +-- Virtual Renderer
     |    +-- Log Entry Component
     
     |
     +-- View Pane (Filter Result View)
          |
          +-- Virtual Renderer
          +-- Log Entry Component

------------------------------------------------------------------------

# 8. Log Entry UI

Example:

``` html
<div class="log-entry">

    <div class="line-number">
        00001
    </div>

    <div class="content">

        <div class="header">
            Time
            Thread
            Level
            Logger
        </div>

        <div class="message">
            Message body
        </div>

    </div>

</div>
```

------------------------------------------------------------------------

# 9. Features

  Feature                   Support
  ------------------------- ---------
  Dark theme                Yes
  Courier New 12px          Yes
  Vertical layout           Yes
  Synchronized Split View   Yes
  Click-to-sync selection   Yes
  NLog parsing              Yes
  Line wrapping             Yes
  Regex search              Yes
  Level filtering           Yes
  Stack trace grouping      Yes
  Live tail                 Yes
  Large file optimization   Planned
  Column resize             Planned

------------------------------------------------------------------------

# 10. Filtering & Interactivity

## Split-View Synchronization
-   Clicking a row in the **Filter Result View** scrolls the **Main View** to that exact log line and highlights it.
-   Selecting a row in the **Main View** highlights the corresponding row in the **Filter Result View** (if it passed the filter).

## Supported Filters

### Level

    ERROR
    WARN
    INFO
    DEBUG
    TRACE

### Logger

Example:

    Hero.Core.Streaming.*

### Regex

Example:

    (PlayerId|BattleId)=\d+

------------------------------------------------------------------------

# 11. Color Rules

    TRACE -> Gray

    DEBUG -> Light Gray

    INFO  -> Blue

    WARN  -> Yellow

    ERROR -> Red

------------------------------------------------------------------------

# 12. Performance Design

## Large File Support

Future design:

    File
     |
     +-- Chunk Reader
     |
     +-- Line Index
     |
     +-- Shared Virtual List Renderer

Only visible lines are rendered in the Main View and Filter View.

------------------------------------------------------------------------

# 13. Comparison

  Feature          klogg       New Viewer
  ---------------- ----------- ------------
  Split View       Yes         Yes
  Huge files       Excellent   Planned
  Regex search     Yes         Yes
  NLog parsing     No          Yes
  Wrap line        No          Yes
  Vertical view    No          Yes
  Unity friendly   Medium      High

------------------------------------------------------------------------

# 14. Future Extensions

## NLog Layout Configuration

Allow user input:

    ${longdate} [${threadid}] ${level} ${logger} - ${message}

Parser automatically adapts.

## Unity Integration

Special formatting:

-   Collapse long type names
-   Highlight Unity objects
-   Detect exception stack traces
-   Filter by subsystem

## Advanced Features

-   Bookmarks
-   Saved filters
-   Log sessions
-   Diff two logs
-   Timeline view