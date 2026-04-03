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
git clone https://github.com/lkende/claude-skills.git ~/Code/claude-skills
```

### 2. Symlink the skill(s) you want

```bash
ln -s ~/Code/claude-skills/advanced-fullpage-screenshot ~/.claude/skills/advanced-fullpage-screenshot
```

### 3. Add the required permission

The stitching step runs an inline Python script via heredoc. Add this to your `~/.claude/settings.json` under `allowedTools`:

```json
{
  "allowedTools": [
    "Bash(python3 << *)"
  ]
}
```

If `allowedTools` already exists, add `"Bash(python3 << *)"` to the array.

### 4. Use it

```
/advanced-fullpage-screenshot https://example.com
```

---

## Requirements

- [Claude Code](https://claude.ai/code)
- Chrome with the [Claude Code Chrome DevTools MCP](https://chromewebstore.google.com/detail/claude-code/...) extension
- Python 3 (Pillow auto-installs on first run)
