---
name: working-with-the-blog
description: Use when writing, editing, or reviewing blog posts, drafts, and ideas, or when modifying the blog service's build, validation, and deployment pipeline.
---

# Working with the Blog

## Overview
This skill guides you through the technical implementation, directory layouts, typography rules, prose voice, and deployment workflows of xqhare.net's personal blog (`blog.xqhare.net`).

## Web Service & Deployment Architecture
* **Isolation**: Served by an Nginx instance in a standalone Docker container, separate from main `xqhare.net` pages.
* **Git-Driven Workflow**: Gitea triggers a webhook on `git push`, executing `hook.sh` to run `build.sh` and `deploy.sh`.
* **Atomic Deployments**:
  1. `./build.sh` generates static HTML under `build/`.
  2. `./deploy.sh` moves `build/` to a timestamped folder: `data/YYYY-MM-DD-HH-MM-SS/`.
  3. The Nginx root points to a symlink `html/` which is atomically updated to the new timestamped folder.
  4. `./rollback.sh` reverts the symlink to the previous folder in case of errors.

## Project Layout
* `pages/posts/[slug]/post.md`: Active, published blog posts. The `[slug]` folder name must match the `title` slug in YAML exactly.
* `ideas/`: Drafts and simmer files (exempt from build validation).

## Metadata (YAML) Rules
Every post must start with a YAML front-matter block containing:
* `title`: lowercase_with_underscores (no spaces, matches directory exactly).
* `display_title`: Readable post title.
* `date`: `YYYY-MM-DD` (**CRITICAL**: format must be exact for reverse alphabetical `ls -r` chronological sorting).
* `author`: Must be exactly `Xqhare`.
* `tags`: List of tags. The single source of truth is the Tags table in `README.md`.
* `abstract`: 1-2 sentence preview.

## Typography Taxonomy
Asterisks (`*`) are strictly reserved for the style taxonomy. Normal grammatical emphasis must use underscores (`_`).

| Element Type | Markdown Syntax | Rendered Example | Description |
| :--- | :--- | :--- | :--- |
| **Personal Projects** | `***Bold Italic***` | `***Talos***` | Pantheon libraries. Must have GitHub link on first mention: `[***Talos***](https://github.com/xqhare/talos)`. |
| **External Tools/Lang** | `*Italic*` | `*rust*`, `*git*`, `*libc*` | Third-party software/languages. |
| **Spec Constants** | `**Bold**` | `**Start of Text**` | Byte/protocol markers. |
| **Code Identifiers** | `` `Backticks` `` | `` `BTreeMap` `` | Variables, types, functions, files. |
| **Terminal Commands** | `` `$ Backticks` `` | `` `$ cargo run` `` | Prefix shell inputs with a `$`. |

## Narrative Style & The "Xqhare Loop"
* **The Hook**: Open in-media-res with a personal anecdote, shower thought, or direct question. Never use generic introductions like *"In this post, I will..."*.
* **The Pivot**: A clear transition to the moment of realization.
* **The Deep Dive**: High-density technical block, code, tables, or diagrams.
* **Philosophical Outro**: Takeaways and internal mantras.
* **Perspective**: Always first-person singular (`I`, `my`, `me`). Never use collective `we` or passive voice.
* **Fail-Forward**: Every post must include at least one design mistake, dead end, or early "shit" version of the project.
* **Accessibility**: Serving format must be `.webp`. Alt text `![Alt Text](image.webp)` must describe the image contents clearly and concisely (avoiding "image of..." prefixes).

## Automated Validation & Self-Learning
* `verify_posts.py` runs during `./build.sh` and blocks build on validation failure.
* **Self-Learning Style rules**: Automatically registers new asterisk patterns in `style_rules.json`.
* **Self-Learning Tags**: Automatically appends new tags to the markdown table in `README.md` and sorts them.
