---
name: document
description: >
  Document code changes across the three standard files immediately after making them.
  Use after every feature, fix, or refactor — before committing or moving to the next task.
  Triggers: "update docs", "document this", "update notes", /document.
---

# /document

Document the change just made. Do this immediately — not at session end.

## File roles

| File | Role | When to update |
|------|------|---------------|
| `CLAUDE.md` (in repo) | Instructions for Claude — how to work in this repo | Always |
| `notes/projects/<project>.md` | Source of truth — full reference, architecture, version history, roadmap | Always |
| `README.md` | User-facing — new tools, endpoints, deployment steps | Only if user-facing change |

## CLAUDE.md — keep minimal

Only include what Claude needs to **act**:
- Stack
- File/directory structure
- Env vars
- Deploy command ("after changes")
- MCP tool names (no descriptions — cortex.md has those)

Do NOT copy architecture detail, endpoint tables, roadmap, or version history into CLAUDE.md. If Claude needs more context, it calls `search_notes("<project>")`.

## notes/projects/<project>.md — source of truth

Update:
- `version:` frontmatter field
- Any changed tools, endpoints, architecture, or config
- Add entry to `### Done ✅` roadmap section: `- <what> (vX.Y.Z) — <one line why/what>`

Then push:
```bash
cd /root/projects/notes && git add projects/<project>.md && git commit -m "<project> vX.Y.Z: <summary>" && git push
```

## Version bump rules

Bump `VERSION` file (or equivalent) in the repo on every merged change:

| Change type | Bump |
|-------------|------|
| Bug fix, refactor, docs, config, small improvement | **Patch** (x.y.Z) |
| New tool, new endpoint, new UI feature, new indexing capability | **Minor** (x.Y.0) |
| Breaking API change, schema migration requiring re-index, auth model change | **Major** (X.0.0) |

## Checklist

1. Determine change type → pick version bump
2. Bump `VERSION` in repo
3. Update `CLAUDE.md` — minimal, instructions only
4. Update `notes/projects/<project>.md` — version frontmatter + tools/config + roadmap Done entry
5. Update `README.md` only if user-facing
6. Push notes to git
7. Commit repo changes (VERSION + CLAUDE.md) with the version in the message
