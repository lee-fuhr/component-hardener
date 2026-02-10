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
