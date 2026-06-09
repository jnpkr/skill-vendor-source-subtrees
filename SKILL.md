---
name: vendor-source-subtrees
description: Guide for vendoring external source repositories with git subtree so coding agents can inspect real dependency code. Use when setting up agent-readable library source, adding or updating repositories under repos/, configuring editor exclusions for vendored code, writing AGENTS.md guidance for read-only reference repositories, or creating project-local pattern files from vendored source.
---

# Vendor Source Subtrees

## Overview

Use git subtrees to make external dependency source code locally explorable for coding agents while keeping application imports, editor behavior, and ownership boundaries clean.

This skill is based on the workflow described in Effect's article "The One Weird Git Trick That Makes Coding Agents More Effect-ive": https://effect.website/blog/the-one-weird-git-trick-that-makes-coding-agents-more-effect-ive/

## Principles

- Prefer real source code over isolated docs snippets or web search when an agent needs idiomatic usage patterns, tests, module structure, or API design.
- Vendor external repositories as read-only reference material, not as application code.
- Keep vendored repositories under a single top-level directory, normally `repos/`.
- Continue importing from normal package dependencies; never import application code from `repos/`.
- Prefer `git subtree` over `git submodule` when the goal is agent reference material: subtrees behave like ordinary directories after clone and avoid extra initialization steps.
- Use `--squash` so the external repository history is collapsed into one commit per add or update.
- Avoid relying on `node_modules` as the reference source because it is often compiled, flattened, ignored, or deoptimized for agent exploration.

## Add A Subtree

Before making changes, identify the dependency repository URL, branch, and target directory. Use `repos/<library>` unless the project already has a convention.

```bash
git subtree add \
  --prefix=repos/<library> \
  <repository-url> \
  <branch> \
  --squash
```

Example:

```bash
git subtree add \
  --prefix=repos/effect \
  https://github.com/Effect-TS/effect.git \
  main \
  --squash
```

After adding a subtree, inspect the resulting diff and commit history shape. Expect a new directory under `repos/` and a squashed subtree commit.

## Update A Subtree

Use the same prefix, repository URL, and branch used when adding the subtree:

```bash
git subtree pull \
  --prefix=repos/<library> \
  <repository-url> \
  <branch> \
  --squash
```

Review updates as third-party source refreshes. Do not mix application changes into the same commit unless explicitly requested.

## Configure VS Code

When a project vendors repositories under `repos/`, add or merge these settings into `.vscode/settings.json` so human editor search, file watching, and auto-imports stay focused on application code:

```json
{
  "typescript.preferences.autoImportFileExcludePatterns": [
    "repos/**"
  ],
  "javascript.preferences.autoImportFileExcludePatterns": [
    "repos/**"
  ],
  "files.exclude": {
    "repos/**": true
  },
  "files.watcherExclude": {
    "repos/**": true
  },
  "search.exclude": {
    "repos/**": true
  }
}
```

When `.vscode/settings.json` already exists, merge these keys without removing existing project settings.

## Configure Agent Instructions

Add explicit guidance to the repository's agent instruction file, usually `AGENTS.md`. Include both the location and the intended usage:

```markdown
## Vendored Repositories

This project vendors external repositories under `repos/`.

- Use vendored repositories as read-only reference material when working with related libraries.
- Prefer examples and patterns from vendored source over generated guesses or web search results.
- Do not edit files under `repos/` unless explicitly asked.
- Do not import from `repos/`; application code should use normal package imports.
```

Add library-specific instructions when useful:

```markdown
When writing Effect code, inspect `repos/effect/` for idiomatic usage, tests, module structure, and API design. Treat it as the source of truth for Effect patterns.
```

If a vendored repository includes an agent-oriented guide such as `LLMS.md`, instruct agents to read it before writing code for that library.

## Create Pattern Files

When the same vendored library area will be used repeatedly, create a small project-local reference under `agent-patterns/`. Keep it practical and specific to the application.

Suggested prompt:

```markdown
Review the implementation, tests, and documentation for `<topic>` in `repos/<library>`.
Create `agent-patterns/<library>-<topic>.md` with the most important patterns to follow in this project.
Include common constructors or APIs, usage examples, error handling patterns, examples adapted from the vendored source, and notes about what to avoid.
```

Pattern files should summarize the useful idioms. They should not mirror an entire upstream documentation set.

## Verification

- Confirm `repos/<library>` exists and contains source, tests, or docs useful to agents.
- Confirm `.vscode/settings.json` excludes `repos/**` without deleting unrelated settings.
- Confirm `AGENTS.md` or the project's equivalent agent guide treats `repos/` as read-only reference material.
- Confirm generated application code imports from normal dependencies, not vendored paths.
