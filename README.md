# doto

**A plain-text task format.** One file, one task per block, one truth
per line — readable in any editor, parseable with a regex, diffable in
git, and structured enough to run a real life on.

```text
!task p-20261125-001
title: Renew the library books
status: next
meta: domain: personal | owner: lumi
due: 2026-11-28
context: errands
log: 2026-11-25 created
```

## Why another format?

todo.txt got the philosophy right — plain text forever — but real use
kept demanding things it can't say: more states than done/not-done,
*blocked* as a fact that composes with any state, a way to bundle
related tasks (paperclips), and provenance for tasks that arrive from
email. `.doto` makes those explicit instead of smuggling them into
tags.

## Highlights

- **Multi-state workflow** — `inbox`, `next`, `scheduled`, `waiting`,
  `someday`, `done`, `dropped`
- **Blocked is a field, not a status** — a task can be `next` *and*
  blocked
- **Paperclips** — a shared `clip:` label bundles related tasks
- **Email stamps** — `sender:`, `subject:`, `received:`, and an
  open-message link ride along with tasks born from mail
- **Append-only log** — every task carries its own history
- **Extensible** — unknown fields are preserved, never mangled

## Read the spec

📄 **[SPEC.md](SPEC.md)** — the full v0.2 specification
📋 **[examples.doto](examples.doto)** — a small annotated example store

## Status

v0.2, in daily production use (500+ tasks) as the single source of
truth behind a web app, native apps, reminder syncing, and an
assistant-driven workflow. The format is stable; the spec is the
contract.

## License

MIT — see [LICENSE](LICENSE). Use the format anywhere.
