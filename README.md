# changekey.to

A fast, mobile-first transposition reference for players of transposing instruments —
the kind of thing you keep open on your phone at the music stand.

It answers three questions, one screen at a time:

- **Play** — the conductor calls out a concert pitch; what do you play on your instrument?
- **Read** — you're playing a part written for another instrument; what key do you read it in, and how far do you transpose?
- **Pick** — you've been handed a part; which of your instruments is easiest to read it on?

## Design goals

- One big, glanceable answer per screen — never a wall of numbers.
- Plain language first (semitone counts and examples), music vocabulary second.
- Usable by a middle-schooler and a pro alike.
- Real notation: staves and key signatures drawn with Bravura (SMuFL) glyph outlines.
- No build step, no framework, works offline.

## Shareable links

State is encoded in the URL, so any view is bookmarkable / QR-able:

| Link | Opens |
| --- | --- |
| `changekey.to/Bb` | your instrument set to B♭ |
| `changekey.to/Bb/play/Eb` | Play, concert E♭ → what you play |
| `changekey.to/Bb/hear/C` | Play (reversed), your written C → its concert pitch |
| `changekey.to/Bb/from/A/2b` | Read, part for A with 2 flats |
| `changekey.to/pick/A/2b` | Pick a horn for an A part in 2 flats |

Notes use `b`/`s` for flat/sharp (e.g. `Fs`, `Bb`); key signatures are a count plus
`b`/`s` (e.g. `2b`, `3s`, `0`). Links are read from the path, the `#hash`, or `?r=`.

## Files

- `index.html` — the whole app: HTML + CSS + vanilla JS, no dependencies, no build.
  The Share panel's QR generator ([qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator),
  MIT) is inlined so the app is a single self-contained file.
- `wrangler.toml` — Cloudflare Worker (static assets) config; `not_found_handling =
  "single-page-application"` serves `index.html` for pretty paths like `/Bb`.
- `.assetsignore` — repo-root files that shouldn't be served.

## Adding or editing instruments

Instruments live in one place: the `CATALOG` array near the top of the `<script>`
in `index.html` (search for **"INSTRUMENTS — contributions welcome"**). Both the
Play and Pick views read from it, so an edit shows up everywhere. No build step —
edit, save, open `index.html`.

Each transposition (`C`, `B♭`, `A`, `E♭`, `F`, `G`, `D`) has an `insts` list. To add
or fix an instrument, edit the matching one. An instrument looks like:

```js
{ n:'alto sax', sound:-9, lo:'Db3', hi:'Ab5', less:true }
```

- **`n`** — display name.
- **`sound`** — how far it *sounds* from the written note, in semitones (`−` = lower).
  B♭ clarinet/trumpet `-2`, tenor sax/bass clarinet `-14` (an octave lower),
  French horn `-7`, alto sax `-9`, bari sax `-21`, E♭ sopranino clarinet `+3`.
  It must belong to that key's class — `sound` mod 12 is the same for every
  instrument in a key (every B♭ instrument is `-2`, `-14`, `-26`, …).
- **`lo` / `hi`** — lowest / highest *concert* (sounding) note it can play, as a name
  like `'D3'` or `'Bb6'` (`b` = flat, `#` = sharp). Used to grey out out-of-range
  results in Advanced mode. Approximate is fine.
- **`less`** — optional; `true` for rare/historical instruments (shown after
  "Less common:").

Leave each key's `shift` value alone — it's the pitch-class transposition and stays
fixed when you add instruments to an existing key.

## Deploy

Deployed as a Cloudflare **Worker with static assets** — no server code, no build step.
`wrangler.toml` points the Worker at the repo root (`[assets] directory = "."`) and sets
`not_found_handling = "single-page-application"` so pretty paths like `/Bb` serve
`index.html`; `.assetsignore` keeps repo-root files (README, config) from being served.

Hosting is via **Workers Builds** connected to this GitHub repo:

- pushes to `main` deploy to production (`changekey.to`);
- pushes to any other branch create a preview version with its own `*.workers.dev`
  preview URL, listed under Workers & Pages → the project → **Deployments**.

Locally, just open `index.html` in a browser — no server needed. To serve it exactly the
way Cloudflare does (SPA routing included), run `npx wrangler dev`. On any other static
host the `#hash` form of the links works with no configuration.

## Credits

Notation glyphs are outlines from [Bravura](https://github.com/steinbergmedia/bravura)
(SIL Open Font License).

## License

[MIT](LICENSE)
