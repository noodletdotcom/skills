# Sandbox rules

A noodlet runs with a strict Content-Security-Policy on an isolated origin
(`*.cdn.noodlet.com`'s sandbox sibling). The policy is the real security boundary —
the SDK is just a convenience library on top of it. Author to these rules and your
lesson works the first time; the publish step warns you when it spots a violation.

## No network

`connect-src 'none'` — `fetch`, `XMLHttpRequest`, `WebSocket`, `EventSource` and
`navigator.sendBeacon` are all blocked. A noodlet must be completely self-contained:
no analytics, no third-party CDNs, no APIs. Precompute or inline anything you'd
otherwise fetch.

## No external scripts

`<script src="https://…">` won't load — only `'self'` and `cdn.noodlet.com` are allowed
in `script-src`. Inline your JavaScript, or bundle library code directly into the
file. Inline `<script>` blocks **are** allowed: the publish step hashes each one
into the CSP so it runs.

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
