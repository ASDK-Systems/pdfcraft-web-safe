# Audit: pdfcraft-web v0.21.1

This repo re-hosts PdfCraft's web/WASM build for embedding in [VARKIS](https://github.com/ASDK-Systems)'s
Office Studio, and this file records a static security audit of that build, done
before it was uploaded here. Companion to [wordcraft-web-safe](https://github.com/ASDK-Systems/wordcraft-web-safe),
[gridcraft-web-safe](https://github.com/ASDK-Systems/gridcraft-web-safe) and
[deckcraft-web-safe](https://github.com/ASDK-Systems/deckcraft-web-safe)'s own `AUDIT.md`, same methodology.

## What was checked

Downloaded `pdfcraft-web-0.21.1.zip` directly from
[storytold/pdfcraft](https://github.com/storytold/pdfcraft)'s `v0.21.1` release,
verified it against that release's own `SHA256SUMS.txt`, then read every line of:

- `index.html` (1.6 KB)
- `pdfcraft-web-dbf16682ba914f2f.js` - the wasm-bindgen glue (183 KB)
- `snippets/eframe-d31a8450ea12dfc1/inline0.js` (3 lines)
- `_headers`, `HOSTING.md`

and grepped the raw main `.wasm` binary's string table (60 MB) for any
`http(s)://`-shaped string, since the glue JS is the only surface through which
compiled Rust can reach the network at all - a `.wasm` module has no ambient network
access of its own.

## Findings

**One real difference from the other three apps: a generic `fetch(url)` binding, not
just one hardcoded same-origin call.** The glue JS exposes `fetch` as a wasm-bindgen
function the compiled Rust code can call with any URL string it constructs, not only
the module's own wasm-load fetch. This is legitimate and documented, not a defect:
PdfCraft fetches its OCR text-recognition models (`models/*.onnx`, relative to the
page, gated by a SHA-256 check against `ATTRIBUTION.toml` before use) only when Scan
& OCR actually runs, and supports an optional `?file=<URL>` startup flag that fetches
a document from a URL the *embedding page* supplies (not document content, and not
used by VARKIS's own integration, which uses the `?host=parent` bridge instead).

**No evidence the desktop app's "check for updates" feature (confirmed from this
repo's own flatpak sandboxing manifest: "Help > Check for updates reaches GitHub")
is compiled into the web build.** Grepped the `.wasm` string table for
`api.github.com`, `check_for_update` and `CheckForUpdate` - zero matches. Only an
`update_check_at_start` *setting name* string was found (a boolean preference key
among several others, e.g. `show_page_edge_shadows` - shared settings-validation
code, not proof the network call itself is compiled into this target). No literal
GitHub API endpoint exists anywhere in the binary.

**`postMessage` appears only as the generic wasm-bindgen wrapper**, used by the
optional `?host=parent` embedding bridge documented in `HOSTING.md` (the same
`craft-web-host` contract gridcraft/deckcraft/wordcraft share - this app's own docs
name "GridCraft, WordCraft and PdfCraft" as sharing it explicitly). VARKIS does not
use the cross-origin mode - the iframe is always same-origin - and VARKIS's own
listener checks the message's `source`/`origin` before acting on it.

**No Python/Pyodide/CDN dependency.** Grepped the glue JS for `pyodide`, `python`,
`cdn.`, `jsdelivr`, `unpkg`, `cloudflare` - zero matches.

**The `.wasm` binary's other URL-shaped strings are inert.** PDF/XMP/XFA metadata
namespace constants (`ns.adobe.com`, `w3.org`, `aiim.org` - required by the PDF/A and
PDF/UA specifications themselves, never fetched), OOXML namespace constants (this
app also imports Office documents via `doc_combine`), and the usual static
about-box/credit links (Discord, getartcraft.com, GitHub issues for egui/wgpu/lopdf/
rerun, and the project's own releases page). None of these are reachable from any
JS import.

**Conclusion: nothing to strip.** Like the other three apps, this is a verbatim
mirror of upstream's own build, not a modified one - no Apache-2.0 4(b) "changed
files" notice applies. The broader `fetch` binding here is a real architectural
difference worth re-checking on a future version bump, not assuming stays a no-op.

## What this repo hosts

`LICENSE-MIT` and `LICENSE-APACHE` are copied verbatim from
`storytold/pdfcraft@v0.21.1`. The `v0.21.1` web build release asset is live:
uploaded via `gh release create`, then re-downloaded fresh and confirmed
byte-for-byte identical to the pinned SHA-256 both before upload and after.
VARKIS's own `ensureWebBuild('pdfcraft')` was run directly against this repo's
release URL (cache cleared first) and verified it downloads, verifies and extracts
correctly.
