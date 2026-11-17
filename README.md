# pdfcraft-web-safe

An audited, re-hosted copy of [storytold/pdfcraft](https://github.com/storytold/pdfcraft)'s
web/WASM build, for embedding in [VARKIS](https://github.com/ASDK-Systems)'s Office
Studio instead of fetching it from storytold's own infrastructure at runtime.

See [`AUDIT.md`](./AUDIT.md) for what was checked, including a real difference from
the other three apps (a generic `fetch` binding, resolved as legitimate - OCR models
and the documented `?file=` flag, not reachable from document content). PdfCraft is
dual MIT/Apache-2.0 - see `LICENSE-MIT` and `LICENSE-APACHE`.

**Status:** live. The `v0.21.1` web build release asset is uploaded and VARKIS's own
`ensureWebBuild('pdfcraft')` has been verified end-to-end against this repo's
`/releases/download/v0.21.1/` URL (download, SHA-256 verify, extract). Native
desktop/CLI binaries are not mirrored here; VARKIS downloads those straight from
storytold, still verified against the same pinned SHA-256 either way.
