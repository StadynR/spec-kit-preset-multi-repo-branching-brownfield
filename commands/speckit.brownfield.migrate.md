---
description: Reverse-engineer brownfield features while preserving multi-repository scope
---

# Migrate Existing Features

Reverse-engineer `spec.md`, `plan.md`, and `tasks.md` for an existing feature, including the child repositories that implement or support it.

## User Input

```text
$ARGUMENTS
```

You **MUST** consider the user input before proceeding (if not empty).

## Prerequisites

1. Verify `.specify/` exists, Git is available, and the project has source code.
2. Verify the constitution exists or ask the user to bootstrap first.
3. Discover child repositories using `multi_repo_branching` settings: parse `.gitmodules` for `submodule`, scan child `.git` directories for `independent`, and choose between them automatically for `auto`.

## Outline

1. Identify the migration target and its files, tests, dependencies, API surface, and database changes across the root and child repositories.
2. Determine which repositories are affected by the feature based on actual implementation files, imports, service calls, submodule usage, and commits. Do not infer impact from repository names alone.
3. Reverse-engineer `spec.md`, `plan.md`, and completed `tasks.md` as in the standard brownfield migration workflow. Mark the spec `status: migrated`.
4. In `plan.md`, add an `Affected Repositories` section under `Project Structure` when applicable:

   ```markdown
   ### Affected Repositories
   | Repo Path | Type | Reason |
   |-----------|------|--------|
   | services/payments | independent | Existing payment implementation |
   ```

5. In the generated `tasks.md`, record branch setup tasks at the beginning of Phase 1 using:
   - Independent: `git -C "<repo_path>" checkout -b "<BRANCH_NAME>"`
   - Submodule: `git submodule update --init "<repo_path>" && git -C "<repo_path>" checkout -b "<BRANCH_NAME>"`
   These are documentation of the coordinated workflow; do not execute them during migration.
6. Report gaps separately for each repository and ask for confirmation before writing artifacts. For `all`, migrate one feature at a time.

## Rules

- Read existing source and history without modifying source code or creating branches.
- Mark all reconstructed implementation tasks `[x]`; leave newly recommended follow-up work unchecked.
- Be honest about inaccessible or uninitialized child repositories.
- Preserve relative child repository paths and repository type in all generated artifacts.