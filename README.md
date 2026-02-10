# skill-auditor

Normalize components from any source into production-ready parts of a robust system.

> Other auditors check if it's safe. This one makes it *good*.

## The problem

You're pulling skills from GitHub repos, official marketplaces, community collections, plugins with bundled skills, and that one thing someone shared in Discord. They all have different:
- Quality levels (some are 50 lines, some are 900)
- Structure (some follow best practices, some don't)
- Safety profiles (some are careful, some are yolo)

Before skill-auditor, you had two choices: blindly install everything, or manually review each component. Neither scales.

## What this does differently

Most auditors stop at "is it safe?" This one goes further:

1. **Security scanning** - Yes, catches dangerous patterns (eval, subprocess, credential access)
2. **Performance optimization** - Restructures bloated skills to actually perform well
3. **Multi-source normalization** - Takes components from anywhere and gets them to a consistent, robust standard
4. **Type-aware rules** - Different standards for skills vs agents vs hooks vs plugins

## Research backing

This skill is grounded in actual research, not intuition:

### Anthropic skill authoring best practices
[platform.claude.com/docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- SKILL.md must be <500 lines (performance degrades beyond this)
- Name ≤64 characters
- Description ≤1024 characters
- Reference depth: 1 level only (nested refs cause partial reads)
- Use progressive disclosure pattern

### "Lost in the Middle" (Stanford/Berkeley, 2023)
[cs.stanford.edu/~nfliu/papers/lost-in-the-middle.arxiv2023.pdf](https://cs.stanford.edu/~nfliu/papers/lost-in-the-middle.arxiv2023.pdf)
- U-shaped recall curve for LLM context
- **First 20%** = primacy zone (high recall) → Put purpose, triggers, usage here
- **Last 10%** = recency zone (high recall) → Put quick reference here
- **Middle 70%** = "lost" zone (low recall) → Use pointers to sub-files, not critical details

## Installation

Copy the `skill-auditor` folder to your Claude Code skills directory:

```bash
# Clone this repo
git clone https://github.com/yourusername/skill-auditor.git

# Copy to Claude Code skills
cp -r skill-auditor ~/.claude/skills/
```

## Usage

```bash
# Full audit (security + optimization)
/skill-auditor /path/to/skill

# Security scan only
/skill-auditor /path/to/skill --security-only

# Optimization analysis only
/skill-auditor /path/to/skill --optimize-only

# Batch audit all installed components
/skill-auditor --all
```

## What it checks

### Security patterns

| Pattern | Risk | Description |
|---------|------|-------------|
| `eval()`, `exec()` | Critical | Arbitrary code execution |
| `subprocess`, `os.system` | Critical | Shell injection |
| `shutil.rmtree`, `rm -rf` | Critical | Recursive file deletion |
| `curl`, `wget`, `requests` | High | Network access |
| Hardcoded credentials | Critical | Exposed secrets |

### Optimization checks

| Check | Limit | Why |
|-------|-------|-----|
| SKILL.md lines | <500 | Performance degradation |
| Name length | ≤64 chars | Anthropic spec |
| Description length | ≤1024 chars | Anthropic spec |
| Content zones | Primacy/recency | LLM recall optimization |

### Risk levels by component type

| Type | Risk | Why |
|------|------|-----|
| Skills | Medium | User-invoked, transparent |
| Agents | High | Autonomous, can take actions |
| Plugins | High | Bundle multiple components |
| Hooks | **Critical** | Auto-execute without confirmation |

## Output format

```
## Audit: example-skill

**Type:** Skill
**Source:** /path/to/skill
**Date:** 2024-01-15

### Security scan
🔴 CRITICAL: eval() found at line 45
🟡 WARNING: Network request at line 89
🟢 PASS: No credential exposure

### Optimization analysis
- Lines: 652 (limit: 500) ❌
- Structure: Missing frontmatter
- Content zones: Critical info in middle (should be in primacy zone)

### Recommendation
⚠️ INSTALL WITH CAUTION: Needs optimization before use
```

## Example: fixing a violation

Before (823 lines):
```
my-skill/
└── SKILL.md (823 lines - violates 500 limit)
```

After restructuring with progressive disclosure:
```
my-skill/
├── SKILL.md (247 lines - compliant)
├── examples/
│   └── detailed-examples.md
├── templates/
│   └── output-templates.md
└── reference/
    └── api-reference.md
```

## Self-documenting

This skill follows all the patterns it teaches:
- SKILL.md is 150 lines (well under 500)
- Uses progressive disclosure (details in `checklists/` and `patterns/`)
- Critical info in first 20%, quick reference at end
- Type-specific guidance in separate files

## File structure

```
skill-auditor/
├── SKILL.md                    # Main skill (150 lines)
├── README.md                   # This file
├── checklists/
│   ├── security.md             # Security patterns to scan for
│   └── optimization.md         # Optimization guidelines
└── patterns/
    ├── skill.md                # Skill-specific patterns
    ├── agent.md                # Agent-specific patterns
    ├── hook.md                 # Hook-specific patterns
    └── plugin.md               # Plugin-specific patterns
```

## License

MIT License - see LICENSE file.

## Contributing

Issues and PRs welcome. The skill itself should remain under 500 lines, so consider contributing to the sub-files in `checklists/` or `patterns/`.
