# Claude Code skills (synced)

My personal Claude Code skills, versioned so they're available in **every project on every device**. Claude Code auto-discovers any `~/.claude/skills/<name>/SKILL.md`, so cloning this repo to `~/.claude/skills` makes all of them usable everywhere.

## Set up on a new device
```bash
cd ~/.claude
# if a skills/ folder already exists, move it aside first:  mv skills skills.bak
git clone <THIS_REPO_URL> skills
```
Then **restart Claude Code** so the new skills load (the skill list is gathered at launch).

## Add or update a skill
```bash
# add a new <name>/SKILL.md folder, then:
cd ~/.claude/skills
git add -A && git commit -m "add <name>" && git push
```
On the other device: `git -C ~/.claude/skills pull`, then restart Claude Code.

## Notes
- New/changed skills are picked up at the **start of a new session**, not mid-session.
- Agent definitions (frontmatter with `model`/`color`/"Use this agent when…") belong in `~/.claude/agents/`, not here — they won't work as skills.
