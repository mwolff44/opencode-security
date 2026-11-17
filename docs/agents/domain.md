# Domain docs

## Layout

This is a single-context repository.
Domain vocabulary belongs in root `CONTEXT.md`; architectural decisions belong
in `docs/adr/`.
Create these lazily when vocabulary or a substantive architectural decision is
agreed.

## Consumer rules

Before exploring or editing, read `CONTEXT.md` if present and relevant ADRs.
If absent, proceed without scaffolding placeholder files.
Use canonical domain terms consistently.
Surface conflicts with an ADR rather than silently overriding it.
Keep implementation details out of the glossary.

A detector reports evidence of a suspected threat; no detection is not proof of
safety.
An enforcement control prevents an operation at a specified boundary.
Keep proposed behavior, static evidence, and runtime-verified guarantees distinct.
