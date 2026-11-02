# rex-sessionizer

**Fuzzy project switching for Rex power users.**

[![License](https://img.shields.io/badge/license-MIT-22c55e?style=flat-square)](LICENSE)
![Version](https://img.shields.io/badge/version-1.0.0-3b82f6?style=flat-square)
[![Rex](https://img.shields.io/badge/built%20for-Rex-6d28d9?style=flat-square)](https://www.superlogical.com/rex/docs/automate/cli)
![Bash](https://img.shields.io/badge/Bash-3.2%2B-4eaa25?logo=gnubash&logoColor=white&style=flat-square)
[![fzf](https://img.shields.io/badge/fzf-required-f59e0b?style=flat-square)](https://github.com/junegunn/fzf)
[![jq](https://img.shields.io/badge/jq-required-1f2937?style=flat-square)](https://jqlang.org)

> Inspired by [ThePrimeagen's tmux sessionizer](https://github.com/theprimeagen/tmux-sessionizer).

## ⚡ At a Glance

`rex-sessionizer` is a keyboard-first project and session switcher for Rex.
It searches your project directories and existing sessions in one `fzf`
prompt, then hands the selected context to the Rex app.

Built for people who live in the terminal and treat context switching as a
first-class operation.

> **Power move:** `rex-sessionizer -s 0 --vsplit` reuses a command pane beside
> your current block instead of spawning duplicates.

## 🧠 Core Features

| Capability | What it gives you |
| --- | --- |
| ⚡ **Fuzzy context switch** | Search sessions and projects in one picker. |
| 📁 **Directory → session** | Turn a project dir into a named Rex session. |
| 🪟 **Reusable command panes** | `-s INDEX` reuses cached windows and splits. |
| 🎯 **App-aware handoff** | Selects the session; pick the app if needed. |

## 🚀 Quick Start

### 1 · Dependencies

Rex, `fzf`, and `jq`.

### 2 · Install

```sh
install -m 755 rex-sessionizer ~/.local/bin/
```

### 3 · Configure

```sh
cp rex-sessionizer.conf.example ~/.config/rex/rex-sessionizer.conf
$EDITOR ~/.config/rex/rex-sessionizer.conf
```

### 4 · Run

```sh
rex-sessionizer
```

> Ensure `~/.local/bin` is on your `PATH`. Session switching requires a running
> Rex server and an app client with **Remote Control** enabled.

## ⌨️ Shell Keybindings

Bind any free chord to launch the picker from your prompt. Replace `ctrl-p` /
`^P` with a chord that is free in your shell.

| Shell | Config file |
| --- | --- |
| fish | `~/.config/fish/config.fish` |
| zsh | `~/.zshrc` |
| bash | `~/.bashrc` |

### fish

```fish
set -g REX_SESSIONIZER $HOME/.local/bin/rex-sessionizer
bind ctrl-p 'commandline -i "$REX_SESSIONIZER"; commandline -f execute'
```

### zsh

```zsh
rex-sessionizer-widget() {
  zle -I
  rex-sessionizer
  zle reset-prompt
}
zle -N rex-sessionizer-widget
bindkey '^P' rex-sessionizer-widget
```

### bash

```bash
bind '"\C-p": "\C-urex-sessionizer\n"'
```

## 📄 License & Contact

MIT © 2026 rex-sessionizer contributors · Questions? [@0xViva](https://github.com/0xViva)
