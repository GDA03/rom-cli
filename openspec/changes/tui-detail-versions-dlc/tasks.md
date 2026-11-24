# Tasks

## 1. Data model and Switch grouping

- [ ] 1.1 Add Title ID helpers: base-TID mask, release number (`vN // 65536`), version tag, region tag, and a choice label builder. Verify with an assert self-check on these samples:
  - `[01004EA01CED8000][v0]` → Base, `[01004EA01CED8800][v196608]` → Update rev 3, `[01004EA01CED9001]` → DLC, all with base `01004EA01CED8000`;
  - `+[v1.2.1][US]` gives the version tag `v1.2.1` and the region `US`.
- [ ] 1.2 Group the matched files in `ArchiveOrg.index_results()` into one `Result` per base TID, with `meta["choices"]`, the "+N upd · M DLC" summary, defaults (`.nsp` base, newest update) and a flag for a missing base. Files without a Title ID stay single rows. Verify that `--list -p switch "zelda tears"` shows one TotK row whose choices include 3 base files, and that the 35 complete groups in the live index each produce one row.
- [ ] 1.3 Make `download()` expand a result into one job per picked choice (or the defaults), and keep `run_cli_download` and `-y` working. Verify in a temp dir: `run_cli_download` on a grouped result downloads the default base (Range-checked `PFS0`, not a full download, using a stubbed `stream`). Also verify that `rom -y -p gba "mario kart super circuit"` still saves the USA file.

## 2. romsfun versions

- [ ] 2.1 Extract `RomsFun.files(result)` from `resolve()`, and make `resolve()` honour `meta["picked"]`. Verify:
  - `files()` returns 7 labelled entries for Mario Kart Super Circuit;
  - picking the Europe entry resolves to a URL containing `Europe`;
  - the default still resolves to the USA file.

## 3. Link-only extras

- [ ] 3.1 Parse the romslab button labels and the nswgame/dlpsgame URLs into typed link-only choices (Base/Update/DLC, `direct: False`). Verify:
  - for the MK8DX romslab post: Base, Update and DLC on datanodes, plus the filekeeper Base;
  - for the MK8DX nswgame post: at least one Update and one DLC link.

## 4. TUI

- [ ] 4.1 Add `DetailScreen` on `i`: header, `RadioSet` for base/file/source, `SelectionList` for Update/DLC (link-only rows disabled), and read-only `TextArea` links. Confirm queues the picks. Verify with headless screenshots:
  - the TotK panel shows 3 base radios and the update list;
  - the romsfun MK8SC panel shows 7 radios;
  - an nswgame panel lists links by type.
- [ ] 4.2 Add the queue panel, with a `ProgressBar` per job, persistent errors and cancel (`x`). Add the `cancel` event to `stream()`, which keeps `.part`. Verify in the harness with a stubbed slow `stream`:
  - three jobs show separate progress;
  - a forced failure keeps its message;
  - cancel stops the job and its `.part` file remains.
- [ ] 4.3 Merge duplicate (title, platform) rows from different sources in the TUI, with `meta["alts"]` offered in the panel. Verify with a "mario kart" headless screenshot: Mario Kart DS for NDS appears once, and its panel offers romsgames and romsfun.
- [ ] 4.4 Order the picker head (Switch, PS4, PS2, GBA, NDS, PSP) and add a type-to-filter `Input`. Verify in the harness: typing "ps" leaves exactly PS1, PS2, PS3, PS4, PS5, PSP and PS Vita.
- [ ] 4.5 Update the README UI section and the `AGENTS.md` TUI notes (detail panel, queue, choices model, cancel). Verify that the documented keys match the bindings.

## Workflow follow-up

- Apply only after `tui-quick-fixes` is applied.
- Archive both changes in order (`tui-quick-fixes` first), so the `tui` main spec exists before this delta is merged.
