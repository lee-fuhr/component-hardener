---
name: skill-auditor
description: Audits skills, agents, plugins, and hooks for security and optimization. Use when installing new components, reviewing external code, or optimizing existing skills. Checks for unsafe patterns, validates structure, recommends improvements based on Anthropic best practices.
---

# Skill auditor

Security audit and optimization for Claude Code components before installation or to improve existing ones.

**Usage:**
- `/skill-auditor [path]` - Full audit (security + optimization)
- `/skill-auditor [path] --security-only` - Security scan only
- `/skill-auditor [path] --optimize-only` - Optimization analysis only
- `/skill-auditor --all` - Batch audit all installed components

## When to use

**ALWAYS run this skill when:**
- Installing anything from external sources
- User points you to a new skill/agent/plugin/hook
- Adding a new marketplace
- Reviewing components you haven't audited before
- Optimizing existing components for performance

## Risk levels by type

| Type | Risk | Why |
|------|------|-----|
| Skills | Medium | User-invoked, transparent |
| Agents | High | Autonomous, can take actions |
| Plugins | High | Bundle multiple components |
| Hooks | **Critical** | Auto-execute without confirmation |

## Audit workflow

### Phase 1: Security scan

Check for dangerous patterns. See [checklists/security.md](checklists/security.md) for full list.

**Critical patterns to grep:**
```bash
grep -rn "eval\|exec\|subprocess\|os\.system" .
grep -rn "shutil\.rmtree\|rm -rf" .
grep -rn "curl\|wget\|requests\." .
```

**Output security findings as:**
- 🔴 CRITICAL: Blocks installation
- 🟡 WARNING: Needs review
- 🟢 PASS: Safe pattern confirmed

### Phase 2: Optimization analysis

Check against Anthropic guidelines. See [checklists/optimization.md](checklists/optimization.md).

**Hard limits:**
| Element | Limit |
|---------|-------|
| SKILL.md body | <500 lines |
| Name | ≤64 chars |
| Description | ≤1024 chars |
| Reference depth | 1 level only |

**Content zone check (Lost in the Middle research):**
- First 20% (primacy): Should have purpose, triggers, usage
- Middle 70%: Should have pointers only, not critical details
- Last 10% (recency): Should have quick reference, related items

### Phase 3: Type-specific review

Apply patterns for the specific component type:
- Skills: [patterns/skill.md](patterns/skill.md)
- Agents: [patterns/agent.md](patterns/agent.md)
- Hooks: [patterns/hook.md](patterns/hook.md)
- Plugins: [patterns/plugin.md](patterns/plugin.md)

### Phase 4: Recommendations

Generate specific recommendations for each violation found.

**CRITICAL: Ask for confirmation before making changes.**

Output format:
```markdown
## Optimization recommendations for [component]

### Violations found
1. [Violation]: [specific issue]
2. [Violation]: [specific issue]

### Proposed changes
1. [Change]: [what will be modified]
2. [Change]: [what will be modified]

### New structure (if restructuring)
```
component/
├── SKILL.md (X lines → Y lines)
├── [new-subfile].md
└── ...
```

**Proceed with changes?** [Yes / No / Show me more detail]
```

## Output format

```markdown
## Audit: [component-name]

**Type:** [Skill / Agent / Plugin / Hook]
**Source:** [path or URL]
**Date:** [date]

### Security scan
🔴 CRITICAL: [blockers]
🟡 WARNING: [concerns]
🟢 PASS: [safe patterns]

### Optimization analysis
- Lines: [X] (limit: 500)
- Structure: [assessment]
- Content zones: [assessment]

### Recommendation
✅ SAFE TO INSTALL (no changes needed)
✅ SAFE TO INSTALL + optimization available
⚠️ INSTALL WITH CAUTION: [concerns]
🚫 DO NOT INSTALL: [blockers]

### Optimization available
[If applicable, list proposed improvements]
```

---

## Quick reference

| Command | Description |
|---------|-------------|
| `/skill-auditor path` | Full audit |
| `/skill-auditor path --security-only` | Security only |
| `/skill-auditor path --optimize-only` | Optimization only |
| `/skill-auditor --all` | Batch audit |

## Related

- [checklists/security.md](checklists/security.md) — Full security patterns
- [checklists/optimization.md](checklists/optimization.md) — Optimization guidelines
- [patterns/](patterns/) — Type-specific patterns
