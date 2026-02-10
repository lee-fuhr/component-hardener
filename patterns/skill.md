# Skill optimization patterns

Type-specific optimization for Claude Code skills.

## Optimal SKILL.md structure

```
┌─────────────────────────────────────────────────────────┐
│  FRONTMATTER                                            │
│  ---                                                    │
│  name: lowercase-hyphenated (≤64 chars)                 │
│  description: Third person, WHAT + WHEN (≤1024 chars)   │
│  ---                                                    │
├─────────────────────────────────────────────────────────┤
│  PRIMACY ZONE (first 20% of lines)                      │
│                                                         │
│  # Skill name                                           │
│  One-line purpose statement.                            │
│                                                         │
│  **Usage:**                                             │
│  - `/skill-name` - basic usage                          │
│  - `/skill-name [arg]` - with argument                  │
│                                                         │
│  ## When to use                                         │
│  - Trigger condition 1                                  │
│  - Trigger condition 2                                  │
├─────────────────────────────────────────────────────────┤
│  POINTER ZONE (middle 70%)                              │
│                                                         │
│  ## Section 1                                           │
│  Brief context. See [detail.md](detail.md) for more.    │
│                                                         │
│  ## Section 2                                           │
│  Brief context. See [patterns.md](patterns.md).         │
│                                                         │
│  [NO detailed implementation here - put in sub-files]   │
├─────────────────────────────────────────────────────────┤
│  RECENCY ZONE (last 10%)                                │
│                                                         │
│  ## Quick reference                                     │
│  | Command | Description |                              │
│  |---------|-------------|                              │
│  | `/cmd1` | Does X      |                              │
│                                                         │
│  ## Related skills                                      │
│  - related-skill-1                                      │
│  - related-skill-2                                      │
└─────────────────────────────────────────────────────────┘
```

## Description best practices

**Good (third person, WHAT + WHEN):**
```yaml
description: Processes PDF files to extract text and tables. Use when working with PDFs, document extraction, or form filling.
```

**Bad (first person, vague):**
```yaml
description: I help you with PDF stuff.
```

## Sub-file organization

```
skill-name/
├── SKILL.md              # <500 lines, overview only
├── patterns/             # Detailed patterns
│   ├── basic.md          # Common use cases
│   └── advanced.md       # Edge cases, complex scenarios
├── examples/             # Concrete examples
│   └── common.md         # Input/output pairs
├── reference/            # Technical details
│   └── api.md            # API reference, schemas
└── scripts/              # Executable utilities (if needed)
    └── helper.py         # Executed, not loaded into context
```

## Refactoring checklist

When a skill exceeds 500 lines:

1. **Identify extractable sections:**
   - Detailed step-by-step instructions → `patterns/`
   - Code examples → `examples/`
   - API reference → `reference/`
   - Background/theory → `concepts/` or remove

2. **Replace with pointers:**
   ```markdown
   # Before (inline)
   ## Authentication
   [50 lines of auth details]

   # After (pointer)
   ## Authentication
   OAuth2 flow with refresh tokens. See [patterns/auth.md](patterns/auth.md).
   ```

3. **Keep in SKILL.md:**
   - Frontmatter (always)
   - Purpose and triggers (primacy zone)
   - Section headers with one-line context
   - Quick reference table (recency zone)

4. **Move to sub-files:**
   - Multi-step workflows
   - Code blocks >10 lines
   - Detailed explanations
   - Edge case handling

## Common violations and fixes

| Violation | Fix |
|-----------|-----|
| >500 lines | Extract to sub-files |
| First-person description | Rewrite as third-person |
| No trigger conditions | Add "## When to use" section |
| Critical info in middle | Move to primacy or recency zone |
| Nested references | Flatten to one level |
| No quick reference | Add table at end |
| Vague description | Add specific trigger keywords |

## Transformation rules

### Missing frontmatter

**Detection:** File doesn't start with `---`
**Fix:** Insert at line 1 (before all content)
**Template:**
```yaml
---
name: [derive from filename, lowercase-hyphenated]
description: [Extract from first paragraph. Third person, WHAT + WHEN. ≤1024 chars]
---
```

### Missing "When to use" section

**Detection:** No `## When to use` heading in first 30 lines
**Fix:** Insert after first heading and purpose statement (primacy zone, ~line 5-10)
**Template:**
```markdown
## When to use

- [Derive from description field - extract trigger scenarios]
- [Look for action verbs in description]
- [Identify input types mentioned]
```

### Missing usage examples

**Detection:** No `**Usage:**` or `## Usage` in first 20 lines
**Fix:** Insert after `# Title` and one-line purpose, before "When to use"
**Template:**
```markdown
**Usage:**
- `/[skill-name]` - [basic invocation]
- `/[skill-name] [arg]` - [with argument if applicable]
```

### Missing quick reference (recency zone)

**Detection:** No `## Quick reference` in last 20 lines
**Fix:** Append to end of file (recency zone)
**Template:**
```markdown
## Quick reference

| Command | Description |
|---------|-------------|
| `/[skill-name]` | [Primary action] |
| `/[skill-name] [arg]` | [With argument] |
```

### Missing related skills section

**Detection:** No `## Related` heading in last 15 lines
**Fix:** Append after quick reference (recency zone, very end)
**Template:**
```markdown
## Related skills

- [Scan for skills mentioned in body, or leave as TODO]
```

### First-person description

**Detection:** Description contains "I ", "my ", "I'll", "I can"
**Fix:** Rewrite description field in frontmatter
**Transform:**
- "I help you process PDFs" → "Processes PDF files for text extraction"
- "I can analyze code" → "Analyzes code for patterns and issues"

### Missing sub-file organization (>500 lines)

**Detection:** File exceeds 500 lines
**Fix:** Create sub-files and replace inline content with pointers
**Template for pointers:**
```markdown
## [Section name]

[One-line context]. See [[section].md]([section].md) for details.
```

### Critical info buried in middle

**Detection:** Important content (triggers, constraints, commands) not in first 20% or last 10%
**Fix:** Extract and relocate
**Primacy zone (first 20%):** Purpose, triggers, usage, constraints
**Recency zone (last 10%):** Quick reference, related skills, key commands
