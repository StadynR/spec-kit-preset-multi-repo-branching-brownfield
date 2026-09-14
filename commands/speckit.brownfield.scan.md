---
description: Auto-discover an existing project and its child repositories
---

# Scan Project

Analyze an existing codebase to discover its technology stack, architecture patterns, module structure, coding conventions, and child Git repositories. This produces a project profile for the brownfield bootstrap workflow.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty). The user may specify a subdirectory, a focus area, or a scan depth.

## Prerequisites

1. Verify the current directory is a Git repository.
2. Verify this is an existing project with source code.

## Outline

1. Detect languages, package managers, frameworks, build tools, CI/CD, and test frameworks.
2. Analyze architecture and map modules, dependencies, and code boundaries.
3. Extract naming, branching, commit, testing, and documentation conventions.
4. Detect existing governance such as `CONTRIBUTING.md`, `ARCHITECTURE.md`, `AGENTS.md`, and `.specify/`.
5. Discover child repositories using the `multi_repo_branching` configuration:
   - Read `.specify/init-options.json` and use `multi_repo_branching.type` (`auto`, `independent`, or `submodule`) and `scan_depth` (default `2`).
   - For `submodule`, parse `.gitmodules` and list each configured `path`.
   - For `independent`, find directories containing their own `.git` directory up to `scan_depth`, excluding the root repository.
   - For `auto`, use submodule discovery when `.gitmodules` exists; otherwise use independent discovery.
   - Use `git check-ignore` while traversing independent repositories. A directory containing its own `.git` remains a valid child even when ignored; do not descend through an ignored directory without `.git`.
   - Record each child repository's path, type, detected stack, purpose, and relationship to the root project.
   - If no configuration exists, use `type = auto` and `scan_depth = 2`.
6. Output a `# Project Profile` with the normal brownfield sections plus:

   ```markdown
   ## Child Repositories
   | Repo Path | Type | Stack/Purpose | Relationship |
   |-----------|------|---------------|--------------|
   | components/auth | independent | Auth service | Root application dependency |
   ```

   The child repository inventory is the evidence used later to determine the `Affected Repositories` section in a feature plan.

## Rules

- **Read-only**: never modify source code or repository state.
- Respect `.gitignore`; never scan `node_modules/`, `vendor/`, `dist/`, `.venv/`, or other ignored directories unless they are child repositories with their own `.git` directory.
- Report only observed facts. Use `Not detected` where evidence is absent.
- Keep child repository paths relative to the root when presenting the profile.