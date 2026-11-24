# rom

Python CLI to find and download ROMs, ISOs and Switch games from the terminal.
It has an interactive UI (Textual) like `epub-search`, name search, downloads with
a progress bar, and a folder organised by platform.

> English-language version of the original Portuguese project by
> [sostenesapollo](https://github.com/sostenesapollo/rom-cli). Thanks to the
> original author for the base work — this fork translates the interface and
> docs to English and tracks the upstream fixes.

## Requirements

- Python 3.8+
- macOS or Linux
- `textual` (`install.sh` installs it into its own venv)

## Installation

```bash
git clone https://github.com/danisiber/rom-cli.git
cd rom-cli
./install.sh
```

This creates a venv in `~/.venvs/rom`, installs `textual`, and publishes `rom`
(and `rom-remote`) into `~/bin`. Use `PREFIX=/usr/local ./install.sh` for a
different destination.

## Usage

```bash
rom                          # opens the platform picker and browses
rom -p switch                # opens straight into the Switch catalog
rom -p switchports           # only the Switch ports/homebrew
rom "metroid"                # searches every source
rom -y -p gba "pokemon"      # downloads the best match without asking
rom --list "metroid"         # lists only, does not download
rom --print-url -p switch …  # prints the direct link (aria2/curl)
rom -l                       # lists the platforms
rom --index                  # builds/prints the catalog index
rom --refresh                # rebuilds the index (ignores the cache)
```

With no query, `rom` opens the **platform picker**; once you pick one, it lists
that platform's whole catalog (pages of 300, `n`/`b` to move) — so you can
browse without typing anything. Typing in the field and pressing `Enter` goes
back to searching.

In the UI: `↑↓` navigate, `space` mark, `d` download (the marked ones or the
current one), `p` switch platform, `n`/`b` page, `ctrl+b` options palette,
`ctrl+a` select all, `ctrl+n` clear, `q` quit.

| Flag | Description |
|------|-------------|
| `-p, --platform` | platform (switch, snes, gba, ps2, nds, wii…) |
| `-y, --yes` | download the best match without the menu |
| `--list` | list only |
| `--print-url` | resolve and print the direct link only |
| `--refresh` | rebuild the catalog index |
| `--index` | download/print the index and exit |
| `-n, --number` | how many results to show (default 15) |
| `-o, --out` | destination folder (default `~/Downloads/roms` or `$ROM_DIR`) |
| `-l, --platforms` | list platforms |

Platforms: `switch`, `switchports`, `ps5`, `ps4`, `ps3`, `psv`, `ps2`, `ps1`,
`psp`, `nds`, `gba`, `snes`, `pc`, `dos`, `nes`, `gb`, `gbc`, `n64`, `3ds`, `gc`,
`wii`, `wiiu`, `xbox`, `xbox360`, `genesis`, `mastersystem`, `gamegear`, `segacd`,
`dreamcast`, `saturn`, `arcade`.

Files go to `~/Downloads/roms/<platform>/`.

## How it works

Each platform has an ordered list of providers (`PLATFORMS` in `bin/rom`); a
search queries all of them and merges the results.

- **RomsFun** (`romsfun.com`): WP REST search + the public download page, which
  hands out a signed `statics.romsfun.com` link. Covers most retro consoles plus
  PS3/PS4/PS Vita, Wii U, Xbox/360, PC and DOS. RomsFun has no Switch files.
- **RomsGames**: search + a POST that returns the file (NES … PS2, Sega, arcade).
- **Switch** — four providers, best first:
  - **romslab** (`romslab.com`, about 9k Switch posts): the post's "Mirror 2"
    filekeeper link resolves to a signed CDN URL in a single request. Base games
    only; updates and DLC sit on captcha-protected datanodes.
  - **Archive.org file index**: the `.nsp`/`.nsz`/`.xci` files inside public
    multi-game items, matched by filename. Login-restricted items are skipped.
    Cached for 7 days in `switchindex.json`; the first Switch search builds it,
    which takes a minute or two.
  - **Archive.org item search** for single-game items.
  - **nswgame** (`nswgf.com`, the Switch sister site of dlpsgame): pixeldrain and
    mediafire links download directly. The other hosts (MegaUp, datanodes, gofile,
    buzzheavier…) sit behind Cloudflare or a captcha, so for those `rom` only
    prints the links.
  - The old dlarchive source closed its public API (HTTP 403) and was removed.
- **Archives stay as downloaded**: `.rar`/`.7z`/`.zip` files are not extracted.
  Extract them yourself (`7z x file.rar`).
- **Switch ports**: Archive.org scan (`switch port` / `homebrew port` /
  `nsp`+`port`), 7-day cache.
- **PS3 and PS Vita**: NoPayStation — downloads `.pkg` straight from Sony's CDN
  (`zeus.dl.playstation.net`), no token needed.
- **PS4 and PS5**: dlpsgame (search via WP REST). The direct download works when
  the post links a supported host (pixeldrain, mediafire, Google Drive, 1fichier);
  when it doesn't, `rom` shows the links so you can grab the file yourself.
- **Best sellers first**: browsing a platform lists the best-selling titles
  (Wikipedia lists, 30-day cache) with ★; picking one searches the providers.
- **Fallback**: Archive.org on platforms with a thin catalogue.
- Caches live in `~/.cache/rom-cli/`; `--refresh` refetches them.
- Ranking is by title match, with a penalty for hacks/translations/betas and a
  preference for `Base` over `Update`/`DLC`.
- Files are validated before being kept: zip, 7z, `Rar!` for `.rar`, `PFS0` for
  `.nsp`, and `PKG`/`CNT` for `.pkg`.
- Downloads are resumable (`.part` + `Range`).

## Remote mode (`rom-remote`)

Runs on the Mac worker (`ssh mac`) and brings the files back. Useful when this
Mac's network blocks the download hosts.

```bash
rom-remote -p switch "mario kart"    # opens the UI on the worker
rom-remote -y -p snes "metroid"      # downloads there and syncs back here
```

Tune it with `ROM_REMOTE_HOST`, `ROM_REMOTE_DIR` and `ROM_DIR`.

## Notice

Only use what you have the right to.
