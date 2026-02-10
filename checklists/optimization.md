# Optimization checklist

Research-based guidelines for optimizing skills, agents, plugins, and hooks.

## Source research

- [Anthropic Skill Authoring Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [Lost in the Middle: How Language Models Use Long Contexts](https://cs.stanford.edu/~nfliu/papers/lost-in-the-middle.arxiv2023.pdf) (Stanford/Berkeley)
- [Claude 4.x Prompting Best Practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-4-best-practices)

## Hard limits (Anthropic official)

| Element | Limit | Consequence of violation |
|---------|-------|--------------------------|
| SKILL.md body | **<500 lines** | Performance degradation, context competition |
| Name field | 64 characters max | Validation failure |
| Description field | 1024 characters max | Validation failure |
| Reference depth | 1 level only | Partial file reads, lost context |
| Files >100 lines | Must have TOC | Navigation failures |

## Lost in the middle effect

From Stanford/Berkeley research on LLM context usage:

```
┌─────────────────────────────────────────┐
│  FIRST 20% — PRIMACY ZONE (high recall) │
│  Put: identity, purpose, triggers       │
├─────────────────────────────────────────┤
│  MIDDLE 70% — LOST ZONE (low recall)    │
│  Put: pointers to sub-files only        │
│  Avoid: critical details, exact syntax  │
├─────────────────────────────────────────┤
│  LAST 10% — RECENCY ZONE (high recall)  │
│  Put: quick reference, cheatsheet       │
└─────────────────────────────────────────┘
```

**Key findings:**
- Accuracy highest at start and end (U-shaped curve)
- Effect strongest when input is <50% of context window
- Middle content often "lost" — Claude may not recall it

## Optimization analysis steps

### Step 1: Measure current state

```bash
# Count lines
wc -l SKILL.md

# Check for sub-files
ls -la *.md patterns/ examples/ reference/ 2>/dev/null

# Find nested references (bad)
grep -n "See \[.*\](.*\.md)" SKILL.md | while read line; do
  file=$(echo "$line" | grep -oP '\]\(\K[^)]+')
  grep -l "See \[.*\](.*\.md)" "$file" 2>/dev/null && echo "NESTED: $file"
done
```

### Step 2: Check against limits

| Check | Command | Target |
|-------|---------|--------|
| Line count | `wc -l SKILL.md` | <500 |
| Name length | `grep "^name:" SKILL.md \| wc -c` | <64 |
| Desc length | `grep "^description:" SKILL.md \| wc -c` | <1024 |
| Has TOC | `grep -c "^## Contents\|^## Table of" SKILL.md` | >0 if >100 lines |
| Reference depth | See nested check above | 0 nested |

### Step 3: Identify content zones

Analyze where critical content appears:

1. **First 20% of lines** — Should contain:
   - [ ] Frontmatter (name, description)
   - [ ] One-line purpose statement
   - [ ] Trigger conditions / "when to use"
   - [ ] Quick usage examples

2. **Middle 70%** — Should contain only:
   - [ ] Section headers with brief context
   - [ ] References to sub-files ("See [detail.md] for...")
   - [ ] No critical implementation details

3. **Last 10%** — Should contain:
   - [ ] Quick reference / cheatsheet
   - [ ] Common gotchas / anti-patterns
   - [ ] Related skills / see also

### Step 4: Generate recommendations

For each violation found, generate specific fix:

| Violation | Recommendation |
|-----------|----------------|
| >500 lines | Extract sections to sub-files |
| Critical info in middle | Move to primacy or recency zone |
| Nested references | Flatten to one level from SKILL.md |
| Missing TOC | Add table of contents at top |
| Description not third-person | Rewrite as "Processes X" not "I process X" |
| Vague triggers | Add specific "when to use" section |

## Optimization output format

```markdown
## Optimization analysis: [component-name]

**Type:** [Skill / Agent / Plugin / Hook]
**Current lines:** [X] (target: <500)
**Sub-files:** [Y]

### Violations found

🔴 CRITICAL: [line count, nested refs, etc.]
🟡 WARNING: [structure issues]
🟢 GOOD: [things done well]

### Content zone analysis

| Zone | Lines | Should contain | Actually contains |
|------|-------|----------------|-------------------|
| Primacy (1-20%) | 1-X | Purpose, triggers | [what's there] |
| Middle (21-90%) | X-Y | Pointers only | [what's there] |
| Recency (91-100%) | Y-Z | Quick ref | [what's there] |

### Recommended changes

1. **[Change 1]**: [specific action]
2. **[Change 2]**: [specific action]
3. **[Change 3]**: [specific action]

### Proposed new structure

```
component-name/
├── SKILL.md (X lines → target Y lines)
├── [new-subfile-1].md
├── [new-subfile-2].md
└── ...
```

**Proceed with optimization?** [Awaiting confirmation]
```
