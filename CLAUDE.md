# Adv360-Pro-ZMK — Context for Future Sessions

This repo is the user's personalized ZMK firmware for a Kinesis Advantage 360 Pro. The base ZMK build harness is upstream; the customization lives in `config/`.

## Files that matter

| Path | Purpose |
|---|---|
| `config/adv360.keymap` | The main keymap. Defines behaviors, layers, and bindings. Almost all real work happens here. |
| `config/macros.dtsi` | Macro definitions (tmux prefix-chord macros, Mac/Windows shortcut macros, bracket-pair macros). Included from the keymap. |
| `config/adv360_left.keymap`, `config/adv360_right.keymap` | Split-side stubs. Currently empty / unused for layout logic — don't edit these expecting effects. |
| `config/keymap.json` | Kinesis configurator export format. Not the source of truth; the `.keymap` file is. |
| `config/info.json`, `version.dtsi`, `west.yml` | Build metadata. Don't touch unless changing board/build config. |
| `Makefile` | Builds firmware in Docker. `make all` builds the right side (also produces full firmware). `make left` builds left only. Output lands in `firmware/`. |
| `prefix-layer-guide.md` | Reference for the prefix layer (layer 6) bindings, with the tmux and herdr meaning of each key side by side. Update this when layer access keys or actions change. |

## Build process

```
make all      # builds firmware via Docker (zmkfirmware/zmk-build-arm)
make clean    # wipe firmware/ and the docker image
```

Firmware UF2 lands in `firmware/`. Flashing is done by putting the keyboard in bootloader mode (via the `&bootloader` binding on the Mod layer) and copying the UF2 to the mounted volume.

## Layer architecture

Seven layers, indexed in `adv360.keymap`:

| Index | Name | Access | Purpose |
|---|---|---|---|
| 0 | BASE | always on | Clean QWERTY. No home-row mod-tap. |
| 1 | KEYPAD | `&tog KEYPAD` (top-right edge key) | Numpad on the right hand. |
| 2 | FN | `&mo FN` (outer corners on row 5) | F-keys. |
| 3 | MOD | `&mo MOD` (top inner key, right side) | Bluetooth, bootloader, version macro, RGB/backlight. |
| 4 | SYM | hold PG_DN (via `ltn`) | Symbols, brackets, F-keys. **Optimized for Go/Rust.** |
| 5 | CMODS | hold BSPC (via `ltq`) | Callum-style sticky one-shot mods on home row. |
| 6 | TMUX | hold DEL (via `ltq`) | One-key multiplexer ops via `Ctrl+A` prefix macros. Drives both tmux and herdr. |

Layers 7–9 are reserved (`Purple`, `Cyan`, `Yellow`).

## Hold-tap behaviors

Two custom hold-tap behaviors live at the top of `adv360.keymap`:

### `ltq` — layer-tap quick (with prior-idle guard)

```
tapping-term-ms = 175
quick-tap-ms = 175
require-prior-idle-ms = 125
flavor = balanced
```

Used for thumbs whose tap is in prose flow (BSPC, DEL). The 125 ms `require-prior-idle-ms` forces the key to resolve as a **tap** if any other key was pressed within 125 ms before it — this prevents accidental layer activation during fast typing (e.g. "to only" would otherwise become "to#nly" if SYM caught the next key).

### `ltn` — layer-tap no-idle

```
tapping-term-ms = 175
quick-tap-ms = 175
flavor = balanced
(no require-prior-idle-ms)
```

Used **only on PG_DN** (the SYM thumb). PG_DN is never typed mid-word, so the prior-idle guard adds no safety, but it would block fast Vim sequences like `vi}`, `ci{`, `da)` from activating SYM. Removing the guard lets the hold fire instantly.

**Rule of thumb when adding new layer-taps:** if the tap key is something you press during typing flow (space, backspace, delete, enter), use `ltq`. If the tap key is something you only press deliberately (pgdn, pgup, end, home), use `ltn`.

## Thumb cluster (the heart of the layout)

Bottom-row thumbs from outside-left to outside-right:

| Key | Tap | Hold |
|---|---|---|
| BSPC | Backspace | CMODS (sticky mods) |
| DEL | Delete | TMUX |
| END | End | — |
| PG_DN | PgDn | SYM |
| ENTER | Enter | — |
| SPACE | Space | — |

**Why SPACE has no hold layer:** SPACE is the most-tapped key. Putting a hold on it forces every space press through hold-tap resolution and risks mis-fires in fast typing. Moving SYM off SPACE was a deliberate trade for clean Vim ergonomics.

**Why TMUX is on DEL (left) and SYM is on PG_DN (right):**
- Tmux nav keys (H/J/K/L) live on the right hand → left-thumb-hold + right-hand-key = clean opposite-hand chord.
- Symbol keys (`}`, `(`, `[`) live on the left hand → right-thumb-hold + left-hand-key = clean opposite-hand chord.
- Both layer-entry keys are off the typing-flow path.

## BASE layer notes

- No home-row mod-tap. The home row is pure QWERTY. Modifiers are reached either via the bridge keys (LCTRL, LALT, LGUI, RCTRL — physical keys preserved) or via the CMODS sticky-mod layer.
- Physical SHIFT remains on the outer columns (LSHFT, RSHFT) and is the right choice for Vim's `"`, `?`, `:` etc. Don't route those through CMODS — sticky-shift via CMODS is a 3-step chain.
- Brackets `[` `]` are on the BASE layer (bottom-right). `{` `}` `(` `)` live on SYM.

## SYM layer rationale (adv360.keymap, layer 4)

Optimized for Go and Rust:

- Left home row: `& { ( [ <` — bracket opens you type all day in struct/func/slice/generics.
- Top row (left): `` ` } ) ] >`` — closing counterparts above.
- Bottom row (left): `| \ ~ \`` — bitwise / escape.
- Right home row: `_ - = +` — assignment family.
- Top row (right): `! @ # $` — Rust macros, attributes.
- Bottom row (right): `^ % * /` — arithmetic.
- F-keys preserved on the top row for debugger shortcuts.

Common compound symbols:
- Go `:=` — type `:` (base layer), then SYM+`=`. Or just `:=` if you don't mind the layer trip.
- Rust `->` — SYM+`_` then `>` (which is on SYM left side, top row position of D).
- Rust `=>` — SYM+`=` then `>`.

## CMODS layer (layer 5)

Sticky one-shot mods on the home row, Mac-optimized (CAGS order):

| Finger | Left hand | Right hand |
|---|---|---|
| Pinky (A / ;) | Ctrl | Ctrl |
| Ring (S / L)  | Alt | Alt |
| Middle (D / K) | Cmd | Cmd |
| Index (F / J) | Shift | Shift |

Cmd on the middle finger because Cmd is the most-used Mac modifier (save/copy/paste/switch). Sticky behavior is `sticky_key_quick_release` (`skq`): release-after-ms 1000, quick-release, ignore-modifiers (so you can chain `Cmd+Shift+key`).

**This layer is mostly redundant** for keyboard-only flows because the physical LCTRL/LALT/LGUI/RCTRL bridge keys are still exposed on the BASE layer. CMODS earns its keep when the other hand is on the mouse and you need one-handed mod combos.

## Prefix layer (layer 6, named TMUX)

All bindings on this layer are **macros** that emit `Ctrl+A` (the prefix) followed by an action key. Definitions are in `macros.dtsi`:

- Pane navigation: `Tmux_Left`/`Right`/`Up`/`Down` → prefix + `h`/`j`/`k`/`l`
- Resize (shifted): `Tmux_Resize_*` → prefix + `H`/`J`/`K`/`L`
- Cross-container nav (ctrl family): `Tmux_Prev_Win`/`Next_Win` → prefix + `C-h`/`C-l`;
  `Tmux_Prev_Space`/`Next_Space` → prefix + `C-k`/`C-j`; `Tmux_Prev_Agent`/`Next_Agent`
  → prefix + `C-p`/`C-n` (herdr only, no-op in tmux)
- `Tmux_Last_Pane` → prefix + `;` (tmux built-in); `Tmux_Space_Picker` → prefix + `w`
- Splits: `Tmux_Split_V`/`H` → prefix + `v`/`s`
- Windows: `Tmux_Win1`–`Win9` → prefix + `1`–`9`; `Tmux_New_Win` → prefix + `c`; `Tmux_Prev_Win`/`Next_Win` → prefix + `Ctrl+H`/`Ctrl+L`
- Copy/paste/reload: `Tmux_Copy_Mode` (`[`), `Tmux_Paste` (`]`), `Tmux_Reload` (`r`)
- Misc: `Tmux_Zoom` (`z`), `Tmux_Close_Pane` (`x`), `Tmux_Detach` (`d`)

The user's tmux config lives at **`~/.config/tmux/tmux.conf`** — there is no `~/.tmux.conf`.
It uses `Ctrl+A` as the prefix (not the default `Ctrl+B`) and has `bind-key C-a send-prefix`,
so `C-a C-a` sends a literal Ctrl+A there. It had drifted from the keymap (its comments still
describe an older layout); `bind -r H/J/K/L resize-pane` and `bind -r C-k/C-j switch-client`
were added to match the current macros. Two layer keys still diverge in tmux: `O` (`C-a r`)
hits a legacy `bind r resize-pane -Z` (zoom) instead of reload, and `A` (`C-a w`) is a
*window* picker there but a *workspace* picker in herdr. tmux reload (`bind R`) is unreachable
from the layer because shift is `&none`. Verify tmux edits in a throwaway server:
`tmux -L test -f ~/.config/tmux/tmux.conf new-session -d && tmux -L test list-keys -T prefix`.

### This layer also drives herdr

The user runs **herdr** (a terminal multiplexer for coding agents) as well as tmux —
separately, never nested. Herdr's prefix was set to `Ctrl+A` in
`~/.config/herdr/config.toml` and several of its stock action keys were rebound so
this layer means the same thing in both tools. tmux *window* ≈ herdr *tab*;
tmux *session* ≈ herdr *workspace*.

Herdr overrides that exist purely to match this layer:
`split_horizontal`→`prefix+s`, `reload_config`→`prefix+r`, `detach`→`prefix+d`,
`previous_tab`/`next_tab`→`prefix+ctrl+h`/`prefix+ctrl+l`,
`previous_workspace`/`next_workspace`→`prefix+ctrl+k`/`prefix+ctrl+j`,
`previous_agent`/`next_agent`→`prefix+ctrl+p`/`prefix+ctrl+n`,
`last_pane`→`prefix+;`, `resize_pane_*`→`prefix+shift+h/j/k/l`,
`edit_scrollback`→`prefix+[`. Displaced defaults were rehomed:
`settings`→`prefix+comma`, `resize_mode`→`prefix+shift+r`.
`workspace_picker` stays on its default `prefix+w`, which layer key `A` now sends.

**The emitted-key rule:** plain letter = move within the pane grid; shift letter = resize
within the grid; ctrl letter = jump out of the container (`C-h`/`C-l` tab, `C-k`/`C-j`
space, `C-p`/`C-n` agent). Follow it when adding bindings.

**There is no shift available on layer 6.** `LSHFT`/`RSHFT` are `&none` deliberately —
holding physical shift during macro playback would turn `&kp LC(A)` into `Ctrl+Shift+A`,
which is not the prefix, breaking every key on the layer. This is why resize occupies its
own row rather than being a shift variant of `HJKL`, and why no new binding may rely on a
shift chord. It is also the standing answer to "why doesn't DEL+H resize?" — `H` is pane
focus; resize is on `X C V B`, which *emit* `C-a H/J/K/L`.

**Reading `herdr config check`:** `invalid keybinding: ...` means unparseable syntax.
`<key>: kept keys.A, disabled keys.B` means two actions claimed one chord and one was
silently dropped. Herdr normalizes `prefix+H` and `prefix+shift+h` to the same key, so
watch for that collision. Note that appending keys to the end of `config.toml` puts them
under the trailing `[[keys.command]]` table and yields `unknown config key` — new `[keys]`
entries must go inside the `[keys]` table, above `[[keys.command]]`.

Two keys have no clean herdr counterpart: `U` (`Ctrl+A [`, tmux copy mode) maps to
`edit_scrollback` instead, and `I` (`Ctrl+A ]`, tmux paste) is unbound because herdr
has no prefix paste action.

**Prefer changing the herdr config over changing the macros** — herdr applies live via
`herdr config check` + `herdr server reload-config`, whereas a macro change costs a
firmware rebuild and reflash.

`prefix-layer-guide.md` at the repo root is the canonical user-facing reference for
which key does what on this layer, in both tools. Keep it in sync when bindings change.

## Macros worth knowing about

In `macros.dtsi`:
- **Bracket-pair macros** (`macro_quotes`, `macro_dquotes`, `macro_braces`, `macro_parens`, `macro_brackets`) emit the pair and arrow-left to land the cursor inside. Currently defined but not bound anywhere — available if the user wants to wire them up.
- **Mac/Windows shortcut macros** (`Mac_Copy`, `Win_Copy`, etc.) wrap common OS chords. Mostly unused in the current bindings; held over from the upstream layout.
- **`macro_kinesis`** types out "Kinesis". Vestigial — upstream default.

## Recent design decisions

These are the load-bearing choices in the current state:

1. **SYM moved off SPACE onto PG_DN.** Original layout had `&ltq SYM SPACE` on the right large thumb. The 125 ms idle guard that protected space from mis-firing into SYM also blocked rapid Vim sequences. Splitting: SYM now on PG_DN with `ltn` (no guard); SPACE is pure space.
2. **TMUX moved off PG_DN onto DEL.** Tmux nav keys are on the right hand, so left-thumb-hold + right-hand-key is the natural chord. DEL is never typed mid-word, so it can also use `ltq` without prose-flow regressions.
3. **`ltq` vs `ltn` split.** Two hold-tap behaviors instead of one global config. `ltq` for prose-flow thumbs (BSPC, DEL); `ltn` for the symbol-flow thumb (PG_DN). The earlier single-behavior approach forced a compromise between prose safety and Vim speed.

## User context

- macOS user (uses Cmd, not Ctrl, as the primary editing modifier).
- Heavy Vim user — writes code in Cursor/VS Code with Vim plugins. Layout decisions weight Vim ergonomics highly.
- Writes Go and Rust — SYM layer is tuned for both.
- Uses tmux daily with `Ctrl+A` prefix, and also uses herdr (also on `Ctrl+A`, configured to match). The two are used separately, never nested.

## When working on this repo

- Source of truth for the layout is `config/adv360.keymap` plus `config/macros.dtsi`. Edit those.
- Don't edit `config/keymap.json` directly — it's a configurator export, not the source.
- When changing thumb cluster assignments, update **both** the binding line **and** the comment block at the top of the BASE layer in `adv360.keymap`. They are linked documentation; one drifting from the other will confuse the next session.
- When changing layer 6 bindings, update **four** places so they don't drift: `prefix-layer-guide.md` (per-row breakdown), the "Prefix layer" sections in `README.md` (herdr + tmux tables), the layer 6 comment block in `adv360.keymap`, and the herdr `[keys]` block in `~/.config/herdr/config.toml`. If the change needs a tmux binding, edit `~/.config/tmux/tmux.conf` directly (back it up first) **and** document it in the README — it is untracked, not off-limits.
- When adding a new hold-tap, decide consciously: prose-flow tap → `ltq`; deliberate-only tap → `ltn`. Add new behaviors rather than over-tuning existing ones.
- Refer to physical keys by their base-layer label (SPACE, BSPC, ENTER, DEL, END, PG_DN), not by descriptive nicknames like "big thumb" or "inner thumb" — those names obscure rather than clarify.
