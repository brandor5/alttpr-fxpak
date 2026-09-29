# alttpr-fxpak

Roll a seed on [alttpr.com](https://alttpr.com), build the ROM, and boot it on
the FXPak Pro — one command, no browser, no file manager.

```console
$ alttpr-fxpak --preset beginner
rolling a beginner seed ...
building ROM ...

/home/you/Games/alttpr/alttpr-JG8k3oAxyB.sfc
permalink: https://alttpr.com/h/JG8k3oAxyB
file select code: Ice Rod, Book, Moon Pearl, Lamp, Ice Rod

sending to the FXPak Pro ...
```

It needs [SNI](https://github.com/alttpo/sni) running with the FXPak Pro
connected. The last step runs SNI's `send_file`, which copies the ROM to the
cart and boots it.

## How it works

alttpr.com never serves a playable ROM, because the randomizer is a patch on a
commercial game. A seed comes back in two pieces:

1. A **BPS patch** that turns a vanilla Japanese 1.0 cartridge dump into the
   randomizer base ROM. This is the same for every seed on a given build, so
   it is downloaded once and cached.
2. A **seed patch** — a list of `{offset: [bytes]}` edits that turn that base
   into your particular seed.

This script applies both locally, writes your cosmetic preferences, fixes the
SNES header checksum, and hands the result to `send_file`.

**You supply the vanilla ROM.** Nothing here downloads copyrighted material.
Your dump is checked by CRC32 against the one the patch was built for, so a
wrong region, a 1.2 dump, or a bad rip is caught before anything reaches the
cart. A 512-byte copier header is detected and stripped, so `.smc` files from
older dumping tools work too.

Pure standard library — no `pip install`, nothing to keep up to date.

## Setup

You need Python 3.11 or newer, SNI, and your own ROM as described above.

From a clone of this repository:

```sh
mkdir -p ~/.local/bin && ln -s "$PWD/alttpr-fxpak" ~/.local/bin/
mkdir -p ~/.config/alttpr-fxpak && cp config.example.toml ~/.config/alttpr-fxpak/config.toml
```

Then edit `~/.config/alttpr-fxpak/config.toml` and point `base_rom` at your
vanilla Japanese 1.0 ROM. That is the only required setting, unless SNI's
`send_file` lives somewhere other than `~/.local/opt/sni/current/`; in that
case set `send_file` as well.

## Usage

```sh
alttpr-fxpak                                    # default open preset, roll and play
alttpr-fxpak --preset crosskeys
alttpr-fxpak --set world_state=inverted --set goal=pedestal
alttpr-fxpak --start-with PegasusBoots          # start with the boots
alttpr-fxpak --hash 9MQ9gBKAMD                  # build someone else's seed
alttpr-fxpak --race                             # no spoiler log, flagged as a race
alttpr-fxpak --dry-run                          # build only, leave the cart alone
alttpr-fxpak --list-presets                     # presets and every valid --set value
```

`--list-presets` reads the live settings document, so it always matches what
the site currently offers rather than a list baked in here.

### Settings

A seed starts from a **preset** (`--preset`, default `default`), and each
`--set KEY=VALUE` then replaces one of the preset's settings. Presets and
`--set` are meant to be combined:

```sh
# The open preset, but with gentler item placement and a sword to start
alttpr-fxpak --set item_placement=basic --set weapons=assured

# Beginner, but inverted and hunting for triforce pieces
alttpr-fxpak --preset beginner --set world_state=inverted --set goal=triforce-hunt
```

`--set` is applied last, so it wins over the preset and over flags like
`--spoilers` and `--race`. If the same key is set twice, the later one wins.
Every value is checked against the site's own list before anything is sent,
so a typo fails immediately with the valid choices.

The presets on alttpr.com are `beginner`, `crosskeys`, `default` (open),
`nightmare`, `quick` and `veetorp`. There is no "casual" preset; "casual"
usually means `item_placement=basic`, as in the first example above.

| Setting | Values | Controls |
|---|---|---|
| `glitches_required` | `none`, `overworld_glitches`, `hybrid_major_glitches`, `major_glitches`, `no_logic` | Which glitches the logic may require of you |
| `item_placement` | `basic`, `advanced` | `basic` is the beginner-friendly placement; `advanced` is the usual race placement |
| `dungeon_items` | `standard`, `mc`, `mcs`, `full` | What leaves its dungeon: nothing; maps and compasses; plus small keys; everything, big keys included |
| `accessibility` | `items`, `locations`, `none` | 100% inventory, 100% locations, or only guaranteed beatable |
| `goal` | `ganon`, `fast_ganon`, `dungeons`, `pedestal`, `triforce-hunt`, `ganonhunt`, `completionist` | What finishes the seed |
| `tower_open` | `0`–`7`, `random` | Crystals needed to enter Ganon's Tower |
| `ganon_open` | `0`–`7`, `random` | Crystals needed before Ganon can be beaten |
| `world_state` | `standard`, `open`, `inverted`, `retro` | Starting state of the world |
| `entrance_shuffle` | `none`, `simple`, `restricted`, `full`, `crossed`, `insanity` | How doors and caves are shuffled |
| `boss_shuffle` | `none`, `simple`, `full`, `random` | Which boss is in which dungeon |
| `enemy_shuffle` | `none`, `shuffled`, `random` | Which enemies appear where |
| `enemy_damage` | `default`, `shuffled`, `random` | How hard enemies hit |
| `enemy_health` | `default`, `easy`, `hard`, `expert` | How much damage enemies take |
| `pot_shuffle` | `on`, `off` | Whether pot contents are shuffled |
| `hints` | `on`, `off` | Telepathic tile and NPC hints |
| `weapons` | `randomized`, `assured`, `vanilla`, `swordless` | `assured` starts you with a sword |
| `item_pool` | `normal`, `hard`, `expert`, `crowd_control` | How generous the item pool is |
| `item_functionality` | `normal`, `hard`, `expert` | How strong the items are |
| `spoilers` | `on`, `off`, `generate`, `mystery` | Spoiler log policy; same as `--spoilers` |
| `pseudoboots` | `true`, `false` | Dash from the start without the boots; the real boots are still in the world |
| `allow_quickswap` | `true`, `false` | Whether the seed allows item quickswap |
| `tournament` | `true`, `false` | Race seed; `--race` also sets this |

This table reflects the site as of September 2026. `--list-presets` prints
the current values, so trust it if the two ever disagree.

True/false settings also accept `on`/`off`, `yes`/`no` and `1`/`0`.

### Cosmetics

`--heart-speed`, `--heart-color`, `--menu-speed`, `--no-music`,
`--reduce-flashing`, `--quickswap` / `--no-quickswap`, and `--sprite`.

`--sprite` takes either a path to a `.zspr` file or the name of a sprite on
alttpr.com, matched case-insensitively and cached after the first download:

```sh
alttpr-fxpak --sprite "Purple Chest"
alttpr-fxpak --sprite ~/sprites/mysprite.zspr
```

Set the ones you always want in `config.toml` and forget about them; the flags
override the config for a single run.

`--no-music` is the one to use with an MSU-1 soundtrack on the cart.

### Starting items

`--start-with ITEM` puts an item in your inventory from the start, and takes
it out of the world so you don't find a second one. Repeat it for more than
one item. A casual boots seed is:

```sh
alttpr-fxpak --set item_placement=basic --set weapons=assured --start-with PegasusBoots
```

That is the open preset with basic item placement, a sword from the start,
and the Pegasus Boots from the start. `--preset beginner --start-with
PegasusBoots` is the same idea in standard mode; `beginner` already assures
the sword.

Item names are alttpr.com's own, such as `PegasusBoots`, `Flippers`,
`MoonPearl` and `Hookshot`. `--list-items` prints all of them, and matching
ignores case.

Starting items need a different generator on alttpr.com, the customizer, and
that brings a few limits:

- **No entrance shuffle.** The customizer returns entrance-shuffled seeds
  without any starting equipment, so the combination is refused.
- **New seeds only.** `--start-with` can't change a seed passed with
  `--hash`. A starting-items seed rebuilds from its hash like any other,
  items included.
- **Three hearts to start**, as usual. The customizer counts starting health
  as equipment, so the tool always includes it.

### Dry runs

`--dry-run` stops after the ROM is written and prints the `send_file` command
it would have run, so you can send it later or inspect the ROM first:

```console
$ alttpr-fxpak --dry-run
/home/you/Games/alttpr/alttpr-JG8k3oAxyB.sfc
permalink: https://alttpr.com/h/JG8k3oAxyB
file select code: Ice Rod, Book, Moon Pearl, Lamp, Ice Rod

dry run: built but not sent. Send it yourself with:
  /home/you/.local/opt/sni/current/send_file /home/you/Games/alttpr/alttpr-JG8k3oAxyB.sfc
```

The seed is still rolled on alttpr.com and the ROM is still written — the cart
is the only thing left alone. `--no-send` is kept as an alias.

## Output

Each run writes two files to `out_dir` (default `~/Games/alttpr`):

- `alttpr-<hash>.sfc` — the ROM, also what gets sent to the cart
- `alttpr-<hash>.json` — permalink, file select code, and the seed's settings

The **file select code** is the five items shown on the in-game file select
screen. Compare it with the other runners before a race to prove everyone is
on the same seed.

With `--save-spoiler` the sidecar also gets the full spoiler log, assuming the
seed was generated with spoilers enabled.

## Troubleshooting

**`send_file exited 1`** — the ROM built fine and is still on disk; only the
transfer failed. Check that SNI is running and the FXPak Pro is connected,
then send the ROM again with `send_file <path to the .sfc>`.

**`base ROM has CRC32 ... the patch expects ...`** — wrong ROM. It must be the
Japanese 1.0 release, 1,048,576 bytes unheadered.

**alttpr.com timed out** — generation genuinely can take a minute or two under
load, and heavier presets take longer. The timeout is five minutes; retry.

## Caches

- `~/.cache/alttpr-fxpak/base/` — base BPS patches, keyed by randomizer build
- `~/.cache/alttpr-fxpak/sprites/` — downloaded `.zspr` files

Both are safe to delete; they refill on the next run.

## License

Apache License 2.0 — see [LICENSE](LICENSE).

The ROM assembly data (cosmetic offsets, value tables, sprite injection, the
header checksum, and the file select code table) and the customizer flags
used for starting items are adapted from
[pyz3r](https://github.com/tcprescott/pyz3r) by Thomas Prescott, also
Apache-2.0. [NOTICE](NOTICE) lists exactly what was taken and what changed.

The license covers this code only. It grants nothing to do with *A Link to
the Past* itself: this tool never includes or downloads the game, and you must
supply your own dump.
