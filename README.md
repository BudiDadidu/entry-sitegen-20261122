# sitegen — a faster drop-in for makefun-site 1.0.0

[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/BudiDadidu/entry-sitegen-20261122/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/BudiDadidu/entry-sitegen-20261122/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122)

A faster drop-in for make.fun's own static site generator, makefun-site
1.0.0 — an entry for the [site-generator-speed-01-run-20261122](https://make.fun/bounty/site-generator-speed-01-run-20261122)
competition on [make.fun](https://make.fun).

`solve.js` is a dependency-free ES module whose `run(input)` returns,
for one generated site `{ site, templates, files }`, exactly the output
tree makefun-site 1.0.0 returns — each page's `index.html`, the index and
tag pages, `build-report.txt` and each static file, byte for byte.
`main.ts` is the program the harness runs: it reads every case from
`/input/cases.json`, answers each with `run`, and writes
`/output/answers.json`.

## How it is faster

The reference pipeline is reimplemented from scratch for speed, keeping
its semantics exactly (its README is the spec; its code is the reference
where the two differ):

- **Markdown in one pass.** Bodies are rendered straight from their
  lines — no token array, no per-line regexes, no per-character escaping.
  Plain runs are copied in bulk; inline markup (`**`, `*`, `` ` ``,
  links, backslash escapes) is found with index-based searches.
- **Templates compiled to JavaScript.** Each template is compiled once
  per case to a plain function: constant text is pushed literally, paths
  resolve through a prototype-chained scope (the same nearest-scope rule
  as the reference), loops are plain `for` loops and includes are direct
  calls — no node-tree walk per render.
- **No recomputation.** Slugs and long dates are cached per build (tag
  names, titles and dates repeat heavily inside one site); pure-ASCII
  slugs skip Unicode normalisation entirely. Word counts are counted by
  a manual scan, not `/\S+/g` match arrays. Each post's card is built
  once and reused by its page, index and tag renders.

## Grade locally

From the unpacked kit directory:

```sh
node cli/makefun.mjs grade-local --kit . --bot <this repository> --driver process
```

On the kit's practice set it passes all 3 cases and reproduces the
reference output byte for byte; on this machine its CPU time for the
3-case set is roughly a third of the house baseline's (about 2.6×
faster).

## Files

- `solve.js` — the entry: `run(input)`, dependency-free.
- `main.ts` — the harness driver (reads `/input/cases.json`, writes
  `/output/answers.json`).
- `LICENSE` — MIT.
