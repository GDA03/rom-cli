# Tasks

## 1. Best-seller marking

- [ ] 1.1 Make `pop_rank()` exact-match only, and make `annotate_popular()` skip titles that contain a `BAD` word. Verify with an assert self-check:
  - "Mario Kart 64: Chaos Blast" gets no pop when "Mario Kart 64" is listed;
  - an exact title gets its pop;
  - "Super Mario Kart (Hack)" gets no pop.
- [ ] 1.2 Re-run `python bin/rom --list -n 15 "mario kart" </dev/null`. Verify that ★ appears only on rows whose titles match list entries exactly, and that the top result is still a Mario Kart title.

## 2. Table presentation

- [ ] 2.1 Add a `direct` class attribute to every source, and render `↓`/`↗` in the first column. Verify with a headless screenshot of a mixed search showing both markers.
- [ ] 2.2 Rebuild the table columns per render and hide Size/Type when they are empty. Verify with screenshots:
  - a retro search ("mario kart" on GBA) has no Size/Type columns;
  - a Switch search shows Type.
- [ ] 2.3 Restyle marked rows to a readable bold `✔` and muted green background. Verify with a screenshot after marking two rows: the titles are legible.
- [ ] 2.4 Replace "todas" with "All" in the picker, `_hint` and `_render_page`. Verify that `grep -n todas bin/rom` returns nothing.

## 3. Keys, footer and status

- [ ] 3.1 Remove the mark-and-palette Enter handler and bind Enter to download the highlighted row. Verify in the harness, with the download function stubbed: Enter calls it with exactly the highlighted row and leaves `app.selected` unchanged.
- [ ] 3.2 Make `p` browse when the input is empty and search otherwise, and add `/` to focus the input. Verify in the harness:
  - with an empty input, picking PS5 sets `browse_items`;
  - with "zelda" in the input, picking Switch triggers a search;
  - `/` gives the Input focus.
- [ ] 3.3 Move key hints into `Binding(show=True)` labels, cut `#status` to one line of messages, and add the "200 of N" truncation notice. Verify with a 120-column screenshot with two rows marked: the status fits on one line and the footer shows the hints. Also verify a `_show` call with 412 fake results reports "200 of 412 results".
- [ ] 3.4 Update the "Known TUI issues" in `AGENTS.md` and the key list in the README UI paragraph. Verify that both match the new bindings.

## Workflow follow-up

- Archive with `/opsx-archive` after review.
- Apply `tui-detail-versions-dlc` after this change. It builds on the footer bindings and on Enter meaning download.
