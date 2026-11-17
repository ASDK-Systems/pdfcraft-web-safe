# pdfcraft-web-safe

An audited, re-hosted copy of [storytold/pdfcraft](https://github.com/storytold/pdfcraft)'s
web/WASM build, for embedding in [VARKIS](https://github.com/ASDK-Systems)'s Office
Studio instead of fetching it from storytold's own infrastructure at runtime.

See [`AUDIT.md`](./AUDIT.md) for what was checked, including a real difference from
the other three apps (a generic `fetch` binding, resolved as legitimate - OCR models
and the documented `?file=` flag, not reachable from document content). PdfCraft is
dual MIT/Apache-2.0 - see `LICENSE-MIT` and `LICENSE-APACHE`.

**Status:** audit complete. The `v0.21.1` web build asset is being uploaded to this
repo's own release next (`gh release create` - the binary is too large for a
file-content API call). Native desktop/CLI binaries are not mirrored here; VARKIS
downloads those straight from storytold, still verified against the same pinned
SHA-256 either way.
