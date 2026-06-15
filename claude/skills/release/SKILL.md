---
name: release
description: >
  This skill should be used when the user wants to cut a release — bump the version, commit,
  push, and verify CI created the tag and release. Triggers on "release this", "cut a release",
  "ship it", "bump version", or "release v...".
---

# /release

To ship a release: bump VERSION, commit with a conventional commit message, push. CI handles the rest.

## What CI Does Automatically (cortex)

On push to `main` with a new VERSION:
1. Reads `VERSION` file
2. Checks if tag `v{VERSION}` already exists — skips release if so
3. Builds and pushes Docker image to GHCR (`ghcr.io/xoudusz/cortex-mcp:latest`, `:vX.Y.Z`, `:X.Y`)
4. Creates git tag `v{VERSION}`
5. Auto-generates release notes from commit subjects (conventional commits → categorized sections)
6. Creates GitHub release
7. Publishes PyPI package from `local/`

## Pre-Release Checklist

Before bumping VERSION, verify:
- [ ] `notes/projects/<project>.md` updated — version frontmatter + Done entry
- [ ] `CLAUDE.md` updated if tools/structure changed
- [ ] All changes committed (no dirty working tree)
- [ ] `/document` has been run

## Version Bump Rules

| Change | Bump | Example |
|--------|------|---------|
| Bug fix, refactor, docs, config | Patch (x.y.**Z**) | 2.1.5 → 2.1.6 |
| New tool, endpoint, UI feature | Minor (x.**Y**.0) | 2.1.6 → 2.2.0 |
| Breaking API, schema migration, auth change | Major (**X**.0.0) | 2.1.6 → 3.0.0 |

## Conventional Commit Format

CI uses commit subjects to generate release notes. Use the correct prefix:

| Prefix | Release Notes Section |
|--------|-----------------------|
| `feat:` / `feature:` | Features |
| `fix:` / `bugfix:` | Fixes |
| `perf:` | Performance |
| `refactor:` | Refactoring |
| `docs:` / `doc:` | Docs |
| `ci:` / `build:` / `chore:` | CI / Infra |
| `test:` / `tests:` | Tests |
| (no prefix) | Other |

Example: `feat: add eval MCP tool` → appears under **Features** in release notes.

## Release Steps

```bash
# 1. Bump VERSION
echo "X.Y.Z" > VERSION

# 2. Commit with conventional prefix matching the change type
git add VERSION
git commit -m "feat: <description>"   # or fix:, perf:, docs:, etc.

# 3. Push — CI does everything else
git push
```

## Verify Release

```bash
gh release list --limit 3          # confirm new release created
gh run list --limit 3              # confirm CI passed
```

## Notes

- Never manually create a tag or release — CI does it; duplicates cause errors
- Never delete and recreate a tag — use a new version instead
- If CI already released the current VERSION (tag exists), bump VERSION again and push
- Release notes pull from ALL commits since previous tag — keep commit messages clean
