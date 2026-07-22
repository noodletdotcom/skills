# Sandbox rules

A noodlet runs with a strict Content-Security-Policy on an isolated origin
(`*.cdn.noodlet.com`'s sandbox sibling). The policy is the real security boundary —
the SDK is just a convenience library on top of it. Author to these rules and your
lesson works the first time. The publish step blocks definite errors and warns about
issues that may be intentional or non-essential.

## Missing createLesson

Call `window.Noodlet.createLesson(...)` once so the lesson can start, report results
and resize correctly. You can alternatively import `createLesson` from `@noodlet/sdk`.
This is required: if the publish checker cannot verify a call, the noodlet is not
published. Minified aliased imports are supported, but do not obfuscate the SDK call
beyond recognition.

## Missing local script

Every relative `<script src>` and relative JavaScript import must resolve to a file
in the upload. Include the missing file or correct its path.

## Invalid JavaScript

Reachable JavaScript must parse successfully. Fix syntax errors before publishing.

## Invalid HTML encoding

HTML files must be UTF-8 so they can be parsed, sandboxed, and have the SDK injected.

## Viewport layout

A noodlet is embedded in a page, not displayed as a full-window app. The host sizes
its iframe to fit the content, so there is no need to stretch the lesson to `100vh`.
Fixed heights are still available when they suit the activity.

## No network

`connect-src 'none'` — `fetch`, `XMLHttpRequest`, `WebSocket`, `EventSource` and
`navigator.sendBeacon` are all blocked. A noodlet must be completely self-contained:
no analytics, no third-party CDNs, no APIs. Precompute or inline anything you'd
otherwise fetch.

## No external scripts

`<script src="https://…">` won't load — only `'self'` and `cdn.noodlet.com` are allowed
in `script-src`. Inline your JavaScript, or bundle library code directly into the
file. Inline `<script>` blocks **are** allowed: the publish step hashes each one
into the CSP so it runs. Only scripts are hashed — inline `<style>` blocks and
`style="…"` attributes need no hashing because `style-src` allows `'unsafe-inline'`;
styles just work.

## No inline handlers

Attributes such as `onclick="…"` are blocked by the script policy. Attach event
listeners from an allowed script with `addEventListener` instead.

## External assets

Images (including SVG), audio, video, fonts and stylesheets may only come from the
lesson itself or from `cdn.noodlet.com`. `data:` URIs are allowed for images. Anything
else is blocked.

Assets uploaded alongside `index.html` (e.g. `shapes.svg`, `audio/click.mp3`) are
served from the same origin as the lesson and are referenced by relative path —
`<img src="shapes.svg">`, `<use href="shapes.svg#triangle">`, etc. There is no need
to inline them. Use separate files freely; only inline when the asset is tiny or
generated dynamically.

## No forms

`form-action 'none'` — a `<form>` can't submit anywhere. Handle input in JavaScript
(read field values, call `preventDefault`) rather than relying on submission.
