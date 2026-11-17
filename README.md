# claude-nvim

Write Claude Code prompts in Neovim instead of Claude's input box.

`claude-nvim` opens one tmux window with Claude Code, a Neovim prompt buffer and a shell.
You write the prompt in Neovim with your normal keymaps, press Enter, and it is sent to Claude.

```
┌─────────────────────┬─────────────────────┐
│ Claude Code         │ shell               │
│ (read replies here) │ (type y for yazi)   │
├─────────────────────┤                     │
│ Neovim prompt       │                     │
│ (write, <CR> sends) │                     │
└─────────────────────┴─────────────────────┘
```

## Files

| File | Installed to | What it does |
|---|---|---|
| `claude-nvim` | `~/.local/bin/claude-nvim` | Bash launcher that creates the tmux layout and starts Claude Code |
| `claude-prompt.lua` | `~/.config/nvim/plugin/claude-prompt.lua` | Neovim plugin: the prompt buffer, sending, drafts and history |

The plugin goes in `plugin/`, not `lua/plugins/`. Neovim loads everything in `plugin/` automatically, so lazy.nvim does not need to know about it.

## Requirements

- macOS or Linux
- tmux 3.x
- Neovim 0.10 or newer (uses `vim.system`, `vim.fn.getregion` and `vim.keycode`)
- [Claude Code](https://claude.com/claude-code) (`claude` on your `PATH`)

## Install

```sh
cp claude-nvim ~/.local/bin/claude-nvim
chmod +x ~/.local/bin/claude-nvim
cp claude-prompt.lua ~/.config/nvim/plugin/claude-prompt.lua
```

Make sure `~/.local/bin` is on your `PATH`. After editing either file here, copy it again to update the installed version.

## Usage

```sh
claude-nvim               # start in the current folder
claude-nvim --continue    # any arguments are passed to claude
```

Run it inside or outside tmux. Outside tmux, it starts a new tmux session first.

### In the prompt buffer

| Key | Mode | Action |
|---|---|---|
| `<CR>` | normal | Send the whole buffer to Claude, then clear it |
| `<C-s>` | normal, insert | Same as `<CR>` |
| `<leader>as` | visual, any buffer | Send the selection as it is |

Insert mode `<CR>` is a normal newline, so multi-line prompts work as expected.

### Commands

| Command | Action |
|---|---|
| `:ClaudePrompt` | Open the prompt buffer (the launcher does this for you) |
| `:ClaudeHistory` | Open the list of every prompt you have sent |

## Settings

Set these as environment variables when starting the launcher:

| Variable | Default | Meaning |
|---|---|---|
| `CLAUDE_FILES_WIDTH` | `50%` | Width of the right pane |
| `CLAUDE_PROMPT_HEIGHT` | `25%` | Height of the prompt pane |
| `CLAUDE_FILES_CMD` | empty | Command typed into the right pane on start. Empty means a plain shell |

Example: `CLAUDE_FILES_CMD=y CLAUDE_FILES_WIDTH=35% claude-nvim`

## How it works

**Sending.** The plugin loads the text into a tmux buffer (`tmux load-buffer`), pastes it into Claude's pane with bracketed paste (`paste-buffer -p`), then presses Enter 150 ms later. Bracketed paste makes the newlines arrive as line breaks instead of submitting the prompt early.

**Finding Claude's pane.** The launcher tags Claude's pane with the tmux option `@claude 1` and passes its pane ID to Neovim as `CLAUDE_PANE`. The plugin only sends to a pane with that tag. If it can't find one, it shows an error and sends nothing. It never guesses, because sending to the wrong pane could paste the prompt into a shell and run it as commands.

**Drafts.** The prompt buffer saves on every change, with no swap file, so closing the window (`prefix &`) loses nothing. There is one draft per project folder, so two Claude windows never share a draft.

## Files it writes

| Path | Contents |
|---|---|
| `~/.local/state/nvim/claude-prompts/<folder>.md` | Unsent draft for each project folder (`/` in the path becomes `%`) |
| `~/.local/state/nvim/claude-prompt-history.md` | Every prompt you have sent, with date and folder |

Nothing else is changed.

## Notes

- **Restoring with tmux-resurrect.** A restored window gets new pane IDs and loses the `@claude` tag, so the prompt pane can't find Claude and won't send. Close the restored window and run `claude-nvim` again.
- **"No Claude pane in this tmux window."** The prompt buffer was opened in a window that `claude-nvim` didn't start, or Claude's pane was closed. Run `claude-nvim` again.

## Uninstall

```sh
rm ~/.local/bin/claude-nvim ~/.config/nvim/plugin/claude-prompt.lua
```

To also remove your drafts and history, delete `~/.local/state/nvim/claude-prompts/` and `~/.local/state/nvim/claude-prompt-history.md`.
