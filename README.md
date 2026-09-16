# Stellar Atlas — Figma Make design prototype

Source repository for the Stellar Atlas interactive design prototype, prepared
for import into Figma Make.

This is a static prototype with mock data. There is no build step, no package
manager, no backend and no API credentials. Serve the directory over HTTP and
open `index.html`.

```
python3 -m http.server 8000
# then open http://localhost:8000/index.html
```

It must be served over HTTP rather than opened as a `file://` URL, because the
loader uses `fetch` (see below).

## Layout

| Path | What it is |
| --- | --- |
| `index.html` | `<x-dc>` template markup, the `<style>` block, the `<script data-dc-script>` tag, and the split-loader |
| `support.js` | The Design Craft runtime |
| `dc-logic/part-01..04.txt` | The prototype's logic, split across four files |
| `brand/` | Label, service and software marks |
| `public/` | Artist, release and editorial imagery |
| `remote/bcbits/` | Locally hosted album cover art |

## Please do not "fix" `dc-logic/`

Those four `.txt` files are **one JavaScript source split at arbitrary byte
offsets**. Individually they are not valid JavaScript and will not parse. They
are deliberately not named `.js` so that nothing tries to execute or lint them.

The split exists for one reason: Figma Make enforces a 1 MB per-file limit, and
this prototype's logic is a single 1.52 MB block. Splitting it keeps every source
file well under the limit (largest is ~382 KB).

Concatenating `part-01` through `part-04` in order reproduces the original source
byte for byte. Nothing was minified, reformatted, reordered or edited.

## How the loader works

`support.js` reads the prototype's logic from the `textContent` of the
`<script data-dc-script>` tag — see `parseDcDocument` in its `parse.ts` — and it
has no `src` support. So the loader at the end of `<body>`:

1. fetches the four chunks and joins them in order,
2. writes the result back into that script tag,
3. and only then injects `support.js`.

Because `support.js` is no longer in `<head>`, the
`x-dc{display:none!important}` rule it normally installs itself
(`hideRawTemplate`) is declared up front in `index.html` instead, so the raw
template never flashes. The rendered result, CSS cascade order and behaviour are
unchanged.

## Notes

- `support.js` loads React and ReactDOM from unpkg with SRI, so the prototype
  needs outbound network access to boot.
- Part of the Landing screen is gated on a `stellar-atlas:experience:v1` key in
  `localStorage`. A brand-new origin renders a first-run state; once that key is
  written it settles into the returning-visitor state. Both are original
  prototype behaviour.
- Five images in `public/` exceed 1 MB. They are the original bytes and were left
  untouched, since recompressing them would change the design's appearance.
