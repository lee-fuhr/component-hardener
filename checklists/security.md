# Security checklist

Detailed security patterns to check during audit.

## Code execution risks

| Pattern | Risk | What to look for |
|---------|------|------------------|
| `eval()`, `exec()` | Critical | Arbitrary code execution |
| `subprocess`, `os.system`, `os.popen` | Critical | Shell injection |
| `__import__()`, `importlib` | High | Dynamic imports |
| `compile()` | High | Code compilation |
| `setattr()`, `getattr()` with user input | Medium | Attribute injection |

## File system risks

| Pattern | Risk | What to look for |
|---------|------|------------------|
| Paths outside `~/.claude/`, `/Users/lee/CC/` | High | Unexpected access |
| Writes to `/usr/`, `/etc/`, `/bin/` | Critical | System modification |
| `shutil.rmtree`, `rm -rf` | Critical | Recursive delete |
| Symlink creation/manipulation | High | Path traversal |
| `open()` with user-controlled paths | Medium | Path injection |

## Network risks

| Pattern | Risk | What to look for |
|---------|------|------------------|
| Hardcoded external URLs | High | Data exfiltration |
| Non-standard ports (not 80/443) | Medium | Covert channels |
| WebSocket to unknown hosts | High | Persistent connections |
| `curl`, `wget`, `requests` to arbitrary URLs | High | Uncontrolled network access |
| DNS lookups to unusual domains | Medium | DNS exfiltration |

## Credential risks

| Pattern | Risk | What to look for |
|---------|------|------------------|
| Hardcoded API keys/tokens | Critical | Exposed secrets |
| Reading `~/.ssh/`, `~/.aws/`, `~/.config/` | Critical | Credential theft |
| `os.environ` for secrets | Medium | Environment leakage |
| Base64 encoded strings | High | Obfuscated secrets |
| Strings matching key patterns (`sk-`, `ghp_`, etc.) | High | API key patterns |

## Obfuscation red flags

| Pattern | Risk | Action |
|---------|------|--------|
| Base64 encoded code blocks | Critical | Decode and review |
| Hex-encoded payloads | Critical | Decode and review |
| Compressed content that's executed | Critical | Decompress and review |
| Minified/uglified code | High | Request readable version |
| Variable names like `x1`, `a`, `_` | Medium | Possible obfuscation |

## Grep patterns for automated scanning

```bash
# Code execution
grep -rn "eval\|exec\|subprocess\|os\.system\|__import__" .

# File system
grep -rn "shutil\.rmtree\|rm -rf\|/usr/\|/etc/\|/bin/" .

# Network
grep -rn "requests\.\|urllib\|curl\|wget\|socket\." .

# Credentials
grep -rn "api_key\|apikey\|secret\|password\|token\|\.ssh\|\.aws" .

# Obfuscation
grep -rn "base64\|decode\|encode\|\\\\x" .
```
