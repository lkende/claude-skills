# claude-skills

A collection of Claude Code skills.

## Skills

| Skill | Description |
|---|---|
| [advanced-fullpage-screenshot](advanced-fullpage-screenshot/SKILL.md) | Full-page screenshot via CSS translateY + Chrome DevTools, with lazy-load pre-scroll, animation freezing, and Pillow stitching |

---

## Installation

### 1. Clone the repo

```bash
git clone https://github.com/lkende/claude-skills.git ~/.claude/skills/lkende-claude-skills
```

### 2. Symlink the skill(s) you want

```bash
ln -s ~/.claude/skills/lkende-claude-skills/advanced-fullpage-screenshot ~/.claude/skills/advanced-fullpage-screenshot
```

### 3. Use it

```
/advanced-fullpage-screenshot https://example.com
```

The stitching step runs an inline Python script via heredoc (`python3 << ...`). Claude will prompt for approval the first time it runs — this is the recommended default.

If you'd prefer to pre-authorize it in a specific project without being prompted, add this to that project's `.claude/settings.local.json`:

```json
{
  "allowedTools": [
    "Bash(python3 << *)"
  ]
}
```

Do not add this to your global `~/.claude/settings.json` — that would pre-authorize heredoc Python execution across all projects.

---

## Requirements

- [Claude Code](https://claude.ai/code)
- Chrome with the [Claude Code Chrome DevTools MCP](https://chromewebstore.google.com/detail/claude-code/...) extension
- Python 3 (Pillow auto-installs on first run)
