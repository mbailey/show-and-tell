---
name: show-and-tell
description: Visual context sharing. LOAD when user says "show me", "open in browser", or "look at".
---

# show-and-tell

Display content for users (files, URLs, commands) or observe their screen context.

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

## Requirements

- **tmux**: Required for pane management
- **Neovim**: For file display (with socket support)
- **Browser**: For URL display (Firefox preferred)
- **nvim-remote**: Optional, enhances Neovim integration
