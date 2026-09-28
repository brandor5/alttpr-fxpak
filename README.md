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

It leans on the SNI install from `../snes-ansible`: the last step is that
setup's `send_file`, which copies the ROM to the cart and boots it.

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

```sh
mkdir -p ~/.local/bin && ln -s /work/alttpr-fxpak/alttpr-fxpak ~/.local/bin/
mkdir -p ~/.config/alttpr-fxpak && cp config.example.toml ~/.config/alttpr-fxpak/config.toml
```

Then edit `~/.config/alttpr-fxpak/config.toml` and point `base_rom` at your
vanilla Japanese 1.0 ROM. That is the only required setting.

## Usage

```sh
alttpr-fxpak                                    # default open preset, roll and play
alttpr-fxpak --preset crosskeys
alttpr-fxpak --set world_state=inverted --set goal=pedestal
alttpr-fxpak --hash 9MQ9gBKAMD                  # build someone else's seed
alttpr-fxpak --race                             # no spoiler log, flagged as a race
alttpr-fxpak --no-send                          # build only, leave the cart alone
alttpr-fxpak --list-presets                     # presets and every valid --set value
```

`--list-presets` reads the live settings document, so it always matches what
the site currently offers rather than a list baked in here.

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
transfer failed. Check SNI and the cart:

```sh
systemctl --user status sni
journalctl --user -u sni -n 30
```

**`base ROM has CRC32 ... the patch expects ...`** — wrong ROM. It must be the
Japanese 1.0 release, 1,048,576 bytes unheadered.

**alttpr.com timed out** — generation genuinely can take a minute or two under
load, and heavier presets take longer. The timeout is five minutes; retry.

## Caches

- `~/.cache/alttpr-fxpak/base/` — base BPS patches, keyed by randomizer build
- `~/.cache/alttpr-fxpak/sprites/` — downloaded `.zspr` files

Both are safe to delete; they refill on the next run.
