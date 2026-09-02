# Advantage 360 — Prefix Layer Guide (tmux / herdr)

This layer drives **two** multiplexers. Both use `Ctrl+A` as their prefix, and they are never
used at the same time, so one set of muscle memory covers both:

| | tmux | herdr |
|---|---|---|
| Prefix | `Ctrl+A` (`~/.config/tmux/tmux.conf`) | `Ctrl+A` (`~/.config/herdr/config.toml`) |
| Top-level container | session | workspace ("space") |
| Middle container | window | tab |
| Leaf | pane | pane |
| Extra | — | agent (a recognized coding agent in a pane) |

Where herdr's stock binding disagreed with tmux, **herdr was changed** so the keyboard layer
stays identical for both. Those overrides live under `[keys]` in
`~/.config/herdr/config.toml`.

## Activating the layer

**Hold the DEL key** (left thumb cluster, second from outside). While holding, every key press
emits `Ctrl+A` (the prefix) + an action key. Tapping DEL produces a normal Delete.

## Two things to know before reading the tables

**"Emits" is not "press."** The Physical Key column is what you press. The Emits column is the
byte sequence the keyboard sends. Resize lives on `X C V B` and *emits* `C-a H/J/K/L` —
pressing `H` focuses a pane and always will.

**There is no shift on this layer.** `LSHFT` and `RSHFT` are deliberately `&none`. Holding
physical shift during macro playback would turn `&kp LC(A)` into `Ctrl+Shift+A`, which is not
the prefix, breaking every key on the layer. That is why resize occupies its own row rather
than being shift+`HJKL`, and why new bindings must never rely on a shift chord.

## The one rule behind the emitted keys

| Emitted shape | Scope |
|---|---|
| `C-a` + plain letter | move *within* the pane grid — `h j k l` |
| `C-a` + shift letter | *resize* within the pane grid — `H J K L` |
| `C-a` + ctrl letter | jump *out* of the current container — `C-h C-l` tabs, `C-k C-j` spaces, `C-p C-n` agents |

## Key Map (physical key positions from base layer)

### Number Row — Window / Tab Select

| Physical Key | Emits | In tmux | In herdr |
|---|---|---|---|
| `1`–`9` | `C-a` `1`–`9` | Select window 1–9 | Select tab 1–9 (`switch_tab`) |

### Left Hand — Upper Row (Q row)

| Physical Key | Emits | In tmux | In herdr |
|---|---|---|---|
| `W` | `C-a` `v` | Vertical split | Vertical split (`split_vertical`) |
| `E` | `C-a` `s` | Horizontal split | Horizontal split (`split_horizontal`) |
| `R` | `C-a` `z` | Zoom/unzoom pane | Zoom/unzoom pane (`zoom`) |

### Left Hand — Home Row (A row) — Pane Navigation

| Physical Key | Emits | In tmux | In herdr |
|---|---|---|---|
| `A` | `C-a` `w` | fzf **window** picker | **Workspace** picker — one level up |
| `S` | `C-a` `h` | Pane left | Pane left (`focus_pane_left`) |
| `D` | `C-a` `j` | Pane down | Pane down (`focus_pane_down`) |
| `F` | `C-a` `k` | Pane up | Pane up (`focus_pane_up`) |
| `G` | `C-a` `l` | Pane right | Pane right (`focus_pane_right`) |

### Left Hand — Bottom Row (Z row) — Pane Resize

| Physical Key | Emits | In tmux | In herdr |
|---|---|---|---|
| `X` | `C-a` `H` | Resize left 5 cells | Resize left (`resize_pane_left`) |
| `C` | `C-a` `J` | Resize down 5 cells | Resize down (`resize_pane_down`) |
| `V` | `C-a` `K` | Resize up 5 cells | Resize up (`resize_pane_up`) |
| `B` | `C-a` `L` | Resize right 5 cells | Resize right (`resize_pane_right`) |

Resize mirrors navigation one row down, same vim HJKL directions. In herdr these resize
**directly** — they do not enter a resize mode, matching tmux.

### Right Hand — Upper Row (Y row)

| Physical Key | Emits | In tmux | In herdr |
|---|---|---|---|
| `U` | `C-a` `[` | Enter copy mode | Open pane scrollback in `$EDITOR` (`edit_scrollback`) |
| `I` | `C-a` `]` | Paste buffer | **Unbound** — herdr has no prefix paste action |
| `O` | `C-a` `r` | **Zoom** (legacy `bind r`), *not* reload | Reload `config.toml` (`reload_config`) |

### Right Hand — Home Row (H row) — Pane Navigation + Last Pane

| Physical Key | Emits | In tmux | In herdr |
|---|---|---|---|
| `H` | `C-a` `h` | Pane left | Pane left |
| `J` | `C-a` `j` | Pane down | Pane down |
| `K` | `C-a` `k` | Pane up | Pane up |
| `L` | `C-a` `l` | Pane right | Pane right |
| `;` | `C-a` `;` | Last pane — tmux built-in | Last pane (`last_pane`) |

### Right Hand — Bottom Row (N row)

| Physical Key | Emits | In tmux | In herdr |
|---|---|---|---|
| `N` | `C-a` `x` | Close pane (confirm) | Close pane (`close_pane`) |
| `M` | `C-a` `d` | Detach session | Detach client (`detach`) |
| `,` | `C-a` `c` | New window | New tab (`new_tab`) |

### Bottom Row — Navigation Cluster

Every cross-container motion lives here. Left half = tabs, right half = workspaces, then
agents continuing rightward.

| Physical Key | Emits | In tmux | In herdr |
|---|---|---|---|
| `←` | `C-a` `C-h` | Previous window | Previous tab (`previous_tab`) |
| `→` | `C-a` `C-l` | Next window | Next tab (`next_tab`) |
| `↑` | `C-a` `C-k` | Previous session | Previous workspace (`previous_workspace`) |
| `↓` | `C-a` `C-j` | Next session | Next workspace (`next_workspace`) |
| `[` | `C-a` `C-p` | *nothing* | Previous agent (`previous_agent`) |
| `]` | `C-a` `C-n` | *nothing* | Next agent (`next_agent`) |

## Herdr actions with no key on this layer

Type `Ctrl+A` then the key on any keyboard:

| Chord | Action |
|---|---|
| `prefix+g` | Navigate mode |
| `prefix+b` | Toggle sidebar |
| `prefix+Tab` / `prefix+Shift+Tab` | Cycle pane next / previous |
| `prefix+Shift+N` | New workspace |
| `prefix+Shift+G` | New git worktree |
| `prefix+Shift+X` | Close tab |
| `prefix+Shift+T` / `Shift+P` / `Shift+W` | Rename tab / pane / workspace |
| `prefix+,` | Settings (moved off `prefix+s`, which the layer needs for splits) |
| `prefix+Shift+R` | Resize mode (moved off `prefix+r`, which the layer needs for reload) |
| `prefix+?` | Help |

Deliberately unbound: indexed `focus_agent` and `switch_workspace` (1–9). Both parse, but
they would cost 9–18 more macros, and `↑↓` / `[]` covers the common case.

## Caveat: literal `Ctrl+A`

With `Ctrl+A` as the prefix, a bare `Ctrl+A` no longer reaches the shell as beginning-of-line.
tmux already has the escape hatch configured (`bind-key C-a send-prefix`), so `C-a C-a` sends a
literal `Ctrl+A` there. Herdr has no documented equivalent, so use `Home` inside herdr panes.

## Two actions the layer cannot reach in tmux

- **Reload** is on `bind R` (shift+R), and shift is unavailable on this layer, so no macro can
  emit `C-a R`. Type it by hand.
- The **`O` key** emits `C-a r`, which hits the legacy `bind r resize-pane -Z` (zoom) rather
  than reload. Redundant with `z`; rebinding `r` to `source-file` would align both tools.

## Keeping this in sync

- **Keyboard:** `config/adv360.keymap` (layer 6) and `config/macros.dtsi` (`Tmux_*` macros — a
  historical name; the layer serves both tools). A macro change needs `make all` and a reflash.
- **herdr:** `~/.config/herdr/config.toml`. Validate with `herdr config check`, apply live with
  `herdr server reload-config` — no reflash. Read the check output carefully: `invalid
  keybinding` means bad syntax, while `<key>: kept keys.A, disabled keys.B` means two actions
  claimed one chord and one was silently dropped.
- **tmux:** `~/.config/tmux/tmux.conf` (not tracked here; note the path — there is no
  `~/.tmux.conf`). Now carries `bind -r H/J/K/L resize-pane` for the `X C V B` keys and
  `bind -r C-k/C-j switch-client -p/-n` for `↑` / `↓`. Everything else is a tmux built-in or
  was already bound. Validate a change with
  `tmux -L test -f ~/.config/tmux/tmux.conf new-session -d && tmux -L test list-keys -T prefix`.
