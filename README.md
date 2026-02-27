# ideation

Multi-agent ideation skill for [Claude Code](https://claude.com/claude-code). Launches a team of specialized agents that explore a concept through structured dialogue between a Free Thinker and a Grounder, arbitrated by the team lead, and documented by a Writer. Three actions control the workflow: **Plan** (pre-flight interview), **Ideate** (depth-aware exploration + conditional production), and **Continue** (versioned session resumption).

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
    ├── lineage.md             # Version chain tracking for continuations
    ├── prd.md
    ├── session-config.yaml    # Session configuration schema
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

### New session (Plan → Ideate)

Every new session starts with the **Plan** action — a brief interview to configure depth, outputs, and research before the team is spawned.

```
/ideation path/to/concept-seed.md
```

Or with an inline description:

```
/ideation "An app that turns voice memos into structured project briefs"
```

Or with no argument (the interview will ask for the concept):

```
/ideation
```

The Plan action will ask you:
1. **Depth** — how deep should exploration go?
   - **Quick** (~15-30 min): 2-3 directions, fast convergence
   - **Standard** (~45-90 min): 3-5 directions, balanced exploration *(default)*
   - **Deep** (~2-3 hrs): 5-8 directions, thorough exploration
   - **Exhaustive** (~3+ hrs): 8+ directions, comprehensive mapping
2. **Outputs** — which deliverables to produce (all, subset, or session artifacts only)
3. *(If needed)* Domain clarification or research confirmation

After confirmation, the Ideate action launches the team with your configured settings.

### Continue a previous session

Resume and build on a previous ideation session. Creates a **new versioned directory** — the parent session is never modified.

```
/ideation continue ideation-distributed-systems-20260219-143052/
```

Or search by keyword (matches against `ideation-*<keyword>*` directories in CWD):

```
/ideation continue distributed-systems
```

Continue mode:
- **Discovers** matching sessions with summaries (date, depth, brief count)
- **Creates a versioned directory** (e.g., `ideation-voice-memos-v2-20260221-091500/`)
- **Runs a mini-interview** asking what to focus on, depth, and outputs
- **References parent materials** (vision doc, briefs, ideation graph) so the team builds on prior work
- **Tracks lineage** in `session/LINEAGE.md`

Versioning scheme:
```
Original:     ideation-voice-memos-20260219-143052/
Continue:     ideation-voice-memos-v2-20260221-091500/
Continue:     ideation-voice-memos-v3-20260222-140000/
Branch:       ideation-voice-memos-v2a-20260222-150000/
```

### Generate a PRD from a session

After an ideation session completes, generate a Product Requirements Document:

```
/ideation prd ideation-distributed-systems-20260219-143052/
```

Or search by keyword:

```
/ideation prd distributed-systems
```

PRD mode is a solo operation — no team is spawned. It reads the session's vision document, briefs, and other artifacts, then produces a `PRD_<concept-slug>.md` that focuses on *what* to build and *why*, preserving the ideation session's intent, emotional logic, and language.

## How It Works

The system separates cognitive modes across distinct roles because combining them in a single agent produces biased output:

- **Generation** should not evaluate its own work
- **Evaluation** should not try to also create
- **Synthesis** should have no perspective to protect
- **Research** should report facts, not generate ideas

A human provides the seed concept. The **Plan** action interviews the user to configure depth and outputs. The **Ideate** action spawns the team: two dialogue agents (Free Thinker + Grounder) explore the space, the Arbiter evaluates idea reports and steers toward convergence using depth-aware rules, and the Writer observes and synthesizes into a vision document. When ideation converges, production agents (conditionally spawned based on output selection) transform the output into distributable artifacts.

### Depth levels

| | Quick | Standard | Deep | Exhaustive |
|---|---|---|---|---|
| Directions explored | 2-3 | 3-5 | 5-8 | 8+ |
| Time estimate | ~15-30 min | ~45-90 min | ~2-3 hrs | ~3+ hrs |
| Convergence | Fast | Balanced | Thorough | Comprehensive |
| Best for | Well-defined concepts, time pressure | General exploration | Complex/ambiguous domains | Foundational concepts |

### Output tiers

**Tier 1** (always produced): Vision doc, briefs, session summary, ideation graph, snapshots, idea reports.

**Tier 2** (user-selectable): Distribution page (HTML), Results PDF, Capsule PDF, PPTX presentation, infographic images. Production agents are only spawned for selected outputs.

**Minimal path**: If no Tier 2 outputs are selected, the production phase is skipped entirely.

## Output

Each invocation creates a uniquely named directory:

```
ideation-<slug>-<YYYYMMDD-HHMMSS>/
  index.html                       # Designed distribution page (if selected)
  RESULTS_<concept>.pdf            # PDF of the distribution page (if selected)
  CAPSULE_<concept>.pdf            # Comprehensive session archive (if selected)
  PRESENTATION_<concept>.pptx      # Slide deck (if selected)
  PRD_<concept>.md                 # Product requirements (generated via /ideation prd)
  images/                          # Infographic images (if selected)

  session/
    session-config.yaml            # Session configuration
    VISION_<concept>.md            # Consolidated vision document
    SESSION_SUMMARY.md             # Session summary
    ideation-graph.md              # Living graph of the dialogue
    LINEAGE.md                     # Version chain (for continuations)
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
