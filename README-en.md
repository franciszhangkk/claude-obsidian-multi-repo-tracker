# claude-obsidian-multi-repo-tracker

[中文](README.md) | [English](README-en.md)

> A Claude Code Skill: CLAUDE.md as a lightweight router (~2K token), Obsidian as an on-demand content library — Claude loads just enough context for each task.

---

## The Problem

When you maintain multiple code repositories, Claude Code wastes thousands of tokens every session re-discovering architecture, finding entry points, and re-learning cross-repo dependencies.

The obvious fix — stuffing all project knowledge into CLAUDE.md — creates a new problem: **a 4K-10K token CLAUDE.md loads in full on every session, most of it irrelevant to the current task.**

---

## The Design

Two-layer separation:

| Layer | File | Loading | Token budget |
|-------|------|---------|-------------|
| Navigation | `CLAUDE.md` | Auto-loaded every session | ~2K (router + hard constraints) |
| Content | Obsidian sub-documents | Claude reads on-demand via route table | ~1-3K (only what the task needs) |

### CLAUDE.md = Route Table, Not a Knowledge Base

Seven sections with a clear purpose each:

```
1. Metadata         ← vault path, related projects
2. Project pitch    ← who uses it, what business problem it solves
3. Doc map          ← Obsidian sub-document index (/sync-docs auto-maintains)
4. Route table      ← user intent → must-read document (precise mapping)
5. Truth source     ← Obsidian is the map, code is the territory
6. Git conventions  ← /commit /pull first
7. Hard constraints ← rules Claude will break if not reminded every session
```

Architecture details, API docs, implementation patterns — all live in Obsidian, fetched via the route table.

### Obsidian = Layered Content Library

```
project/<name>/
├── 概览.md (Overview)      ← business context, tech stack, key modules
├── activeContext.md         ← current focus + last 5 commits (machine-maintained)
├── progress.md              ← completed features + decision log
├── 01-架构/ (Architecture)  ← architecture docs, cross-repo diagrams
├── 02-功能/ (Features)      ← feature module docs
├── 03-产品/ (Product)       ← product requirements
├── 04-参考资料/ (Reference) ← API docs, debug commands
└── 05-测试与调试/ (Testing) ← bug records, test cases
```

### Map vs Territory Principle

- **Obsidian is the map**: design intent, past decisions, current focus — things code can't tell you
- **Code is the territory**: function signatures, actual behavior, call chains — Obsidian can drift, always Read actual files before changing code
- **When they conflict**: fix small discrepancies in Obsidian immediately; for large ones, stop and ask the user

---

## How It Differs

| Approach | Multi-repo | Token efficiency | Code↔docs sync | Drift detection |
|----------|:----------:|:---------------:|:--------------:|:---------------:|
| **This project** | ✅ | ✅ route on demand | ✅ | ✅ `/update-memory` |
| [Cline Memory Bank](https://docs.cline.bot/features/memory-bank) | ❌ | ❌ reads 6 files every session | ❌ | ❌ |
| [obsidian-mind](https://github.com/breferrari/obsidian-mind) | ❌ | ❌ | ❌ | ❌ |
| [claude-code-memory-setup](https://github.com/lucasrosati/claude-code-memory-setup) | ❌ | ❌ | ⚠️ | ❌ |

---

## What This Is Not

- ❌ **Not** a second brain / Zettelkasten / PARA — project tracking only, not life notes
- ❌ **Not** RAG / vector retrieval — pure markdown, Claude reads directly
- ❌ **Not language-locked** — Go / Python / Rust / JS all work
- ❌ **No hooks** — commands are invoked explicitly; no PreToolUse / Stop hook automation

---

## Quick Start (5 minutes)

### 1. Install

```bash
git clone https://github.com/franciszhangkk/claude-obsidian-multi-repo-tracker.git \
  ~/.claude/skills/claude-obsidian-multi-repo-tracker
```

### 2. Initialize your vault

```
/init-vault
```

Prompts for your vault path, then generates the base structure (homepage, CLAUDE.md, workflow guide).

### 3. Add a project

```
/add-project ai-agents ~/code/ai-agents
```

Creates the three-file set (overview, activeContext, progress + five category dirs) in Obsidian, and writes a v0.5-structured CLAUDE.md in your code repo.

### 4. Daily workflow

| Scenario | Command |
|----------|---------|
| Commit code + sync docs + update focus | `/commit` |
| Pull code + update docs | `/pull` |
| Sync docs only | `/sync-docs` |
| Switch task / update current focus | `/update-active` |
| Full drift audit | `/update-memory` |

---

## All Commands

| Command | When | What it does |
|---------|------|-------------|
| `/init-vault` | First time | Initialize Obsidian vault structure |
| `/add-project` | New project | Generate templates + detect language + write CLAUDE.md |
| `/commit` | Every commit | Stage + Chinese message + sync-docs + **auto-infer focus draft** |
| `/pull` | Every pull | Stash-protected pull + batch doc sync |
| `/sync-docs` | After commits | Update activeContext recent changes + refresh doc map |
| `/update-active` | When switching tasks | Infer focus from git signals, one-tap confirm → activeContext |
| `/update-memory` | Weekly / post-feature | Full audit: Code ↔ Obsidian drift, decide per item |

---

## Codex Users

Vault structure + `AGENTS.md` work. Commands (`/commit`, `/pull`, `/sync-docs`) are Claude Code only. See [docs/codex-compatibility.md](docs/codex-compatibility.md).

---

## Docs

- [INSTALL.md](INSTALL.md) — Installation and configuration
- [SKILL.md](SKILL.md) — Skill trigger conditions
- [docs/设计原则.md](docs/设计原则.md) — Design principles (Chinese)
- [examples/demo-vault/](examples/demo-vault/) — Minimal example vault

---

## Changelog

**v0.5**: CLAUDE.md refactored to route table + 7-section skeleton; project pitch section added; recent changes moved to activeContext; token reduced ~60%

**v0.4**: Added doc map, route checklist, map-vs-territory principle, `/update-memory` command

**v0.3**: Implemented `/sync-docs` `/commit` `/pull`, full demo-vault, Layer 1 Git conventions

---

## Contributing

PRs and issues welcome — see [CONTRIBUTING.md](CONTRIBUTING.md). Scope is intentionally narrow (project tracking only), so feature requests should align with that.

## Acknowledgments

- Conceptual roots: [Andrej Karpathy's LLM Wiki](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- Structural reference: [Cline Memory Bank](https://docs.cline.bot/features/memory-bank)
- Automation inspiration: [Self-Evolving Memory with Hooks](https://www.mindstudio.ai/blog/self-evolving-claude-code-memory-obsidian-hooks)

## License

MIT
