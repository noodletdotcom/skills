# The lesson contract

A noodlet reports a score back to the teacher through the Noodlet SDK. Call
`createLesson` once, render your activity in `onStart`, and report a result when the
student finishes. The lesson never receives student identity — the host attaches it
server-side.

```html
<!doctype html>
<html>
  <head><meta charset="utf-8" /><title>My Lesson</title></head>
  <body>
    <div id="app"></div>
    <script>
      const ctx = window.Noodlet.createLesson({
        onStart(ctx, info) {
          // info.entryPoint is "main" for now. Build your UI here.
          const btn = document.createElement('button')
          btn.textContent = 'Finish'
          btn.addEventListener('click', () => {
            // score / maxScore map to a 0..1 mastery fraction; passed is a real pass/fail.
            ctx.submitResult({ score: 3, maxScore: 3, passed: true })
            ctx.complete({ score: 3, maxScore: 3, passed: true })
          })
          document.getElementById('app').appendChild(btn)
        },
      })
    </script>
  </body>
</html>
```

## Context methods

`onStart(ctx, info)` hands you a context with:

- `reportProgress({ fraction })` — 0..1 progress as the student works.
- `saveState(obj)` — persist resumable state.
- `submitResult({ score, maxScore, passed, response?, duration? })` — record an attempt. `duration` is elapsed time in **milliseconds**; it is stored as an ISO 8601 duration string (e.g. `PT1M30S`).
- `complete(result?)` — mark the lesson finished (pass the final result).
- `requestNext()` — ask the host to advance to the next activity.
- `reportError(msg)` — surface a problem.

`score / maxScore` becomes a 0..1 mastery fraction; `passed` is a genuine pass/fail.
The iframe auto-resizes — don't set a fixed height.

## Getting the SDK

> **AI authoring note:** You should not need to fetch the SDK script or npm package
> to understand the API — `createLesson`, `onStart`, `submitResult`, `complete`, and
> the rest are all documented in this file. Include the script tag in the lesson HTML
> you write, but do not fetch or read the SDK source itself.

`window.Noodlet` is **injected automatically** at publish — whether or not a script tag
for it is already present (the injection is idempotent). Two ways to author against it:

- **Hand-written HTML.** Include
  `<script src="https://cdn.noodlet.com/sdk/noodlet-sdk.js"></script>` in the file. The tag
  is optional once published (the global is injected either way), but including it means
  the lesson runs correctly when opened directly in a browser during authoring — and
  critically, the SDK will surface sandbox violation warnings to the user in that mode,
  flagging network calls and blocked external scripts before you publish.
- **Bundler / npm.** Install the package and import it directly:

  ```sh
  npm install @noodlet/sdk
  ```
  ```js
  import { createLesson } from '@noodlet/sdk'
  ```

  The bundled copy is what runs locally (with dev-mode warnings). Publish still injects
  the global on top — harmless, the assignment is idempotent (last-write-wins).

  Keep the build unminified — set `build.minify: false` in Vite (or the equivalent for
  your bundler). Readable source keeps the publish warnings actionable and lets you and
  the platform audit exactly what runs in front of students.

See the **sandbox-rules** reference for the hard constraints every lesson must respect,
and the **design** reference for designing lessons for children.
