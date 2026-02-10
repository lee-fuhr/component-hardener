# Hook optimization patterns

Type-specific optimization for Claude Code hooks.

**Hooks are highest risk** — they run automatically without explicit invocation.

## Hook risk profile

| Aspect | Risk level | Why |
|--------|------------|-----|
| Auto-execution | Critical | No user confirmation |
| Blocking capability | Critical | Can halt all operations |
| Shell access | High | Direct command execution |
| Event frequency | High | May run thousands of times |

## Optimal hook structure

Hooks should be **minimal and transparent**.

```
┌─────────────────────────────────────────────────────────┐
│  TRIGGER (absolutely clear)                             │
│                                                         │
│  Event: PreToolUse / PostToolUse / Notification         │
│  Matcher: [exact conditions]                            │
│  Blocking: true/false                                   │
├─────────────────────────────────────────────────────────┤
│  PURPOSE (one line)                                     │
│                                                         │
│  "Prevents accidental commits to main branch"           │
├─────────────────────────────────────────────────────────┤
│  COMMAND (minimal, auditable)                           │
│                                                         │
│  Single script call with clear purpose                  │
│  No chained commands unless necessary                   │
├─────────────────────────────────────────────────────────┤
│  BYPASS (mandatory)                                     │
│                                                         │
│  How to disable if it breaks:                           │
│  - Comment out in settings.json                         │
│  - Environment variable override                        │
└─────────────────────────────────────────────────────────┘
```

## Hook definition example

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "command": "/path/to/guard-script.sh",
        "blocking": true,
        "timeout": 5000
      }
    ]
  }
}
```

## Hook audit checklist

### 1. Trigger clarity
- [ ] Event type is documented
- [ ] Matcher conditions are explicit
- [ ] Blocking behavior is intentional

### 2. Command safety
- [ ] Script path is absolute or well-defined
- [ ] No arbitrary input passed to shell
- [ ] Timeout is set (default 60s may be too long)
- [ ] Exit codes are meaningful

### 3. Bypass mechanism
- [ ] Can be disabled without editing code
- [ ] Documented how to bypass
- [ ] Tested that bypass works

### 4. Performance
- [ ] Executes in <1 second typical
- [ ] Doesn't make network calls (or documents why)
- [ ] Handles errors gracefully

## Red flags in hooks

| Pattern | Risk | Action |
|---------|------|--------|
| `curl` / `wget` in hook | Critical | Data exfiltration risk |
| No timeout | High | Can hang indefinitely |
| Blocking on all events | High | Performance impact |
| Complex shell pipeline | Medium | Hard to audit |
| Reading environment vars | Medium | May leak secrets |
| No error handling | Medium | Silent failures |

## Hook vs skill decision

**Use a hook when:**
- Action must happen automatically
- User shouldn't need to remember
- Consistency is critical (pre-commit checks)

**Use a skill when:**
- User should control invocation
- Action needs context/judgment
- Failure is recoverable

## Common violations and fixes

| Violation | Fix |
|-----------|-----|
| Unclear trigger | Document event + matcher explicitly |
| No bypass | Add disable mechanism |
| Network in hook | Move to skill or justify in docs |
| No timeout | Set explicit timeout (recommend 5s) |
| Complex command | Extract to script, call script |
| Silent failure | Add logging, meaningful exit codes |

## Transformation rules

### Missing trigger documentation

**Detection:** Hook JSON exists but no accompanying markdown documentation
**Fix:** Create `[hook-name].md` alongside or document inline
**Template:**
```markdown
# [Hook name]

**Event:** [PreToolUse / PostToolUse / Notification]
**Matcher:** [Tool name or pattern]
**Blocking:** [true/false]

## Purpose

[One-line description of what this hook prevents or enforces]
```

### Missing bypass mechanism

**Detection:** No bypass documentation in hook file or config
**Fix:** Add bypass section (recency zone of hook docs)
**Template:**
```markdown
## Bypass

To disable this hook:
1. Comment out in `.claude/settings.json` (temporary)
2. Set environment variable: `SKIP_[HOOK_NAME]=1` (per-session)
3. Remove from hooks array (permanent)
```

### No timeout specified

**Detection:** Hook definition lacks `"timeout"` field
**Fix:** Add timeout to hook JSON
**Template:**
```json
{
  "matcher": "[existing]",
  "command": "[existing]",
  "blocking": [existing],
  "timeout": 5000
}
```
**Note:** Default 60s is too long. Recommend 5000ms (5s) for blocking hooks.

### Complex inline command

**Detection:** Hook command contains pipes (`|`), multiple commands (`;`, `&&`), or >50 chars
**Fix:** Extract to script file, reference script in hook
**Template:**
```json
{
  "command": "/path/to/hooks/[hook-name].sh"
}
```
**Script template:**
```bash
#!/bin/bash
# [Hook name] - [purpose]
# Bypass: SKIP_[HOOK_NAME]=1

[[ -n "$SKIP_[HOOK_NAME]" ]] && exit 0

# [Logic here]
exit 0  # or exit 1 to block
```

### Missing error handling in script

**Detection:** Hook script lacks exit code handling or error cases
**Fix:** Add error handling to script
**Template additions:**
```bash
set -euo pipefail

# Meaningful exit codes
# 0 = allow (or success for post-hooks)
# 1 = block (for blocking pre-hooks)
# 2 = error (something went wrong)

trap 'echo "Hook error: $?" >&2; exit 2' ERR
```

### Network calls in hook

**Detection:** Hook contains `curl`, `wget`, `fetch`, or opens URLs
**Fix:** Either remove network call OR document with justification
**Template for justified network:**
```markdown
## Network usage

This hook makes network calls to [destination] for [reason].

**Risk mitigation:**
- Timeout: [X]ms
- Fallback if unreachable: [behavior]
- Data transmitted: [what data, is it sensitive?]
```

### Blocking on overly broad matcher

**Detection:** Blocking hook with matcher like `*` or very common tool
**Fix:** Narrow matcher or make non-blocking
**Questions to ask:**
- Does this REALLY need to run on every [tool] call?
- Can matcher be more specific? (e.g., `Bash` → only certain commands)
- Should it be non-blocking with notification instead?

### Missing purpose statement

**Detection:** Hook exists but purpose unclear (no comment, no docs)
**Fix:** Add purpose in primacy zone
**Template:**
```markdown
## Purpose

[What does this hook prevent?] / [What does this hook enforce?]

Without this hook: [What bad thing would happen?]
```
