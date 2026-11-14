# Nemeq Specification

Design documents for the Nemeq operating system.

Nemeq is an open-source ecosystem for embodied AI agents. This repository
holds the specification: terminology, architecture, and the principle of
Quoriosity that makes agents verifiable.

## Documents

- [00 Glossary](00-glossary.md) — full vocabulary, brand term first
- [01 NEMEQ expansion](01-nemeq-expansion.md) — official definition
- [02 Architecture](02-architecture.md) — kernel, agents, zones, sleep
- [03 Quoriosity](03-quoriosity.md) — verification principle and its rules

## Conventions

- **Brand term first in the glossary**: `Sittu (Sleep)`, `Tush (Incarnation)`.
  In prose the English term may lead at first mention: "sleep (Sittu)".
- **Latin letters only** — no diacritics in code, filenames or brand terms.
  Academic transliterations (`nēmequ`, `bārû`) may appear only in citations.
- **One concept — one name.** `incarnation` is the English term, `Tush` its
  brand. `embodiment` survives only as a technical path
  (`/var/nemeq/embodiments/`), never as a prose term.
- **Numbers live in data, not in prose.** Any numeric claim must carry
  a link to its source, or be marked as unverified.

## Links

- Main site: [nemeq.org](https://nemeq.org)
- Developer portal: [nemeq.dev](https://nemeq.dev)

## Status

v0.2 — sleep and incarnation model rewritten (Sittu/Tebu/Umu, Ug/Zinuntu,
memory bridge). Breaking changes expected until v1.0.
