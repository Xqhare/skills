---
name: nomos-cli
description: Use when configuring the Nomos workspace, validating task files, migrating legacy projects, or querying task execution queues using the nomos CLI.
---

# Nomos CLI and Configuration

## Overview
This skill guides the use of the Nomos command-line interface and configuration system to manage project tasks.

**REQUIRED BACKGROUND:** You must understand the basic file syntax described in the `nomos` skill.

## Global Configuration
On first run, Nomos initializes a global configuration at `~/.config/nomos/config.json` (or fallback `~/.nomos/config.json`).
Example structure:
```json
{
  "search_bases": [
    "/home/xqhare/Adytum/Programming/rust/"
  ],
  "files": {
    "personal_tasks": "/home/xqhare/Adytum/personal.nomos"
  },
  "version": 1,
  "default_prio_level": 5
}
```
- **search_bases**: Folders crawled by Nomos to find projects.
- **files**: Direct mappings for standalone files.

## Project Configuration (`nomos.json`)
Located in project subdirectories inside `search_bases`.
Example structure:
```json
{
  "task_files": [
    "TODO.nomos",
    "docs/roadmap.nomos"
  ],
  "version": 1,
  "default_prio_level": 5
}
```
- If no `nomos.json` is present, Nomos defaults to searching for `nomos`, `todo`, `tasks`, or `roadmap` files with a `.nomos` extension (or legacy `.md`/`.txt`).

## Version Resolution Precedence
When parsing task files, format version is resolved as follows:
1. **In-File Metadata Override**: First non-empty line is `<!-- nomos: X -->` (where X is `0` or `1`).
2. **Project Configuration**: `version` key in `nomos.json`.
3. **Global Configuration**: `version` key in `config.json`.
4. **Extension Inference**: `.md` defaults to v0 (legacy); `.nomos` defaults to v1.

## CLI Commands
- **Validate**: Checks global config, project configs, and referenced task files for correct formatting and parsability.
  ```bash
  nomos validate
  ```
- **All**: Lists all tasks across crawl paths, sorted topographically (Kahn sort).
  ```bash
  nomos all
  ```
- **Next**: Displays the single next task to work on.
  ```bash
  nomos next
  ```
- **Update**: Migrates legacy v0 files (alphabetical priorities, mandatory delimiters) to v1 (`.nomos`).
  ```bash
  nomos update
  ```

## Filtering Flags
These flags can be applied to `all` and `next`:
- `-p=<project>`: Limit output to a specific project (case-insensitive).
- `-s=<status>`: Filter by status (e.g., `-s=open`, `-s=in_progress`, `-s=done`, `-s=blocked`, `-s=deferred`, `-s=cut`).
- `-n <number>`: Limit output to N tasks (e.g. `nomos all -n 5`).
