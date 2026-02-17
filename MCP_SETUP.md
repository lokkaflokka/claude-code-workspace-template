# MCP Workspace Setup

Guide for setting up MCP tools in a work context, isolated from personal infrastructure.

---

## Why Isolated MCP

If you have personal MCP servers, the work machine must NOT connect to them. Personal and work MCP are completely separate: separate instances, configs, packages, credentials.

---

## How It Works

The hub uses a **plugin manifest interface**. Each domain package exports a manifest declaring its tools. The hub discovers and registers them automatically.

```
~/.config/mcp-toolkit-hub/config.yaml
    |
mcp-toolkit-hub (orchestrator — generic, no domain logic)
    | loads manifest.ts from each enabled package
domain packages (your tools)
    |
namespaced tools (package_toolname)
```

### Adding a Package

1. Create a package with `src/lib/manifest.ts`:

```typescript
import { z } from 'zod';

export const manifest = {
  name: 'my-package',
  version: '1.0.0',
  tools: [
    {
      name: 'my_tool',
      description: 'What this tool does',
      schema: {
        query: z.string().describe('The search query'),
      },
      handler: async (args: any) => {
        // Your logic here
        return `Result: ${args.query}`;
      },
    },
  ],
};
```

2. Add to config:
```yaml
# ~/.config/mcp-work-hub/config.yaml
schema_version: "1.0"
packages:
  my-package:
    path: ~/work-mcp/my-package
    enabled: true
```

3. Build both: `npm run build` in package and hub.

**No hub source changes needed.** The hub is a generic registrar.

---

## Work MCP Setup (Phased)

### Phase 0: No MCP (Week 1-2)

Claude Code works well without MCP. Focus on knowledge capture first. MCP adds complexity — earn the need.

### Phase 1: Read-Only Connectors (Week 3-4)

When a tool would save real time:

1. Clone mcp-toolkit-hub (it's generic — no personal logic to strip)
2. Create a work domain package with `manifest.ts`
3. Create a work config at `~/.config/mcp-work-hub/config.yaml`
4. Build and register in Claude Code config

**Likely first candidates:**

| Tool | What It Does | Sensitivity |
|------|-------------|-------------|
| GitHub/GitLab | Repo search, PR listing | Low |
| Jira/Linear | Ticket queries, board status | Low |
| Warehouse metadata | Schema inspection, explain plans | Medium — no raw data |

### Phase 2: Write-Capable Tools (Month 2+)

After trial period and company-credentialed access:
- Ticket creation (with confirmation gates)
- PR assistance
- Report generation

**Rule:** Every write-capable tool requires explicit user confirmation.

---

## Security Checklist

- [ ] Work MCP config is separate from personal config
- [ ] No personal credentials in work config
- [ ] No work credentials in personal config
- [ ] All write-capable tools have confirmation gates
- [ ] `.env` files are gitignored
- [ ] No secrets in package source code

---

## Personal vs Work MCP

| Aspect | Personal | Work |
|--------|----------|------|
| Config | `~/.config/mcp-toolkit-hub/config.yaml` | `~/.config/mcp-work-hub/config.yaml` |
| Packages | Your personal domain packages | Work-specific packages |
| Credentials | Personal OAuth/API keys | Work OAuth/API keys |
| Machine | Personal laptop | Work laptop |

They never share configs, credentials, or data paths.
