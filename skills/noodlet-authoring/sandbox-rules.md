# Sandbox rules

A noodlet runs with a strict Content-Security-Policy on an isolated origin
(`*.cdn.noodlet.com`'s sandbox sibling). The policy is the real security boundary —
the SDK is just a convenience library on top of it. Author to these rules and your
lesson works the first time. The build step blocks definite errors and warns about
issues that may be intentional or non-essential.

## Missing createLesson

Call `window.Noodlet.createLesson(...)` once so the lesson can report results. You
can alternatively import `createLesson` from `@noodlet/sdk`. A lesson without the
call still builds and plays — Noodlet starts and resizes it — but it is **unscored**:
the teacher sees that it was opened, never a mark. The build says so with this
warning, so wire the call whenever marks matter. Minified aliased imports are
supported, but do not obfuscate the SDK call beyond recognition, or the checker
cannot verify it and the lesson is treated as unscored.

## Missing local script

Every relative `<script src>` and relative JavaScript import must resolve to a file
in the upload. Include the missing file or correct its path.

## Invalid JavaScript

Reachable JavaScript must parse successfully. Fix syntax errors before building.

## Invalid HTML encoding

HTML files must be UTF-8 so they can be parsed, sandboxed, and have the SDK injected.

## Viewport layout

A noodlet's viewport is its iframe, not the surrounding browser window. Using
`min-height: 100vh` on `body` or a root shell is encouraged when the lesson should
fill the available iframe height. Put outer spacing and the page background on that
shell rather than in a first child's collapsing vertical margin. Flex or grid can
then deliberately distribute any extra height when the activity benefits from it.

## Missing asset

An `<img>`/`<source>`/`<video>`/`<audio>`/`<track>` `src`, a CSS `url(...)`, or a
path a script hands to the browser to load (`img.src = "…"`,
`setAttribute("src", "…")`, `new Audio("…")`) should resolve to a file in the upload.
A path joined from pieces, such as `"assets/" + kind + ".png"`, is reported only when
no uploaded file could fit it (`assets/*.png`). Any other string in a script is left
alone, however much it looks like a file name: it may be the name a download is saved
under. The build therefore misses some missing files; the teacher's preview reports
those when the browser fails to load them. It's a non-blocking warning — include the
file at that path, or fix the path.

## No external network

`connect-src 'self'` — `fetch` and `XMLHttpRequest` can load the lesson's own files
by relative path, and nothing else:

```js
const data = await (await fetch('data.json')).json();
// or, in a module script:
import data from './data.json' with { type: 'json' };
```

Every other origin is blocked, `cdn.noodlet.com` included: no analytics, no third-party
APIs, no remote data. The lesson's origin serves its uploaded files and nothing more,
so there is no server for `WebSocket`, `EventSource` or `sendBeacon` to talk to. Ship
any data the lesson needs as a file in the upload, or inline it.

## No external scripts

Scripts run only from the lesson's own files and `cdn.noodlet.com` (`script-src`).
A library named in a `<script src="https://…">` tag **in the HTML file** is fine if
it comes from one of these public CDNs — the build downloads it once and serves the
copy from the lesson's own files, pinned so it never changes under a class:

- `cdnjs.cloudflare.com`
- `cdn.jsdelivr.net`
- `unpkg.com`
- `cdn.tailwindcss.com`

Pin a version in the URL. A script from anywhere else stops the build: inline it, or
upload the file alongside the lesson. What the build cannot do is see a script the
lesson asks for *while it is running* — a tag created in JavaScript, a dynamic
`import()`, an `import … from 'https://…'` in a module — and those are blocked.

Inline `<script>` blocks **are** allowed: the build step hashes each one into the
CSP so it runs. A script created at runtime has no hash and is blocked, which rules
out compiling JSX in the browser with Babel. Only scripts are hashed — inline
`<style>` blocks and `style="…"` attributes need no hashing because `style-src`
allows `'unsafe-inline'`; styles just work.

## No inline handlers

Attributes such as `onclick="…"` are blocked by the script policy. **Attach
listeners with `addEventListener`.** As a safety net, handlers written as attributes
in the HTML file itself are moved into a script by the build, with the same
behaviour (`this`, `event`, `return false`), so they keep working. Handlers that
only exist inside JavaScript strings are not — markup built at runtime, such as
`el.innerHTML = '<button onclick="pick(1)">'` or `setAttribute('onclick', …)`,
stays blocked and the button does nothing. For markup you generate, put what the
handler needs in a `data-` attribute and use one listener on the container:

```js
list.addEventListener('click', (event) => {
  const button = event.target.closest('[data-answer]');
  if (button) pick(Number(button.dataset.answer));
});
```

`href="javascript:…"` is blocked the same way: use a button, or `href="#"` with a
click listener that calls `preventDefault`.

## Dialogs suppressed

The lesson frame can't show the browser's own pop-ups. `alert()`, `confirm()` and
`prompt()` do nothing there: `confirm()` always answers Cancel and `prompt()` always
answers nothing, so an "Are you sure?" button never does anything.

**Use Noodlet's pop-ups instead.** They are on the SDK, look the same in every
lesson, and return promises (see the contract reference):

```js
button.addEventListener('click', async () => {
  if (await window.Noodlet.confirm('Start again?')) restart();
});
```

`window.Noodlet.alert(message)` and `window.Noodlet.prompt(message, initial)` work
the same way. Or show the message in the page, which is often better for children.

As a safety net the build points a native call at Noodlet's version wherever it can
do so without changing how the lesson behaves — a call inside a click handler, say.
Where it can't (the answer is read by other code, or the call is in a method, a
loop callback or top-level code), it warns with this rule and the call stays dead.

## External assets

Images (including SVG), audio, video, fonts and stylesheets may only come from the
lesson itself or from `cdn.noodlet.com`. `data:` URIs are allowed for images. Anything
else is blocked — a hotlinked image, an embedded video, a font fetched at runtime.

One exception, as for scripts: a stylesheet named in a `<link rel="stylesheet">` tag
in the HTML file is copied into the lesson by the build, with the fonts it refers
to, when it comes from the CDNs listed under "No external scripts" or from
`fonts.googleapis.com` (whose fonts come from `fonts.gstatic.com`). So a Google
Fonts link in `<head>` works as written.

Assets uploaded alongside `index.html` (e.g. `shapes.svg`, `audio/click.mp3`) are
served from the same origin as the lesson and are referenced by relative path —
`<img src="shapes.svg">`, `<use href="shapes.svg#triangle">`, etc. There is no need
to inline them. Use separate files freely; only inline when the asset is tiny or
generated dynamically.

## No forms

`form-action 'none'` — a `<form>` can't send its data anywhere. A form is still fine
as a way to collect an answer: the `submit` event fires (on its button, or on Enter),
so handle it in JavaScript — read the field values and call `preventDefault`. What
does not work is a form with an `action` that expects a server to receive it.

## Found when the lesson runs

The build reads a lesson; it never runs it. Some problems only exist once it is
running — a button wired up in a way the sandbox blocks, a library that will not
start. When the teacher tries the lesson in their preview, Noodlet records what the
browser blocked and adds it to that build as `findings` (in `last_build`, next to the
build's own `issues`). **After the teacher has tried a lesson, read `last_build`
again**: a finding is a real failure a student would hit, with the rule and the fix.
Findings only cover what the teacher actually pressed, and are cleared by the next
build. They use the rules above, and these:

### No eval

Running text as code — `eval`, `new Function`, a string passed to `setTimeout` — is
blocked. Libraries that evaluate strings (Alpine.js, Vue's in-page templates) won't
start. Use plain JavaScript, or a build of the library that doesn't.

### No injected scripts

Only scripts that were in the file when it was built are allowed to run. A script
created while the lesson is running — which is how Babel runs JSX in the browser — is
blocked. Compile beforehand and put the result in the file.

### Popups blocked

A lesson stays in its frame: `window.open` and links with `target="_blank"` open
nothing. Show the content in the lesson, or have the teacher share the link with the
class separately.

### Navigation blocked

A lesson cannot send its frame to another site: `location.href = …`,
`location.assign`, a `<meta http-equiv="refresh">` or an ordinary link to an outside
address. The request is never made, but the lesson is gone — the learner is left
looking at a blank frame. Keep every step on the lesson's own page (moving between
the lesson's own files is fine), and have the teacher share outside links with the
class separately.

### Script error

The lesson's own code threw an error, and whatever it was doing stopped. This is a
bug in the lesson, not something the sandbox did.

### Sandbox blocked

Something else the sandbox does not allow. The finding names the policy directive.
