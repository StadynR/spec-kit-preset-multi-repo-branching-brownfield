---
description: Generate brownfield spec-kit configuration for a multi-repository project
---

# Bootstrap Spec-Kit

Generate a customized spec-kit configuration for an existing codebase, preserving the boundaries and branch workflow of independent child repositories and Git submodules.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Prerequisites

1. Verify the current directory is a Git repository and `.specify/` exists.
2. Load the profile from `/speckit.brownfield.scan`, or perform the scan inline if it is unavailable.
3. Discover child repositories using the `multi_repo_branching` settings described below.

## Outline

1. Confirm the observed project profile and child repository list with the user.
2. Generate `.specify/memory/constitution.md` from observed conventions. Include repository boundaries, ownership, dependency direction, and the rule that changes spanning child repositories must identify each affected repository.
3. Customize the spec, plan, and tasks templates from the actual project structure.
4. Ensure the plan workflow records an `Affected Repositories` section under `Project Structure` when a feature affects child repositories. Use this table shape:

   ```markdown
   ### Affected Repositories
   | Repo Path | Type | Reason |
   |-----------|------|--------|
   | components/auth | independent | Feature changes authentication behavior |
   ```

5. Ensure the tasks workflow creates Phase 1 branch setup tasks at the start of the phase for every affected repository:
   - Independent: `git -C "<repo_path>" checkout -b "<BRANCH_NAME>"`
   - Submodule: `git submodule update --init "<repo_path>" && git -C "<repo_path>" checkout -b "<BRANCH_NAME>"`
   - Mark these setup tasks `[P]` when they have no dependency on one another.
6. If generating `AGENTS.md`, assign ownership by repository path and document cross-repository coordination.
7. Present the bootstrap plan and ask for confirmation before writing. Never overwrite existing artifacts without showing the proposed merge or diff.

## Multi-Repository Discovery

- Read `.specify/init-options.json`; defaults are `type = auto` and `scan_depth = 2`.
- `submodule` parses `.gitmodules`.
- `independent` finds child directories containing `.git` up to the configured depth.
- `auto` selects `submodule` when `.gitmodules` exists and `independent` otherwise.
- A child repository remains valid even when ignored by its parent repository. Paths must be recorded relative to the root.

## Rules

- Derive all configuration from observed project evidence.
- Preserve existing standards and merge with existing spec-kit customizations.
- Do not create branches during bootstrap; branch creation belongs in generated feature tasks.
- Report child repositories that cannot be inspected instead of guessing their contents.