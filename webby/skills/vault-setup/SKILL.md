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
   - Otherwise, default to `/home/croche/Vault/Notes`
   - Verify the path exists using `ls`

2. **Check if MCPVault is already configured:**
   ```bash
   claude mcp list --scope user
   ```
   If an `obsidian` server is already listed, tell the user and ask if they want to reconfigure.

3. **Register MCPVault at user scope:**
   ```bash
   claude mcp add obsidian --scope user -- npx @bitbonsai/mcpvault@latest [vault-path]
   ```

4. **Report the result:**
   - Confirm the server was registered
   - List the vault path configured
   - Explain that all projects now have access to the vault via `mcp__obsidian__*` tools
   - Note that Claude Code needs to be restarted for the new MCP server to take effect

5. **Recommend global install for faster startup:**
   ```
   npm install -g @bitbonsai/mcpvault
   ```
   This avoids the `npx` fetch delay on each session start. If globally installed, the user can reconfigure with:
   ```bash
   claude mcp add obsidian --scope user -- mcpvault [vault-path]
   ```

## Output Format

```
### Vault Setup Complete

MCPVault registered at user scope.
- Server: obsidian
- Vault: [path]
- Scope: user (available in all projects)

Restart Claude Code for the MCP server to activate.

For faster startup, install globally:
  npm install -g @bitbonsai/mcpvault

Available tools after restart:
  /webby:daily          — read today's daily note
  /webby:session-log    — log session summary to daily note
  /webby:vault-query    — search the vault
  /webby:vault-save     — save new knowledge to the vault
```

## Notes

- This only needs to be run once per machine
- The vault path is machine-specific. On a new machine, run this with the correct path
- The plugin's `.mcp.json` also includes MCPVault as a fallback, but user-scope registration ensures it works from every project
