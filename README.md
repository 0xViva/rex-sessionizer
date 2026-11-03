# rex-sessionizer

**Fuzzy project switching for Rex power users.**

[![License](https://img.shields.io/badge/license-MIT-22c55e?style=flat-square)](LICENSE)
![Version](https://img.shields.io/badge/version-1.2.0-3b82f6?style=flat-square)
[![Rex](https://img.shields.io/badge/built%20for-Rex-6d28d9?style=flat-square)](https://www.superlogical.com/rex/docs/)
![Bash](https://img.shields.io/badge/Bash-3.2%2B-4eaa25?logo=gnubash&logoColor=white&style=flat-square)
[![fzf](https://img.shields.io/badge/fzf-required-f59e0b?style=flat-square)](https://github.com/junegunn/fzf)
[![jq](https://img.shields.io/badge/jq-required-1f2937?style=flat-square)](https://jqlang.org)

> Inspired by [ThePrimeagen's tmux sessionizer](https://github.com/theprimeagen/tmux-sessionizer).

https://github.com/user-attachments/assets/5502d208-934b-4634-9e69-7d0c74adafbd

## ⚡ At a Glance

`rex-sessionizer` is a keyboard-first project and session switcher for Rex.
It searches your project directories and existing sessions in one `fzf`
prompt, then hands the selected context to the Rex app.

Built for people who live in the terminal and treat context switching as a
first-class operation.

> **What is Rex?** Throughout this project, **Rex** means the terminal at
> [superlogical.com/rex](https://www.superlogical.com/rex/docs/) — the
> multiplexer for all work, with a macOS app and Linux/FreeBSD servers, plus
> the `rex` CLI used here to control it. This is not the Rex programming
> language or any other tool named Rex.

> **Power move:** `rex-sessionizer -s 0 --vsplit` reuses a command pane beside
> your current block instead of spawning duplicates. Add `-t SESSION` to run
> it in another session.

## 🧠 Core Features

| Capability | What it gives you |
| --- | --- |
| ⚡ **Fuzzy context switch** | Search sessions and projects in one picker. |
| 📁 **Directory → session** | Turn a project dir into a named Rex session. |
| 🪟 **Reusable command panes** | `-s INDEX` reuses windows and splits. |
| 🎯 **Client-aware handoff** | Picks the sole app client, or the one from `-C`. |
| 🌐 **Multi-server aware** | `-S/--server` targets another Rex host. |
| 🧪 **Safe to inspect** | `--list` and `--dry-run` change nothing. |

## 🚀 Quick Start

### 1 · Dependencies

- [**Rex**](https://www.superlogical.com/rex/docs/) — the terminal this tool
  drives. Install the macOS app (or a Linux/FreeBSD server) and make sure the
  `rex` CLI is on your `PATH`. The app client needs **Remote Control** enabled
  for session switching.
- **`fzf`** — fuzzy finder for the picker
- **`jq`** — parses Rex's JSON output

```sh
# macOS (Homebrew)
brew install fzf jq

# Debian / Ubuntu
sudo apt install fzf jq

# Fedora
sudo dnf install fzf jq
```

### 2 · Install

```sh
git clone https://github.com/0xViva/rex-sessionizer
cd rex-sessionizer
mkdir -p ~/.local/bin
install -m 755 rex-sessionizer ~/.local/bin/
```

Ensure `~/.local/bin` is on your `PATH`:

```sh
# fish
fish_add_path ~/.local/bin

# zsh / bash
export PATH="$HOME/.local/bin:$PATH"
```

### 3 · Configure (required)

There are no built-in search paths: the picker and `--list` need
`REX_SESSIONIZER_SEARCH_PATHS` to be set. Create the config and point it at
**your** project directories:

```sh
mkdir -p ~/.config/rex
cat > ~/.config/rex/rex-sessionizer.conf <<'EOF'
REX_SESSIONIZER_SEARCH_PATHS=(
  ~/code
  ~/src
)
EOF
$EDITOR ~/.config/rex/rex-sessionizer.conf
```

If you cloned the repo, `rex-sessionizer.conf.example` has every option.

> The paths above are placeholders. If none of them exist, the picker will
> list only Rex sessions and no project directories.

Without a config file you can still pass a directory or session explicitly
(`rex-sessionizer ~/code/my-project`), and `-s/--session` still works inside a
Rex terminal. Only project discovery requires the search paths.

### 4 · Run

```sh
rex-sessionizer
```

To verify the install without changing anything:

```sh
rex-sessionizer --list              # sessions + discovered project directories
rex-sessionizer --dry-run           # show what the picker would do
```

> Session switching requires a running
> [Rex](https://www.superlogical.com/rex/docs/) server and a Rex app client with
> **Remote Control** enabled. Without a client, the session is still created
> and `rex-sessionizer` prints a warning instead of failing silently. Use
> `--no-switch` to skip the handoff explicitly.

## 🪟 Session Commands

`-s INDEX` runs a command from `REX_SESSIONIZER_COMMANDS` in a reusable window
(or split) of the current session. This is the same idea as the original
tmux sessionizer's long-running session commands.

```sh
# ~/.config/rex/rex-sessionizer.conf
REX_SESSIONIZER_COMMANDS=(
  "lazygit"
  "nvim"
  "htop"
)
```

```sh
rex-sessionizer -s 0              # lazygit in a new/reused window
rex-sessionizer -s 1 --vsplit     # nvim in a split to the right
rex-sessionizer -s 2 --hsplit     # htop in a split below
rex-sessionizer -s 0 -t alpha     # run in the "alpha" session instead
```

- `-s` normally needs to be run inside a Rex terminal (`REX_SESSION`), so Rex
  knows which session to use. `-t/--target` removes that limitation.
- A pane is reused only while it is still alive. Panes are found by a label
  that encodes the command index, the command, and the working directory; no
  cache has to be kept in sync.
- If you edit a command or change directory, the next `-s` creates a fresh pane
  rather than silently reusing the old one.
- Set `REX_SESSIONIZER_KEEP_OPEN=1` if you want one-shot commands (for example
  `go test ./...`) to leave their pane open after they exit. The default is `0`,
  matching the original behavior: one-shot panes disappear and are recreated
  the next time you run them.

### Editor and Rex bindings

```lua
-- Neovim, from inside a Rex terminal
vim.keymap.set("n", "<C-f>", "<cmd>silent !rex window new picker -- rex-sessionizer<CR>")
vim.keymap.set("n", "<M-h>", "<cmd>silent !rex-sessionizer -s 0<CR>")
vim.keymap.set("n", "<M-t>", "<cmd>silent !rex-sessionizer -s 1<CR>")
```

## ⌨️ Rex Keybindings

Bind the picker in Rex itself rather than in fish, bash, or zsh, so the same
chord works from every pane, tab, and session. Rex reads its bindings from
`~/.config/rex/init.lua` (or `$XDG_CONFIG_HOME/rex/init.lua`).

Add this to `~/.config/rex/init.lua`:

```lua
rex.action{
  name  = "rex_sessionizer",
  title = "Open Rex Sessionizer",
  run   = function()
    rex.session.new_window{
      layout = rex.layout.block{
        flavor  = "com.superlogical.terminal.shell",
        options = { command = { "sh", "-lc", "rex-sessionizer" } },
      },
    }
  end,
}

rex.bind("ctrl+p", "rex_sessionizer")
```

Replace `ctrl+p` with any chord that is free. Rex uses `cmd`, `ctrl`, `alt`,
and `shift` as modifiers; see the
[Keyboard Shortcuts](https://www.superlogical.com/rex/docs/customize/keyboard-shortcuts)
docs for the full key syntax and sequences.

Then apply it:

```sh
rex config check     # validate init.lua (optional)
rex config reload    # make the binding live
```

`rex config reload` only works on the machine running the Rex server. After
that, `Ctrl+P` opens a fresh picker window from anywhere in Rex. You can also
run the action by name from the command palette or with `rex do rex_sessionizer`.

## ⚙️ Configuration

The global config is `~/.config/rex/rex-sessionizer.conf` (or
`$XDG_CONFIG_HOME/rex/rex-sessionizer.conf`). A `rex-sessionizer.conf` in the
directory you launch from is sourced **after** the global file, so it can
override settings for that directory.

> **Security note:** both config files are shell scripts, so they can run
> arbitrary commands. Only launch `rex-sessionizer` from directories you trust.

`REX_SESSIONIZER_SEARCH_PATHS` has no built-in default: set it in the config,
or the picker and `--list` refuse to run.

| Variable | Default | Purpose |
| --- | --- | --- |
| `REX_SESSIONIZER_SEARCH_PATHS` | **required** (no built-in default) | Directories to scan. |
| `REX_SESSIONIZER_EXTRA_SEARCH_PATHS` | empty | Extra directories appended to the list. |
| `REX_SESSIONIZER_MAX_DEPTH` | `1` | Default scan depth. |
| `REX_SESSIONIZER_EXCLUDES` | `(.git)` | Directory names to prune. |
| `REX_SESSIONIZER_INCLUDE_HIDDEN` | `0` | Include hidden directories. |
| `REX_SESSIONIZER_FOLLOW_SYMLINKS` | `0` | Follow symlinked directories. |
| `REX_SESSIONIZER_COMMANDS` | empty | Commands used by `-s/--session`. |
| `REX_SESSIONIZER_CLIENT` | `$REX_CLIENT` | App client label or ID. |
| `REX_SESSIONIZER_SERVER` | empty (Rex's `$REX_SERVER` applies) | Rex server label or address. |
| `REX_SESSIONIZER_TARGET` | empty | Default session for `-s`. |
| `REX_SESSIONIZER_KEEP_OPEN` | `0` | Keep panes open after their command exits. |
| `REX_SESSIONIZER_HYDRATE` | `1` | Source `.rex-sessionizer` in new sessions. |
| `REX_SESSIONIZER_SWITCH` | `1` | Switch the app to the selected session. |
| `REX_SESSIONIZER_LOG` | empty | `echo`, `file`, or empty. |
| `REX_SESSIONIZER_LOG_FILE` | `$XDG_DATA_HOME/rex-sessionizer/rex-sessionizer.log` | Log path for `LOG=file`. |

Search entries can be written as `DIR:DEPTH` to override the global depth:

```sh
REX_SESSIONIZER_SEARCH_PATHS=(
  ~/code
  ~/work:3
  ~/personal
)
```

### Hydration

When a project directory is turned into a new session, `rex-sessionizer` types
`. <file>` into that session's terminal, where `<file>` is the first of:

1. `<project>/.rex-sessionizer`
2. `~/.rex-sessionizer`

This is useful for setting environment variables, changing directory, or
starting a shell prompt. Disable it with `--no-hydrate` or
`REX_SESSIONIZER_HYDRATE=0`.

## 🩺 Troubleshooting

| Symptom | Check |
| --- | --- |
| Nothing happens after selecting a session | Is the Rex app connected with **Remote Control** on? The script prints a warning when no app client is connected. |
| `session target ... is ambiguous` | Two sessions share a label. Rename one, or pass a full session ID. |
| `-s` says `REX_SESSION is not set` | Run it inside a Rex terminal, or pass `-t/--target`. |
| The picker only shows sessions | Every configured search path was missing, pruned, or hidden. Check `--list` and `REX_SESSIONIZER_INCLUDE_HIDDEN`. |
| An old command pane is reused | The pane label includes the command and cwd; if you expected a new pane, check that the config actually changed. |
| No logs | Set `REX_SESSIONIZER_LOG=echo` or `REX_SESSIONIZER_LOG=file`. |

## 📄 License & Contact

MIT © 2026 rex-sessionizer contributors · Questions? [@0xViva](https://github.com/0xViva)
