# 1. Record architecture decisions

Date: 2026-07-05

## Status

Accepted

## Context

Significant technical decisions in CargoTrack (choice of database, messaging pattern, auth strategy, service boundaries, etc.) need a durable record of *why* they were made, not just *what* was decided — otherwise that reasoning is lost as soon as the people who made the call move on or forget.

## Decision

We will use Architecture Decision Records, as described by Michael Nygard, and keep them in `cargotrack-docs/adr/`.

Each ADR is a short markdown file with these sections: Status, Context, Decision, Consequences. ADRs are numbered sequentially and are not edited after acceptance — a decision that changes gets a new ADR that supersedes the old one.

## Consequences

- Future contributors can see why a decision was made without needing to ask the original author.
- Decisions that turn out to be wrong are easy to trace and reverse deliberately, rather than silently worked around.
- Adds a small amount of process overhead for genuinely significant decisions — not intended for routine implementation choices.
