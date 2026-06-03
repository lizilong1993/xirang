# XiRang / 息壤

[中文](./README.md)

**AI-maintained second knowledge system for Obsidian**

> Capture everything. Let knowledge grow.

XiRang archives conversation attachments, pasted screenshots, web clippings, and manual drops into a raw inbox. AI Agents auto-register, classify, source-note, and maintain MOCs.

### Key Features

- **Persona layer** — per-project user profiling (preferences, patterns, pitfalls), inspired by TencentDB Agent Memory's four-tier memory pyramid
- **Self-optimizing skills** — weekly cron reviews conversations, distills missing rules into `skills/`, auto-improves over time

## How it works

```
paste screenshot / upload file / save web page
       ↓
  raw/inbox/{cowork,screenshot,webclip}
       ↓
  AI Agent: register → classify → source note → MOC → Persona
```

## Setup

Place repo files in your Obsidian vault root.

**Claude Code**: `/goal follow "/path/to/vault/GOAL.md"`

**Other agents**: paste GOAL.md content to the agent and ask it to execute in vault root.

## Inbox structure

```
vault/
├── raw/inbox/00_webclip/     # Web Clipper saves
├── raw/inbox/20_cowork/      # Files from chat
├── raw/inbox/30_screenshot/  # Pasted screenshots
├── raw/inbox/40_manual/      # Manual drops
├── raw/inbox/50_to_review/   # Awaiting review
├── wiki/{source-notes,persona,cards,moc}
├── logs/source-registry.csv
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

Suggested frontmatter: `capture_channel`, `processing_status`, `domain`, `project`, `source_url`, `captured_at`.

## Agent behavior

| You do | AI responds |
|--------|-------------|
| Upload file in chat | Save to `20_cowork/`, register, source note |
| Paste screenshot | Save to `30_screenshot/`, extract info, source note |
| Web Clipper saves page | Scan `00_webclip/`, classify, source note |
| Ask a question | Search vault first, then web or model knowledge |

## Credits

- **Andrej Karpathy** — LLM Wiki concept
- **TencentDB Agent Memory** — Four-tier memory pyramid inspiring Persona layer
- [claude-obsidian](https://github.com/AgriciDaniel/claude-obsidian)
- [obsidian-skills](https://github.com/kepano/obsidian-skills)

## License

MIT
