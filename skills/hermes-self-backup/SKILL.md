---
name: hermes-self-backup
description: "Securely back up and restore your entire Hermes Agent configuration, skills, cron jobs, and state."
version: 2.0.0
author: Ody Thomas / JARVIS
license: MIT
platforms: [linux, macos]
metadata:
  hermes:
    tags: [devops, backup, restore, hermes, migration, github, security]
    homepage: https://github.com/NousResearch/hermes-agent
    related_skills: [hermes-agent, hermes-auto-updater]
---

# Hermes Self-Backup

Back up your entire Hermes Agent identity — skills, configuration, cron jobs, profiles, and persona — to a **private GitHub repository**. Restore or clone to any machine in minutes.

## What gets backed up

| Item | How |
|---|---|
| `config.yaml` | Full config (no secrets) |
| `skills/` | All user-created and hub-installed skills |
| `profiles/` | Per-profile configs and skills |
| `cron/` | Scheduled job definitions |
| `scripts/` | Utility scripts |
| `.persona.md` | Custom persona file |
| Local secrets (optional) | **Encrypted with GPG** if you choose |

## What is NEVER backed up

- Raw secret files with API keys (unless you explicitly encrypt them)
- `auth.json` with OAuth tokens
- `sessions/` (transient, large)
- `hermes-agent/` source (reinstallable via `hermes update`)
- `logs/`

## Requirements

- `git` and `gh` CLI installed and authenticated (`gh auth login`)
- Hermes Agent installed
- Optional: `gpg` for encrypting secret files

## Quick Start

### 1. Install the skill

```bash
hermes skills install hermes-self-backup
```

### 2. Run setup

```bash
bash ~/.hermes/skills/devops/hermes-self-backup/scripts/setup.sh
```

This asks for a GitHub repo name and generates `hermes-backup.sh` and `hermes-restore.sh` in your `~/.hermes/scripts/` folder. It reuses your existing `gh` authentication — no extra tokens needed.

### 3. Back up

```bash
~/.hermes/scripts/hermes-backup.sh
```

First run creates a private GitHub repo and pushes all config/state. On subsequent runs it commits and pushes deltas.

### 4. Restore (on a new machine)

```bash
~/.hermes/scripts/hermes-restore.sh
```

You'll be prompted for any encrypted files and missing configuration.

## Security features

1. **Private repo default** — the backup repo is created as private.
2. **Explicit deny-list** — a `.gitignore` in the staging directory blocks all secret patterns.
3. **No silent overwrites** — restore prompts before replacing existing files.
4. **Live secret protection** — restore refuses to overwrite existing secret files unless you explicitly confirm.

## Automation

You can cron the backup so it runs automatically:

```bash
hermes cron create "0 2 * * 0" --name "hermes-weekly-backup" \
  --no-agent --script "hermes-backup.sh"
```

This backs up every Sunday at 2 AM.

## Files shipped

| File | Purpose |
|---|---|
| `scripts/setup.sh` | Interactive setup: generates backup/restore scripts locally |
| `SKILL.md` | This documentation |

After setup, these are generated in `~/.hermes/scripts/`:

| File | Purpose |
|---|---|
| `hermes-backup.sh` | Push Hermes state to GitHub |
| `hermes-restore.sh` | Pull Hermes state from GitHub |

## Troubleshooting

**"Not authenticated with gh"**
Run `gh auth login` and try again. For headless/CI use, set `GH_TOKEN` env var.

**"Push failed"**
Ensure your `gh` token has `repo` scope.

**Encrypted file won't decrypt**
You must use the **same passphrase** you used when encrypting. GPG does not store this passphrase.

## License

MIT
