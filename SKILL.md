---
name: component-hardener
description: Hardens Claude Code components (skills, agents, hooks, plugins) for security and performance. Actively restructures to match best practices. Use when installing external components, optimizing existing ones, or ensuring structural compliance.
---

# Component hardener

Makes Claude Code components **safe** and **performant**. Actively restructures components to match best practices — not just auditing, but fixing.

**Usage:**
- `/component-hardener [path]` - Harden component (fix + report)
- `/component-hardener [path] --audit-only` - Report without modifying
- `/component-hardener --all` - Harden all installed components
- `/component-hardener --batch [type]` - Harden all of one type (skill/agent/hook)

## When to use

**ALWAYS run when:**
- Installing anything from external sources
- User points to a new component
- Optimizing existing components
- Checking structural compliance

**NEVER skip for:**
- Hooks (critical risk - auto-execute)
- Agents (high risk - autonomous)
- Plugins (bundle multiple components)

## What it does

### Security hardening
Scans for dangerous patterns and blocks unsafe components. See [checklists/security.md](checklists/security.md).

- 🔴 CRITICAL: Blocks installation (eval, exec, network exfiltration)
- 🟡 WARNING: Needs review (shell access, file deletion)
- 🟢 PASS: Safe pattern confirmed

### Structural hardening
**Actively fixes** these issues (not just reports):

**For skills:**
- Adds "When to use" section if missing (primacy zone)
- Adds quick reference table if missing (recency zone)
- Moves buried critical info to appropriate zones
- Adds related skills section
- Extracts inline content >50 lines to sub-files

**For agents:**
- Adds "Core constraints" section if missing
- Adds "Decision authority" section if missing
- Adds "Output format" section if missing
- Adds "When to escalate" section if missing

**For hooks:**
- Adds bypass mechanism (SKIP_HOOK_[name]=1)
- Adds timeout if missing
- Documents network dependencies
- Adds error handling if missing

**For plugins:**
- Audits all bundled components
- Verifies manifest completeness
- Checks for hidden components

## Hardening workflow

### Step 1: Security scan
```bash
# Critical patterns
grep -rn "eval\|exec\|subprocess\|os\.system" .
grep -rn "shutil\.rmtree\|rm -rf" .
grep -rn "curl\|wget\|requests\." .
```

If 🔴 CRITICAL found → **STOP. Do not proceed.**

### Step 2: Identify component type
Detect type from structure:
- `SKILL.md` → Skill
- `AGENT.md` or agent definition → Agent
- Hook registration in settings.json → Hook
- `manifest.json` with components → Plugin

### Step 3: Apply type-specific hardening
Load the appropriate pattern file and **apply fixes**:
- Skills: [patterns/skill.md](patterns/skill.md)
- Agents: [patterns/agent.md](patterns/agent.md)
- Hooks: [patterns/hook.md](patterns/hook.md)
- Plugins: [patterns/plugin.md](patterns/plugin.md)

### Step 4: Write fixed version
**Default behavior:** Write the fixed component, report what changed.

```markdown
## Hardened: [component-name]

**Type:** Skill
**Source:** /path/to/component

### Changes made
1. ✅ Added "When to use" section (primacy zone)
2. ✅ Added quick reference table (recency zone)
3. ✅ Extracted 80-line template to patterns/template.md
4. ✅ Added related skills section

### Structure (before → after)
- Lines: 450 → 185
- Sub-files created: 2
- Zone compliance: ❌ → ✅

### Security
🟢 PASS: No dangerous patterns found
```

## Output format

```markdown
## Hardened: [component-name]

**Type:** [Skill / Agent / Hook / Plugin]
**Source:** [path]
**Mode:** [Fix / Audit-only]

### Security
🔴 CRITICAL: [blockers - STOP if found]
🟡 WARNING: [concerns]
🟢 PASS: [confirmations]

### Structural changes
[List of fixes applied]

### Verification
- [ ] Component loads without error
- [ ] Zone distribution correct (20/70/10)
- [ ] All sub-files readable
- [ ] For hooks: bypass mechanism works
```

---

## Quick reference

| Command | Description |
|---------|-------------|
| `/component-hardener path` | Harden (fix + report) |
| `/component-hardener path --audit-only` | Report without modifying |
| `/component-hardener --all` | Harden all components |
| `/component-hardener --batch skill` | Harden all skills |

## Risk levels

| Type | Risk | Priority |
|------|------|----------|
| Hooks | **Critical** | Harden first |
| Agents | High | Harden second |
| Plugins | High | Harden third |
| Skills | Medium | Harden last |

## Related

- [checklists/security.md](checklists/security.md) — Security patterns
- [checklists/optimization.md](checklists/optimization.md) — Optimization guidelines
- [patterns/](patterns/) — Type-specific transformation patterns
