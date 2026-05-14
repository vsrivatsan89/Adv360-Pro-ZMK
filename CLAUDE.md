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
| `tmux-layer-guide.md` | Reference for the tmux layer bindings. Update this when tmux layer access keys or actions change. |

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
| 6 | TMUX | hold DEL (via `ltq`) | One-key tmux ops via `Ctrl+A` prefix macros. |

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

## Tmux layer (layer 6)

All tmux bindings are **macros** that emit `Ctrl+A` (the tmux prefix) followed by an action key. Definitions are in `macros.dtsi`:

- Navigation: `Tmux_Left`/`Right`/`Up`/`Down` → prefix + `h`/`j`/`k`/`l`
- Resize (shifted): `Tmux_Resize_*` → prefix + `H`/`J`/`K`/`L`
- Splits: `Tmux_Split_V`/`H` → prefix + `v`/`s`
- Windows: `Tmux_Win1`–`Win9` → prefix + `1`–`9`; `Tmux_New_Win` → prefix + `c`; `Tmux_Prev_Win`/`Next_Win` → prefix + `Ctrl+H`/`Ctrl+L`
- Copy/paste/reload: `Tmux_Copy_Mode` (`[`), `Tmux_Paste` (`]`), `Tmux_Reload` (`r`)
- Misc: `Tmux_Zoom` (`z`), `Tmux_Close_Pane` (`x`), `Tmux_Detach` (`d`)

The user's tmux config uses `Ctrl+A` as the prefix (not the default `Ctrl+B`). Don't change the prefix in the macros without also changing `~/.tmux.conf`.

`tmux-layer-guide.md` at the repo root is the canonical user-facing reference for which key does what on this layer. Keep it in sync when bindings change.

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
- Uses tmux daily with `Ctrl+A` prefix.

## When working on this repo

- Source of truth for the layout is `config/adv360.keymap` plus `config/macros.dtsi`. Edit those.
- Don't edit `config/keymap.json` directly — it's a configurator export, not the source.
- When changing thumb cluster assignments, update **both** the binding line **and** the comment block at the top of the BASE layer in `adv360.keymap`. They are linked documentation; one drifting from the other will confuse the next session.
- When changing tmux layer bindings, update `tmux-layer-guide.md` too.
- When adding a new hold-tap, decide consciously: prose-flow tap → `ltq`; deliberate-only tap → `ltn`. Add new behaviors rather than over-tuning existing ones.
- Refer to physical keys by their base-layer label (SPACE, BSPC, ENTER, DEL, END, PG_DN), not by descriptive nicknames like "big thumb" or "inner thumb" — those names obscure rather than clarify.
