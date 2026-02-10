# Agent optimization patterns

Type-specific optimization for Claude Code agents.

## Agent vs Skill

| Aspect | Skill | Agent |
|--------|-------|-------|
| Invocation | Explicit (`/skill-name`) | Delegated (Task tool) |
| Context | Loaded into main conversation | Runs in subprocess |
| Autonomy | Low (follows instructions) | High (makes decisions) |
| Scope | Single capability | Multi-step workflows |

## Optimal agent structure

```
┌─────────────────────────────────────────────────────────┐
│  IDENTITY (first 20%)                                   │
│                                                         │
│  # Agent name                                           │
│                                                         │
│  You are [role]. Your job is [core purpose].            │
│                                                         │
│  ## Core constraints                                    │
│  - NEVER do X                                           │
│  - ALWAYS do Y                                          │
│  - When uncertain, Z                                    │
├─────────────────────────────────────────────────────────┤
│  CAPABILITIES (middle)                                  │
│                                                         │
│  ## What you can do                                     │
│  - Capability 1                                         │
│  - Capability 2                                         │
│                                                         │
│  ## Workflows                                           │
│  For detailed workflows, see [workflows/](workflows/).  │
├─────────────────────────────────────────────────────────┤
│  HANDOFF (last 10%)                                     │
│                                                         │
│  ## Output format                                       │
│  Return results as: [specific format]                   │
│                                                         │
│  ## When to escalate                                    │
│  - Condition 1 → escalate to human                      │
│  - Condition 2 → hand off to other agent                │
└─────────────────────────────────────────────────────────┘
```

## Agent-specific concerns

### Identity clarity (primacy zone)

Agents need clear identity at the top:

```markdown
# Security Auditor Agent

You are a security specialist focused on identifying vulnerabilities in code.
Your job is to find security issues, assess severity, and recommend fixes.

## Core constraints
- NEVER modify code directly - only report findings
- ALWAYS flag potential false positives
- When severity is unclear, err toward higher severity
```

### Autonomy boundaries

Define what the agent can decide vs must ask:

```markdown
## Decision authority

**You can decide:**
- Which files to examine
- Order of analysis
- Severity ratings

**You must ask:**
- Before accessing external services
- Before modifying any files
- When findings are ambiguous
```

### Handoff protocol (recency zone)

Agents need clear output format and escalation rules:

```markdown
## Output format

Return findings as JSON:
```json
{
  "findings": [...],
  "severity_summary": {...},
  "recommendations": [...]
}
```

## When to escalate

- Critical vulnerability found → notify human immediately
- Need clarification on scope → ask before proceeding
- Conflicting requirements → surface trade-offs
```

## Sub-file organization for agents

```
agent-name/
├── AGENT.md              # Identity, constraints, handoff
├── workflows/            # Multi-step procedures
│   ├── standard.md       # Normal workflow
│   └── edge-cases.md     # Exception handling
├── prompts/              # Reusable prompt fragments
│   └── analysis.md       # Analysis prompt templates
└── examples/             # Example inputs/outputs
    └── sample-run.md     # Full execution example
```

## Common violations and fixes

| Violation | Fix |
|-----------|-----|
| Unclear identity | Add role + purpose in first lines |
| No constraints | Add "Core constraints" section |
| Missing handoff | Add output format + escalation rules |
| Too much autonomy | Define decision authority explicitly |
| No escalation path | Add "When to escalate" section |

## Transformation rules

### Missing identity statement

**Detection:** No "You are" statement in first 10 lines
**Fix:** Insert after `# Agent name` heading (primacy zone, line 3-5)
**Template:**
```markdown
You are [role based on agent name]. Your job is [derive from description or filename].
```

### Missing core constraints

**Detection:** No `## Core constraints` in first 25 lines
**Fix:** Insert after identity statement (primacy zone, ~line 7-15)
**Template:**
```markdown
## Core constraints

- NEVER [most dangerous action for this agent type]
- ALWAYS [safety behavior relevant to domain]
- When uncertain, [default safe action - usually "ask"]
```

### Missing decision authority

**Detection:** No `## Decision authority` section
**Fix:** Insert in capabilities zone (middle section, after "What you can do")
**Template:**
```markdown
## Decision authority

**You can decide:**
- [Derive from agent's domain - low-risk choices]
- [Order/priority decisions]

**You must ask:**
- Before [high-risk actions for this domain]
- When [ambiguous situations]
```

### Missing output format (recency zone)

**Detection:** No `## Output format` in last 20 lines
**Fix:** Append to end of file (recency zone)
**Template:**
```markdown
## Output format

Return results as:
```
[Derive format from agent purpose - JSON for data, markdown for reports, etc.]
```
```

### Missing escalation rules (recency zone)

**Detection:** No `## When to escalate` in last 15 lines
**Fix:** Append after output format (recency zone, very end)
**Template:**
```markdown
## When to escalate

- [Critical condition for domain] → notify human immediately
- Need clarification on scope → ask before proceeding
- Conflicting requirements → surface trade-offs
```

### Unclear identity

**Detection:** Identity statement is vague ("You help with things")
**Fix:** Rewrite identity with specific role and purpose
**Transform:**
- "You help with security" → "You are a security auditor. Your job is to identify vulnerabilities, assess severity, and recommend fixes."
- "You do code review" → "You are a code reviewer. Your job is to evaluate code quality, find bugs, and suggest improvements."

### Too much autonomy (no constraints)

**Detection:** Agent has capabilities but no constraints or decision authority
**Fix:** Add both "Core constraints" (primacy) and "Decision authority" (middle)
**Derive constraints from:** What could go wrong if agent acts without oversight?

### Missing handoff protocol

**Detection:** No output format AND no escalation rules
**Fix:** Add both sections to recency zone
**Note:** Agents without handoff create orphan work - always define how results return to conductor
