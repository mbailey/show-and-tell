---
description: Observe what the user is currently viewing (screen context)
argument-hint: "[options] [target]"
---

# /show-and-tell:look

Capture what the user is currently viewing with tmux hierarchy and pane content.

## Usage

```
/show-and-tell:look [OPTIONS] [TARGET]
```

## Targets

- (none) - Current pane content
- `%<id>` - Specific pane by ID
- `%1,%2,%3` - Multiple panes (comma-separated)
- `window` - All panes in current window
- `<session>:<window>` - All panes in specific window

## Options

- `-l, --lines N` - Lines of scrollback history (default: visible)
- `-H, --hierarchy` - Show tmux hierarchy only (no content)
- `-p, --preserve-blanks` - Preserve exact blank line spacing

## Examples

```
/show-and-tell:look
/show-and-tell:look -H
/show-and-tell:look -l 100
/show-and-tell:look %15,%16
/show-and-tell:look window
```

## Implementation

Run the look command from this plugin:

```bash
${CLAUDE_PLUGIN_ROOT}/bin/look $ARGUMENTS
```
