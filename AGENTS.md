# AGENTS.md

## Scope
These instructions apply only to this repo.

## Blog basics
- Jekyll posts live in `_posts/`.
- Filename format: `YYYY-MM-DD-title.md`.
- Default layout is `post` via `_config.yml`.
- Current convention: **no front matter** (see existing post).
- Timezone: `America/Sao_Paulo` (use this when picking dates).

## Post creation workflow
- Keep the author's text intact and **only fix spelling** by default.
- Do not rewrite sentences, change meaning, or adjust style unless explicitly asked.
- Preserve Markdown structure, code blocks, links, and headings.
- If the user provides a title/date/slug, use those verbatim.
- If not provided:
  - Derive `title` from the first H1 if present; otherwise infer a short title from the content.
  - Derive `slug` by lowercasing, replacing spaces with hyphens, and removing non‑alphanumerics.
  - Use today’s date in `America/Sao_Paulo`.

## File handling
- Create a new file in `_posts/` using the required filename format.
- Write the corrected text into the file.
- Do not add front matter unless the user explicitly asks for it.

## Git workflow
- Stage only the new/modified post file.
- Commit with message: `post: <slug>` unless the user provides a different message.
- Push to the current branch unless the user specifies another branch.

## Clarify when needed
Ask the user only if a decision is ambiguous and cannot be inferred (e.g., conflicting title/date, or they want grammar edits beyond spelling).
