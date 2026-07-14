---
name: nomos
description: Use when creating, editing, or parsing Nomos task files (.nomos) or embedding tasks in Markdown documents.
---

# Nomos File Syntax and Basics

## Overview
Nomos is a decentralized, text-based project management system. This skill details the task list syntax for Nomos files (`.nomos`) or embedded tasks in standard Markdown documents.

## File Format
Nomos task lists are standard Markdown lists. Lines that do not start with a task `- ` or note `* ` at the correct indentation level are ignored by the parser.

## Task Line Syntax
The syntax of a task line is:
```text
- [Status] (Priority) Title :: [InceptionDate] [CompletionDate] Description +kindTag @locationTag #genericTag keyTag=valueTag dep="Dependency Title"
```

### Status
- `[ ]` Open
- `[/]` In Progress
- `[x]` Done
- `[B]` Blocked
- `[C]` Cut
- `[D]` Deferred

### Priority
Enclosed in parentheses `(0)` through `(9)` before the title:
- `(1)` is the highest standard priority, `(9)` is the lowest.
- `(0)` is Extremely Important. **Usage is discouraged** and reserved strictly for critical incidents (e.g., production outage, security breach).
- Defaults to `5` if omitted.

### Title and Delimiter
- **Title**: Must be unique within the project.
- **Delimiter**: Double-colon with surrounding spaces (` :: `) is optional under v1 for simple tasks containing only a title, but mandatory if dates, descriptions, tags, or dependencies follow.

### Dates
Optional dates in `YYYY-MM-DD` format:
- One date: `[InceptionDate]`
- Two dates: `[InceptionDate] [CompletionDate]` (CompletionDate is only allowed if status is resolved: `[x]` or `[C]`).

### Tags
- **+Kind**: `+bug`, `+feature` (leading `+`, no whitespace)
- **@Location**: `@src/main.rs` (leading `@`)
- **#Generic**: `#idea`, `#todo` (leading `#`, no whitespace)
- **Key=Value**: Custom metadata (e.g. `est=2d`, `owner="Xqhare and Partner"`). Double quotes are supported for values containing spaces.

### Dependencies
- Defined via metadata tags: `dep="Dependency Title"` (intra-project) or `dep="project_name:Dependency Title"` (inter-project).
- Double quotes are required around task titles.

## Indentation and Hierarchy
- **Child/Subtasks**: Indented by exactly **4 spaces** under their parent task. Indentation is syntactically significant.
- **Notes**: Start with `* ` and must be indented by **one level deeper than the task they describe** (the same level as child tasks, i.e., parent task indent + 4 spaces). Notes cannot define dependencies.
- **Behavior**: Parent tasks implicitly depend on their child subtasks and cannot be marked Done (`[x]`) until all child tasks are resolved (Done or Cut).

## Cross References
- **CLI operations**: See `nomos-cli` skill.
