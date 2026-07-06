# XiRang / 息壤

[中文](./README.zh.md)

**AI-maintained second knowledge system for Obsidian**

> Capture everything. Let knowledge grow.

Drop files, paste screenshots, save web pages, or let your agents sync their conversations — XiRang routes everything through a pipeline that registers, classifies, distills, and archives your knowledge. No Obsidian plugins needed.

**Modular design.** Core gives you a working ingest pipeline. 8 optional modules (Entity System, Knowledge Cards, Multi-Agent Sync, Scheduled Pipelines, Knowledge Radar, Knowledge Interview, Memory System, Paper Discovery) can be added during setup. The agent asks what you need, then builds it.

## How it works

```
paste screenshot / upload file / save web page / agent chat ends
       ↓
  raw/inbox/{cowork,screenshot,webclip,agent_chat}
       ↓
  AI Agent: register → classify → source note → Persona → MOC → archive
       ↓  (modules add: entity split → card → radar → interview → ...)
  archive raw/archived/ → Git commit
```

### Core

- **Auto-ingest** — receive → QC → register → source note → Persona → MOC → archive → Git, zero clicks
- **Persona layer** — per-project profiling (preferences, patterns, pitfalls), updated on every intake
- **Retrieval-first** — answers draw from: cards → persona → source-notes → registry → memory → web
- **Safe archival** — raw/archived/ is immutable, Git-tracked

### Optional modules

| Module | What it does |
|--------|-------------|
| Entity System | Split entities from source notes into wiki articles, dual-axis classify, force wikilinks |
| Knowledge Cards | ≥3 same-domain sources → structured card with evidence table + conflict tracking |
| Multi-Agent Sync | Auto-sync sessions from Claude Code / Codex / Reasonix / ChatGPT and more |
| Scheduled Pipelines | Daily sync+extract+report, weekly distill+health-check+skill review |
| Knowledge Radar | Cross-project tool & tech recommendations |
| Knowledge Interview | Detect knowledge gaps and ask targeted questions |
| Memory System | Short-term project briefs + long-term distill + cross-project persistent context |
| Paper Discovery | Detect DOI/arXiv/PMID and fetch full-text via multi-tool pipeline |

## Setup

Copy repo files into your Obsidian vault root.

**Claude Code**:

```bash
/goal follow "/path/to/your/vault/GOAL.md"
```

**Other agents** (Codex, Cowork, Reasonix): paste GOAL.md content into chat and ask the agent to run it in vault root.

## Core inbox structure

Modules add extra dirs (`wiki/articles/`, `wiki/radar/`, `schedule/`). See GOAL.md.

```
vault/
├── raw/
│   ├── inbox/{00_webclip,20_cowork,30_screenshot,40_manual,50_to_review,70_agent_chat}/
│   └── archived/YYYY-MM/
├── wiki/source-notes/
├── wiki/persona/
├── wiki/moc/
├── wiki/index.md
├── logs/source-registry.csv
├── logs/reports/
├── skills/
├── AGENTS.md           ← AI rules
├── CLAUDE.md           ← points to AGENTS.md
└── GOAL.md
```

## Screenshot ingestion

Paste or upload screenshots in chat. Agent saves to `30_screenshot/`, extracts visible state (errors, paths, UI), writes a source note.

If it can't read attachment bytes, it says so — no pretending.

## Web Clipper

Install [Obsidian Web Clipper](https://obsidian.com/clipper), set:

- Note location: `raw/inbox/00_webclip`
- Note name: `{{date|date:"YYYY-MM-DD"}} - {{title}}`

Frontmatter: `capture_channel`, `processing_status`, `domain`, `project`, `source_url`, `captured_at`.

## Agent behavior (Core)

| You do | Agent does |
|--------|-----------|
| Upload a file | Save to `20_cowork/`, register, source note, archive |
| Paste a screenshot | Save to `30_screenshot/`, extract info, source note, archive |
| Web Clipper saves a page | Scan `00_webclip/`, classify, source note, archive |
| Drop files into inbox | Scan, process, archive, Git commit |
| Ask a question | Search vault → memory → web → model knowledge (tagged) |

Modules extend the pipeline: A adds entity split after source notes, B distills cards when ≥3 sources accumulate, C syncs multi-agent sessions, etc.

## Credits

- **Andrej Karpathy** — LLM Wiki concept
- **TencentDB Agent Memory** — memory pyramid inspiring the Persona layer
- **isEris** — comparison of Obsidian plugin vs IDE+API distillation approaches
- [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian) — upstream reference
- [obsidian-skills](https://github.com/kepano/obsidian-skills) — upstream reference
- **[Reasonix](https://reasonix.ai)** — agent orchestration framework, used for daily pipeline
- **DeepSeek** — GOAL.md modularization and agent interaction design
- Long-term production iteration — knowledge reconciliation, card distillation, agent chat capture, safe archival

## License

MIT
