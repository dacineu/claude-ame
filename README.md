# Claude-AME (Assistant Manager & Expertiser)

**Unified management platform for Claude Code assistants and autonomous agents**

Claude-AME provides centralized management, intelligent routing, performance analytics, and quality assurance for fleets of Claude Code assistants. Monitor agent health, optimize task distribution, track costs, and get intelligent recommendations for skill improvements—all from a single CLI interface.

---

## Quick Status

- **Phase**: Inception → Phase 1 Planning
- **Version**: 0.1.0
- **GSD Milestone**: Pre-M1 (requirements complete)
- **See**: `.planning/STATE.md` for detailed status

---

## Key Features

- **Agent Registry** - Register and manage all Claude assistants with metadata
- **Health Monitoring** - Automatic health checks with circuit breakers
- **Intelligent Routing** - Task assignment based on capability, performance, cost
- **Performance Analytics** - Token usage, latency, success rates, cost tracking
- **Output Validation** - JSON Schema, regex, custom validators
- **Skill Recommender** - Gap analysis, consolidation suggestions
- **claude-mem Integration** - Persistent storage via MCP

---

## Project Structure (GSD)

```
claude-ame/
├── .planning/
│   ├── PROJECT.md         # Vision & goals
│   ├── REQUIREMENTS.md    # Detailed FRs/NFRs
│   ├── ROADMAP.md         # 5-phase development plan
│   ├── CONFIG.json        # Configuration template
│   └── STATE.md           # Current project status
├── src/
│   └── skills/
│       └── claude-ame-operator/
│           └── SKILL.md   # Operator skill definition
├── scripts/                # Helper scripts
├── tests/                  # Test suite
├── docs/                   # Documentation (to be generated)
├── examples/               # Sample configs & usage
├── dist/                   # Built packages (.skill)
├── README.md               # Project overview (this file)
├── STATE.md                # Quick status reference
└── LICENSE                 # MIT License
```

---

## Quick Start (Once Installed)

```bash
# Initialize configuration
claude-ame config init

# Register an agent
claude-ame agent register my-helper \
  --name "My Helper" \
  --skills python,testing \
  --endpoint http://localhost:8080/health

# Monitor health
claude-ame health list

# Route a task
claude-ame task route --requires python,testing
```

---

## Documentation

See `.planning/` for full project documentation:
- **PROJECT.md** - Vision, goals, success metrics
- **REQUIREMENTS.md** - Detailed functional & non-functional requirements
- **ROADMAP.md** - 5-phase development plan with milestones
- **STATE.md** - Current project status and next actions

Operator skill reference: `src/skills/claude-ame-operator/SKILL.md`

---

## Development Status

- **Phase 1 (Foundation)**: Planning complete, ready to begin implementation
- **Deliverables**: Agent registry, CLI framework, configuration management
- **Estimated Effort**: 9 weeks total for v1.0

---

## License

MIT
