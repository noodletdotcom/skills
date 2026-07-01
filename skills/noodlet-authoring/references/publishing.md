# Publishing via the teacher web app

Open the teacher app at https://teachers.noodlet.com → **Noodlets**.

## Create and upload

1. **New noodlet** — give it a title.
2. **Upload files** — `index.html` is required. Add any assets (SVG, images, audio,
   CSS) at their relative paths. A whole `dist/` folder from a bundler is fine.
3. **Publish** — the platform sandboxes the lesson and returns:
   - A **preview URL**: open it to confirm the lesson runs and reports a score.
   - **Warnings**: each names the sandbox rule it hit and the fix (see
     [sandbox-rules](https://cdn.noodlet.com/skill/references/sandbox-rules.md)). Address them and
     publish again. Publishing only fails outright if there is no `index.html`.

Re-publishing keeps the noodlet's URL and all class assignments intact.

## Assign to a class

**Classes** → open a class → **Assignments** → add the noodlet.
See [classes](https://cdn.noodlet.com/skill/references/classes.md) for creating classes and
enrolling students.

---

Prefer tools over the web app? Connect the Noodlet MCP server
(`claude mcp add --transport http noodlet https://mcp.noodlet.local/mcp`) —
`edit_noodlet`, `publish_noodlet`, `create_class`, `assign_noodlet`, and
`list_progress` run the same pipeline without leaving the conversation.
