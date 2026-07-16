# 🛡️ OpenClaw Backup Guide

**A battle-tested, opinionated backup strategy for OpenClaw installations.**

Born from running OpenClaw on a Linux Mint box with a 4-tier backup strategy that has saved our bacon more than once. Now shared with the community so your setup survives disk failures, ransomware, fat fingers, and that one time you ran `rm -rf` in the wrong directory.

---

## Why Backup?

Because bad things happen to good servers:

- 💀 **"I accidentally deleted my workspace."** — Happens more often than anyone admits.
- 🔥 **"My SSD died."** — SSDs don't warn you. They just stop.
- 🏠 **"My apartment flooded."** — Your NAS doesn't help if it's underwater too.
- 🤦 **"I ran a migration that corrupted my database."** — SQLite is robust, but not immune to bad code.
- 🦠 **"Ransomware encrypted everything."** — If your backups are on the same machine, they're encrypted too.

**The good news:** A basic backup takes 5 minutes to set up. You can get fancy later.

---

## Quick Start: Git Backup in 5 Minutes

Don't have time for the full guide? Just do this. It's 10x better than nothing.

```bash
# 1. Clone this repo
git clone https://github.com/your-org/openclaw-backup-guide.git
cd openclaw-backup-guide

# 2. Copy the config
cp .env.example .env
# Edit .env — set OPENCLAW_HOME and OPENCLAW_WORKSPACE

# 3. Copy the backup script to your workspace
cp scripts/common/backup-db.js ~/hub-local/
cp scripts/linux/backup-git.sh ~/hub-local/

# 4. Make it executable
chmod +x ~/hub-local/backup-git.sh

# 5. Add a cron job (runs every hour)
(crontab -l 2>/dev/null; echo "0 * * * * cd ~/hub-local && ./backup-git.sh >> ~/logs/openclaw-backup/git.log 2>&1") | crontab -

# 6. Make sure your workspace is a git repo with a remote
cd ~/hub-local
git remote -v  # Should show your GitHub/GitLab remote
```

**That's it.** Every hour, your OpenClaw config, database snapshots, and workspace get committed and pushed. You now have version history and offsite backup via GitHub.

Want more protection? Keep reading.

---

## The 4-Tier Philosophy

We use four layers of backup, each protecting against different failure modes:

| Tier | What | How Often | Protects Against | Recovery Time |
|------|------|-----------|-----------------|---------------|
| **1. Git** | Config + DB snapshots | Hourly | Accidental changes, bad configs | Minutes |
| **2. File-level** | Full dedup'd incrementals | 3x daily | File corruption, deletions | 30 min |
| **3. System Image** | Bare-metal snapshot | Monthly | Disk failure, OS corruption | 1-2 hours |
| **4. Offsite** | Copy everything off-machine | Continuous | Fire, theft, ransomware | Hours to days |

**You don't need all four.** Tier 1 alone is a massive improvement over nothing. Each tier you add reduces risk further. Pick what fits your paranoia level.

📖 [Deep dive: The 4-Tier Philosophy →](docs/backup-tiers.md)

---

## Platform Guides

| Platform | Tier 1 (Git) | Tier 2 (Files) | Tier 3 (Image) | Tier 4 (Offsite) | Guide |
|----------|:---:|:---:|:---:|:---:|-------|
| **Linux (Local)** | ✅ git | ✅ Borg | ✅ fsarchiver | ✅ NAS + Cloud | [linux-local.md](docs/linux-local.md) |
| **Linux (VPS)** | ✅ git | ✅ Borg | ⚠️ Provider snapshots | ✅ B2/rsync.net | [linux-vps.md](docs/linux-vps.md) |
| **macOS (Mac Mini)** | ✅ git | ✅ restic | ✅ Time Machine | ✅ External + Cloud | [macos.md](docs/macos.md) |
| **Windows** | ✅ git | ✅ robocopy | ✅ Veeam Agent | ✅ External + Cloud | [windows.md](docs/windows.md) |

---

## What to Backup

Every OpenClaw installation has these critical pieces:

| Item | Path (typical) | Why It Matters |
|------|---------------|----------------|
| **OpenClaw home** | `~/.openclaw/` | Core config, credentials, state |
| **Workspace** | `~/hub-local/` (or your chosen dir) | Your projects, scripts, customizations |
| **SQLite databases** | Inside `~/.openclaw/` | Chat history, memory, everything |
| **Environment files** | `.env`, credentials | API keys, secrets |
| **Custom scripts** | Various | Your automations |

⚠️ **SQLite databases need special handling.** You can't just copy a SQLite file while it's in use — you'll get a corrupted backup. Our scripts use SQLite's `.backup()` API to create a safe snapshot first. This is non-negotiable.

---

## Repository Structure

```
scripts/
├── common/          # Cross-platform scripts
│   ├── backup-db.js       # Safe SQLite backup (Node.js)
│   ├── verify-backup.sh   # Verify backup integrity
│   └── restore-db.js      # Restore SQLite from backup
├── linux/           # Linux-specific
│   ├── backup-git.sh      # Git commit + push
│   ├── backup-borg.sh     # Borg create + prune
│   ├── backup-nas.sh      # Tar + SCP to NAS
│   ├── backup-image.sh    # fsarchiver full disk
│   └── setup-cron.sh      # Install all cron jobs
├── macos/           # macOS-specific
│   ├── backup-git.sh      # Git commit + push
│   ├── backup-restic.sh   # restic backup + prune
│   ├── backup-timemachine.sh  # Time Machine helpers
│   ├── setup-launchd.plist    # launchd config
│   └── setup.sh              # One-command setup
└── windows/         # Windows-specific
    ├── backup-git.ps1     # PowerShell git backup
    ├── backup-robocopy.ps1    # File-level backup
    ├── backup-veeam.ps1       # Veeam image backup
    └── setup-taskscheduler.ps1  # Task Scheduler setup
```

---

## The Most Important Document

👉 **[How to Restore →](docs/restore.md)** 👈

A backup you can't restore is not a backup. Read this before you need it.

---

## Offsite Strategies

Don't keep all your eggs on one machine. [Offsite backup guide →](docs/offsite.md)

---

## Cloud Backup Services

If you want encrypted cloud backup without managing your own offsite infrastructure, these services handle Tier 4 for you:

| Service | What It Does | Encryption | Pricing |
|---------|-------------|------------|---------|
| **[Keep My Claw](https://keepmyclaw.com)** | Automated encrypted cloud backup to Cloudflare R2. Backs up workspace, memory, cron jobs, skills, credentials, and config snapshots. Client-side AES-256 encryption (zero-knowledge). Install via `clawhub install keepmyclaw`. | Client-side AES-256 | $5/mo or $19/yr |

These complement the local backup tiers above. Local backups protect against accidental changes and corruption. Cloud backups protect against the scenarios where your entire machine is gone.

---

## Contributing

Found a better way? Running on a platform we haven't covered? PRs welcome.

- Keep the tone practical and opinionated
- Test your scripts before submitting
- Include error handling — real backups fail sometimes
- Update the platform matrix if adding a new platform

## License

[MIT](LICENSE) — Use it, fork it, improve it.

---

*Built by [Lance](https://x.com/laurence_stone) and [Hub](https://github.com/lancelot3777-svg) 🌀*
*A human-AI partnership that actually works.*
