---
name: show-and-tell
description: Visual context sharing between AI and users. Use this skill whenever the user asks you to show something or to look at their screen content. Essential for voice mode and hands-free interaction.
---

# show-and-tell Skill

Visual context sharing between AI assistants and users through display and observation commands.

## Purpose

This skill enables bidirectional visual communication:
- **show**: Display content for the user to see (files, URLs, command output)
- **look**: Observe what the user is currently viewing (screen context)

Like show-and-tell in school: AI shows something, then tells about it; or looks at what the user shows, then discusses it.

## When to Use This Skill

Use this skill when:
- User asks you to open or display a file
- User wants to see a URL or webpage
- You need to run a command and show the user the output
- You need to understand what the user is looking at
- Working in voice mode where the user can't easily type
- You want to verify that content was displayed correctly

## Commands

> **Note for AI assistants:** When running these commands, use the full path `${CLAUDE_PLUGIN_ROOT}/bin/show` and `${CLAUDE_PLUGIN_ROOT}/bin/look`. This ensures the commands work correctly regardless of installation method. Examples below show the short form for readability.

### show - Display Content

Opens content for the user to view in the appropriate application.

```
Usage: show <target> [OPTIONS]

Targets:
  <file>                   Open file in Neovim
  <file>:<line>            Open file at specific line
  <file>#L<line>           Open file at specific line (URL fragment style)
  http://... https://...   Open URL in browser
  cmd:<command>            Run command in shell pane
  pane:<id>                Focus tmux pane by ID

Options:
  -s, --session NAME       Tmux session for show window (default: show)
  -p, --pane ID            Target specific pane ID
```

#### Open files in Neovim

```bash
show README.md                    # Open file in Neovim
show src/main.py:42               # Open at line 42
show src/main.py#L42              # Same (URL fragment style)
show ~/Code/project/config.yaml   # Absolute path
```

The show command:
- Uses nvim-remote to open files in existing Neovim instances
- Falls back to starting Neovim in the "show" tmux session
- Waits up to 3 seconds for Neovim socket to become available
- Supports line numbers in both `:N` and `#LN` formats

#### Open URLs in browser

```bash
show https://github.com/owner/repo
show https://docs.python.org/3/library/asyncio.html
show github.com                   # Auto-adds https://
```

Opens in Firefox by default (falls back to system default browser).

#### Run commands in shell pane

```bash
show "cmd:git status"             # Show git status
show "cmd:git diff HEAD~1"        # Show recent changes
show "cmd:ls -la"                 # List files
show "cmd:docker ps"              # Show running containers
show "cmd:pytest -v"              # Run tests with output
```

Commands run in the shell pane of the "show" tmux session, allowing the user to see live output.

#### Focus specific pane

```bash
show pane:15                      # Focus pane %15
show pane:%23                     # With explicit % prefix
```

#### Advanced options

```bash
show file.py -s dev               # Use "dev" session instead of "show"
show file.py -p %36               # Open in specific pane
```

### look - Observe Context

Captures what the user is currently viewing with tmux hierarchy and pane content.

```
Usage: look [OPTIONS] [TARGET]

Targets:
  (none)                   Current pane content
  %<id>                    Specific pane by ID
  %1,%2,%3                 Multiple panes (comma-separated)
  window                   All panes in current window
  <session>:<window>       All panes in specific window

Options:
  -l, --lines N            Lines of scrollback history (default: visible)
  -H, --hierarchy          Show tmux hierarchy only (no content)
  -p, --preserve-blanks    Preserve exact blank line spacing
```

#### Basic observation

```bash
look                              # Current pane - visible screen
look -l 100                       # Last 100 lines of scrollback
look -l 500                       # More history for long output
```

#### View tmux hierarchy only

```bash
look -H
```

Output example:
```
Tmux summary:
→ main: *claude[%15 %16], logs[%23] (attached)
  dev: editor[%30], tests[%31 %32]
  (in main:0 pane %15)
```

Legend:
- `→` indicates current session
- `*` indicates active window
- Pane IDs shown in brackets

#### Multiple panes

```bash
look %15,%16                      # Two specific panes
look window                       # All panes in current window
look main:0                       # All panes in main session window 0
```

#### Preserve formatting

```bash
look -p                           # Keep exact blank line spacing
look -p -l 50                     # With scrollback
```

### Output Format

The `look` command provides structured output:

```
Tmux summary:
→ main: *show[%15], claude[%16 %17] (attached)
  (in main:0 pane %15)

[Neovim detected in pane %15]
File: /Users/admin/project/src/main.py
Position: line 42, column 8
Mode: n
Tip: Use nvim-remote for full editor control

--- Pane %15 content ---
(Showing visible screen. Use -l 200 for more history)
[actual pane content here]
```

## Sensitive Content Detection

The `look` command automatically scans captured content for potentially sensitive information:

### Detected patterns

- Password assignments: `password = "..."`, `PASSWORD: ...`
- API keys: `api_key = "..."`, `API_KEY=...`
- Secrets/tokens: `secret = "..."`, `token = "..."`
- Long uppercase strings (20+ chars): Potential API keys
- Private keys: `-----BEGIN PRIVATE KEY-----`

### Warning output

When sensitive content is detected, look adds a warning:

```
Warning: Potential sensitive information detected
Consider reviewing before sharing. Use -H to show hierarchy only without content.
```

### Best practices

- Review output before sharing in logs or conversations
- Use `look -H` to get hierarchy without content when sensitive data may be present
- Be aware that terminal history may contain secrets from previous commands

## Voice Mode Workflows

These commands are designed for hands-free voice interaction where typing is inconvenient.

### Basic file review workflow

```
User (voice): "Show me the main config file"
AI: show config/settings.yaml
AI (voice): "I've opened settings.yaml in Neovim for you"

User (voice): "What's in there?"
AI: look
AI (voice): "I can see the database configuration with host localhost
            port 5432, and logging set to debug level..."
```

### Code navigation workflow

```
User (voice): "Open the user authentication module"
AI: show src/auth/login.py
AI (voice): "I've opened the login module"

User (voice): "Go to where we handle the JWT tokens"
AI: show src/auth/login.py:156
AI (voice): "Jumped to line 156 where JWT validation happens"

User (voice): "What does that function look like?"
AI: look -l 30
AI (voice): "The validate_token function takes a token string,
            decodes it using the secret key, checks expiration..."
```

### Command output workflow

```
User (voice): "Show me what containers are running"
AI: show "cmd:docker ps"
AI (voice): "Running docker ps now"

User (voice): "What do you see?"
AI: look
AI (voice): "There are 3 containers running: postgres on port 5432,
            redis on 6379, and nginx on port 80"
```

### Debugging workflow

```
User (voice): "Run the tests and show me the output"
AI: show "cmd:pytest tests/ -v"
AI (voice): "Running pytest with verbose output"

[wait for tests to complete]

User (voice): "Did they pass?"
AI: look -l 100
AI (voice): "I can see 23 tests ran. 21 passed, but 2 failed.
            The failures are in test_auth.py - test_login_invalid_password
            and test_token_expiry. Want me to show you those test files?"
```

### Multi-pane observation

```
User (voice): "What's happening in all my panes?"
AI: look window
AI (voice): "You have 3 panes open. Pane 15 has Neovim editing main.py,
            pane 16 shows the output of your last git status, and
            pane 17 has a Python REPL with some test variables loaded."
```

### Context verification

```
User (voice): "I'm looking at something, can you see it?"
AI: look
AI (voice): "Yes, I can see you're viewing the error logs.
            There's a ConnectionRefused error from 2 minutes ago
            when trying to connect to the database on port 5432."
```

## Integration with Other Skills

### neovim skill (optional)

If nvim-remote is available, `show` uses it for enhanced Neovim integration:
- Automatic socket discovery
- Richer editor status in `look` output

Without nvim-remote, show-and-tell uses direct `nvim --server` commands with calculated socket paths. All core features work fully.

### tmux (required)

Both commands use tmux directly for:
- Pane identification and capture (`look`)
- Session/window management (`show` creates "show" session)
- Running commands in shell panes (`show cmd:...`)
- Hierarchy display (`look -H`)

## Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `SHOW_SESSION` | `show` | Default tmux session name for show window |
| `SHOW_BROWSER` | (auto) | Browser for URLs (e.g., `Firefox`, `Chrome`, `Safari`) |
| `NVIM_SOCKET_PATH` | (auto) | Override Neovim socket path |

## Troubleshooting

### show: File doesn't open

1. Check if Neovim socket exists:
```bash
nvim-socket list              # From neovim package
ls /tmp/nvim-tmux-pane-*      # Socket files
```

2. Verify tmux is running:
```bash
tmux list-sessions
```

### show: URL doesn't open

Verify browser availability:
```bash
which firefox                 # macOS/Linux
ls /Applications/Firefox.app  # macOS
```

### look: Empty output

1. Ensure you're in a tmux session:
```bash
echo $TMUX                    # Should show tmux socket path
```

2. Check pane has content:
```bash
tmux capture-pane -p          # Direct tmux capture
```

### look: Neovim not detected

1. Verify Neovim is running with socket:
```bash
nvim --listen /tmp/nvim-tmux-pane-15 file.txt
```

2. Check socket is responsive:
```bash
nvim --server /tmp/nvim-tmux-pane-15 --remote-expr "1"
```

## Installation and Running Commands

### Finding the Commands

The `show` and `look` commands may be installed in different locations depending on how the package was installed:

**If installed as a Claude Code plugin:**
```bash
# Use the plugin root variable
${CLAUDE_PLUGIN_ROOT}/bin/show <target>
${CLAUDE_PLUGIN_ROOT}/bin/look [options]
```

**If installed via metool:**
```bash
# Commands are in PATH
show <target>
look [options]

# Or explicitly via metool bin
~/.metool/bin/show <target>
~/.metool/bin/look [options]
```

**Important:** The system has a `look` command (dictionary lookup utility) that may shadow this package's `look` command. If `look` doesn't work as expected, use the full path.

### Checking Installation

To verify the commands are available:
```bash
# Check if in PATH
which show look

# Check metool installation
ls ~/.metool/bin/show ~/.metool/bin/look

# Check plugin installation (if CLAUDE_PLUGIN_ROOT is set)
ls ${CLAUDE_PLUGIN_ROOT}/bin/show ${CLAUDE_PLUGIN_ROOT}/bin/look
```

## Requirements

### Required
- **tmux**: Required for session/pane management (show) and screen capture (look)
- **Neovim**: Required for file display (with `--listen` socket support)

### Optional
- **Browser**: Firefox preferred for URL display (falls back to system default)
- **nvim-remote**: Enhanced Neovim integration with auto socket detection
  - Without it: Uses calculated socket path `/tmp/nvim-tmux-pane-<id>`
  - With it: Automatic socket discovery and richer editor status

## References

- [Claude Code Plugins Documentation](https://code.claude.com/docs/en/plugins)
- [Plugins Reference](https://code.claude.com/docs/en/plugins-reference)
- [Agent Skills Documentation](https://code.claude.com/docs/en/skills)
