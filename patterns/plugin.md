# Plugin optimization patterns

Type-specific optimization for Claude Code plugins.

## Plugin structure

Plugins bundle multiple components:

```
plugin-name/
├── manifest.json         # Plugin metadata + component list
├── skills/               # Bundled skills
│   ├── skill-1/
│   │   └── SKILL.md
│   └── skill-2/
│       └── SKILL.md
├── hooks/                # Bundled hooks (HIGHEST RISK)
│   └── pre-commit.sh
├── agents/               # Bundled agents
│   └── agent-1.md
└── mcp/                  # MCP server configs
    └── server.json
```

## Plugin audit priority

Audit components in this order (highest risk first):

1. **Hooks** — Run automatically, can block operations
2. **MCP servers** — External connections, persistent
3. **Agents** — Autonomous, can take actions
4. **Skills** — User-invoked, most transparent

## Manifest review

```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "What this plugin does",
  "author": "author-name",
  "repository": "https://github.com/...",
  "skills": ["skill-1", "skill-2"],
  "hooks": ["pre-commit"],
  "agents": ["agent-1"],
  "mcp": ["server"]
}
```

### Manifest checklist

- [ ] Author is identifiable
- [ ] Repository is accessible
- [ ] All components are listed
- [ ] Version follows semver
- [ ] No hidden components (check filesystem vs manifest)

## MCP server concerns

MCP servers create persistent connections to external services.

**Audit questions:**
- What service does it connect to?
- What credentials does it need?
- What data can it access?
- Can it be used offline?

**Red flags:**
- Connects to unknown/unusual hosts
- Requires broad permissions
- No documentation of data flow
- Credentials stored in plain text

## Plugin installation checklist

### Before installing

1. [ ] Review manifest.json
2. [ ] Audit all hooks (priority 1)
3. [ ] Review MCP server configs
4. [ ] Audit all agents
5. [ ] Audit all skills
6. [ ] Check repository reputation

### During installation

1. [ ] Install from source, not binary
2. [ ] Verify checksums if provided
3. [ ] Review what gets written where

### After installation

1. [ ] Test each component individually
2. [ ] Verify hooks don't break workflow
3. [ ] Check MCP connections work
4. [ ] Document any required setup

## Common violations and fixes

| Violation | Fix |
|-----------|-----|
| Hidden components | Compare manifest to filesystem |
| Undocumented hooks | Require hook documentation |
| Unknown MCP hosts | Verify or reject |
| No author info | Research or reject |
| Bundled secrets | Require credential setup step |
| Missing version | Pin or reject |

## Marketplace trust levels

| Source | Trust level | Action |
|--------|-------------|--------|
| `anthropics/claude-plugins-official` | High | Audit hooks only |
| Known author with history | Medium | Full audit |
| Unknown author | Low | Full audit + sandbox test |
| No repository | Very low | Reject unless compelling reason |
