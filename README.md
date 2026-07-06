# XiRang / 息壤

[中文](./README.zh.md)

**AI-maintained second knowledge system for Obsidian**

> Capture everything. Let knowledge grow.

XiRang is a modular, AI-maintained second knowledge system for Obsidian. Drop files, paste screenshots, save web clippings, or let your AI agents sync their conversations — everything flows through a pipeline that registers, classifies, distills, and archives your knowledge automatically.

XiRang uses a **modular design**: Core provides a minimal ingest pipeline, and 8 optional modules (Entity System, Knowledge Cards, Multi-Agent Sync, Scheduled Pipelines, Knowledge Radar, Knowledge Interview, Memory System, Paper Discovery) can be enabled on demand. During initialization, the agent guides you through module selection and asks only what it needs to know.

## Why XiRang

Most knowledge vaults grow into graveyards — collected but never curated. Plugin-based approaches (Templater + Text Generator + Dataview + Copilot) suffer from silent batch failures, cryptic errors, and complex configuration. XiRang bypasses all that: AI agents call LLM APIs directly, no Obsidian plugins required.

## How it works

```
paste screenshot / upload file / save web page / agent chat ends
       ↓
  raw/inbox/{cowork,screenshot,webclip,agent_chat}
       ↓
  AI Agent: register → classify → source note → Persona → MOC → archive
       ↓  (optional modules extend: entity split → card → radar → interview → ...)
  archive raw/archived/ → Git commit
```

### Core (always enabled)

- **Auto-ingest pipeline** — receive → QC filter → register → source note → Persona update → MOC → archive → Git, zero manual steps
- **Persona layer** — per-project user profiling (preferences, patterns, pitfalls) updated on every intake
- **Retrieval-first** — answer questions by searching: cards → persona → source-notes → registry → memory → web
- **Safe archival** — raw/archived/ is an immutable evidence chain, Git-tracked

### Optional modules (choose during init)

| Module | Description |
|--------|-------------|
| Entity System | Auto-split entities from source notes into wiki articles, dual-axis classification, wikilink enforcement |
| Knowledge Cards | ≥3 same-domain sources → structured card with evidence table and conflict tracking |
| Multi-Agent Sync | Auto-sync sessions from Claude Code / Codex / Reasonix / ChatGPT and more |
| Scheduled Pipelines | Daily sync+extract+report, weekly distillation+health-check+skill optimization |
| Knowledge Radar | Cross-project tool/technology recommendation engine |
| Knowledge Interview | Proactively discover knowledge gaps and ask targeted questions |
| Memory System | Short-term project briefs + long-term distillation + cross-project persistent context |
| Paper Discovery | Auto-detect DOI/arXiv/PMID and fetch full-text via multi-tool pipeline |

## Setup

Place the repo files in your Obsidian vault root.

**Claude Code**:

```bash
/goal follow "/path/to/your/vault/GOAL.md"
```

**Other agents** (Codex, Cowork, Reasonix, etc.): paste GOAL.md content to the agent and ask it to execute in your vault root.

## Core inbox structure

Optional modules add extra directories (e.g. `wiki/articles/`, `wiki/radar/`, `schedule/`). See GOAL.md for details.

```
vault/
├── raw/
│   ├── inbox/{00_webclip,20_cowork,30_screenshot,40_manual,50_to_review,70_agent_chat}/
│   └── archived/YYYY-MM/  # Post-process archive (Git tracked)
├── wiki/source-notes/  # Lightweight source index + citation
├── wiki/persona/       # Dynamic-category user profiles
├── wiki/moc/           # Maps of Content
├── wiki/index.md       # Vault index
├── logs/source-registry.csv  # with status + confidence
├── logs/reports/       # Report directory
├── skills/             # Modular AI operation rules
├── AGENTS.md           ← AI rules (entry point)
├── CLAUDE.md           ← points to AGENTS.md
└── GOAL.md
```

## Screenshot ingestion

Paste or upload screenshots directly in chat. The agent saves to `raw/inbox/30_screenshot/`, extracts visible content (UI state, errors, paths), and creates a source note.

If the agent cannot access attachment bytes, it must report that honestly — no pretending.

## Web Clipper

Install [Obsidian Web Clipper](https://obsidian.com/clipper) browser extension with:

- Note location: `raw/inbox/00_webclip`
- Note name: `{{date|date:"YYYY-MM-DD"}} - {{title}}`

Suggested frontmatter: `capture_channel: webclip`, `processing_status: new`, `domain`, `project`, `source_url`, `captured_at`.

## Agent behavior (Core)

| You do | AI responds |
|--------|-------------|
| Upload a file in chat | Save to `20_cowork/`, register, source note, archive |
| Paste a screenshot | Save to `30_screenshot/`, extract info, source note, archive |
| Web Clipper saves a page | Scan `00_webclip/`, classify, source note, archive |
| Drop files into inbox | Scan, process, archive, Git commit |
| Ask a question | Search vault → memory → web → model knowledge (tagged) |

> Enabling optional modules extends the pipeline automatically: Module A adds entity split after source notes, Module B distills cards when ≥3 sources accumulate, Module C syncs multi-platform agent sessions, etc.

## Credits

- **Andrej Karpathy** — LLM Wiki concept inspiring the knowledge organization
- **TencentDB Agent Memory** — Four-tier memory pyramid inspiring the Persona layer
- **isEris** — Practical comparison of Obsidian plugin vs IDE+API script distillation approaches
- [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian) — Upstream reference
- [obsidian-skills](https://github.com/kepano/obsidian-skills) — Upstream reference
- **[Reasonix](https://reasonix.ai)** — Agent orchestration and knowledge registration framework, daily pipeline tool
- **DeepSeek** — GOAL.md modularization and agent interaction design assistance
- **Practice-driven** — Knowledge reconciliation, card distillation, agent chat capture, safe archival evolved from long-term production iteration

## License

MIT
