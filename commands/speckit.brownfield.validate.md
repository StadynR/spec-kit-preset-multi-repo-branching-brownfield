---
description: Validate brownfield configuration and multi-repository coordination
---

# Validate Bootstrap

Verify that the brownfield bootstrap artifacts match the actual project, including all configured child repositories and the branch setup workflow.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Prerequisites

1. Verify `.specify/` exists and the current directory is a Git repository.
2. Verify at least one bootstrap artifact exists.

## Outline

1. Validate the constitution, templates, and `AGENTS.md` against the actual root project as in the standard brownfield validation workflow.
2. Discover child repositories using `.specify/init-options.json`:
   - Read the `multi_repo_branching` object; use `type = auto` and `scan_depth = 2` when it is absent.
   - `submodule`: parse `.gitmodules`.
   - `independent`: find child `.git` directories up to `scan_depth`.
   - `auto`: choose submodules when `.gitmodules` exists, otherwise independent repositories.
3. Validate multi-repository configuration:
   - `type` is `auto`, `independent`, or `submodule` and `scan_depth` is within `1`-`10`.
   - Every configured submodule path exists or is reported as missing.
   - Every discovered child repository is represented in repository ownership or module documentation.
   - Plan and tasks templates describe the `Affected Repositories` table and the correct branch command for each repository type.
   - Branch setup tasks use the current feature branch and appear before other Phase 1 tasks.
4. Report evidence-based passes, drift, and actionable fixes. This command is read-only.

## Rules

- Never scan ignored directories unless they contain their own `.git` directory.
- Do not treat a missing child repository checkout as a valid reason to skip validation; report it clearly.
- Do not modify configuration or repository state.