# Save Snapshot — PWA package

Installable web-app version of Blake's Save Snapshot collection, built to be
added to the AYN Thor's homescreen (Chrome → *Add to Home screen* /
*Install app*). It opens standalone, full-screen, and works offline.

## What's in this folder

| File | What it is |
|---|---|
| `index.html` | The Save Snapshot tool itself (same file as `~/workspace/your_files/save-snapshot.html`, plus PWA hooks). 17 snapshots preloaded; drag-and-drop / file-picker upload still works fully offline. |
| `manifest.webmanifest` | App name, icons, colours, `display: standalone`. |
| `sw.js` | Service worker. Precaches the whole app shell (page, manifest, icons, all box art) and runtime-caches PokeAPI sprites. Versioned per build, so a new deploy replaces the old cache. |
| `icons/` | Floppy-disk logo app icons (192 / 512 / 512 maskable / 180 Apple touch). Generated locally, no hotlinking. |
| `art/` | The 11 games' real box cover art, downloaded and bundled locally (libretro thumbnail archive — Bulbapedia's archive blocks automated downloads, so the packager falls back to libretro automatically). |

## Rebuild (after the tool changes — Zelda, new games, new snapshots)

```sh
python3 ~/workspace/emulator/build_pwa.py
```

It reads the current `~/workspace/your_files/save-snapshot.html`, localises
any new remote box-art URLs it finds (Bulbapedia first, libretro fallback,
with a map in the script for Pokémon + the six Zelda games), regenerates the
icons only if missing, and writes a fresh `sw.js` with a new cache version.
Art downloads are cached in `~/workspace/emulator/pwa-art-cache/`, so
rebuilds are fast. Re-run the command any time the tool file changes, then
re-deploy (below).

## Hosting — recommended: GitHub Pages

Why: Blake already has GitHub and uses GitHub Actions; Pages is free, serves
HTTPS automatically (required for installable PWAs), and updates flow from a
plain `git push`.

Exact steps (Blake's go-ahead needed — nothing here is connected yet):

1. **Connect GitHub** (one time): in Muse, open the GitHub connector and
   authorise; install the Muse GitHub App on the repo you will use.
2. **Create a repo**, e.g. `save-snapshot` (public is fine and free; private
   repos also work with Pages but need a Pro account for private Pages —
   keep it public).
3. **Push this folder's contents to the repo's default branch** (the files
   belong at the repo root: `index.html`, `manifest.webmanifest`, `sw.js`,
   `icons/`, `art/`). Either via git, or via the GitHub web UI's
   *Add file → Upload files*.
4. **Enable Pages**: repo → *Settings → Pages* → *Build and deployment* →
   *Source: Deploy from a branch* → branch `main`, folder `/ (root)` → Save.
   Wait ~1 minute. The app is live at
   `https://<username>.github.io/save-snapshot/`.
5. **On the Thor**: open that URL in Chrome → menu (⋮) → *Add to Home
   screen* → *Install*. The floppy-disk logo appears on the homescreen and
   opens the collection standalone, offline-capable.

**Update flow:** rebuild with `build_pwa.py` → commit + push the changed
files → Pages redeploys in ~1 minute → the Thor's installed app checks for a
new service worker on next launch and updates itself (old cache is deleted
by the new worker). No reinstall needed.

### Alternatives (not recommended)

- **Netlify Drop**: drag this folder onto `app.netlify.com/drop` — instant
  HTTPS URL, no repo needed. Needs a free Netlify account; updates mean
  re-dragging the folder (or connecting a repo for continuous deploys).
- **Cloudflare Pages**: dashboard upload or `wrangler pages deploy`; needs a
  free Cloudflare account; same instant HTTPS.
- **Vercel**: a Vercel connector exists in this environment; `vercel deploy`
  from the folder. Also fine, but one more account for no benefit here.
- **Muse artifact public link** (`save-snapshot` space, if shared): this
  gives a plain homescreen **shortcut**, not an installable PWA. Chrome's
  *Add to Home screen* works on any HTTPS page, but without a manifest and
  service worker it opens in a browser tab (address bar, no offline), and
  the artifact page cannot carry a manifest/service worker of its own. Its
  one advantage: the link always shows the latest space, because the space
  itself updates from the daily Drive watch.

## Honest limits

- **Snapshots are frozen at build time.** The PWA is a static bundle: it
  shows the collection as of the last build/deploy. It cannot reach Blake's
  Google Drive (a static page has no Google auth), so the Drive auto-refresh
  lives in the Muse space + the daily `save-snapshot-drive-watch` job. Fresh
  snapshots arrive when a new build is deployed.
- **The manual upload path works fully offline.** Drop any Red/Blue/Yellow
  (32 KB) or FireRed/LeafGreen/Ruby/Sapphire/Emerald (128 KB) save on the
  page and it parses in the browser, nothing uploaded. (Zelda/other games
  once their readers land in the tool.)
- **Sprites need network the first time.** Party/Pokédex sprites load from
  PokeAPI at runtime and are cached after first view; offline, a sprite that
  was never cached falls back to the Poké Ball placeholder — the card still
  renders with all its text stats. Box art is bundled, so it always works
  offline.

## Verification (done Nov 14, 2026, in this VM)

- Manifest parses; name/display/start_url/theme/background present; all four
  icons exist at their declared sizes (checked with PIL).
- `sw.js` passes `node --check`; precache lists exactly the 18 files on disk
  (index, manifest, 4 icons, 11 box arts), none missing.
- All 18 precache URLs + manifest + sw.js return HTTP 200 from a local
  server, manifest served as `application/manifest+json`.
- Both inline `<script>` blocks in `index.html` pass `node --check`.
- Zero remote box-art URLs remain in the page; all 11 art files are valid
  512×512 PNGs (visually confirmed: Red = Charizard box art).
- **Not verifiable here:** a live install prompt / service-worker
  registration in a real browser — this VM's Chromium refuses to start
  outside the live browser. The first real test is opening the deployed URL
  in Chrome on the Thor and using *Add to Home screen*.
