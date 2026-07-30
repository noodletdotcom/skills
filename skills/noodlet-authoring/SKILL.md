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
bundler, prefer readable, unminified output: it keeps the build checks legible and
lets you (and the platform) see exactly what runs in front of students.

> **AI note — build what they asked for.** Never invent the topic. If the request
> doesn't say what the noodlet should teach or test, ask that one question and wait
> for the answer — a guessed topic wastes the teacher's time however good the
> activity is.
> Everything else is yours to decide: age group, activity type, length, structure.
> Pick sensible defaults and build, then name the choices you made in a line or two
> when you hand over the preview ("pitched at Year 4, five questions, retry on a
> wrong answer") so the teacher knows what they can push back on.

> **AI note — build it, don't describe it.** Making the lesson real is your job, not a
> set of instructions you hand over. Two separate steps: **build** turns the files into
> a playable lesson only the teacher can see — private, repeatable, no sign-off needed —
> and **assign** gives it to a class, the one step that reaches students, so check that
> one first. "Publish it", "make it live" or "get it to my class" means *build it, then
> assign it if I have a class*, never *send me the steps*. Use the Noodlet MCP tools if
> they're connected, else the device flow (you request a code, the teacher approves it
> in their browser, you upload). Report back in their words ("it's ready — here's a link
> to try it"), not tool names.

> **AI note — who you're talking to.** Assume the teacher is non-technical. You own
> the technical side end-to-end: writing the code, uploading, building, and fixing
> any warnings yourself. Talk to them the way you'd talk to a colleague in a staff
> room — what the lesson does, a link to try it, and what happens next. Don't surface
> IDs, tokens, HTTP calls, CSP/sandbox internals, file sizes, or tool mechanics; say
> "it's ready — here's the link to try it", not which checks it passed. Only step down
> into technical detail when something genuinely needs their decision, or when they
> ask technical questions themselves — then match the level they set.


## Before you start

Aim for a working first pass quickly. A topic is enough to begin — infer a sensible age group, activity type, and structure from context, then build it. The topic is the one thing you must not invent: if the request doesn't say what to teach or test, ask for it and wait. Once the teacher can see a preview, that's the right moment to ask refinement questions (age group, difficulty, number of questions, style, etc.) — and to tell them, briefly, which of those you decided for them.

## Workflow

Write the lesson → build it → assign it to a class → students play and scores appear in the teacher's progress view.

**You upload and build it**, not the teacher — don't stop at a finished file and a set of steps. Use the Noodlet MCP tools if they're connected, else the device flow in [publishing.md](publishing.md); handing over the upload steps is the fallback for when you can't do it yourself.

> **AI note:** You should not need to fetch any files from `/sdk/` to understand how to author a noodlet — the full SDK contract is in `contract.md` below. Include the script tag in the lesson HTML you write, but do not fetch or read the SDK source itself.

## Sub-files — read as you work

Fetch these progressively as each step becomes relevant, not all at once:

- [contract.md](contract.md) — the SDK API: createLesson, onStart, submitResult, complete — and how to get the SDK.
- [sandbox-rules.md](sandbox-rules.md) — hard CSP rules (no network, no external scripts, no inline handlers) — read before writing code.
- [design.md](design.md) — design considerations for your audience: feedback style, retries, competition, and interface clarity.
- [publishing.md](publishing.md) — uploading and building a noodlet yourself (device flow) — read this before telling anyone to upload anything; also covers the teacher web app.
- [classes.md](classes.md) — creating classes, enrolling students, assigning noodlets, tracking progress.
