# 📂 Vault Watcher Skill

[简体中文](./README.md) | 🌐 English

**AI Agent Skill for Obsidian vault file change monitoring → Feishu/Lark group notifications**

An AI-powered skill that guides you through installing and managing [Vault Watcher](https://github.com/manwithshit/vault-watcher) — just tell your AI agent what you want, and it handles the rest.

---

## What Problem Does This Solve?

If you have an AI Agent (like CoPaw) working in the background — organizing notes, archiving files, updating to-dos — you face a problem:

> **The AI says "done", but you have no idea what changed or where.**

Vault Watcher monitors your Obsidian vault and sends structured Feishu/Lark cards whenever files change. This **skill** wraps the entire setup into a conversational experience — no manual script editing required.

---

## 🚀 AI One-Click Install

> [!TIP]
> **Copy the prompt below and send it to your AI Agent (Claude Code / Cursor / Codex, etc.) — it will guide you through the entire installation.**
>
> ```
> Install vault-watcher for me.
> It's an Obsidian vault file change monitor that pushes notifications to Feishu/Lark.
> Project: https://github.com/manwithshit/vault-watcher-skill
> Guide me through the configuration (vault path, watch directories, Feishu credentials),
> then deploy it.
> ```
>
> If your agent has the vault-watcher skill installed, it will automatically enter the guided setup flow.

---

## What the Skill Does

| User Intent | Action |
|-------------|--------|
| "Install vault-watcher" | → Interactive setup wizard (7 steps) |
| "vault-watcher status" | → Check service, show logs |
| "Stop vault-watcher" | → Manage launchd service |
| "Change watch directory" | → Edit config and restart |
| "Uninstall vault-watcher" | → Clean removal |

## How It Works

```
fswatch (8s collect) → vault-watcher.sh (10s debounce + classify + diff) → vault-notify.py (Feishu card)
```

Managed by macOS `launchd`: auto-start on boot, auto-restart on crash.

Detects 5 event types: ➕ New / ✏️ Modified / 📦 Archived / 🗑️ Deleted / 📋 Diff details

## Skill Structure

```
vault-watcher-skill/
├── SKILL.md                  # Entry point: triggers + routing + workflow
├── scripts/
│   └── install.sh            # One-click installer (generates sh/py/plist, loads launchd)
└── references/
    ├── setup-guide.md        # Step-by-step config collection guide
    └── management.md         # Status / start / stop / uninstall / troubleshoot
```

## Requirements

- **macOS** (uses `launchd` + `fswatch`)
- **fswatch**: `brew install fswatch` (auto-installed during setup)
- **Python 3**: ships with macOS
- **Feishu/Lark app**: with `im:message` permission

## Core Features

| Feature | Description |
|---------|-------------|
| **5 event types** | New / Modified / Archived / Deleted / **Diff details** |
| **Diff tracking** | Shows exactly which lines were added/removed for a tracked file |
| **Smart dedup** | Multiple changes within a short window are merged into one notification |
| **Archive detection** | Distinguishes "moved to archive" from "truly deleted" |
| **iCloud noise filter** | Auto-ignores `.icloud` placeholders, temp files, etc. |
| **Zero dependencies** | Only `fswatch` (brew) + Python 3 + bash 3.2 |
| **Self-healing** | macOS `launchd` managed — auto-restarts on crash |
| **Privacy-first** | Runs locally — only file paths are sent to your own Feishu Bot |

## Limitations

- **macOS only** (depends on `launchd` + `fswatch`)
- **Diff tracks a single file** (the one you care most about)
- **bash 3.2 constraint** (macOS built-in version, no associative arrays)

---

## License

MIT
