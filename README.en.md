# XiRang / 息壤

[中文](./README.md)

**AI-maintained second knowledge system for Obsidian**

> Capture everything. Let knowledge grow.

XiRang archives conversation attachments, pasted screenshots, web clippings, manual drops, and agent chat logs into a raw inbox. AI Agents auto-register, classify, source-note, reconcile conflicts, distill knowledge cards, archive originals, and commit to Git.

**New in this version**: Knowledge conflict detection & reconciliation, automatic knowledge card distillation, agent chat auto-capture, safe archival with Git version tracking.

### Key Features

- **Persona layer** — per-project user profiling (preferences, patterns, pitfalls), inspired by TencentDB Agent Memory's four-tier memory pyramid
- **Knowledge reconciliation** — auto-detect factual conflicts between new and existing sources, mark stale/outdated entries
- **Knowledge card distillation** — auto-distill structured knowledge cards when a domain accumulates ≥3 source notes
- **Self-optimizing skills** — weekly cron reviews conversations, distills missing rules into `skills/`, auto-improves over time

## How it works

```
paste screenshot / upload file / save web page / agent chat ends
       ↓
  raw/inbox/{cowork,screenshot,webclip,agent_chat}
       ↓
  AI Agent: register → classify → source note → reconcile → card → MOC → Persona
       ↓
  archive raw/archived/ → Git commit
```

## Setup

Place repo files in your Obsidian vault root.

**Claude Code**: `/goal follow "/path/to/vault/GOAL.md"`

**Other agents**: paste GOAL.md content to the agent and ask it to execute in vault root.

## Inbox structure

```
vault/
├── raw/
│   ├── inbox/{00_webclip,20_cowork,30_screenshot,40_manual,50_to_review,70_agent_chat}/
│   └── archived/YYYY-MM/  # Post-process archive (Git tracked)
├── wiki/{source-notes,persona,cards,moc}
├── logs/source-registry.csv  # with status + confidence columns
├── logs/reports/             # cron reports (daily/weekly/monthly/quarterly/annual + reconciliation)
├── skills/                   # modular AI operation rules
├── AGENTS.md          ← AI rules (core)
├── CLAUDE.md          ← points to AGENTS.md
└── GOAL.md
```

## Screenshot ingestion

Primary path: paste/upload screenshot into chat → AI saves to `30_screenshot/`, extracts visible content, creates source note.

If agent can't access attachment bytes, it must report that honestly.

## Web Clipper

Install [Obsidian Web Clipper](https://obsidian.com/clipper), configure:

- Note location: `raw/inbox/00_webclip`
- Note name: `{{date|date:"YYYY-MM-DD"}} - {{title}}`

Suggested frontmatter: 