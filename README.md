# Multi-Repo Branching for Spec Kit + Brownfield Extension

A community preset for [Spec Kit](https://github.com/github/spec-kit) that coordinates feature branch creation across multiple git repositories. It includes configuration files for the agents provided by the [Spec Kit Brownfield extension](https://github.com/Quratulain-bilal/spec-kit-brownfield).

## Problem

In multi-module projects where sub-components maintain independent git histories (stored within a shared root) or use git submodules, Spec Kit only operates on the root repository. Developers must manually create matching feature branches in each affected sub-component.

## What This Preset Does

| Phase | Without Preset | With Preset |
|-------|---------------|-------------|
| **Plan** | Analyzes root repo only | Discovers child repos, identifies which are affected by the feature |
| **Tasks** | No branch setup tasks | Generates Phase 1 tasks to create feature branches in affected repos |

The preset supports two types of child repositories:

- **Independent repos** — subdirectories with their own `.git` (not formal git submodules)
- **Git submodules** — formally registered in `.gitmodules`

## Installation

### Brownfield Extension

Follow the steps at https://github.com/Quratulain-bilal/spec-kit-brownfield#installation

### Preset Installation

```bash
# Install from GitHub release
specify preset add --from https://github.com/StadynR/spec-kit-preset-multi-repo-branching-brownfield/archive/refs/tags/v1.0.0.zip
# Or install from local directory (for development)
specify preset add --dev /path/to/spec-kit-preset-multi-repo-branching

# Verify installation
specify preset list
specify preset resolve speckit.plan
```

## Configuration

Add to `.specify/init-options.json` in your project root:

```json
{
  "multi_repo_branching": {
    "scan_depth": 3,
    "type": "auto"
  }
}
```

### Options

| Option | Values | Default | Description |
|--------|--------|---------|-------------|
| `type` | `"independent"`, `"submodule"`, `"auto"` | `"auto"` | How to discover child repos |
| `scan_depth` | `1`–`10` | `2` | Max directory depth to scan |

### Type Modes

- **`independent`** — Scans for directories containing `.git` that are NOT the repo root. Uses `git check-ignore` to filter gitignored directories during traversal. A directory with its own `.git` is always reported even if gitignored.
- **`submodule`** — Parses `.gitmodules` in the repo root to find registered submodule paths. No filesystem scanning needed.
- **`auto`** (default) — Checks if `.gitmodules` exists. If yes, uses submodule mode. If no, uses independent mode.

### Configuration Not Required

If `.specify/init-options.json` doesn't exist or has no `multi_repo_branching` key, the preset uses defaults: `type = "auto"`, `scan_depth = 2`.

## How It Works

### During `/speckit.plan`

1. After running the setup script, the AI agent reads your config
2. Based on the `type` setting:
   - **Independent**: Runs `find` to locate `.git` directories, filters with `git check-ignore`
   - **Submodule**: Parses `.gitmodules` for submodule paths
   - **Auto**: Checks for `.gitmodules`, falls back to independent scan
3. Cross-references discovered repos against the feature spec
4. Documents affected repos in the plan's **Affected Repositories** section

### During `/speckit.tasks`

1. Reads the "Affected Repositories" section from `plan.md`
2. Generates Phase 1 setup tasks for branch creation:
   - **Independent**: `git -C "<path>" checkout -b "<BRANCH>"`
   - **Submodule**: `git submodule update --init "<path>" && git -C "<path>" checkout -b "<BRANCH>"`

## Examples

### Independent Repos (Monorepo Without Submodules)

Project structure:
```
my-project/           # Root repo (.git)
├── components/
│   ├── auth/         # Independent repo (.git)
│   └── api/          # Independent repo (.git)
├── libs/
│   └── shared/       # Independent repo (.git)
└── .specify/
    └── init-options.json
```

Config:
```json
{
  "multi_repo_branching": {
    "type": "independent",
    "scan_depth": 3
  }
}
```

### Git Submodules

Project structure:
```
my-project/           # Root repo (.git)
├── vendor/
│   ├── lib-a/        # Submodule
│   └── lib-b/        # Submodule
├── .gitmodules
└── .specify/
    └── init-options.json
```

Config:
```json
{
  "multi_repo_branching": {
    "type": "submodule"
  }
}
```

### Auto-Detection (No Config Needed)

If you don't add any config, the preset auto-detects:
- Checks for `.gitmodules` → submodule mode
- No `.gitmodules` → independent scan with depth 2

## What's Overridden

This preset overrides **commands only** — no template overrides. Core templates are used as-is, and the commands instruct the AI agent to add multi-repo sections when generating plan.md and tasks.md.

| File | Type | Core Behavior Preserved | Additions |
|------|------|------------------------|-----------|
| `speckit.plan` | command | ✅ Full workflow | + Steps 3-4: discover repos, identify affected, add section to plan.md |
| `speckit.tasks` | command | ✅ Full workflow | + Branch-creation task generation in Phase 1 |
| `speckit.brownfield.scan` | command | ✅ Brownfield discovery | + Child repository discovery and repository inventory in the project profile |
| `speckit.brownfield.bootstrap` | command | ✅ Brownfield bootstrap | + Multi-repo boundaries, `Affected Repositories`, and branch setup guidance |
| `speckit.brownfield.validate` | command | ✅ Brownfield validation | + Validation of child repositories, ownership, and branch setup commands |
| `speckit.brownfield.migrate` | command | ✅ Brownfield migration | + Cross-repository feature scope and documented branch setup tasks |

For the brownfield workflow, the preset allows to:

1. Discover independent repositories and Git submodules using `multi_repo_branching` configuration.
2. Preserve repository boundaries when generating the constitution, templates, and `AGENTS.md`.
3. Record affected repositories in `plan.md` using an `Affected Repositories` table.
4. Add branch creation tasks for affected repositories at the start of Phase 1 in `tasks.md`.
5. Include repository scope and branch coordination when migrating existing features.

The Copilot agent files generated by the Brownfield extension receive the same instructions:

- `.github/agents/speckit.brownfield.scan.agent.md`
- `.github/agents/speckit.brownfield.bootstrap.agent.md`
- `.github/agents/speckit.brownfield.validate.agent.md`
- `.github/agents/speckit.brownfield.migrate.agent.md`

### Why commands-only?

Overriding templates requires copying the entire core template to add a few lines. When upstream updates templates, presets become stale. By keeping template overrides out, the AI uses the latest core templates and adds multi-repo sections dynamically via command instructions.

Command overrides are clearly marked with `<!-- PRESET: multi-repo-branching START/END -->` comments around additions for easy diffing when upstream changes. See [Keeping Up with Upstream](#keeping-up-with-upstream) for the update workflow.

## Keeping Up with Upstream

This preset overrides core `speckit.plan`, `speckit.tasks`, and the four brownfield commands. When Spec Kit or the Brownfield extension releases a new version, the upstream commands may change. The core plan and tasks files include a tracking header:

```html
<!-- Based on spec-kit v0.5.1 (SHA: aa2282e) — core content from github/spec-kit -->
```

### Checking for drift

1. Note the version/SHA in the tracking header
2. Compare against the latest Spec Kit release on [github/spec-kit](https://github.com/github/spec-kit/releases)
3. If there's a new release, diff the core commands:
   ```bash
   # Fetch the core command from the version your preset is based on
   git show v0.5.1:commands/speckit.plan.md > /tmp/old-plan.md

   # Fetch the core command from the new release
   git show v0.6.0:commands/speckit.plan.md > /tmp/new-plan.md

   # See what changed
   diff /tmp/old-plan.md /tmp/new-plan.md
   ```

### Applying updates

1. Open the preset's command file (e.g., `commands/speckit.plan.md`)
2. Look for `<!-- PRESET: multi-repo-branching START -->` and `<!-- PRESET: multi-repo-branching END -->` markers — these are the preset additions
3. Update the core content (everything **outside** the markers) to match the new upstream version
4. Update the tracking header with the new version and SHA
5. Test, commit, and retag

### When composition lands

Spec Kit has a composition feature in development (`strategy: wrap`, `strategy: append`). Once merged, a new version of this preset will be released that uses thin wrapper commands referencing core content via `{CORE_TEMPLATE}` — eliminating the need to track upstream changes manually. At that point, the preset will automatically stay in sync with core updates without requiring new releases for each Spec Kit version.

## Stacking

This preset is designed to work well when stacked with other presets. It only adds sections to templates and steps to commands — it does not remove or reorder any core behavior.

If another preset also overrides one of these commands, the one with higher priority wins. Use `--priority` when installing to control order.

## Removing

```bash
specify preset remove multi-repo-branching
```

## License

MIT
