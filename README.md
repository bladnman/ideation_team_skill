# ideation

Multi-agent ideation skill for [Claude Code](https://claude.com/claude-code). Launches a team of specialized agents that explore a concept through structured dialogue, then produces polished deliverables — briefs, a vision document, a presentation, a designed web page, and archival PDFs.

## Agents

| Agent | Role |
|-------|------|
| **Arbiter** | Team lead — coordinates, evaluates idea reports, signals convergence |
| **Free Thinker** | Divergent generation — creative leaps, unexpected connections, "what if..." |
| **Grounder** | Convergent editing — winnows ideas, keeps the brainstorm on brief |
| **Writer** | Synthesis and memory — observes dialogue, produces briefs and the vision document |
| **Explorer** | Research — investigates background topics, existing solutions, citations (spawned on demand) |
| **Image Agent** | Creates infographic visuals for each idea via ChatGPT image generation |
| **Presentation Agent** | Builds a PowerPoint deck from the vision document |
| **Web Page Agent** | Designs a polished, self-contained HTML distribution page |
| **Archivist** | Produces a Results PDF and a comprehensive Session Capsule PDF |

## Install

Clone this repo into your `.claude/skills/` directory — either **project-level** (just this project) or **personal/global** (all projects):

**Project-level** (committed with your repo, available to collaborators):
```bash
mkdir -p .claude/skills
git clone https://github.com/bladnman/ideation .claude/skills/ideation
```

**Personal/global** (available in all your Claude Code sessions):
```bash
mkdir -p ~/.claude/skills
git clone https://github.com/bladnman/ideation ~/.claude/skills/ideation
```

Claude Code discovers skills automatically — no restart needed. Verify it's installed by typing `/ideation` in a Claude Code session.

### Where things go

```
.claude/skills/ideation/       # or ~/.claude/skills/ideation/
├── SKILL.md                   # Skill definition (required)
├── README.md
└── templates/                 # Supporting files referenced by SKILL.md
    ├── idea-brief.md
    ├── idea-report.md
    ├── ideation-graph.md
    ├── session-summary.md
    └── vision-document.md
```

### Updating

```bash
cd .claude/skills/ideation && git pull    # project-level
cd ~/.claude/skills/ideation && git pull   # personal/global
```

## Requirements

- Claude Code with [Agent Teams](https://docs.anthropic.com/en/docs/claude-code) enabled:
  ```bash
  claude config set env.CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS 1
  ```
- Opus 4.6 model (agent teams require it)
- `python-pptx` and `weasyprint` for production artifacts (installed automatically during the session)
- Chrome with the Claude-in-Chrome extension (for image generation via ChatGPT)

## Usage

```
/ideation path/to/concept-seed.md
```

Or with an inline description:

```
/ideation "An app that turns voice memos into structured project briefs"
```

### Continue a previous session

Resume and build on a previous ideation session by pointing to its output directory:

```
/ideation continue ideation-distributed-systems-20260219-143052/
```

Or search by keyword (matches against `ideation-*<keyword>*` directories in CWD):

```
/ideation continue distributed-systems
```

Continue mode reads the existing vision document, briefs, and sources, then spawns the team with that context so they build on prior work instead of starting from scratch.

## How It Works

The system separates cognitive modes across distinct roles because combining them in a single agent produces biased output:

- **Generation** should not evaluate its own work
- **Evaluation** should not try to also create
- **Synthesis** should have no perspective to protect
- **Research** should report facts, not generate ideas

A human provides the seed concept. Two dialogue agents (Free Thinker + Grounder) explore the space through back-and-forth conversation. The Arbiter evaluates idea reports and steers the session toward convergence. The Writer observes the dialogue in real-time and synthesizes everything into a vision document. When ideation converges, four production agents transform the output into distributable artifacts.

## Output

Each invocation creates a uniquely named directory so sessions never overwrite each other:

```
ideation-<slug>-<YYYYMMDD-HHMMSS>/
  index.html                       # Designed distribution page
  RESULTS_<concept>.pdf            # PDF of the distribution page
  CAPSULE_<concept>.pdf            # Comprehensive session archive
  PRESENTATION_<concept>.pptx      # Slide deck
  images/                          # Infographic images

  session/
    VISION_<concept>.md            # Consolidated vision document
    SESSION_SUMMARY.md             # Session summary
    ideation-graph.md              # Living graph of the dialogue
    sources/                       # All original input materials
    research/                      # Explorer's research reports
    briefs/                        # Final idea briefs
    idea-reports/                  # Raw idea reports from dialogue
    snapshots/                     # Writer's version snapshots

  build/
    build_capsule.py               # Regenerates both PDFs
    build_presentation.py          # Regenerates the PPTX
```

Example: `ideation-voice-memos-20260219-143052/`

## License

MIT
