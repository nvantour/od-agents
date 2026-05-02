# od-agents

Claude Code skills en agents voor het FACT-ACT experiment proces van Online Dialogue.

## Installatie

```bash
git clone https://github.com/nvantour/od-agents.git
```

Voeg het pad toe aan je Claude Code settings (`~/.claude/settings.json`):

```json
{
  "skillsDirectories": ["/pad/naar/od-agents/skills"]
}
```

## Structuur

```
od-agents/
├── skills/       # Claude Code skills (slash commands)
└── agents/       # Claude subagents
```

## Skills per FACT-ACT stap

| Stap | Skill | Omschrijving |
|------|-------|--------------|
| Analyze (voor) | `analyze-before-hypothese` | Hypothese schrijven in Als-Dan-Omdat format |
| Create | `create-figma-analyse` | Figma design analyseren voor A/B test |
| Create | `create-ab-test-init` | Nieuw experiment volledig opzetten |
| Create | `create-dom-inspect` | DOM inspecteren en selectors documenteren |
| Create | `create-ab-test-development` | Variatiecode schrijven (JS, tracking, CSS) |
| Analyze (na) | `analyze-after-resultaten` | Testresultaten analyseren en concluderen |

## Updates

```bash
git pull
```
