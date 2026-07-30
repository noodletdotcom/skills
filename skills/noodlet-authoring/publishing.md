# Building and assigning a noodlet

Uploading and building the lesson is something you do, not something you explain.
Two routes:

1. **The Noodlet MCP server**, if it's connected — `edit_noodlet`, `build_noodlet`,
   `create_class`, `assign_noodlet` and `list_progress` run this same pipeline without
   leaving the conversation (`claude mcp add --transport http noodlet https://mcp.noodlet.com/mcp`).
2. **The device flow below**, if you have a shell — any coding agent, no MCP needed.

Only if neither is available does the teacher do it themselves in the web app (last
section, where the button is labelled **Publish**) — paste them the lesson file.

## Build it yourself (device flow)

Authorization works like `gh auth login`: you request a code, the teacher approves it
in their browser, and you poll for a token. They can create an account on the way, so
this works for a teacher who has never used Noodlet.

1. Request access:

   ```sh
   curl -sS -X POST https://api.noodlet.com/agent/device \
     -H 'Content-Type: application/json' \
     -d '{"client_name": "<your product name, e.g. Claude Code>"}'
   ```

   The response has `device_code` (secret — for your polling only),
   `user_code`, `verification_uri_complete`, `interval`, and `expires_in`.

2. Show the teacher `verification_uri_complete` as a clickable link and tell
   them the `user_code`, e.g. *"Open this link and approve the request — the
   page should show the code `WDJB-MJHT`."* They can log in or create an
   account on the way; the approval page comes back automatically (even after
   e-mail verification).

3. Poll every `interval` seconds until approved:

   ```sh
   curl -sS -X POST https://api.noodlet.com/agent/token \
     -H 'Content-Type: application/json' \
     -d '{"device_code": "<device_code>"}'
   ```

   While pending this returns `{"error": "authorization_pending"}`; on approval
   it returns `{"access_token": ..., "expires_in": ...}`. Stop on
   `access_denied` or `expired_token` (start over for a fresh code).

   If polling isn't possible in your environment, the approval page also offers
   the teacher the token behind a "give it to the agent manually" reveal — you
   can ask them to paste it to you instead.

4. Build it with the token (send it as `Authorization: Bearer <access_token>`
   on every call):

   Titles in API requests are plain text. Do not HTML-escape them: send
   `Lesson & Test`, not `Lesson &amp; Test`.

   ```sh
   # Create a draft — response has noodlet_id and source_url. The body also
   # takes optional "description" (shown to teachers/students), "teacher_notes"
   # (teacher-only), and "agent_notes" (notes to future-you about how the lesson
   # is built/designed) — set them when they'd genuinely help, not on every noodlet.
   curl -sS -X POST https://api.noodlet.com/teacher/noodlets \
     -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' \
     -d '{"title": "Fractions Quiz", "agent_notes": "Plain HTML+JS, no bundler"}'

   # Upload each file under its own name (repeat per file, keeping relative paths).
   # No renaming needed — only with several HTML files must the entry be index.html.
   curl -sS -X PUT "<source_url>/fractions-quiz.html" \
     -H "Authorization: Bearer $TOKEN" --data-binary @fractions-quiz.html

   # Build it — the endpoint is named /publish, after the button in the web app
   curl -sS -X POST "https://api.noodlet.com/teacher/noodlets/<noodlet_id>/publish" \
     -H "Authorization: Bearer $TOKEN" -H 'Content-Type: application/json' -d '{}'
   ```

   The response has a `preview_url` (share it with the teacher — it opens their
   preview page in the Noodlet web app, after signing in, and never expires) and
   `warnings` (sandbox rules the lesson hit). Fix warnings, re-upload the changed
   files, and build again — rebuilding keeps the noodlet's URL.

The token lasts a few hours and covers creating, updating, and building this
teacher's noodlets — not class or student management (the teacher does that in
the web app). Never ask for the teacher's password or session cookie.

Run this whole flow quietly. The teacher only ever needs three things from it:
the approval link and code (step 2), the preview link once it's built, and — if
a warning needs a content decision — a plain-language description of the change.
Everything else (tokens, noodlet ids, curl output, warning internals, upload
sizes) is your plumbing; keep it out of the conversation.

A built noodlet is private to its owner until it's assigned to a class, so building
on their behalf needs no sign-off of its own — the approval in step 2 is the
permission. Assigning it to students is where you check with them first.

## If the teacher does it in the web app

Only when you have neither MCP tools nor a shell — give them the lesson file and
these steps. Open the teacher app at https://teachers.noodlet.com → **Noodlets**.

1. **New noodlet** — give it a title.
2. **Upload files** — a single HTML file of any name is fine; only when there are
   several HTML files must the entry one be named `index.html`. Add any assets (SVG,
   images, audio, CSS) at their relative paths. A whole `dist/` folder from a bundler
   is fine.
3. **Publish** — the platform sandboxes the lesson and returns:
   - A **preview URL**: open it to confirm the lesson runs and reports a score.
   - **Issues**: each names the sandbox rule it hit and the fix (see
     [sandbox-rules](sandbox-rules.md)). Address them and
     publish again. Errors such as missing `createLesson`, missing local scripts,
     invalid JavaScript, or code the sandbox is guaranteed to block prevent publish.
     Advisory layout and offline-compatibility warnings do not.

Re-publishing keeps the noodlet's URL and all class assignments intact.

## Assign to a class

**Classes** → open a class → **Assignments** → add the noodlet.
See [classes](classes.md) for creating classes and
enrolling students.
