# Splat Segment Inspector

**→ [b-connolly.github.io/splat-segment-inspector](https://b-connolly.github.io/splat-segment-inspector/)**

A single-page tool for Gaussian splat captures published as **3D Tiles**. Paste
a hosted `tileset.json` and it fetches, gunzips and SPZ-decodes the tiles and
merges them in their projected frame; segment the result, correct the classes
by hand, and export the corrections.

It exists because ArcGIS cannot select or filter individual splats —
`GaussianSplatLayer` has no `filter`, no renderer and no query, and Pro's
documentation says plainly that *"individual Gaussian splats cannot be
selected"*. Judging a segmentation, picking clusters out of it, and correcting
the ones the rules got wrong all have to happen somewhere else.

## What it does

- **Decodes 3D Tiles in the browser.** No server, no Pro, no download step.
  On a 1,091-tile construction capture that is 3,288,740 splats in about
  30 seconds, landing on the same bounding box and anchor the Python
  implementation produces.
- **Renders real Gaussians.** Each splat's 3D covariance projected and
  eigen-decomposed into an oriented screen-space ellipse, sorted back to front
  every frame in a worker. Positions stay float32 throughout.
- **Segments.** Ground from a low quantile on a coarse grid, vertical faces
  from each splat's own covariance, grouping in plan rather than in 3D — which
  is what separates a column from the slab it is bolted to.
- **Selects and reclassifies.** Cluster, box, lasso and brush, through the
  cloud or visible-surface only; add and subtract latched or held; make classes
  of your own beyond the built-in ground/structure/column.
- **Exports what changed**, as runs of splat indices — kilobytes, not
  megabytes — to be replayed onto the original PLY.

## Your data stays yours

Everything runs in the page. Files are read through the File API and tiles are
fetched straight from their host to your browser; nothing is uploaded, stored
or sent anywhere, and the site itself is a single static HTML file with no
analytics and no backend. The only third-party request it makes is the IBM Plex
stylesheet from Google Fonts.

## The rest of the toolchain

This page is the interactive half of **SplatTilesKit**, a Python toolbox and
ArcGIS Pro toolbox that converts Gaussian-splat 3D Tiles to 3DGS PLY and back
with position and CRS preserved, builds the LOD pyramid ArcGIS publishes,
segments a capture, and turns the result into per-class splat layers plus a
selectable multipatch feature class.

The corrections this page exports are applied with:

```bash
splat-tiles relabel labelled.ply --edits labelled.edits.json -o fixed.ply
```

## Building it

There is nothing to build. `index.html` is one self-contained file — no
dependencies, no bundler, no server. Open it from disk and it behaves
identically to the hosted copy.

It is generated from `python/viewer/segment-inspector.html` in the
SplatTilesKit repository; edit it there.
