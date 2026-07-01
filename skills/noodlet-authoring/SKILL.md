---
name: noodlet-authoring
description: >-
  Builds and publishes sandboxed interactive lessons ("noodlets") for students.
  Use when a teacher wants to create a quiz, game, drag-and-drop activity, or
  any interactive lesson — and whenever the words "noodlet", "lesson", or
  "publish" appear in an authoring context.
---

# Authoring noodlets

A **noodlet** is a single self-contained interactive HTML activity — a lesson, quiz,
game, or test — that runs in a locked-down sandbox and reports a score back to the teacher. You write the HTML; the platform
sandboxes it and makes it playable by a class.

A minimal lesson is one HTML file with an inline `<script>` that calls
`window.Noodlet.createLesson(...)`. A built `dist/` folder works too — upload every
file at its relative path, with an `index.html` at the root. If you build with a
bundler, prefer readable, unminified output: it keeps the publish checks legible and
lets you (and the platform) see exactly what runs in front of students.


## Before you start

Aim for a working first pass quickly. A topic or subject is enough to begin — infer a sensible age group, activity type, and structure from context, then build it. Only ask a clarifying question if the request is genuinely blank (no subject at all). Once the teacher can see a preview, that's the right moment to ask refinement questions (age group, difficulty, number of questions, style, etc.).

## Workflow

Write the lesson → publish via the teacher web app → assign to a class → students play and scores appear in the teacher's progress view.

> **AI note:** You should not need to fetch any files from `/sdk/` to understand how to author a noodlet — the full SDK contract is in `references/contract.md` below. Include the script tag in the lesson HTML you write, but do not fetch or read the SDK source itself.

## Sub-files — read as you work

Fetch these progressively as each step becomes relevant, not all at once:

- [references/contract.md](references/contract.md) — the SDK API: createLesson, onStart, submitResult, complete — and how to get the SDK.
- [references/sandbox-rules.md](references/sandbox-rules.md) — hard CSP rules (no network, no external scripts, no inline handlers) — read before writing code.
- [references/design.md](references/design.md) — design considerations for your audience: feedback style, retries, competition, and interface clarity.
- [references/publishing.md](references/publishing.md) — uploading and publishing a noodlet via the teacher web app.
- [references/classes.md](references/classes.md) — creating classes, enrolling students, assigning noodlets, tracking progress.
