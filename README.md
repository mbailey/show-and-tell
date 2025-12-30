# show-and-tell

Visual context sharing between AI assistants and users.

Like the childhood classroom activity where kids show something and tell about it:
- **show**: AI displays content for the user (files in Neovim, URLs in browser, commands in tmux)
- **look**: AI observes what the user is viewing (screen context, active panes)

These are complementary - look verifies what show displayed.

## Requirements

### Required

- **tmux** - Terminal multiplexer for pane management and context capture
  - Used by both `show` and `look` commands
  - `show` creates a dedicated "show" session for content display
  - `look` captures pane content and displays hierarchy

### Recommended

- **neovim package** - Provides `nvim-remote` and `nvim-socket` utilities
  - `show` uses nvim-remote to open files in existing Neovim instances
  - `look` detects Neovim and queries file/position via socket
  - Install: `mt package install neovim`

- **tmux package** - Enhanced tmux utilities and Claude Code skill
  - Complements show-and-tell with tmux management
  - Install: `mt package install tmux`

### Optional

- **Browser** - For URL display (Firefox preferred, configurable via SHOW_BROWSER)

## Installation

```bash
mt package install show-and-tell
```

Or add to your working set:

```bash
mt package add metool-packages-dev/show-and-tell
```

## Commands

### show

Display content for the user:

```bash
show path/to/file.py          # Open file in Neovim
show path/to/file.py:42       # Open file at line 42
show https://example.com      # Open URL in browser
show "cmd:git status"         # Run command in shell pane
```

### look

Observe what the user is viewing:

```bash
look                          # Capture current pane context
look --hierarchy              # Show tmux session/window/pane layout
```

## Integration with Other Packages

### neovim package

The show-and-tell package uses these utilities from the neovim package:

- **nvim-remote**: Opens files in existing Neovim instances
  - `show file.py` → `nvim-remote edit file.py`
  - `show file.py:42` → `nvim-remote edit file.py 42`

- **nvim-socket**: Manages Neovim socket discovery
  - Auto-detects sockets at `/tmp/nvim-tmux-pane-*`
  - Used by `look` to query editor state

### tmux package

Both packages work together for terminal management:

- show-and-tell's `look -H` shows tmux hierarchy
- tmux package provides window/pane utilities
- Combined: Navigate with tmux skill, display with show, observe with look

## Privacy

The `look` command captures screen content. Use responsibly and only when contextually appropriate.

## See Also

- [SKILL.md](SKILL.md) - Claude Code skill for AI integration
