# dotfiles

Claude Code config, skills, and hooks for vibecode (192.168.68.107).

## Structure

```
claude/
  skills/
    grill-me/        # /grill-me — relentless design interview
    handoff/         # /handoff — session handoff doc generator
    document/        # /document — post-change documentation checklist
    skill-creator/   # /skill-creator — scaffold new skills (init_skill.py)
    github-actions/  # /github-actions — CI/CD failure triage via gh CLI
    webapp-testing/  # /webapp-testing — Playwright UI testing
    release/         # /release — VERSION bump, CI flow, conventional commits
```

## Setup

```bash
git clone https://github.com/Xoudusz/dotfiles.git ~/dotfiles
ln -s ~/dotfiles/claude/skills ~/.claude/skills
```

## Adding skills

```bash
mkdir ~/dotfiles/claude/skills/<name>
# write SKILL.md
cd ~/dotfiles && git add -A && git commit -m "add skill: <name>" && git push
```
