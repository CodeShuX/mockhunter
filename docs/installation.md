# Installation

Detailed installation steps for MockHunter. For the quick path, see [README.md](../README.md#installation).

## Prerequisites

### 1. Claude Code

MockHunter is a Claude Code skill. You need [Claude Code](https://claude.com/claude-code) installed.

```bash
# Verify Claude Code is installed
claude --version
```

If not installed, follow the [official install guide](https://docs.claude.com/en/docs/claude-code).

### 2. Playwright MCP

MockHunter uses Playwright MCP to drive a real browser. Most Claude Code users already have it configured.

**Verify:** in any Claude Code session, ask: *"Can you open a browser via Playwright?"* — if Claude can do it, you're set.

**If not configured:** add the Playwright MCP server to your Claude Code config. See [microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp) for setup. The minimal config is roughly:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

Add to `~/.claude/settings.json` under `mcpServers`. Restart Claude Code after editing.

## Install MockHunter

### Option A — Symlink (recommended)

```bash
git clone https://github.com/CodeShuX/mockhunter.git ~/mockhunter
mkdir -p ~/.claude/skills
ln -s ~/mockhunter/skill/SKILL.md ~/.claude/skills/mockhunter.md
```

Pros: easy to update with `cd ~/mockhunter && git pull`. The symlink stays valid.

### Option B — Copy

```bash
git clone https://github.com/CodeShuX/mockhunter.git /tmp/mockhunter
mkdir -p ~/.claude/skills
cp /tmp/mockhunter/skill/SKILL.md ~/.claude/skills/mockhunter.md
rm -rf /tmp/mockhunter
```

Pros: no symlink. Cons: must repeat to update.

### Option C — Project-local

If you want MockHunter only for a single project:

```bash
cd /path/to/your/project
mkdir -p .claude/skills
git clone https://github.com/CodeShuX/mockhunter.git .mockhunter-source
ln -s .mockhunter-source/skill/SKILL.md .claude/skills/mockhunter.md
echo ".mockhunter-source/" >> .gitignore
```

## Verify

In any Claude Code session, type:

```
/mockhunter:mockhunter
```

If MockHunter is installed, the skill will start and ask for a target URL. If nothing happens, see [Troubleshooting](#troubleshooting).

## Updating

```bash
cd ~/mockhunter
git pull
```

If you used the copy method (Option B), repeat the install.

## Uninstall

```bash
rm ~/.claude/skills/mockhunter.md
rm -rf ~/mockhunter
```

## Troubleshooting

### `/mockhunter:mockhunter` does nothing

- Restart Claude Code (the skill list is loaded at startup)
- Verify the symlink exists: `ls -la ~/.claude/skills/mockhunter.md`
- Verify the source file exists: `cat ~/mockhunter/skill/SKILL.md | head`

### Playwright not opening browser

- Check Playwright MCP is configured (see Prerequisites above)
- Ask Claude directly: *"Use Playwright to open https://example.com"* — if that fails, the MCP isn't wired up

### "Permission denied" on the skill file

```bash
chmod 644 ~/mockhunter/skill/SKILL.md
```

### Symlink breaks after moving the source folder

Re-create it:

```bash
rm ~/.claude/skills/mockhunter.md
ln -s /new/path/to/mockhunter/skill/SKILL.md ~/.claude/skills/mockhunter.md
```

### MockHunter takes >10 minutes

That's within v0.1.0's expected range for complex pages. To speed up:
- Audit one tab at a time (tell the skill to focus on a specific section)
- Skip Phase 3 (test interactivity) if you only care about provenance
- Provide DB connection upfront — it skips guesswork

If consistently >20 minutes on simple pages, file an issue with the URL and a description of the page.
