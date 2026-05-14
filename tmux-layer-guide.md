# Advantage 360 — Tmux Layer Guide

## Activating the Tmux Layer

**Hold the DEL key** (left thumb cluster, second from outside). While holding, every key press sends `Ctrl+A` (tmux prefix) + the action key automatically. Tapping the same key produces a normal Delete.

## Key Map (physical key positions from base layer)

### Number Row — Window Select

| Physical Key | Action | Sends |
|---|---|---|
| `1`–`5` | Select window 1–5 | `Ctrl+A` `1`–`5` |
| `6`–`9` | Select window 6–9 | `Ctrl+A` `6`–`9` |

### Left Hand — Upper Row (Q row)

| Physical Key | Action | Sends |
|---|---|---|
| `W` | Vertical split | `Ctrl+A` `v` |
| `E` | Horizontal split | `Ctrl+A` `s` |
| `R` | Zoom/unzoom pane | `Ctrl+A` `z` |

### Left Hand — Home Row (A row) — Pane Navigation

| Physical Key | Action | Sends |
|---|---|---|
| `S` | Pane left | `Ctrl+A` `h` |
| `D` | Pane down | `Ctrl+A` `j` |
| `F` | Pane up | `Ctrl+A` `k` |
| `G` | Pane right | `Ctrl+A` `l` |

### Left Hand — Bottom Row (Z row) — Pane Resize

| Physical Key | Action | Sends |
|---|---|---|
| `X` | Resize left 5 cells | `Ctrl+A` `H` (shift+h) |
| `C` | Resize down 5 cells | `Ctrl+A` `J` (shift+j) |
| `V` | Resize up 5 cells | `Ctrl+A` `K` (shift+k) |
| `B` | Resize right 5 cells | `Ctrl+A` `L` (shift+l) |

Resize keys mirror navigation one row down, same vim HJKL directions.

### Right Hand — Upper Row (Y row)

| Physical Key | Action | Sends |
|---|---|---|
| `U` | Enter copy mode | `Ctrl+A` `[` |
| `I` | Paste buffer | `Ctrl+A` `]` |
| `O` | Reload tmux.conf | `Ctrl+A` `r` |

### Right Hand — Home Row (H row) — Pane Navigation (duplicate)

| Physical Key | Action | Sends |
|---|---|---|
| `H` | Pane left | `Ctrl+A` `h` |
| `J` | Pane down | `Ctrl+A` `j` |
| `K` | Pane up | `Ctrl+A` `k` |
| `L` | Pane right | `Ctrl+A` `l` |

### Right Hand — Bottom Row (N row)

| Physical Key | Action | Sends |
|---|---|---|
| `N` | Close pane (confirm) | `Ctrl+A` `x` |
| `M` | Detach session | `Ctrl+A` `d` |
| `,` | New window | `Ctrl+A` `c` |

### Bottom Row — Arrow Area

| Physical Key | Action | Sends |
|---|---|---|
| `LEFT` arrow | Previous window | `Ctrl+A` `Ctrl+H` |
| `RIGHT` arrow | Next window | `Ctrl+A` `Ctrl+L` |
