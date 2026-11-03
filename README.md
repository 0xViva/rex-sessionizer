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

One executable, one config file, and one Rex binding. Nothing needs to be
sourced from your shell or arranged in a dotfiles repo.

### 1 · Dependencies

- [**Rex**](https://www.superlogical.com/rex/docs/) — install the app or a
  Linux/FreeBSD server and make sure the `rex` CLI is on your `PATH`. The app
  client needs **Remote Control** enabled for session switching.
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

The Rex binding runs the picker on the machine hosting the Rex server, so
`rex`, `rex-sessionizer`, `fzf`, and `jq` must be installed there too. For a
local server, that is the same machine.

### 2 · Install

```sh
git clone https://github.com/0xViva/rex-sessionizer
cd rex-sessionizer
mkdir -p ~/.local/bin
install -m 755 rex-sessionizer ~/.local/bin/
```

`~/.local/bin` is a common choice; any directory on your `PATH` works. If
`~/.local/bin` is not already on your `PATH`, add it in your shell's startup
file (for example, `export PATH="$HOME/.local/bin:$PATH"` in bash/zsh, or
`fish_add_path ~/.local/bin` in fish). Then verify:

```sh
rex-sessionizer --version
```

With a remote Rex server, install the script on that machine too.

### 3 · Configure project directories

The picker needs at least one search path. Create the config file next to
Rex's `init.lua` (skip the `cat` if the file already exists):

```sh
config_dir="${XDG_CONFIG_HOME:-$HOME/.config}/rex"
mkdir -p "$config_dir"
cat > "$config_dir/rex-sessionizer.conf" <<'EOF'
REX_SESSIONIZER_SEARCH_PATHS=(
    ~/code
)
EOF
```

Replace `~/code` with the directory or directories that contain your
projects. Paths that do not exist are skipped. If `XDG_CONFIG_HOME` is unset,
the file is `~/.config/rex/rex-sessionizer.conf`. With a remote Rex server,
create this file on the server machine.

Check what the picker will see:

```sh
rex-sessionizer --list
```

`--list` prints Rex sessions and discovered project directories without
changing anything. Without a config file you can still pass a directory or a
session explicitly (`rex-sessionizer ~/code/my-project`); only project
discovery needs search paths.

### 4 · Bind it in Rex

Add this to Rex's `init.lua` (`~/.config/rex/init.lua`, or
`$XDG_CONFIG_HOME/rex/init.lua` when `XDG_CONFIG_HOME` is set). The server
reads this file, so for a remote server edit it on the server machine:

```lua
rex.action{
  name  = "sessionizer",
  title = "Open Rex Sessionizer",
  run   = function()
    local layer, err = rex.session.new_layer{
      bounds = { 0.1, 0.1, 0.8, 0.8 },
      focus  = true,
      layout = rex.layout.block{
        flavor  = "com.superlogical.terminal.shell",
        options = { command = { "rex-sessionizer" } },
      },
    }
    if err ~= nil then error(err, 0) end
    return { layer_id = layer.layer_id }
  end,
}

rex.bind("ctrl+shift+p", "sessionizer")
```

`ctrl+shift+p` is an example; use any free chord. Rex accepts `cmd`, `ctrl`,
`alt`, and `shift`, plus sequences such as `ctrl+a>f`. See the
[Keyboard Shortcuts](https://www.superlogical.com/rex/docs/customize/keyboard-shortcuts)
docs for the full syntax.

Apply the config on the machine running the Rex server:

```sh
rex config check
rex config reload
```

Then your chord floats the picker over the active window, from any pane, tab,
or session. You can also run it from the command palette or with
`rex do sessionizer`.

Rex runs the action on the server machine. If it cannot find
`rex-sessionizer`, put its directory on the `PATH` that the server's
environment sees, or use the absolute path in the action:

```lua
      options = {
        command = { "/absolute/path/to/rex-sessionizer" },
      },
```

The action uses `rex.session.new_layer`, so the picker is a popup in the
current window instead of a separate tab. For a dedicated window, replace
`new_layer` with `new_window` and drop `bounds`.

### 5 · Use it

```sh
rex-sessionizer                    # pick from any terminal
rex-sessionizer ~/code/my-project  # open a project or existing session
rex-sessionizer --list             # print sessions and projects
```

Add `--dry-run` to preview a selection without creating or switching anything.

`--popup` opens the picker as a floating layer from a shell or editor binding
inside Rex; the Rex action above already does this, so you do not need it
there.

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
# ${XDG_CONFIG_HOME:-$HOME/.config}/rex/rex-sessionizer.conf
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

These commands can be bound in an editor or shell; use `-t/--target` when
running them outside Rex.

## ⚙️ Configuration

The global config is `~/.config/rex/rex-sessionizer.conf` (or
`$XDG_CONFIG_HOME/rex/rex-sessionizer.conf`). A `rex-sessionizer.conf` in the
directory you launch from is sourced **after** the global file, so it can
override settings for that directory. Both are read on the machine where
`rex-sessionizer` runs; with the Rex binding, that is the server machine.

> **Security note:** both config files are shell scripts, so they can run
> arbitrary commands. Only launch `rex-sessionizer` from directories you trust.

`REX_SESSIONIZER_SEARCH_PATHS` has no built-in default: set it in the config,
or the picker and `--list` refuse to run.

Every variable, its default, and the accepted search-entry syntax are
documented in
[`rex-sessionizer.conf.example`](rex-sessionizer.conf.example). Use it as a
reference, or copy it over your config to start from the full set.

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
| The Rex binding does nothing, or says `rex-sessionizer: command not found` | The action runs on the Rex server. Install the script there, put its directory on that machine's `PATH`, or use an absolute path in `command`. |
| `session target ... is ambiguous` | Two sessions share a label. Rename one, or pass a full session ID. |
| `-s` says `REX_SESSION is not set` | Run it inside a Rex terminal, or pass `-t/--target`. |
| The picker only shows sessions | Every configured search path was missing, pruned, or hidden. Check `--list` and `REX_SESSIONIZER_INCLUDE_HIDDEN`. |
| An old command pane is reused | The pane label includes the command and cwd; if you expected a new pane, check that the config actually changed. |
| No logs | Set `REX_SESSIONIZER_LOG=echo` or `REX_SESSIONIZER_LOG=file`. |

## 📄 License & Contact

MIT © 2026 rex-sessionizer contributors · Questions? [@0xViva](https://github.com/0xViva)
