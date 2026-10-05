# tmux + kiro-cli Cheat Sheet

Environment: tmux 3.4, prefix = `Ctrl-b` (default), SSH → tmux → kiro-cli.
Prefix notation below: `C-b x` means press `Ctrl-b`, release, then `x`.

## TL;DR

1. `ssh` in → `tmux attach -t wip`.
2. One **window per project dir**; launch `kiro-cli chat` in each.
3. SSH drops? Reattach — nothing was lost.
4. Process stopped? `cd` back and `kiro-cli chat --resume`.

---

## Mental model (read this first)

- A `kiro-cli chat` session's **working directory is fixed** to where you launched it.
  You don't `cd` mid-session — you launch one session per project dir.
- **Conversations are scoped per-directory.** `--resume` only finds sessions
  started in the *current* dir. So: **one tmux window per project** maps 1:1
  onto how resume works.
- tmux keeps the kiro-cli process alive **across SSH disconnects**. Reconnect,
  reattach, and you're exactly where you left off.

---

## The core workflow

```bash
# connect, then attach your named session
ssh you@host
tmux attach -t wip            # or just: tmux attach  (attaches most recent)

# window per project
cd ~/code/cat9music && kiro-cli chat     # window 1
# C-b c  -> new window
cd ~/code/wip       && kiro-cli chat     # window 2
```

Switch windows: `C-b <number>`  •  window list/picker: `C-b w`

---

## Reconnect after a dropped SSH link (the big win)

```bash
ssh you@host
tmux attach -t wip        # process kept running; you're back mid-conversation
```

If the kiro-cli process itself was stopped (not just detached):

```bash
cd ~/code/wip
kiro-cli chat --resume          # resume most recent conversation in THIS dir
kiro-cli chat --resume-picker   # pick from a list
kiro-cli --list                 # root-level alias for the picker
kiro-cli chat --list-sessions   # list saved sessions for this dir
```

Resume restores the model that was active when the session was saved.

---

## tmux essentials (prefix = C-b)

| Action                         | Keys / Command                 |
|--------------------------------|--------------------------------|
| New window                     | `C-b c`                        |
| Next / previous window         | `C-b n` / `C-b p`              |
| Go to window N                 | `C-b 0..9`                     |
| Window picker                  | `C-b w`                        |
| Rename current window          | `C-b ,`                        |
| Detach (leave it all running)  | `C-b d`                        |
| Split pane vertical / horiz.   | `C-b %` / `C-b "`              |
| Move between panes             | `C-b <arrow>`                  |
| Scrollback (copy mode)         | `C-b [`  (q to exit)           |
| Search in copy mode            | `C-b [` then `C-r` / `C-s`     |

### Session management (from a shell)

```bash
tmux new -s wip         # create a named session
tmux ls                 # list sessions
tmux attach -t wip      # attach to one
tmux kill-session -t 2  # kill a session by name/number
```

You currently have sessions: `wip` (attached), `2`, `3`.

---

## kiro-cli over SSH/tmux — tips

### Rendering looks garbled through tmux?
```bash
kiro-cli settings chat.allowAnimations false   # kill spinners/progress anim
kiro-cli settings chat.allowAsciiArt false     # ASCII instead of box-drawing
kiro-cli chat --legacy-mode                    # fall back to legacy TUI
```

### Line wrapping weird after a reattach/resize?
```bash
kiro-cli chat --wrap auto      # default; also: always | never
```

### Esc not registering cleanly over SSH? Rebind cancel.
```bash
kiro-cli settings chat.keybindings.cancelStream "ctrl+g"
# other rebindable keys: closeMenu, quit  (syntax: ctrl+g, shift+tab, etc.)
```

### In-session keyboard shortcuts (inside kiro-cli)
| Action                      | Keys    |
|-----------------------------|---------|
| Search command history      | `Ctrl-r`|
| Cancel current op / exit    | `Ctrl-c`|
| History up / down           | `Up/Down`|
| Cancel streaming (default)  | `Esc`   |

> Note: `C-b` is tmux's prefix, so it won't collide with kiro-cli's `Ctrl-r`/`Ctrl-c`.

---

## Headless mode (handy for scripted runs in a window)

```bash
kiro-cli chat --no-interactive --trust-tools=read,grep "find all TODOs in src/"
kiro-cli chat --no-interactive --trust-all-tools "run the tests and summarize"
```
Headless needs a query arg and trust flags (otherwise it hangs on approvals).

---

## Config locations (for reference)

| What                         | Path                          |
|------------------------------|-------------------------------|
| Global CLI settings          | `~/.kiro/settings/cli.json`   |
| Workspace settings override  | `.kiro/settings/cli.json`     |
| Global agents                | `~/.kiro/agents/`             |
| Workspace agents             | `.kiro/agents/`               |
| Global steering (rules)      | `~/.kiro/steering/`           |
| Workspace steering           | `.kiro/steering/`             |
| Saved sessions               | `~/.kiro/sessions/cli/`       |

Relocate the whole kiro root with `KIRO_HOME=/path kiro-cli chat`.

---

