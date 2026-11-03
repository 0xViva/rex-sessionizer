# rex-sessionizer

**Fuzzy project switching for Rex power users.**

[![License](https://img.shields.io/badge/license-MIT-22c55e?style=flat-square)](LICENSE)
![Version](https://img.shields.io/badge/version-1.2.0-3b82f6?style=flat-square)
[![Rex](https://img.shields.io/badge/built%20for-Rex-6d28d9?style=flat-square)](https://www.superlogical.com/rex/docs/)
![Bash](https://img.shields.io/badge/Bash-3.2%2B-4eaa25?logo=gnubash&logoColor=white&style=flat-square)
[![fzf](https://img.shields.io/badge/fzf-required-f59e0b?style=flat-square)](https://github.com/junegunn/fzf)
[![jq](https://img.shields.io/badge/jq-required-1f2937?style=flat-square)](https://jqlang.org)

> Inspired by [ThePrimeagen's tmux sessionizer](https://github.com/theprimeagen/tmux-sessionizer).



https://github.com/user-attachments/assets/c007c652-b7fd-4c85-9897-33f1111237a3



## ⚡ At a Glance

`rex-sessionizer` is a keyboard-first project and session switcher for Rex.
It searches your project directories and existing sessions in one `fzf`
prompt, then hands the selected context to the Rex app.

> **What is Rex?**  **Rex** means the terminal at
> [superlogical.com/rex](https://www.superlogical.com/rex/docs/) — the
> multiplexer for all work, with a macOS app and Linux/FreeBSD servers, plus
> the `rex` CLI used here to control it. 

> **Power move:** `rex-sessionizer -s 0 --vsplit` reuses a command pane beside
> your current block instead of spawning duplicates. Add `-t SESSION` to run
> it in another session.

## 🧠 Core Features

| Capability | What it gives you |
| --- | --- |
| ⚡ **Fuzzy context switch** | Search sessions and projects in one picker. |
| 📁 **Directory → session** | Turn a project dir into a named Rex session. |
| 🪟 **Reusable command panes** | `-s INDEX` reuses windows and splits. |
| 🪟 **Floating picker** | `--popup` opens the picker over the current Rex window. |
| 🎯 **Client-aware handoff** | Picks the sole app client, or the one from `-C`. |
| 🌐 **Multi-server aware** | `-S/--server` targets another Rex host. |
| 🧪 **Safe to inspect** | `--list` and `--dry-run` change nothing. |

## 🚀 Setup

Needs [Rex](https://www.superlogical.com/rex/docs/), `fzf`, and `jq`. Enable
**Remote Control** in the Rex app for session switching.

**1. Install and configure**

```sh
git clone https://github.com/0xViva/rex-sessionizer
config="${XDG_CONFIG_HOME:-$HOME/.config}/rex"
mkdir -p ~/.local/bin "$config"
install -m 755 rex-sessionizer/rex-sessionizer ~/.local/bin/
echo 'REX_SESSIONIZER_SEARCH_PATHS=(~/code)' > "$config/rex-sessionizer.conf"
```

Replace `~/code` with your project root and ensure `~/.local/bin` is on
`PATH` (edit an existing config instead of rerunning `echo`).

**2. Bind it in Rex**

Add this to `~/.config/rex/init.lua` (or `$XDG_CONFIG_HOME/rex/init.lua`):

```lua
rex.bind("ctrl+shift+p", function()
  rex.session.new_layer{
    bounds = { 0.1, 0.1, 0.8, 0.8 },
    focus = true,
    layout = rex.layout.block{
      flavor = "com.superlogical.terminal.shell",
      options = { command = { "rex-sessionizer" } },
    },
  }
  return true
end)
```

**3. Reload**

```sh
rex config reload
```

Press `Ctrl+Shift+P`. Run `rex-sessionizer --list` to check discovery. Done.

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
| The Rex binding does nothing, or says `rex-sessionizer: command not found` | The binding runs on the Rex server. Install the script there, put its directory on that machine's `PATH`, or use an absolute path in `command`. |
| `session target ... is ambiguous` | Two sessions share a label. Rename one, or pass a full session ID. |
| `-s` says `REX_SESSION is not set` | Run it inside a Rex terminal, or pass `-t/--target`. |
| The picker only shows sessions | Every configured search path was missing, pruned, or hidden. Check `--list` and `REX_SESSIONIZER_INCLUDE_HIDDEN`. |
| An old command pane is reused | The pane label includes the command and cwd; if you expected a new pane, check that the config actually changed. |
| No logs | Set `REX_SESSIONIZER_LOG=echo` or `REX_SESSIONIZER_LOG=file`. |

## 📄 License & Contact

MIT © 2026 rex-sessionizer contributors · Questions? [@0xViva](https://github.com/0xViva)
