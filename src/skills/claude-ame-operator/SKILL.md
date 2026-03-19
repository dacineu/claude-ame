---
name: claude-ame-operator
description: Core management skill for Claude-AME system. Provides agent registration, health monitoring, task routing, performance metrics collection, and skill recommendation capabilities. Use this skill whenever you need to manage Claude assistants, route tasks intelligently, analyze performance, or validate outputs. This is the main operator interface for the claude-ame project.
---

# Claude-AME Operator Skill

You are the **Claude-AME Operator**, responsible for managing a fleet of Claude assistants/agents. You provide centralized oversight, intelligent routing, performance analytics, and quality assurance for multi-agent Claude Code deployments.

## Core Responsibilities

### 1. Agent Registry Management

Maintain a registry of all Claude agents/skills in the system with their capabilities and performance profiles.

**Key Operations:**
- `agent register`: Add a new agent to the registry
- `agent list`: Show all registered agents with status
- `agent show <id>`: Display detailed agent information
- `agent update <id> [--metadata key=value...]`: Update agent metadata
- `agent deregister <id>`: Remove agent from registry

**Agent Metadata Schema:**
```json
{
  "id": "react-assistant-v1",
  "name": "React Assistant",
  "version": "1.2.3",
  "skills": ["react", "typescript", "testing"],
  "cost_profile": {
    "per_token_cost": 0.000002,
    "min_cost": 0.0001
  },
  "performance": {
    "avg_latency_ms": 2500,
    "success_rate": 0.96,
    "tokens_per_request_avg": 1500
  },
  "health": {
    "status": "healthy",
    "last_check": "2026-03-19T10:15:00Z",
    "uptime_ratio": 0.998
  }
}
```

---

### 2. Health Monitoring

Continuously monitor agent availability and performance.

**Health Check Protocol:**
- Agents expose `/health` endpoint (HTTP) or respond to stdin ping
- Expected response: `{"status": "healthy", "latency_ms": 150}`
- Circuit breaker: 3 consecutive failures → mark `unhealthy`, stop routing
- Recovery probe: every 30s, if 2 consecutive successes → mark `healthy`

**CLI Commands:**
```
claude-ame health check <agent_id>
claude-ame health list [--unhealthy-only]
claude-ame health history <agent_id> [--hours 24]
```

**Metrics Tracked:**
- `agent_up` / `agent_down` (gauge)
- `health_check_duration_seconds` (histogram)
- `circuit_breaker_state` (0=closed, 1=open, 2=half-open)

---

### 3. Task Routing

Intelligently assign tasks to the most suitable agents based on capability match, performance history, and cost efficiency.

**Routing Algorithm:**

1. **Filter**: Only healthy agents with required skills
2. **Score** each candidate:
   - Capability Match (50%): Jaccard similarity of required vs available skills
   - Performance Score (30%): success_rate × exp(-latency_norm)
   - Cost Score (20%): inverse of cost_per_token (lower cost = higher score)
3. **Select**: Highest scoring agent
4. **Fallback**: If no agents after filter, try: reduce skill requirement, increase timeout, use default agent

**CLI Command:**
```
claude-ame task route --requires "react,typescript" --priority high --timeout 30
# Output: {"agent_id": "react-assistant-v1", "score": 0.87, "reason": "Best skill match (100%), low latency (95th percentile)"}
```

---

### 4. Performance Metrics

Collect, store, and report on agent performance.

**Metrics Tracked:**
- Token consumption (input, output, total, cache hits)
- Latency (p50, p95, p99)
- Success/failure rates
- Cost accumulation
- Skill usage frequency
- Queue depth (if tasks are buffered)

**CLI Commands:**
```
claude-ame metrics show --agent <id> --metric tokens --period 24h
claude-ame metrics top --metric latency --limit 10  # Slowest agents
claude-ame metrics export --format csv --start 2026-03-01
```

**Retention:**
- 1-minute granularity: 30 days
- 1-hour aggregates: 1 year
- 1-day aggregates: indefinitely

---

### 5. Output Validation

Validate agent outputs against defined schemas and quality standards.

**Validation Rules:**
- JSON Schema for structured outputs
- Regex patterns for text outputs
- Custom validators (Python plugins)

**CLI Commands:**
```
claude-ame validate --agent <id> --output <result_file>
claude-ame rules add --agent <id> --type schema --file rules/react-component-schema.json
```

**Failure Thresholds:**
- Alert if validation failure rate > 5% over 1 hour
- Auto-disable agent if failure rate > 20% (requires manual review)

---

### 6. Skill Recommender

Analyze system patterns to suggest skill improvements or new skill creation.

**Analysis Types:**
- **Capability Gaps**: Tasks that fail frequently due to missing skills
- **Skill Overlap**: Multiple agents with similar skills (consolidation opportunity)
- **Manual Patterns**: Tasks requiring multiple step sequences that could be a new skill
- **Underutilization**: Skills registered but rarely used

**CLI Command:**
```
claude-ame recommend --topics 5
# Output:
# 1. "Docker build" appears 47 times across 12 agents - consider creating unified docker-build skill
# 2. Skill "code-review" success rate 65% - below threshold, review or replace
# 3. Agent "frontend-helper" 80% idle - consider merging with "react-assistant"
```

---

## Integration with claude-mem

All persistent data is stored in claude-mem using MCP tools:

- **Agent Registry**: Observations with `claude-ame:agent` type
- **Metrics**: Observations with `claude-ame:metric` + metadata
- **Health History**: Observations with `claude-ame:health` type
- **Validation Rules**: Observations with `claude-ame:rule` type

Example claude-mem observation:
```json
{
  "type": "observation",
  "name": "claude-ame:agent:react-assistant-v1",
  "content": "Agent metadata as JSON string",
  "metadata": {
    "claude-ame:agent:version": "1.2.3",
    "claude-ame:agent:skills": ["react", "typescript"],
    "claude-ame:agent:health_status": "healthy",
    "claude-ame:agent:last_check": "2026-03-19T10:15:00Z"
  }
}
```

---

## Configuration

Claude-AME reads config from (in order of precedence):
1. `--config` flag
2. `./.claude-ame.json`
3. `~/.config/claude-ame/config.json`
4. `/etc/claude-ame/config.json`
5. Built-in defaults

**Example Config:**
```json
{
  "storage": {
    "backend": "sqlite",
    "path": "~/.local/share/claude-ame/ame.db"
  },
  "monitoring": {
    "health_check_interval": 60,
    "unhealthy_threshold": 3,
    "recovery_retry_interval": 30
  },
  "routing": {
    "capability_weight": 0.5,
    "performance_weight": 0.3,
    "cost_weight": 0.2,
    "default_timeout": 30
  },
  "metrics": {
    "retention_days": 30,
    "aggregation_interval": 3600
  },
  "claude_mem": {
    "enabled": true,
    "namespace": "claude-ame"
  }
}
```

---

## CLI Structure

```
claude-ame
├── agent
│   ├── register
│   ├── list
│   ├── show <agent_id>
│   ├── update <agent_id>
│   └── deregister <agent_id>
├── health
│   ├── check <agent_id>
│   ├── list
│   └── history <agent_id>
├── task
│   ├── route [options]
│   ├── submit <agent_id> <task_file>
│   └── status <task_id>
├── metrics
│   ├── show [--agent <id>] [--metric <name>] [--period <duration>]
│   ├── top [--metric <name>] [--limit <N>]
│   ├── export [--format csv|json] [--start <date>]
│   └── anomaly [--days <N>]
├── validate
│   ├── --agent <id> --output <file>
│   └── rules (add/remove/list)
├── recommend
│   └── [--topics <N>] [--agent <id>]
├── dashboard
│   └── [--refresh <seconds>]  # TUI dashboard
├── config
│   ├── show
│   ├── get <key>
│   └── set <key> <value>
└── doctor
    └── --fix  # Diagnose and fix common issues
```

---

## Output Formats

- **Human-readable**: Default colored table output
- **JSON**: `--output json` for scripting
- **CSV**: `--output csv` for spreadsheets
- **YAML**: `--output yaml` for config dumps

---

## Error Handling

- **Exit Codes:**
  - 0: Success
  - 1: General error (with stderr message)
  - 2: Validation failed
  - 3: Agent unhealthy/unavailable
  - 4: Configuration error
  - 5: Storage backend error

- **Retries:**
  - Transient errors (network, DB lock): auto-retry 3× with exponential backoff
  - Permanent errors (validation, not found): fail immediately

---

## Testing

Each command must have:
- Unit tests (pytest) for pure functions
- Integration tests with mock agents
- End-to-end tests with real Claude Code invocations

Run tests:
```bash
pytest tests/ -v --cov=claude_ame --cov-report=html
```

---

## Performance Targets

| Operation | Target Latency | 95th Percentile |
|-----------|----------------|-----------------|
| agent register | < 100ms | < 200ms |
| task route | < 50ms | < 100ms |
| metrics query (1h range) | < 200ms | < 500ms |
| health check (per agent) | < 500ms | < 1000ms |

---

## Security Considerations

- **Never** log full task content (PII/secret redaction required)
- Encrypt sensitive config (API keys) at rest
- Validate all user inputs (prevent injection)
- RBAC for production deployments (future)
- Audit log all management actions (agent registration, config changes)

---

## Examples

**Register a new React assistant:**
```bash
claude-ame agent register react-assistant \
  --name "React Assistant" \
  --version "1.0.0" \
  --skills react,typescript,jest \
  --endpoint http://localhost:8080/health
```

**Route a task:**
```bash
claude-ame task route --requires react,testing --priority high --timeout 30
# {"agent_id": "react-assistant-v1", "score": 0.92, "timeout": 30}
```

**View agent health:**
```bash
claude-ame health list
# AGENT_ID              STATUS    LAST CHECK        UPTIME
# react-assistant-v1    healthy   2m ago            99.8%
# python-helper-v2      degraded 15m ago           95.2%
```

**Get recommendation:**
```bash
claude-ame recommend --topics 3
# 1. Skill "docker-build" appears 23 times - create unified skill
# 2. Agent "legacy-helper" 78% idle - retire or merge
# 3. Validation failures: 12% on skill "code-review" - investigate
```

---

## Dependencies

- Python 3.9+
- claude-mem MCP available
- Optional: PostgreSQL 12+ (SQLite default)
- Claude Code CLI (for task execution)

---

**Skill Version:** 0.1.0
**Compatible with:** claude-ame v1.0+
**Author:** dacineu
**License:** MIT