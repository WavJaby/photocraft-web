# PhotoCraft: unofficial browser experience

Community hosting mirror intended to make PhotoCraft easier to try without installing it. PhotoCraft is made by the ArtCraft Team and the PhotoCraft contributors; this host is not affiliated with or endorsed by them.

[Original source](https://github.com/storytold/photocraft) · [Official website](https://getartcraft.com/apps/photocraft) · [Official desktop downloads](https://github.com/storytold/photocraft/releases/latest)

## Layout and use

`index.html`, `site.css` and `site.js` provide the independent hosting page, dismissible notice and upstream links. `app/` contains the official v0.3.0 web application plus upstream license notices. No upstream application code or brand assets are modified. The hosting page uses plain text attribution and no extracted ArtCraft logos.

Serve the site over HTTP/HTTPS; opening HTML with `file://` is unsupported. Build preview: `node scripts/build-site.cjs` prints a dry-run when the pinned Wasm is present; `node scripts/build-site.cjs --write` restores missing Wasm and creates `_site/`, refusing an existing output directory. Serve with `python -m http.server 8765 --bind 127.0.0.1 --directory _site`, then visit http://localhost:8765/. Serving the source root instead uses the restored single-file input. GitHub Pages uses the Actions workflow in `.github/workflows/pages.yml`; Wasm and generated compressed parts never enter Git. Relative URLs support project subpaths. The official module reads `?webgl` and `?cpu` directly from the main page. The notice preference is stored only in this browser's localStorage; "使用說明" reopens it, and clearing the checkbox restores reminders. If storage is blocked, reminders return next visit.

## Browser hosting

The default backend selection tries WebGPU where available. A reported GPU startup error retries the unchanged application once with `?webgl`; explicit `?webgl` or `?cpu` overrides disable that retry. Download errors, slow startup and failures after editing starts never trigger a backend reload. WebGL2 is a compatibility backend, not a guaranteed speed improvement; not all editor work runs on the GPU.

The hosting notice and navigation default to English. A browser whose first preferred language starts with `zh` receives Traditional Chinese (`zh-Hant`), including `zh-CN` and `zh-Hans`; no Simplified Chinese catalog is used. Other or missing language preferences fall back to English. This does not change the language of the unmodified PhotoCraft editor.

The upper toolbar can collapse; its choice persists in localStorage. When collapsed, its toggle sits to the left of the editor's Discord controls, with clearance based on the v0.3.0 default layout. Credits float transparently at the lower right and consume no layout row. The hosting shell uses PhotoCraft v0.3.0 ProMedium color values, including the blue accent inherited from Pro; it does not follow later editor theme changes automatically.

`scripts/build-site.cjs` checks the official Wasm size/SHA-256, gzip-compresses it, and creates content-addressed 512 KiB parts + manifest inside the deployment artifact. `service-worker.js` downloads four parts concurrently through a bounded window, checks each part's SHA-256, orders the compressed stream, and decompresses it into the original Wasm bytes. One retry per failed part is allowed; missing/corrupt parts stop the download with a visible error instead of silently starting another full transfer. Settings live in `delivery-config.js`.

Streaming compilation can proceed while downloading, but the response cannot reach EOF until the complete decoded size/SHA-256 matches the pinned official release; successful compilation/instantiation therefore requires verified original bytes. Only that verified binary enters the application cache. Percentage measures compressed download bytes for parts, decoded bytes for the original single-file path; compilation/GPU startup remains indeterminate. Both paths execute the same official binary. Cache failure does not block startup. Missing browser decompression support uses the verified original single-file path; blocked/unsupported service workers use the official loader without wrapper byte progress. The cache contains application code only, never edited images/documents. Cache eviction/private browsing can require another download. This is not a full offline application.

Below 900 CSS pixels, a separate zoom row fits the unmodified editor into a 960 CSS-pixel virtual viewport by default; 50%/75%/100% choices persist locally. The hosting toolbar initially collapses on small screens; its toggle stays in the hosting controls rather than covering the editor. Portrait controls are small; landscape is preferable. This adjusts display scale only, not browser feature availability or the editor's touch/keyboard behavior. Mobile verification uses browser device emulation, not a physical-device compatibility claim.

Earlier HTTP Range probes in isolated Chrome transferred the same 1 MiB twice in forward/reverse order with browser caching disabled: one request took 27.0–34.3 s; four took 25.3–28.6 s; sixteen took 29.9–31.0 s; thirty-two took 30.3–32.5 s. All returned HTTP 206 without gzip. These probes differ from the physical compressed resources now generated for deployment and cannot establish their performance. Pages already gzipped the original file, so compressed parts do not provide a new 50% reduction over its existing wire size. Cold-start speed depends on the CDN route and must be measured; more parts do not guarantee more throughput. Repeat-visit caching removes the large transfer.

Validation: `node --test tests/delivery.test.cjs` exercises concurrent delivery, retries, corruption, missing parts, timeout, decompression/integrity failure, compatibility paths, cache failure and cache retirement. When `_site/` exists, it also reconstructs every generated part and compares the bytes with the original artifact. This is unit/selftest evidence; actual editor startup and browser lifecycle require separate runtime checks.

## Web vs desktop: v0.3.0 source audit

The shared Rust engine and egui UI provide the same foundation; web functionality is not identical to desktop functionality. This is static/configuration evidence, not a comprehensive runtime parity test.

| Area | Web difference | Evidence in upstream v0.3.0 |
|---|---|---|
| Files | Picker imports bytes; saves download new files. Filesystem-dependent commands are disabled, including direct path/folder workflows | `apps/photocraft-web/src/web.rs:services`; `crates/engine/src/file_cmds.rs:native` |
| Recovery and presets | Web services omit crash-recovery hooks and persistent preset storage; brush presets are session-only | `apps/photocraft-web/src/web.rs:services`; `crates/ui-egui/src/lib.rs:Services` |
| Clipboard | Desktop image clipboard callbacks are not wired in web services; internal editing clipboard is a separate mechanism | `apps/photocraft-web/src/web.rs:services`; `crates/ui-egui/src/lib.rs:Services` |
| Fonts | No installed-system-font scan; optional craft-fonts are not embedded in the web release | `crates/text/src/lib.rs:global`; `crates/text/build.rs` |
| Performance and automation | Wasm jobs run inline; browser memory/GPU limits apply; no desktop TCP control server or desktop CLI executable | `crates/engine/src/jobs.rs:run`; `apps/photocraft-web/src/main.rs` |

Source: https://github.com/storytold/photocraft/tree/v0.3.0. Desktop also remains alpha: https://github.com/storytold/photocraft/blob/v0.3.0/docs/roadmap.md. Do not imply that all desktop features are complete or that browser-specific limitations also apply to desktop. Save work explicitly and evaluate desktop separately.

## Licenses and provenance

PhotoCraft is MIT OR Apache-2.0 at your option; see LICENSE-MIT and LICENSE-APACHE. Hosting-page code added here is MIT under LICENSE-MIT. Preserve [upstream NOTICE](app/NOTICE), [asset attribution](app/ATTRIBUTION.md) and their referenced license texts. ArtCraft trademarks have [separate terms](app/docs/brand/LICENSE-brand.txt).

Official binary source: https://github.com/storytold/photocraft/releases/tag/v0.3.0. Original release files were verified byte-for-byte against the locally downloaded release, not independently authenticated against release checksums. Added notices were taken from the upstream v0.3.0 tag. No complete transitive Rust-dependency license audit has been performed.

## Single-page hosting and updates

The community bootstrap in `app-loader.js` mounts the official canvas in the main document; no iframe is created. Original JS/Wasm are verified against `upstream-files.json`. The host, loader and layout are community code, not an official build or endorsement. Keep upstream copyright, licenses, NOTICE and third-party attributions; do not extract ArtCraft brand marks into the host.

Wasm and compressed parts are ignored. CI downloads the exact archive pinned by URL + SHA-256, restores the verified Wasm, generates a fresh Pages artifact, and deploys it without committing binaries. WordCraft previews must be archived as GitHub Release assets because upstream CI artifacts expire. Browser asset caches retire prior Wasm hashes on service-worker activation. Existing Git history is not rewritten by this change.

1. Select an explicit official web ZIP and verify its published SHA-256; never silently follow latest.
2. Run `node scripts/update-upstream.cjs --archive HTTPS_ZIP_URL --sha256 SHA256 --version VERSION` to inspect the update, then repeat with `--write`. The command rejects changed archive/bootstrap contracts; review upstream licensing, supplemental notices and web/desktop differences.
3. Run `node scripts/build-site.cjs --output _site-review --write` and `node --test tests/*.test.cjs`; perform fresh-browser import/edit/export, mobile and renderer-failure checks.
4. Commit only source, configuration and provenance metadata, then push. Pages builds its current version from scratch; neither the old nor the new Wasm/parts enter Git.

For a clean checkout, `node scripts/update-upstream.cjs --restore` restores only the pinned Wasm. `--restore --archive LOCAL_ZIP` accepts a local copy with the same pinned archive hash.
