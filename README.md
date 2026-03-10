# Software Architect Skill

A Claude Code skill that designs software architecture for any project type. Given a project description, it interviews you about requirements, then produces a comprehensive `ARCHITECTURE.md` with component diagrams, database schemas, API contracts, folder structures, and phased build sequences.

## Install

### Option 1: Add to your project (recommended)

Clone into your project's `.claude/skills/` directory:

```bash
mkdir -p .claude/skills
git clone https://github.com/jskallebak/software-architect.git .claude/skills/software-architect
```

Or add as a git submodule:

```bash
git submodule add https://github.com/jskallebak/software-architect.git .claude/skills/software-architect
```

### Option 2: Install globally

Clone to your home `.claude/skills/` so it's available in all projects:

```bash
mkdir -p ~/.claude/skills
git clone https://github.com/jskallebak/software-architect.git ~/.claude/skills/software-architect
```

### Option 3: Reference in CLAUDE.md

Add this to your project's `CLAUDE.md` or `~/.claude/CLAUDE.md`:

```markdown
## Skills
When asked to design architecture, read and follow the skill at: .claude/skills/software-architect/SKILL.md
```

## Usage

Just ask Claude Code to design your project:

- "Design the architecture for a real-time chat app"
- "I want to build an e-commerce platform with 50k users"
- "Architecture for a CLI tool that scrapes job postings"
- "How should I structure a React Native fitness app?"

The skill will:

1. **Interview you** — asks about scale, team, constraints, tech preferences
2. **Consult references** — draws from 5 reference guides covering architecture patterns, design patterns, tech stacks, documentation best practices, and decision frameworks
3. **Generate ARCHITECTURE.md** — complete with Mermaid diagrams, tech stack tables, data models, API designs, and phased build sequences

## What's included

```
software-architect/
├── SKILL.md                                    # Main skill instructions
└── references/
    ├── system-architecture-patterns.md         # 30+ architecture patterns
    ├── design-patterns.md                      # GoF, DDD, clean architecture
    ├── tech-stack-decision-guide.md            # Framework/DB/infra decisions
    ├── architecture-documentation-guide.md     # ADRs, C4 model, arc42, RFCs
    └── architecture-decision-frameworks.md     # ATAM, risk-driven, heuristics
```

## Benchmark results

Tested across 8 diverse project types (mobile app, ML platform, IoT, legacy migration, static site, fintech API, multiplayer game, data pipeline):

| Metric | With Skill | Without Skill |
|--------|-----------|--------------|
| Pass rate | **100%** (56/56) | 73% (41/56) |
| Mermaid diagrams | Always (1-6/doc) | Never |
| Build sequences | Always | 50% of the time |
| Tradeoff depth | ~14 comparisons/doc | ~7 comparisons/doc |
