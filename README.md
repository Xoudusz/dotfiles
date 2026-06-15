# dotfiles

Claude Code config, skills, and hooks for vibecode (192.168.68.107).

## Structure

```
claude/
  skills/
    grill-me/   # /grill-me — relentless design interview
    handoff/    # /handoff — session handoff doc generator
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
