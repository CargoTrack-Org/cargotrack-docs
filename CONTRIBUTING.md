# Contributing to CargoTrack Docs

## General

- Write in Markdown (`.md`). One topic per file — prefer several short, focused files over one long one.
- Every folder should have a `README.md` that explains what belongs in it, even if the folder is otherwise empty or sparsely populated.
- Use relative links between docs (e.g. `[core-service API](../api/core-service.md)`) so links still work if the repo is renamed or mirrored.

## Diagrams

- Store exported diagram images (`.png`, `.svg`) in `architecture/diagrams/`.
- If a diagram has an editable source (draw.io `.drawio`, Excalidraw `.excalidraw`, Figma link, etc.), commit the source file next to the exported image, or add a link to it in the relevant README.
- Prefer SVG or high-resolution PNG for anything containing text.

## Presentations

- Store slide decks in `presentations/`.
- Naming convention: `YYYY-MM-DD-<short-title>.pptx` (e.g. `2026-07-05-evaluation-demo.pptx`), so ordering by filename matches chronological order.
- If a deck is exported to PDF for easier viewing, commit both the source `.pptx` and the `.pdf`.

## Screenshots

- Store screenshots under `screenshots/<area>/`, grouped by feature or page (e.g. `screenshots/frontend/`, `screenshots/admin-dashboard/`).
- Naming convention: `<feature>-<short-description>.png` (e.g. `shipments-list-view.png`).
- Avoid committing screenshots that contain real credentials, tokens, or personal data — use the demo/test account.

## Architecture Decision Records (ADRs)

- Use `adr/` for any decision that would be expensive to reverse or non-obvious to a future reader (choice of database, messaging pattern, auth strategy, etc.).
- Copy `adr/0001-record-architecture-decisions.md` as a template for new ADRs, and number them sequentially.
