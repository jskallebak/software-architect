# Software Architect Plugin

A Claude Code plugin that designs software architecture for new projects and reviews existing codebases. It can interview you about requirements and produce a comprehensive `ARCHITECTURE.md`, or analyze your existing codebase and generate an `ARCHITECTURE_REVIEW.md` with findings and recommendations.

## Install

### As a Claude Code Plugin (recommended)

```bash
claude plugin add --marketplace github:jskallebak/software-architect
```

Then enable it:
```bash
claude plugin enable software-architect
```

### Manual install

Clone to your project or global skills directory:

```bash
# Project-local
mkdir -p .claude/skills
git clone https://github.com/jskallebak/software-architect.git .claude/skills/software-architect

# Or global (all projects)
mkdir -p ~/.claude/skills
git clone https://github.com/jskallebak/software-architect.git ~/.claude/skills/software-architect
```

## Usage

### Design mode (new projects)

Ask Claude Code to design your project:

- "Design the architecture for a real-time chat app"
- "I want to build an e-commerce platform with 50k users"
- "Architecture for a CLI tool that scrapes job postings"
- "How should I structure a React Native fitness app?"

The skill will:

1. **Interview you** — asks about scale, team, constraints, tech preferences
2. **Consult references** — draws from 6 reference guides covering architecture patterns, design patterns, tech stacks, documentation best practices, decision frameworks, and diagram creation
3. **Generate ARCHITECTURE.md** — complete with Mermaid diagrams, tech stack tables, data models, API designs, and phased build sequences

### Review mode (existing projects)

Ask Claude Code to review your codebase:

- "Review the architecture of this project"
- "What do you think of this codebase?"
- "Audit this project's structure"
- "How can I improve the architecture?"

The skill will:

1. **Explore the codebase** — strategically reads entry points, configs, schemas, and module boundaries
2. **Assess architecture** — evaluates structure, data design, API consistency, dependencies, security, scalability, and code quality
3. **Generate ARCHITECTURE_REVIEW.md** — with executive summary, what's working well, prioritized improvements (critical/important/nice-to-have), pattern and tech stack assessments, and a recommended roadmap

## What's included

```
skills/software-architect/
├── SKILL.md                                    # Main skill instructions
└── references/
    ├── system-architecture-patterns.md         # 30+ architecture patterns
    ├── design-patterns.md                      # GoF, DDD, clean architecture
    ├── tech-stack-decision-guide.md            # Framework/DB/infra decisions
    ├── architecture-documentation-guide.md     # ADRs, C4 model, arc42, RFCs
    ├── architecture-decision-frameworks.md     # ATAM, risk-driven, heuristics
    └── excalidraw-diagrams.md                  # Excalidraw MCP integration guide
```

## Benchmark results

Tested across 8 diverse project types (mobile app, ML platform, IoT, legacy migration, static site, fintech API, multiplayer game, data pipeline):

| Metric | With Skill | Without Skill |
|--------|-----------|--------------|
| Pass rate | **100%** (56/56) | 73% (41/56) |
| Mermaid diagrams | Always (1-6/doc) | Never |
| Build sequences | Always | 50% of the time |
| Tradeoff depth | ~14 comparisons/doc | ~7 comparisons/doc |
