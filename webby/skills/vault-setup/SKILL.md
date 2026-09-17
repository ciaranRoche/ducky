---
name: vault-setup
description: One-time setup to configure MCPVault for cross-project Obsidian vault access.
  Run this once per machine.
allowed-tools:
  - Bash
argument-hint: '[vault-path]'
---

# Vault Setup: Configure Cross-Project Access

Register the MCPVault MCP server at user scope so every Claude Code session can access the Obsidian vault, regardless of which project directory you're in.

## Behavior

1. **Determine the vault path:**
   - If the user provided a path as an argument, use that
   - Otherwise use `$OBSIDIAN_VAULT` if set, else `$HOME/Obsidian/Notes`
   - Verify the path exists and contains a `.obsidian/` directory using `ls`. If not, stop and ask for the right path; do not register a non-vault directory.
   - Verify the vault uses the layout webby expects: `_AGENT_MANIFEST.md`, `10_Daily/`, `20_Work/`, `30_Personal/`, and `80_System/Templates/` at the root. If any are missing (e.g. only legacy `Daily/`, `Work/`, `Templates/` exist), warn the user that the skills will not find their notes and continue only if they confirm.

2. **Check if MCPVault is already configured:**
   ```bash
   claude mcp list --scope user
   ```
   If an `obsidian` server is already listed, show its current path and ask before reconfiguring.

3. **Register MCPVault at user scope:**
   ```bash
   claude mcp add obsidian --scope user -- npx @bitbonsai/mcpvault@latest "<vault-path>"
   ```

4. **Report the result:** server name, vault path, scope, and that Claude Code needs a restart.

5. **Recommend global install for faster startup:**
   ```bash
   npm install -g @bitbonsai/mcpvault
   claude mcp add obsidian --scope user -- mcpvault "<vault-path>"
   ```

## Output Format

```
### Vault Setup Complete

MCPVault registered at user scope.
- Server: obsidian
- Vault: <path>
- Scope: user (available in all projects)

Restart Claude Code for the MCP server to activate.

Skills after restart:
  /webby:daily          read today's daily note
  /webby:todo           add or complete todos
  /webby:work           pick a todo and start
  /webby:session-log    log the session to the daily note
  /webby:standup        recap the last session log
  /webby:review         weekly roll-up
  /webby:vault-query    search the vault
  /webby:vault-save     save new knowledge to the vault
```

## Notes

- Once per machine. The vault path is machine-specific.
- User-scope registration is the only configuration the plugin relies on. The plugin ships no `.mcp.json`, so there is no hardcoded path to drift.
