# Domain Documentation

This repo uses a **single-context** layout for domain documentation.

## Structure

- `CONTEXT.md` — Domain glossary at the repo root
- `docs/adr/` — Architecture Decision Records

## Consumer Rules

1. Read `CONTEXT.md` to understand domain vocabulary before making changes
2. Check `docs/adr/` for past architectural decisions that may constrain implementation
3. Use domain terms consistently in code, comments, issues, and PRDs
4. When adding new domain concepts, update `CONTEXT.md`
5. When making architectural decisions, create a new ADR in `docs/adr/`
