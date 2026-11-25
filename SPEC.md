# The .doto Format — Specification v0.2

A `.doto` file is a plain-text task store: a sequence of task blocks,
separated by blank lines, that a human can read and edit in any text
editor and a program can parse with a regular expression. One file is
the whole store. There is no database, no index, and no lock-in — the
file *is* the interface.

This document describes the format as implemented by the reference
tooling (the Doto web app and `doto_task.py`) and as exercised by a
production store of 500+ tasks.

## 1. Design principles

1. **One truth per line.** Every fact about a task lives on exactly one
   line, as `key: value`. Nothing is nested; nothing spans lines.
2. **Human first.** The file must stay readable and hand-editable.
   Any tool that writes the file must preserve blocks and fields it
   does not understand.
3. **Append-friendly history.** A task's `log:` field accumulates a
   dated audit trail instead of overwriting the past.
4. **Extensible by convention.** New fields are added by using a new
   key. Readers ignore unknown keys; writers preserve them.

## 2. File structure

- Encoding: UTF-8. Line endings: LF.
- A file is zero or more **task blocks** separated by exactly one
  blank line. There is no file header.
- A block begins with a header line:

  ```
  !task <id>
  ```

  followed by one or more field lines.

- A field line matches `^([a-z_]+):\s*(.*)$` — a lowercase key, a
  colon, and a single-line value. Values never contain newlines;
  writers collapse internal whitespace to single spaces.

Example:

```
!task p-20261125-001
title: Renew the library books
status: next
meta: domain: personal | owner: lumi
due: 2026-11-28
context: errands
log: 2026-11-25 created
```

## 3. Task identifiers

An `<id>` is an opaque, unique-within-the-file string. The canonical
form is:

```
p-YYYYMMDD-<suffix>
```

where the date is the creation date and `<suffix>` is a sequence
number (`001`, `002`, …) or a short random token (`a1b2c3`). Tools
that rewrite the file may renumber ids that do not match the canonical
form, so external references should be treated as best-effort; look
tasks up by title when an id may have been renumbered.

## 4. Core fields

| Key       | Required | Meaning |
|-----------|----------|---------|
| `title`   | yes      | The task, in one line. |
| `status`  | yes      | Lifecycle state (see §5). May carry pipe-delimited annotations (§6). |
| `log`     | no       | Append-only history (see §7). |

## 5. Status vocabulary

| Status      | Meaning |
|-------------|---------|
| `inbox`     | Captured, not yet triaged. |
| `next`      | Actionable now. |
| `scheduled` | Has a date or a future trigger. |
| `waiting`   | Blocked on someone or something external (see also `blocked:`). |
| `someday`   | Parked deliberately; not actionable. |
| `done`      | Completed. Terminal. |
| `dropped`   | Abandoned without completion. Terminal. |

The first five are the **open** statuses. `done` and `dropped` are
**closed**.

Note that *blocked* is deliberately **not** a status: a task can be
simultaneously `next` and blocked. Blockedness is a field (§8), so it
composes with any status.

## 6. Pipe-delimited subfields

A field value may carry ` | `-separated segments. The first segment is
the field's primary value; later segments are `key: value` annotations:

```
status: done | domain: personal | owner: lumi
project: Household | area: Kitchen
meta: domain: personal | owner: lumi | priority: high
```

Readers that only need the primary value take the text before the
first `|`. Because `|` is structural inside values, writers must not
emit a bare `|` inside a segment's text (write `/` instead).

## 7. The log field

`log:` records what happened to a task, newest entries appended, never
rewritten:

```
log: 2026-11-25 created via doto_task.py
log: 2026-11-26 completed via doto_task.py
```

Multiple `log:` lines may appear in a block. (Legacy imports may pack
several entries into one line separated by ` ; `.) Each entry begins
with an ISO date where known.

## 8. Optional fields

| Key        | Meaning |
|------------|---------|
| `meta`     | Pipe-delimited annotations: `domain:`, `owner:`, `priority:` (`high` \| `normal`). |
| `project`  | Project name; may carry an `area:` subfield (§6). |
| `area`     | Area of responsibility (standalone form). |
| `due`      | Due date, `YYYY-MM-DD`. |
| `start`    | Earliest start / tickler date, `YYYY-MM-DD`. |
| `context`  | Where or with what the task can be done. |
| `next`     | The concrete next action, when the title alone is not enough. |
| `blocked`  | What the task is waiting on. A field, not a status (§5). |
| `clip`     | **Paperclip label.** Tasks sharing a `clip:` value form a bundle, presented together in supporting apps. |
| `tags`     | Comma- or space-separated labels. |
| `ref`      | External reference, conventionally `<system>:<id>` (e.g. `todinder-fu:td-114`, `things:R5LR…`). |
| `source`   | Where the task came from (e.g. `things-import`, `pocket-direct`). |

## 9. Email provenance stamps

A task created from an email carries up to four stamp fields, written
independently — a task may have any subset:

| Key        | Meaning |
|------------|---------|
| `email`    | A web URL that opens the source message (enables an "open email" link in apps). |
| `sender`   | The message's sender. |
| `subject`  | The message's subject. |
| `received` | When the message was received, `YYYY-MM-DD`. |

## 10. Conformance

A conforming **reader** must:

- Split the file into blocks on blank lines and ignore blocks whose
  first line does not start with `!task `.
- Read the first occurrence of each key in a block (except `log:`,
  which accumulates).
- Treat ids as opaque.
- Determine openness from the first `|`-segment of `status:`.

A conforming **writer** must:

- Preserve unknown fields and unknown blocks verbatim.
- Keep values single-line and whitespace-collapsed.
- Never emit a bare `|` inside a field segment.
- Append to `log:` rather than editing existing entries.
- Separate blocks with exactly one blank line.

## 11. Relationship to todo.txt

.doto shares todo.txt's convictions — plain text, one task store in
one file, no proprietary tooling required to read it. It differs where
real use demanded more structure: a multi-state status vocabulary
instead of done/not-done, blockedness as a composable field, paperclip
bundling, and first-class provenance stamps for tasks born from email.
A todo.txt file cannot express those without the same kind of
extensions .doto simply makes explicit.

## 12. Version history

- **v0.2** — Current. Adds `clip:` paperclips, independent email
  stamps, `meta:` annotations, and the follow-up `ref:` conventions.
- **v0.1** — Initial format: blocks, core fields, status vocabulary.

*The format is in daily production use. Suggestions welcome via issues.*
