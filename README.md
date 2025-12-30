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

### Optional Enhancements

- **nvim-remote** - Enhanced Neovim socket integration
  - Provides automatic socket detection and richer editor status
  - Without it: show-and-tell uses calculated socket paths (works fine)
  - With it: More flexible socket discovery across multiple Neovim instances

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

## Optional Integrations

### nvim-remote (optional)

If nvim-remote is available, show-and-tell uses it for enhanced features:

- **Auto socket detection**: Finds the best Neovim socket automatically
- **Richer status**: More detailed editor state in `look` output

Without nvim-remote, show-and-tell:
- Uses calculated socket paths: `/tmp/nvim-tmux-pane-<pane_id>`
- Uses direct `nvim --server` commands for file operations
- **All core features work fully**

## Privacy

The `look` command captures screen content. Use responsibly and only when contextually appropriate.

## See Also

- [SKILL.md](SKILL.md) - Claude Code skill for AI integration
