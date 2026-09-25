# AGENTS.md — changekey.to

Orientation for an AI/agent working on this repo. Read this first.

## What this is

**changekey.to** — a fast, mobile-first transposition reference for players (and
arrangers) of transposing instruments. Live at https://changekey.to. Three modes,
one glanceable answer per screen:

- **Play** — convert between a written note and concert pitch (a "Concert | Written"
  direction toggle). Has an **Advanced** mode with per-instrument octave detail.
- **Read** — a part written for instrument X in key sig Y: what key do you read it in
  on your instrument, and how far to transpose. (Advanced octave view: NOT built yet.)
- **Pick** — handed a part, which of your instruments reads most easily (ranked by
  fewest accidentals; star to pin the ones you own).

## Architecture & hard constraints

- **One file: `index.html`.** All HTML + CSS + vanilla JS, no framework, no build step,
  no dependencies, no network calls at runtime (the QR library and Bravura notation
  glyphs are inlined). Keep it that way — don't add sibling `.js`/`.css`/font files
  (they complicate the static-asset deploy; we inlined the QR lib for exactly this).
- Mobile-first (~375px baseline), light/dark via `prefers-color-scheme`, colors as CSS
  variables on `:root`.
- Plain ES5-ish vanilla JS (var, function declarations). Match the surrounding style.

## Repo files

- `index.html` — the whole app.
- `wrangler.toml` — Cloudflare **Worker (static assets)** config: `[assets] directory = "."`
  and `not_found_handling = "single-page-application"` (serves index.html for pretty
  paths like `/Bb`). `name` must match the Cloudflare project name (`changekey`).
- `.assetsignore` — repo-root files not served (README, config, etc.).
- `README.md` — public readme, incl. "Adding or editing instruments".
- `LICENSE` — MIT.

## Core data model (search these names in index.html)

- **`CATALOG`** (~line 324) — one entry per transposition (`c, bb, a, eb, f, g, d`). Each has:
  - `key` — display key (`'B♭'`).
  - `shift` — signed semitone move **concert→written pitch class** (written above concert
    = positive), used for key math. `C 0, B♭ +2, A +3, E♭ -3, F +7, G +5, D -2`.
  - `insts` — the single instrument list **both Play and Pick derive from**. Each:
    `{ n:'alto sax', sound:-9, lo:'Db3', hi:'Ab5', less:true }`
    - `sound` — full **written→sounding** transposition in semitones incl. octaves
      (negative = sounds lower). Its value mod 12 is constant within a key (every B♭
      instrument is -2, -14, -26 …). Examples: B♭ clarinet -2, tenor sax/bass clarinet
      -14, alto sax -9, bari sax -21, E♭ sopranino +3, French horn -7.
    - `lo`/`hi` — **concert (sounding)** range as note names (`'D3'`,`'Bb6'`; `b`/`#`).
      Converted to MIDI at load by `nm()`. Used to grey out (strike) out-of-range results.
    - `less` — optional; rare/historical (shown after "Less common:").
- **`nm(name)`** — note name → MIDI (C4 = 60).
- **`exList(insts)`** — Pick's example text, derived from `insts` (common, then "Less common:").
- **`groupsOf(insts)`** — groups a class's insts by `sound` (octave), highest-sounding
  first. The Advanced Play block iterates these.
- Keys use a **fifths** integer (−7..+7): `FIFTH_NAME`/`FIFTH_MINOR`, `reduceFifths(f)`
  (normalizes to −5..+6, fewest accidentals), `keyName(f)` → "X major / Y minor" (relative
  minor is always shown). Transposition of a key sig: `read = reduceFifths(Wf + (Hshift −
  partShift)*7)`.

## Modes & state

`var state = {mode, pins[], instrument, playPc, playDir, playOct, advanced, readPart,
readSig, pickPart, pickSig}` — persisted in `localStorage['transpose.v1']`. **`advanced`
and `playOct` are intentionally NOT in the URL** (local prefs, keep links clean).

- `playDir`: `'play'` = Concert direction (input a concert pitch → written note);
  `'hear'` = Written direction (input a written note → concert pitch). Toggle labels are
  **"Concert" / "Written"**.
- `readSig`/`pickSig` are fifths ints; `readPart`/`pickPart` are the instrument the part
  is written for (a *different* concept from `instrument`, which is the one in hand).

## Advanced mode (Play) — the octave view (already shipped)

Header **Advanced** switch (`role="switch"`, `aria-checked`). When on, Play gains an octave
stepper on the keyboard and an "instruments & octaves" block rendered for **every** class
via `groupsOf`. Two directions are inverses:

- **Written** (`hear`): one written note in → each octave-group's **sounding** pitch varies
  (framing B). Header "Where it sounds". Play column uniform, Sounds varies (with "8ve
  lower" flags).
- **Concert** (`play`): one target sounding note in → each group's **written** note varies
  (framing A). Header "What octave to write". Sounds uniform, Play varies with per-group
  `readTag(-sound)` (compound intervals like ↑M9, ↑M13).

Other rules: headline shows an **octave only for single-voice classes** (`!multi`); multi-
voice classes show a **pitch class** (the octave is instrument-specific and lives in the
block). Instruments are **struck through** (`.oor`) when the sounding pitch is outside their
`lo`/`hi`; a whole row struck if all are (`.alloor`). Result caption is a sentence that
flows into the big note ("To hear concert A♭3 on a B♭ instrument, the written note is …").

**Read and Pick do NOT yet have an advanced octave view** — that's the next task.

## URL routing (search `buildRoute`, `applyRoute`, `currentRoute`, `normAcc`)

- `<inst>` → set instrument, Play. `<inst>/play/<note>` (note is concert),
  `<inst>/hear/<note>` (note is written), `<inst>/from/<part>[/<key>]` (Read),
  `pick/<part>[/<key>]` (Pick).
- Notes: `Bb`, `Fs`, `F#`; also spelled forms (`Bflat`, `F-sharp`, `2 flats`) via `normAcc`.
  Keys: count + `b`/`s` (`2b`, `3s`, `0`). Canonical output is `b`/`s`.
- Read from path (on http), `#hash`, or `?r=`. `updateURL` writes the **path** on
  http(s) and the **hash** on `file:`, and only **after the user interacts** (a fresh
  load keeps the shared link as-is).

## Notation (Bravura / SMuFL)

Treble staff + key signature drawn as inline SVG in `staffSVG(fifths, heightPx)`. The
clef/sharp/flat/natural are **inlined SVG path outlines** (`GLYPH.gClef` etc.) extracted
from the OFL Bravura font — 1000 units/em = 4 staff spaces, no font file. Used in Read.
For any new notation, reuse `staffSVG`/`GLYPH`; don't add a font.

## Dev & test workflow

- No build. Edit `index.html`, then serve locally to test:
  `py -3 -m http.server 8145` (Windows; `py -3` is the real Python — the `python`/`winget`
  in PATH are Store stubs that fail with "Permission denied"). Open
  `http://localhost:8145/index.html` in the **built-in browser** (`mcp__Claude_Browser__*`).
- **The desktop file preview loads local files as a `data:` URL, which strips the hash and
  path** — so URL-routing must be tested over the local **http** server, not a `file://`
  preview.
- Prefer driving/asserting via `javascript_tool` (set `state.*`, call `renderPlay()`, read
  the DOM) for exact checks; screenshot at mobile (`resize_window` preset `mobile` = 375×812)
  for layout, then reset to `desktop` when done. Kill the server after (`pkill -f
  "http.server"`).
- Verify transposition math against known cases (e.g. concert B♭ on a B♭ trumpet → play C;
  A-clarinet part in 2 flats → concert G major (1♯), read A major (3♯) on B♭, live shift
  down 1 semitone; bass clarinet sounds an octave below a soprano B♭).

## Deploy & branching

- Hosted as a **Cloudflare Worker with static assets**, built by **Workers Builds** from
  GitHub `vickyharp/changekey`. **`main` → production (changekey.to)**; other branches →
  **preview versions** at `https://<version>-changekey.vicky-harp.workers.dev` (the URL
  changes every build; find it under Workers & Pages → project → Deployments).
- Workflow: build features on a branch, push, verify on the preview URL, then merge to
  `main` to ship. Don't push to `main` until the user approves.
- `gh` CLI lives at `"/c/Program Files/GitHub CLI/gh.exe"` (not on the bash PATH), authed as
  `vickyharp`.

## Commit conventions

- End every commit message with:
  `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`
- **This Git Bash breaks on heredocs inside `$(...)` and on em-dashes/smart quotes.** Write
  the commit message to a file in the scratchpad and `git commit -q -F <file>`, using plain
  ASCII (hyphens, straight quotes) in the message.
- Commits use the user's GitHub **noreply** email (privacy); it's already configured.

## Design principles (don't regress these)

- One big, glanceable answer per screen. "Too busy" is the cardinal sin.
- Plain language first: semitone counts + a concrete example; interval names secondary.
- Usable by a middle-schooler and a pro. Advanced detail is **opt-in** (Advanced off by
  default); the base experience stays simple.
- Say **"written note"**, not "your note". Always show the **relative minor** with a key
  ("A major / F♯ minor"). Key signatures are entered by **count** (♯/♭), names optional.
- Niche instruments are marked inline with "Less common:" — **no** reordering or badges.
  In Pick, users **star** the instruments they own; nothing is required.
- Instrument data must stay **contributor-friendly** (note-name ranges, documented in the
  `CATALOG` comment and README). Both Play and Pick must derive from the same `insts`.
