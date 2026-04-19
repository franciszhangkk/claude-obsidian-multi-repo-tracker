# claude-obsidian-multi-repo-tracker

[中文](README.md) | [English](README-en.md)

> A Claude Code Skill that turns your Obsidian vault into a project-tracking knowledge base across multiple code repositories.

**Core idea**: Make Claude Code read Obsidian *before* it starts working — instead of re-scanning your codebase from scratch every session. Documentation = context cache.

---

## Why this exists

When you maintain multiple code repositories at once (frontend, backend, shared proto library, scaffolding, ...), Claude Code wastes thousands of tokens every session re-discovering the architecture, finding entry points, and re-learning cross-repo dependencies. Worse, the answers it gives are inconsistent.

What this skill solves:

- ✅ **Each project has a "live document"** (architecture, current focus, progress) — Claude reads ~5 KB and is ready to work
- ✅ **Cross-repo dependencies in one place** (vault homepage as central index)
- ✅ **Code → docs auto-sync** (`/sync-docs`, `/commit`, `/pull`)
- ✅ **Global work log** automatically records what you did each day
- ✅ **Inspired by Cline Memory Bank**, but supports multi-project + cross-repo

---

## How it differs from other approaches

| Project | Single project | Cross-repo | Code↔docs sync | Chinese templates | Codex compatible |
|---------|:--------------:|:----------:|:--------------:|:-----------------:|:----------------:|
| **This project** | ✅ | ✅ | ✅ | ✅ | ⚠️ Partial |
| [Cline Memory Bank](https://docs.cline.bot/features/memory-bank) | ✅ | ❌ | ❌ | ❌ | ❌ |
| [obsidian-second-brain](https://github.com/eugeniughelbur/obsidian-second-brain) | ✅ | ❌ | ❌ | ❌ | ❌ |
| [claude-obsidian (Karpathy LLM Wiki)](https://github.com/AgriciDaniel/claude-obsidian) | N/A | ❌ | ❌ | ❌ | ❌ |
| [claude-code-memory-setup](https://github.com/lucasrosati/claude-code-memory-setup) | ✅ | ❌ | ⚠️ | ❌ | ❌ |

> Note: "Chinese templates" means the bundled templates ship in Chinese by default. The skill itself is **language-agnostic** — Go, Python, Rust, JS, anything works. Templates can be translated.

---

## What this is *not*

- ❌ **Not** a second brain / Zettelkasten / PARA system — it only tracks software projects, not your life notes
- ❌ **Not** RAG / vector retrieval — pure markdown, Claude reads it directly
- ❌ **Not language-locked** — Go / Python / Rust / JS all work; templates contain no language-specific assumptions

---

## Quick start (5 minutes)

### 1. Install the skill

```bash
git clone https://github.com/franciszhangkk/claude-obsidian-multi-repo-tracker.git \
  ~/.claude/skills/claude-obsidian-multi-repo-tracker
```

### 2. Initialize your Obsidian vault

In Claude Code:

```
/init-vault
```

It will ask for your vault path, then generate:

```
<your vault>/
├── 首页.md (Homepage)              # Cross-project index
├── 开发工作流指南.md (Workflow Guide)
├── CLAUDE.md                        # Global instructions for Claude
├── AGENTS.md                        # Global instructions for Codex (optional)
└── 项目/ (Projects)                 # Project documentation directory
```

### 3. Add your code projects

```
/add-project ai-agents ~/code/ai-agents
```

This generates a template under `项目/ai-agents/` (overview, activeContext, progress, plus five category folders), and writes a `CLAUDE.md` in your code repo pointing back to this documentation.

### 4. Daily usage

| Scenario | Command |
|----------|---------|
| Sync code changes to docs | `/sync-docs` |
| Commit code + sync docs | `/commit` |
| Pull code + update docs | `/pull` |
| Update current focus | `/update-active` |
| Weekly review (digest journal into progress) | `/weekly-digest` |

---

## Codex users

You can use the vault structure + `AGENTS.md`, but commands and hooks are Claude Code only. See [docs/codex-compatibility.md](docs/codex-compatibility.md).

---

## Full documentation

- [INSTALL.md](INSTALL.md) — Detailed installation and configuration
- [SKILL.md](SKILL.md) — Skill trigger conditions and command list
- [docs/对比.md](docs/对比.md) — Detailed comparison with other approaches (Chinese)
- [docs/设计原则.md](docs/设计原则.md) — Design principles, why it's built this way (Chinese)
- [examples/demo-vault/](examples/demo-vault/) — A minimal example vault

---

## Contributing

PRs and issues welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). The project is intentionally small in scope (project tracking only, not a second brain), so feature requests should align with that.

## Acknowledgments

- Conceptual roots: [Andrej Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- Structural reference: [Cline Memory Bank](https://docs.cline.bot/features/memory-bank)
- Automation inspiration: [Self-Evolving Memory with Hooks](https://www.mindstudio.ai/blog/self-evolving-claude-code-memory-obsidian-hooks)

## License

MIT
