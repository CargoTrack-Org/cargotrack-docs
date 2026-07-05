# Contributing to CargoTrack Docs

## General

- Write in Markdown (`.md`). One topic per file — prefer several short, focused files over one long one.
- Every folder should have a `README.md` that explains what belongs in it.
- Use relative links between docs (e.g. `[core-service API](../api/core-service.md)`) so links still work if the repo is renamed or mirrored.

## Presentations

Architecture diagrams and product screenshots live inside the slide deck in `presentations/`, not as separate image files — see [presentations/README.md](presentations/README.md) for the naming convention. If a diagram or screenshot needs to be referenced from a markdown doc outside the deck, link to the relevant slide rather than duplicating the image.

## Architecture Decision Records (ADRs)

Use `adr/` for any decision that would be expensive to reverse or non-obvious to a future reader (choice of database, messaging pattern, auth strategy, etc.). Copy `adr/0001-record-architecture-decisions.md` as a template for new ADRs, and number them sequentially.
