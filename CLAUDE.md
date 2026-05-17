# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A pair of standalone, self-contained HTML pages (`list1.html`, `list2.html`) that display the song request repertoire for the Korean duo **태사비애 · 지애** (TaeSaBiAe & Jiae). Each file is a complete page with inline CSS and inline JavaScript — there is **no build system, no package manager, and no server**. Open the file directly in a browser to run it.

The `src/Main.java` + `html_jiae.iml` + `.idea/` scaffolding is leftover IntelliJ project metadata from the "New Java Project" template. It is **not** part of the deliverable and should be ignored when working on the HTML files. Do not introduce Java code, a build step, or anything that depends on the Java module to make the HTML pages work.

## Working with the pages

- Edit `list1.html` / `list2.html` directly. No transpile, no bundle.
- To preview: open the file in a browser (no localhost required). The pages do fetch Google Fonts via CDN, so they need network access for fonts but otherwise work offline.
- `list2.html` is an expanded variant of `list1.html` — it splits Korean into 4 sections and Pop into 3, and uses a horizontally scrollable top nav. Many CSS rules and the entire `<script>` block are duplicated between the two files. **If you change behavior, update both files** unless the user explicitly says one only.

## Architecture (single-file pattern)

Each HTML file follows the same three-layer structure inside one document:

1. **Data is in the DOM.** There is no JSON, no data file. Every artist is a hand-authored `<div class="artist-row">` containing `<div class="artist-name">` (with optional `<small>` for the Korean transliteration of an English artist's name) and `<div class="artist-songs">` containing one `<span class="song-tag">` per song. Sections are `<section class="section" id="...">` and grouped under a `.section-header`. To add/remove songs, edit the markup directly.

2. **Client-side search (in the `<script>` at the bottom).** On load, the script snapshots the original HTML/text of each `.artist-name` and `.song-tag` into `dataset.original` / `dataset.text`. The single `input` event handler then:
   - Lower-cases the query and matches against the snapshots.
   - Re-renders innerHTML with `<mark class="highlight">` wrapping matched substrings.
   - Hides non-matching rows via the `.hidden` class and entire sections via `.section-hidden` when all rows in that section are hidden.
   - Calls `restore()` when the query is empty to put the original HTML back.

   The snapshot-in-dataset approach is load-bearing: highlighting mutates innerHTML, so anything that re-reads `.textContent` during search would see the marked-up version. Preserve this pattern when editing.

3. **Section nav + scroll spy.** The `.scroll-area` `<main>` is the scroll container (the page itself is `overflow: hidden`). Nav links use `data-target` and `scrollArea.scrollTo` with a manually computed offset (`getBoundingClientRect` delta + current `scrollTop`) — `scrollIntoView` is intentionally avoided because the scroll root is the inner `<main>`, not the document. An `IntersectionObserver` rooted on `.scroll-area` (with `rootMargin: '-30% 0px -60% 0px'`) toggles the `.active` class on whichever nav link's section is centered in the viewport.

## Conventions worth preserving

- Section IDs (`korean`, `pop` in list1; `kr-1..4`, `pop-1..3` in list2) are referenced by `data-target` on nav links and by `data-count-for` on the `.section-count` spans. Renaming a section ID requires updating both references.
- The `.section-count` spans are populated at runtime — leave them empty in the markup.
- The English-artist convention is `<div class="artist-name">English Name<small>한국어 표기</small></div>`. The `<small>` is part of the searchable text via `nameEl.textContent`, so a Korean query against an English artist matches the transliteration. Don't strip `<small>` or change to a sibling element without revisiting the search logic.
