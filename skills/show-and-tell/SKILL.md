---
name: show-and-tell
description: Visual context sharing. LOAD when user says "show me", "open in browser", or "look at".
---

# show-and-tell

Display content for users (files, URLs, commands) or observe their screen context.

> **Abstraction layer**: show-and-tell provides high-level "show" and "look" commands that work regardless of whether the user is in tmux, VS Code, or another environment. See [Design Philosophy](docs/commands.md#design-philosophy) for details.

## Quick Reference

| Command | Example | Description |
|---------|---------|-------------|
| `show <file>` | `show README.md:42` | Open file in Neovim at line |
| `show <url>` | `show github.com` | Open URL in browser |
| `show "cmd:..."` | `show "cmd:git log"` | Run command in shell pane |
| `look` | `look -l 100` | Capture pane content |
| `look -H` | `look -H` | Show tmux hierarchy only |

## When to Use

- User asks to open/display a file or URL
- User says "show me" or "look at my screen"
- Working in voice mode (hands-free interaction)
- Need to verify what user is viewing

## Usage

**AI assistants:** Use full path `${CLAUDE_PLUGIN_ROOT}/bin/show` and `${CLAUDE_PLUGIN_ROOT}/bin/look`.

### show - Display Content

```bash
show README.md                    # Open file
show src/main.py:42               # Open at line 42
show https://github.com/repo      # Open URL
show "cmd:git status"             # Run command
show pane:15                      # Focus pane
```

### look - Observe Context

```bash
look                              # Current pane
look -l 100                       # Last 100 lines
look -H                           # Hierarchy only
look %15,%16                      # Multiple panes
look window                       # All panes in window
```

## Documentation

- [Commands](docs/commands.md) - Full command reference
- [Voice Mode](docs/voice-mode.md) - Hands-free workflows
- [Troubleshooting](docs/troubleshooting.md) - Common issues

## Chrome Browser Integration

When Claude-in-Chrome MCP is available, prefer using it for URLs to enable interactive browser control.

### Why Use Chrome MCP

- **Interactive control**: Scroll, click, and interact with page elements
- **Page inspection**: Read page content with `read_page` tool
- **Form filling**: Use `form_input` for automated form completion
- **Screenshots**: Capture visual state with `computer` action: screenshot
- **Better demos**: Live browser interaction for presentations

### Detection and Fallback Pattern

Before showing a URL, check if you have access to `mcp__claude-in-chrome__` tools:

```
AI wants to show URL
    │
    ▼
Does AI have mcp__claude-in-chrome__* tools?
    │
    ├─ YES ──► Call tabs_context_mcp to verify connection
    │              │
    │              ├─ Connected ──► Use navigate tool
    │              │
    │              └─ Not connected ──► Fall back to show command
    │
    └─ NO ───► Use show command directly
```

### Example: Show URL with Chrome MCP

```bash
# Step 1: Check if Chrome MCP is connected
# Call: mcp__claude-in-chrome__tabs_context_mcp

# Step 2: If connected, create a tab and navigate
# Call: mcp__claude-in-chrome__tabs_create_mcp (get a new tab)
# Call: mcp__claude-in-chrome__navigate with the URL and tabId

# Step 3: If not connected, fall back to show command
show https://example.com
```

### Troubleshooting Chrome MCP

| Issue | Solution |
|-------|----------|
| `tabs_context_mcp` fails | Chrome extension not connected; use `show` command |
| Tab ID invalid | Call `tabs_context_mcp` to get fresh tab IDs |
| Extension disconnected mid-session | Graceful fallback to `show` command |
| No Chrome MCP tools available | MCP not configured; use `show` command |

### Setup

To use Chrome MCP:
1. Install the Claude-in-Chrome browser extension
2. Open Chrome and click the Claude extension icon
3. The extension connects to Claude Code via MCP

When the extension is not connected, `tabs_context_mcp` will fail, and you should fall back to the standard `show` command.

## Requirements

- **tmux**: Required for pane management
- **Neovim**: For file display (with socket support)
- **Browser**: For URL display (Firefox preferred, Chrome with MCP preferred for interactive control)
- **nvim-remote**: Optional, enhances Neovim integration
