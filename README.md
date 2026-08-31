# Project page — Towards Understanding Humanoid Control through Network Weight Analysis

Anonymous build, for use during double-anonymous review.

```
site/
├── index.html            # the whole page — no build step, no dependencies
├── assets/
│   ├── fig/              # figures extracted from the paper PDF (webp)
│   └── video/            # ← put the video files here (currently empty)
└── README.md
```

## 1. Videos — already included

All four clips are cut from `0827_720p.mp4` and re-encoded, so the page works as shipped.

| File | Source range | Length | Size |
|---|---|---|---|
| `assets/video/final.mp4` | 1.0 – 28.9 s (whole reel) | 27.9 s | 7.6 MB |
| `assets/video/walking.mp4` | 1.5 – 8.4 s | 6.9 s | 1.8 MB |
| `assets/video/dancing.mp4` | 9.0 – 20.0 s | 11.0 s | 2.7 MB |
| `assets/video/spinkick.mp4` | 23.6 – 28.9 s | 5.3 s | 1.2 MB |

All are H.264 High @ 1280×720/30, audio stripped, `+faststart`. **The original was HEVC
(H.265)**, which Chrome on Windows/Linux and Firefox refuse to decode in an `<video>` tag —
that transcode is not optional. Poster frames are pulled from the encoded clips so there is
no visual jump when playback starts.

To re-cut a clip:

```bash
ffmpeg -y -ss <start> -t <dur> -i 0827_720p.mp4 -an -c:v libx264 -profile:v high \
  -crf 28 -preset slower -pix_fmt yuv420p -g 60 -movflags +faststart \
  -vf "scale=1280:-2,fps=30" assets/video/<name>.mp4

# and refresh its poster
ffmpeg -y -ss 0.3 -i assets/video/<name>.mp4 -frames:v 1 poster.png
```

CRF 28 was chosen by eye against the source — the foliage in this scene is expensive to
encode, and anything below CRF 26 pushes the teaser past 15 MB for no visible gain.

## 2. Anonymity

The page ships anonymised: no names, no affiliations, no institutional links, and the
footer credits only the upstream template. Two things to keep in mind while the paper is
under review:

- **Do not put this URL in the submitted PDF.** IEEE RAS double-anonymous rules count an
  identifying link as an anonymity violation. Use `anonymous.4open.science` if you need to
  link a repo.
- **The video you attach to the submission must be anonymised separately** — no lab logo,
  no names in subtitles, faces blurred. The version on this page can stay as-is.

To de-anonymise after acceptance, edit these spots in `index.html`:

1. the `.authors` / `.aff` / `.venue` block in `<header class="hero">`
2. the `href="#"` placeholders on the Paper / arXiv / Code buttons
3. the `@inproceedings` entry in the BibTeX section
4. the `nav .tag` label and the footer line

## 3. Deploy to GitHub Pages

```bash
git init && git add . && git commit -m "project page"
git branch -M main
git remote add origin git@github.com:<user>/<repo>.git
git push -u origin main
# then: repo Settings → Pages → Source: main / root
```

Served at `https://<user>.github.io/<repo>/`. Choose a repo name that does not identify
you while the paper is under review.

## 4. What's interactive

The Weight Space Analysis section runs: explorer, kinematic prior, RL vs IL, subspace
similarity, domain randomisation. Subsections are separated by a short left rule; the
full-width `<hr class="rule">` is reserved for the h2 sections.

- **Weight space explorer** — one control panel over motion × layer scope × training
  comparison, 15 views, covering Figs. 4a, 5, 6, 9, 10, 11 and 12. Arrow keys step
  through motions. Click any figure to zoom: rasters open at their own pixel grid,
  the vector three-layer figures at 3× (`VECTOR_ZOOM` in `index.html`; raise it freely, the
  browser only rasterises the tiles on screen so the factor costs nothing).
- **Domain randomisation switcher** — the same motion control over the DR / no-DR /
  difference figures.
- **RL vs IL panels** — the per-joint importance curves of the RL teachers and their
  distilled IL students, three motions side by side (Fig. 7), each click-to-enlarge.
- **Generalisation chart** — Table III plotted as three small multiples with hover
  tooltips and a table view.
- **t-SNE switcher** — Figs. 3 and 7 under one motion toggle.
- Collapsible tables for the architecture config, the embedding ablation, the domain
  randomisation terms (Table XI), the distillation setup (Tables IX–X), and the 29-joint
  reference (Table XV).

The page is light-mode only on purpose: every figure is a heatmap rendered on white, and
a dark theme would either invert the colour scales or leave white slabs floating on a
dark page.

## 5. Figure provenance

Most images in `assets/fig/` were extracted from the submitted PDF, so they are the
rasterised versions.

The six three-layer figures are the exception: `w_{walking,dancing,spinkick}_L3_{rl,rlil}.svg`
are **vector**, exported straight from matplotlib. Every heatmap cell is a `<rect>` and
every label is a glyph outline, so they stay sharp at any zoom — the page opens them at
10x. They go through two passes. `svgopt.py` rewrites matplotlib's per-cell rectangle paths
as `<rect class="cN">` against a 256-entry colour table; `svggrid.py` then notices the
cells sit on a regular grid and collapses each panel into one `<path>` per colour with
integer cell coordinates under a `translate/scale` group. Together: **7.4 MB -> 683 KB** raw for the RL figures and **14.7 MB -> 1.2 MB** for the
RL-vs-IL ones, 135 KB / 254 KB over gzip (GitHub Pages gzips by default); 40,093 and 80,186
DOM nodes drop to a few hundred.
Renders faster than the matplotlib original and differs from it only by antialiasing on
shared cell edges.

Still rasterised, and worth re-exporting as SVG if you have the sources: `method.webp`
(Fig. 2) and `jointwise.webp` (Fig. 4b), where the text is small.
