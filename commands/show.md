---
description: Display files, URLs, or command output for the user to see
argument-hint: <target>
---

# /show-and-tell:show

Display content for the user to view in the appropriate application.

## Usage

```
/show-and-tell:show <target>
```

## Targets

- `<file>` - Open file in Neovim
- `<file>:<line>` - Open file at specific line
- `http://...` or `https://...` - Open URL in browser
- `cmd:<command>` - Run command in shell pane
- `pane:<id>` - Focus tmux pane by ID

## Examples

```
/show-and-tell:show README.md
/show-and-tell:show src/main.py:42
/show-and-tell:show https://github.com/owner/repo
/show-and-tell:show cmd:git status
```

## Implementation

Run the show command from this plugin:

```bash
${CLAUDE_PLUGIN_ROOT}/bin/show $ARGUMENTS
```
