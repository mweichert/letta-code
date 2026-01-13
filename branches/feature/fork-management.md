# feature/fork-management

Fork management infrastructure for maintaining a declarative, rebased fork of letta-ai/letta-code.

## Components

| File | Purpose |
|------|---------|
| `fork.yaml` | Declarative configuration defining upstream and branches |
| `scripts/build-fork.py` | Build script for syncing, rebasing, and merging |
| `CLAUDE.md` | Documentation including fork management instructions |
| `branches/` | Per-branch documentation |

## Usage

```bash
# Full rebuild: sync upstream, rebase branches, build fork, push
uv run scripts/build-fork.py

# Preview changes without executing
uv run scripts/build-fork.py --dry-run
```

## How It Works

1. **Sync Upstream**: Fetches `upstream/main` and resets local `main` to match
2. **Rebase Branches**: Topologically sorts and rebases each branch onto its base
3. **Build Fork**: Creates fresh `fork` branch from `main`, merges all feature/bugfix branches
4. **Auto-Tag**: If fork changed, creates versioned tag (e.g., `0.12.8-fork.1`)

## Notes

- The `fork` branch is rebuilt from component branches - direct edits will be overwritten
- Changes to fork infrastructure (fork.yaml, build script, docs) belong in this branch
- Version extraction uses `package.json` (adapted from Python/pyproject.toml original)
