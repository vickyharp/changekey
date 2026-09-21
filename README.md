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
- `_redirects` — SPA fallback so pretty paths (`/Bb`) resolve on Cloudflare Pages / Netlify.

## Deploy

Static hosting, no backend. On Cloudflare Pages the `_redirects` file makes the
pretty paths work; on any static host the `#hash` form works with no config.

## Credits

Notation glyphs are outlines from [Bravura](https://github.com/steinbergmedia/bravura)
(SIL Open Font License).
