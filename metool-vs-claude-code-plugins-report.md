# Metool Package Structure vs Claude Code Plugin Structure

## Executive Summary

This report compares the metool package management system with Claude Code's official plugin architecture. While both systems share similar goals (organizing reusable code units with AI assistance capabilities), they have significant structural differences. The good news is that **metool skills are already compatible** with Claude Code's skill discovery, and a **bridge approach** is more practical than a full convention change.

---

## 1. Current Structure Comparison

### Metool Package Structure

```
module-name/
└── package-name/
    ├── README.md        # Required - Human documentation
    ├── SKILL.md         # Optional - Claude Code skill (AI assistance)
    ├── bin/             # Executables -> ~/.metool/bin/
    ├── shell/           # Functions, aliases -> sourced on startup
    │   ├── functions
    │   ├── aliases
    │   ├── completions/
    │   ├── environment
    │   └── path
    ├── config/          # Dotfiles (dot- prefix) -> ~/
    ├── lib/             # Library functions (not symlinked)
    ├── libexec/         # Helper scripts (not in PATH)
    └── docs/            # Additional documentation
```

**Installation:** GNU Stow symlinks + two-stage skill symlinks
- Package -> `~/.metool/skills/<name>` -> `~/.claude/skills/<name>`

### Claude Code Plugin Structure

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json      # Required - Plugin manifest
├── commands/            # Slash commands (.md files)
├── agents/              # Subagent definitions (.md files)
├── skills/              # Agent Skills (subdirectories)
│   └── skill-name/
│       └── SKILL.md     # Required for each skill
├── hooks/
│   └── hooks.json       # Event handler configuration
├── .mcp.json            # MCP server definitions
└── README.md            # Plugin documentation
```

**Installation:** Via `/plugin` command, copied to cache, or local directory reference

---

## 2. What's Already Compatible

### SKILL.md Format (Fully Compatible)

The metool SKILL.md format **already matches** Claude Code's skill requirements:

| Requirement | Metool | Claude Code | Status |
|------------|--------|-------------|--------|
| YAML frontmatter | Yes | Yes | Compatible |
| `name` field (required) | Yes | Yes | Compatible |
| `description` field (required) | Yes | Yes | Compatible |
| Lowercase/hyphen naming | Yes | Yes | Compatible |
| Size guidelines (<5k words) | Yes | Yes | Compatible |

**Example metool SKILL.md (already valid):**
```yaml
---
name: audio
description: Audio processing and music analysis tools. This skill should be used when...
---

# Audio Tools
...
```

### Skill Discovery Path (Compatible)

- Metool symlinks to `~/.claude/skills/<package-name>/`
- Claude Code scans `~/.claude/skills/*/SKILL.md`
- **Result: Metool skills are already discoverable**

### Progressive Disclosure Pattern (Compatible)

Both systems recommend:
- Keep SKILL.md concise (navigation layer)
- Reference `docs/` for detailed content
- Use the Pareto principle (20% commands for 80% use cases)

---

## 3. Key Differences Requiring Attention

### 3.1 Plugin Manifest vs No Manifest

| Aspect | Metool | Claude Code Plugin |
|--------|--------|-------------------|
| Manifest file | None | `.claude-plugin/plugin.json` (required) |
| Version tracking | Git tags | `version` field in manifest |
| Author metadata | README.md | `author` object in manifest |
| Dependencies | README + runtime checks | Not formally specified |

### 3.2 Component Directories

| Component | Metool Location | Claude Code Plugin Location |
|-----------|-----------------|----------------------------|
| Skills | Package root (`SKILL.md`) | `skills/<name>/SKILL.md` |
| Commands | N/A (shell aliases) | `commands/*.md` |
| Agents | N/A | `agents/*.md` |
| Hooks | N/A | `hooks/hooks.json` |
| MCP config | N/A | `.mcp.json` |
| Executables | `bin/` | Not specified (use scripts/) |
| Shell functions | `shell/` | Not supported |
| Dotfiles | `config/` | Not supported |

### 3.3 Installation Mechanism

| Aspect | Metool | Claude Code Plugin |
|--------|--------|-------------------|
| Tool | GNU Stow | Built-in cache system |
| Location | `~/.metool/` | Plugin cache or local |
| Sharing | Git repos + working sets | Marketplace or git URLs |
| Scope | Shell + Claude Code | Claude Code only |

### 3.4 New Claude Code Plugin Features (Not in Metool)

- **Slash Commands** (`/command`) - Interactive workflows in markdown
- **Agents** - Specialized subagent definitions
- **Hooks** - Event handlers (PreToolUse, SessionStart, etc.)
- **MCP Servers** - External tool integration via `.mcp.json`
- **Output Styles** - Custom output formatting
- **LSP Servers** - Language server protocol integration

---

## 4. Specific Recommendations

### 4.1 Do Not Change: Core Metool Structure

Metool serves a **different purpose** than Claude Code plugins:

- **Metool**: Shell environment management (bin/, shell/, config/) + optional AI skills
- **Claude Code plugins**: Pure AI assistant extensions

**Recommendation:** Keep metool structure intact. It serves shell users well.

### 4.2 Add Optional: Plugin Manifest Generation

Create a tool to generate `.claude-plugin/plugin.json` from metool metadata:

```bash
mt package plugin-manifest <package-name>
```

This would:
1. Parse `README.md` for description
2. Use git tags for version
3. Generate compliant `plugin.json`

### 4.3 Add Optional: Commands Directory Support

Allow packages to include slash commands:

```
package-name/
├── SKILL.md
├── commands/         # NEW: Optional slash commands
│   ├── do-thing.md
│   └── another.md
└── ...
```

Metool could symlink or copy `commands/` to the appropriate location.

### 4.4 Add Optional: Hooks Support

Allow packages to include hooks:

```
package-name/
├── SKILL.md
├── hooks/            # NEW: Optional hooks
│   └── hooks.json
└── ...
```

### 4.5 Consider: MCP Server Integration

For packages that need MCP servers:

```
package-name/
├── SKILL.md
├── .mcp.json         # NEW: Optional MCP config
└── ...
```

### 4.6 Update SKILL.md Template

Add optional frontmatter field for tool restrictions:

```yaml
---
name: package-name
description: ...
allowed-tools: Read, Grep, Bash  # Optional: restrict Claude's tools
---
```

---

## 5. Bridge Approach vs Full Conversion

### Option A: Bridge Approach (Recommended)

**Philosophy:** Metool packages remain shell-first tools with optional Claude Code enhancements.

**Implementation:**
1. Keep existing metool structure unchanged
2. Add optional plugin-compatible directories (commands/, hooks/, agents/)
3. Create `mt package export-plugin` to generate standalone Claude Code plugin
4. Add `mt package plugin-manifest` for those who want manifest files

**Pros:**
- No breaking changes
- Backward compatible
- Serves both shell users and Claude Code users
- Allows gradual adoption of new features

**Cons:**
- Two mental models (metool vs plugin)
- Export step needed for pure plugin distribution

### Option B: Full Conversion to Plugin Structure

**Philosophy:** Metool becomes a Claude Code plugin package manager.

**Implementation:**
1. Change package root to include `.claude-plugin/`
2. Move SKILL.md into `skills/<name>/`
3. Add commands/, agents/, hooks/ as standard directories
4. Keep bin/, shell/, config/ for shell integration

**Pros:**
- Native plugin compatibility
- Easier plugin marketplace publishing

**Cons:**
- **Breaking change** for all existing packages
- Loses simplicity of "SKILL.md at root"
- Adds complexity for simple shell tools
- Migration effort for 50+ existing packages

### Recommendation: Bridge Approach

The bridge approach is superior because:

1. **Metool's identity is shell tooling**: Plugins are Claude Code-specific; metool is broader
2. **Skills already work**: No migration needed for current functionality
3. **Incremental enhancement**: Add plugin features as needed per package
4. **No forced complexity**: Simple packages stay simple

---

## 6. Breaking Changes and Migration

### If Keeping Bridge Approach (Recommended)

**No breaking changes required.** Current packages continue working.

Optional enhancements are additive:
- Add `commands/` if you want slash commands
- Add `hooks/` if you want event handlers
- Add `.mcp.json` if you need MCP servers
- Run `mt package plugin-manifest` if you want a manifest

### If Full Conversion Were Chosen

**Required migrations:**

1. Move `SKILL.md` to `skills/<package-name>/SKILL.md`
2. Create `.claude-plugin/plugin.json` for each package
3. Update `mt package install` to handle new structure
4. Update skill symlink logic

**Migration script sketch:**
```bash
# For each package with SKILL.md
mkdir -p skills/"$package_name"
mv SKILL.md skills/"$package_name"/
mkdir -p .claude-plugin
echo '{"name":"'$package_name'","version":"1.0.0",...}' > .claude-plugin/plugin.json
```

---

## 7. Action Items

### Immediate (No Breaking Changes)

1. **Document plugin compatibility** in metool docs
2. **Update SKILL.md template** to include `allowed-tools` example
3. **Add validation** for SKILL.md frontmatter field lengths (name: 64 chars, description: 1024 chars)

### Short-term (Additive Features)

4. **Add `mt package plugin-manifest`** command
5. **Support commands/ directory** in packages
6. **Support hooks/ directory** in packages
7. **Support .mcp.json** in packages

### Medium-term (Optional)

8. **Add `mt package export-plugin`** for standalone plugin export
9. **Consider agents/ support** for subagent definitions
10. **Explore plugin marketplace** integration

---

## 8. Conclusion

Metool and Claude Code plugins serve overlapping but distinct purposes. Metool excels at shell environment management with AI skill support, while Claude Code plugins focus purely on AI assistant extensions.

The current metool skill format is **already compatible** with Claude Code. Rather than disrupting the existing ecosystem, a **bridge approach** that adds optional plugin-compatible features is the most practical path forward.

This allows:
- Existing packages to continue working unchanged
- New packages to optionally include plugin features (commands, hooks, agents)
- Export capability for those who want to publish pure Claude Code plugins
- Gradual adoption without forced migration

---

## Sources

- [Plugins reference - Claude Code Docs](https://code.claude.com/docs/en/plugins-reference)
- [claude-code/plugins/README.md - GitHub](https://github.com/anthropics/claude-code/blob/main/plugins/README.md)
- [Agent Skills - Claude Code Docs](https://code.claude.com/docs/en/skills)
- [Plugin Structure - Claude Skills](https://claude-plugins.dev/skills/@anthropics/claude-plugins-official/plugin-structure)
- [Skill authoring best practices - Claude Docs](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
- [skills/skill-creator/SKILL.md - Anthropic GitHub](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md)
