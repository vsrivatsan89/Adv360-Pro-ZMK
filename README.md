# Kinesis Advantage 360 Pro ZMK Config

# Home setup 
use the small USB cable in the book shelf. Mod+1 to flash the left keyboard and then MOD + 3 for the right keyboard

## My layout

Seven layers. The source of truth is [`config/adv360.keymap`](config/adv360.keymap) plus
[`config/macros.dtsi`](config/macros.dtsi) — *not* `config/keymap.json`, which is only a
configurator export.

| # | Name | Access | Purpose |
|---|---|---|---|
| 0 | BASE | always on | Clean QWERTY, no home-row mod-tap |
| 1 | KEYPAD | tap top-right edge key | Numpad on the right hand |
| 2 | FN | hold outer corners, row 5 | F-keys |
| 3 | MOD | hold top inner key, right side | Bluetooth, bootloader, version macro, RGB |
| 4 | SYM | **hold PG_DN** | Symbols and brackets, tuned for Go/Rust |
| 5 | CMODS | **hold BSPC** | Callum-style sticky mods on the home row |
| 6 | TMUX | **hold DEL** | Multiplexer prefix layer — see below |

Layer 6 emits `Ctrl+A` (a multiplexer prefix) followed by an action key, collapsing every
two-step prefix chord into one key. `Ctrl+A` is the prefix in **both** herdr and tmux, and
the two are never run nested, so the same layer drives whichever one is in front of me.

## Prefix layer (hold DEL) → herdr

Herdr's prefix is `ctrl+a`, set in `~/.config/herdr/config.toml`. Herdr's vocabulary: a
**workspace** ("space") holds **tabs**, which hold **panes**, and panes may hold **agents**.

> **How to read these tables.** The first column is the key you **physically press** while
> holding DEL. The second is the byte sequence the keyboard **emits** — those are not keys
> to press. Resize, for example, lives on `X C V B` and *emits* `C-a H/J/K/L`; pressing `H`
> itself focuses a pane and always will.
>
> There is no shift on this layer — `LSHFT`/`RSHFT` are deliberately `&none`, because
> holding physical shift during macro playback would turn `C-a` into `Ctrl+Shift+A` and
> break every key on the layer. That is why resize gets its own row instead of being
> shift+`HJKL`.

### Navigating: four axes

The emitted keys follow one rule — **plain letter moves inside the pane grid, ctrl+letter
jumps out of the current container**:

| Axis | Hold DEL + | Emits | Herdr action |
|---|---|---|---|
| **Pane**, directional | `S` `D` `F` `G` or `H` `J` `K` `L` | `C-a` `h` `j` `k` `l` | Focus pane left / down / up / right |
| **Pane**, last used | `;` | `C-a` `;` | Toggle to the previous pane |
| **Tab** | `←` `→` | `C-a` `C-h` / `C-a` `C-l` | Previous / next tab |
| **Workspace** | `↑` `↓` | `C-a` `C-k` / `C-a` `C-j` | Previous / next workspace |
| **Agent** | `[` `]` | `C-a` `C-p` / `C-a` `C-n` | Previous / next agent |
| **Workspace**, pick | `A` | `C-a` `w` | Open the workspace picker |

Why these keys: `← →` sit on the left half of the bottom row and `↑ ↓` on the right, so
**left hand = tabs, right hand = workspaces** — and vertical arrows match the sidebar's
vertical stack. `[` `]` continue rightward for agents, reusing the vim `[q`/`]q` idiom.
`;` is both free on the home row and tmux's own built-in last-pane key.

### Acting on panes

| Hold DEL + | Emits | Herdr action |
|---|---|---|
| `W` | `C-a` `v` | Vertical split |
| `E` | `C-a` `s` | Horizontal split |
| `R` | `C-a` `z` | Zoom / unzoom pane |
| `X` `C` `V` `B` | `C-a` `H` `J` `K` `L` | Resize pane left / down / up / right — direct, no resize mode |
| `N` | `C-a` `x` | Close pane |
| `U` | `C-a` `[` | Open pane scrollback in `$EDITOR` |

Resize mirrors pane navigation exactly one row down, same vim `HJKL` directions.

### Tabs and session

| Hold DEL + | Emits | Herdr action |
|---|---|---|
| `1`–`9` | `C-a` `1`–`9` | Select tab 1–9 |
| `,` | `C-a` `c` | New tab |
| `M` | `C-a` `d` | Detach client |
| `O` | `C-a` `r` | Reload `~/.config/herdr/config.toml` |

### One dead key

`I` (emits `C-a` `]`) is **unbound** under herdr — it has no prefix paste action, relying on
the system clipboard and `copy_on_select` instead.

### Herdr features with no key on this layer

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
| `prefix+,` | Settings |
| `prefix+Shift+R` | Resize mode |
| `prefix+?` | Help |

Still unbound by choice: indexed `focus_agent` and `switch_workspace` (1–9). Both parse
fine, but they would cost 9–18 more macros and a second row of number-key semantics, and
`↑↓` / `[]` already covers the common case.

### The rebinds that make this work

Herdr ships on `ctrl+b` with several action keys that disagree with tmux. These overrides in
`~/.config/herdr/config.toml` close the gap:

| Setting | Set to | Herdr default |
|---|---|---|
| `prefix` | `ctrl+a` | `ctrl+b` |
| `split_horizontal` | `prefix+s` | `prefix+minus` |
| `reload_config` | `prefix+r` | `prefix+shift+r` |
| `detach` | `prefix+d` | `prefix+q` |
| `previous_tab` / `next_tab` | `prefix+ctrl+h` / `prefix+ctrl+l` | `prefix+p` / `prefix+n` |
| `previous_workspace` / `next_workspace` | `prefix+ctrl+k` / `prefix+ctrl+j` | unset |
| `previous_agent` / `next_agent` | `prefix+ctrl+p` / `prefix+ctrl+n` | unset |
| `last_pane` | `prefix+;` | unset |
| `resize_pane_left/down/up/right` | `prefix+shift+h/j/k/l` | unset — mode-based only |
| `edit_scrollback` | `prefix+[` | `prefix+e` |
| `settings` | `prefix+comma` | `prefix+s`, which the layer needs for splits |
| `resize_mode` | `prefix+shift+r` | `prefix+r`, which the layer needs for reload |

Left on herdr defaults because they already matched: `switch_tab`, `focus_pane_*`,
`split_vertical`, `zoom`, `close_pane`, `new_tab`, and `workspace_picker` (`prefix+w`).

Config changes apply live — no rebuild, no reflash:

```shell
herdr config check          # validate; names any invalid or conflicting binding
herdr server reload-config  # apply to the running server
```

Note that `herdr config check` reports two distinct failures. `invalid keybinding: ...` means
the key syntax is unparseable. `<key>: kept keys.A, disabled keys.B` means two actions claim
the same chord and one was silently dropped — worth reading for, since herdr normalizes
bare uppercase (`prefix+H`) and `prefix+shift+h` to the same key.

## Prefix layer (hold DEL) → tmux

When I'm running **tmux and not herdr**, the same keys work — this layer was built for tmux
first. `~/.config/tmux/tmux.conf` uses `Ctrl+A` as the prefix, not `Ctrl+B`. Mapping back: a
tmux **session** ≈ a herdr workspace, a tmux **window** ≈ a herdr tab, and tmux has no
concept of an agent.

### Navigating

| Hold DEL + | Emits | tmux action |
|---|---|---|
| `S` `D` `F` `G` or `H` `J` `K` `L` | `C-a` `h` `j` `k` `l` | Pane left / down / up / right |
| `;` | `C-a` `;` | Last pane — **tmux built-in**, needs no config |
| `←` `→` | `C-a` `C-h` / `C-a` `C-l` | Previous / next window |
| `↑` `↓` | `C-a` `C-k` / `C-a` `C-j` | Previous / next session |
| `[` `]` | `C-a` `C-p` / `C-a` `C-n` | *nothing* — no tmux counterpart to agents |
| `A` | `C-a` `w` | fzf **window** picker (`bind w`) — note: windows, not sessions |

### Acting on panes

| Hold DEL + | Emits | tmux action |
|---|---|---|
| `W` | `C-a` `v` | Vertical split |
| `E` | `C-a` `s` | Horizontal split |
| `R` | `C-a` `z` | Zoom / unzoom pane |
| `X` `C` `V` `B` | `C-a` `H` `J` `K` `L` | Resize pane 5 cells left / down / up / right |
| `N` | `C-a` `x` | Close pane (confirm) |
| `U` | `C-a` `[` | Enter copy mode |
| `I` | `C-a` `]` | Paste buffer |

### Windows and session

| Hold DEL + | Emits | tmux action |
|---|---|---|
| `1`–`9` | `C-a` `1`–`9` | Select window 1–9 |
| `,` | `C-a` `c` | New window |
| `M` | `C-a` `d` | Detach session |
| `O` | `C-a` `r` | **Zoom pane** — `bind r resize-pane -Z`, a leftover from an older keymap. Not reload. See below. |

### The tmux config side

The real file is **`~/.config/tmux/tmux.conf`** (not `~/.tmux.conf`), and it is not tracked in
this repo. It had drifted from the keymap — its comments still described an older layout where
resize was on `C-a` + arrows and `R` sent `C-a r`. Two additions bring it back in line:

```tmux
# Resize: the layer's X C V B emit C-a H/J/K/L, which were unbound
bind -r H resize-pane -L 5
bind -r J resize-pane -D 5
bind -r K resize-pane -U 5
bind -r L resize-pane -R 5

# Sessions: the layer's UP/DOWN emit C-a C-k / C-a C-j
bind -r C-k switch-client -p
bind -r C-j switch-client -n
```

Both are applied. The pre-existing `bind -r Left/Down/Up/Right resize-pane` lines are now
orphaned — no macro emits `C-a Left` any more — but they are kept as a manual fallback.

Still working off tmux defaults or existing bindings: `h j k l` (`select-pane`), `;`
(`last-pane`), `1`–`9` (`select-window`), `z` (zoom), `v` `s` (splits), `[` `]` (copy/paste),
`x` `d` `c`, `w` (fzf window picker), and `C-h` / `C-l` (prev/next window). `C-p` / `C-n` are
unbound, so the agent keys are harmless no-ops.

**Two things the layer cannot reach in tmux:**

- **Reload.** It lives on `bind R` — shift+R — and shift is unavailable on layer 6, so no
  macro can emit `C-a R`. Type `C-a` then `Shift+R` by hand.
- **Reload via the `O` key.** `O` emits `C-a r`, which hits `bind r resize-pane -Z` (zoom).
  That binding is redundant with `z` (which the `R` key already reaches), so rebinding
  `r` to `source-file` would make `O` mean reload in both tools. Left alone for now because it
  changes existing behaviour rather than filling a gap.

### Where the two tools genuinely differ

| Key | tmux | herdr |
|---|---|---|
| `[` `]` | Nothing | Previous / next agent |
| `↑` `↓` | Previous / next session | Previous / next workspace |
| `O` (`C-a` `r`) | Zoom pane (legacy `bind r`) | Reload config |
| `A` (`C-a` `w`) | Window picker | **Workspace** picker — one container level up |
| `U` (`C-a` `[`) | Copy mode, with its own keybindings | Scrollback opened in `$EDITOR` |
| `I` (`C-a` `]`) | Paste buffer | Unbound — no equivalent action |
| `X` `C` `V` `B` | Resize by 5 cells | Resize directly, no mode to exit |

Both tools swallow a bare `Ctrl+A`, so it no longer reaches the shell as beginning-of-line.
tmux already has the escape hatch configured — `bind-key C-a send-prefix`, so `C-a C-a` sends a
literal `Ctrl+A`. Herdr has no documented equivalent, so use `Home` inside herdr panes.

### Changing these bindings

The keyboard side lives in [`config/macros.dtsi`](config/macros.dtsi) as the `Tmux_*` macros
(a historical name — the layer now serves both tools), wired up on layer 6 of
[`config/adv360.keymap`](config/adv360.keymap). Editing a macro costs a `make all` and a
reflash, so prefer adjusting the herdr or tmux config instead. The full per-row breakdown is
in [prefix-layer-guide.md](prefix-layer-guide.md).

## Modifying the keymap

[The ZMK documentation](https://zmk.dev/docs) covers both basic and advanced functionality and has a table of OS compatibility for keycodes. Please note that the RGB Underglow, Backlight and Power Management sections are not relevant to the Advantage 360 Pro's custom ZMK fork. For more information see [this note](#note)

* If you would like to continue using GitHub we recommend using Nick Coutsos’s keymap editor: https://nickcoutsos.github.io/keymap-editor/.
* If you would prefer to leave GitHub and firmware flashing behind you can perform a one-time firmware update to gain access to Clique. Get started here: https://kinesis-ergo.com/360p-clique-upgrade/.

Certain ZMK features (e.g. combos) require knowing the exact key positions in the matrix. They can be found in both image and text format [here](assets/key-positions.md)

## Building the Firmware with GitHub Actions

### Setup

1. Fork this repo.
2. Enable GitHub Actions on your fork.

### Build firmware

1. Push a commit to trigger the build.
2. Download the artifact.

## Building the Firmware in a local container

### Setup

#### Software

* Either Podman or Docker is required, Podman is chosen if both are installed.
* Make is also required

#### Windows specific

* If compiling on Windows use WSL2 and Docker [Docker Setup Guide](https://docs.docker.com/desktop/windows/wsl/).
* Install make using `sudo apt-get install make` inside the WSL2 instance.
* The repository can be cloned directly into the WSL2 instance or accessed through the C: mount point WSL provides by default (`/mnt/c/path-to-repo`).

#### macOS specific

On macOS [brew](https://brew.sh) can be used to install the required components.

* docker
* [colima](https://github.com/abiosoft/colima) can be used as the docker engine

```shell
brew install docker colima
colima start
```
> Note: On Apple Silicon (ARM based) systems you need to make sure to start colima with the correct architecture for the container being used.
> ```
> colima start --arch x86_64
> ```

#### Ubuntu/Debian specific

```shell
sudo apt-get install docker make
```

### Building the firmware

1. Execute `make` to build firmware for both halves or `make left` to only build firmware for the left hand side.
2. Check the `firmware` directory for the latest firmware build. The first part of the filename is the timestamp when the firmware was built.

### Cleanup

The built docker container and compiled firmware files can be deleted with `make clean`. This might be necessary if you updated your fork from V2.0 to V3.0 and are encountering build failures.

Creating the docker container takes some time. Therefore `make clean_firmware` can be used to only clean firmware without removing the docker container. Similarly `make clean_image` can be used to remove the docker container without removing compiled firmware files.

## Flashing firmware

Follow the programming instruction on page 8 of the [Quick Start Guide](https://kinesis-ergo.com/wp-content/uploads/Advantage360-Professional-QSG-v8-25-22.pdf) to flash the firmware.

### Overview

1. Extract the firmwares from the archive downloaded from the GitHub build job (If using the cloud builder) or the firmware folder (If building locally).
1. Connect the left side keyboard to USB.
1. Press Mod+macro1 to put the left side into bootloader mode; it should attach to your computer as a USB drive.
1. Copy `left.uf2` to the USB drive and it will disconnect.
1. Power off both keyboards (by unplugging them and making sure the switches are off).
1. Turn on the left side keyboard with the switch.
1. Connect the right side keyboard to USB to power it on.
1. Press Mod+macro3 to put the right side into bootloader mode to attach it as a USB drive.
1. Copy `right.uf2` to the mounted drive.
1. Unplug the right side keyboard and turn it back on.
1. Enjoy!

> Note: There are also physical reset buttons on both keyboards which can be used to enter and exit the bootloader mode. Their location is described in section 2.7 on page 9 in the [User Manual](https://kinesis-ergo.com/wp-content/uploads/Advantage360-ZMK-KB360-PRO-Users-Manual-v3-10-23.pdf) and use is described in section 5.9 on page 14. 

> Note: Some operating systems wont always treat the drive as ejected after the settings-reset file is flashed or may throw a spurious error, this doesn't mean that the flashing process has failed.

### Upgrading from V2 to V3

If you encounter a git conflict when updating your repository to V3.0 please follow the instructions on how to resolve it [here](UPGRADE.md).

Updating from V2.0 based firmwares to V3.0 based firmwares can be a rather complex process. There are reset files for every major firmware revision as well as documentation on the update process available [here](https://kinesis-ergo.com/support/kb360pro/#firmware-updates).

## Versioning

Starting on 11/15/2023 the Advantage 360 Pro will now automatically record the compilation date, branch and Git commit hash in a macro that can be accessed with Mod+V. This will type out the following string: YYYYMMDD-XXXX-YYYYYY, where XXXX is the first 4 characters of the Git branch and YYYYYY is the Git commit hash. In addition to this the builds compiled by GitHub actions are now timestamped and also record the commit hash in the filename. 

## N-Key Rollover

By default this keyboard has NKRO enabled, however for compatibility reasons the higher ranges are not enabled. If you want to use F13-F24 or the INTL1-9 keys with NKRO enabled you can change `CONFIG_ZMK_HID_KEYBOARD_EXTENDED_REPORT=n` to `CONFIG_ZMK_HID_KEYBOARD_EXTENDED_REPORT=y` in [adv360_left_defconfig](/config/boards/arm/adv360/adv360_left_defconfig#L65)

## Battery reporting

By default reporting the battery level over BLE is disabled as this can cause some computers to spontaneously wake up repeatedly. If you'd like to enable this functionality change `CONFIG_BT_BAS=n` to  `CONFIG_BT_BAS=y` in [adv360_left_defconfig](/config/boards/arm/adv360/adv360_left_defconfig#L58).

## Modifier indicator color

The color of the CAPS/NUM/SCROLL LOCK indicator LEDs may be configured by specifying a hexadecimal RGB color code. For example, `CONFIG_ZMK_RGB_UNDERGLOW_MOD_COLOR=0xFF0000` would give red indicator colors. In order to set the indicator color on both modules, ensure that both [adv360_left_defconfig](/config/boards/arm/adv360/adv360_left_defconfig) and [adv360_right_defconfig](/config/boards/arm/adv360/adv360_right_defconfig) have been updated.

## Layer colors

A total of 32 layers are supported by ZMK, with the highest currently active layer displayed using the layer LEDs on each of the left and right modules. All possible colors are listed below; for the first 8 layers the same color is displayed on both modules. After that, only the right module color will cycle through until "rolling over", which will cause the left module color to change as well (and this then repeats). To avoid confusion, the black/off LED color is only used for layer 0.

| Layer # | L/R | Layer # | L/R | Layer # | L/R | Layer # | L/R |
| ---: | :---: | ---: | :---: | ---: | :---: | ---: | :---: |
| 0 | <img valign='middle' src='assets/swatches/000000.svg'/> <img valign='middle' src='assets/swatches/000000.svg'/> | 8 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> | 16 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> | 24 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> |
| 1 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> | 9 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> | 17 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> | 25 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> |
| 2 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> | 10 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> | 18 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> | 26 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> |
| 3 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> | 11 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> | 19 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> | 27 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> |
| 4 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> | 12 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> | 20 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> | 28 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> |
| 5 | <img valign='middle' src='assets/swatches/FF00FF.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> | 13 | <img valign='middle' src='assets/swatches/FFFFFF.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> | 21 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/0000FF.svg'/> | 29 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> |
| 6 | <img valign='middle' src='assets/swatches/00FFFF.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> | 14 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/FFFFFF.svg'/> | 22 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/FF0000.svg'/> | 30 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/00FFFF.svg'/> |
| 7 | <img valign='middle' src='assets/swatches/FFFF00.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> | 15 | <img valign='middle' src='assets/swatches/0000FF.svg'/> <img valign='middle' src='assets/swatches/00FF00.svg'/> | 23 | <img valign='middle' src='assets/swatches/00FF00.svg'/> <img valign='middle' src='assets/swatches/FF00FF.svg'/> | 31 | <img valign='middle' src='assets/swatches/FF0000.svg'/> <img valign='middle' src='assets/swatches/FFFF00.svg'/> |

## Changelog

The changelog for both the config repo and the underlying ZMK fork that the config repo builds against can be found [here](CHANGELOG.md).

## Beta testing

The Advantage 360 Pro is always getting updates and refinements. If you are willing to beta test you can follow [this guide from ZMK](https://zmk.dev/docs/features/beta-testing#testing-features) on how to change where your config repo points to. The `west.yml` file that is mentioned is located in config/. [This link](config/west.yml) can take you to the file. Typically you will only need to change the `revision: ` to match the beta branch. There is currently no beta branch available for testing.

Feedback on beta branches should be submitted as a GitHub issue on the base ZMK repository as opposed to this config repository.

In the event of a major update the beta branch may not be compatible with the current mainline version of the config repository. If this is the case it will be detailed here along with instructions on how to update.

## Note

By default this config repository references [a customised version of ZMK](https://github.com/ReFil/zmk/tree/adv360-z3.5) with Advantage 360 Pro specific functionality and changes over [base ZMK](https://github.com/zmkfirmware/zmk). The Kinesis fork is regularly updated to bring the latest updates and changes from base ZMK however will not always be completely up to date, some features such as new keycodes will not be immediately available on the 360 Pro after they are implemented in base ZMK.

Whilst the Advantage 360 Pro is compatible with base ZMK (The pull request to merge it can be seen [here](https://github.com/zmkfirmware/zmk/pull/1454) if you want to see how to implement it) some of the more advanced features (the indicator RGB leds) will not work, and Kinesis cannot provide customer service for usage of base ZMK. Likewise the ZMK community cannot provide support for either the Kinesis keymap editor, nor any usage of the Kinesis custom fork.

## Other support

Further support resources can be found on Kinesis.com:

* https://kinesis-ergo.com/support/kb360pro/#firmware-updates
* https://kinesis-ergo.com/support/kb360pro/#manuals

In the event of a hardware issue it may be necessary to open a support ticket directly with Kinesis as opposed to a GitHub issue in this repository.
* https://kinesis-ergo.com/support/kb360pro/#ticket

