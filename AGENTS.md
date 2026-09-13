# AGENTS.md

## Scope
These instructions apply only to this repo.

## Blog basics
- The site uses Hugo with the Hextra theme.
- Blog posts live in `content/blog/`.
- Use YAML front matter with at least `title` and `date`.
- Keep post URLs at the site root; this is configured by the `blog` permalink in `hugo.yaml`.
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
- Create a new file in `content/blog/` named `<slug>.md`.
- Add YAML front matter containing the derived `title` and `date`.
- Write the corrected text into the file.

## Git workflow
- Stage only the new/modified post file.
- Commit with message: `post: <slug>` unless the user provides a different message.
- Push to the current branch unless the user specifies another branch.

## Verification
- Run `hugo --gc --minify` after changing site configuration, layouts, or content.
- A change is complete only when the command exits successfully.

## Clarify when needed
Ask the user only if a decision is ambiguous and cannot be inferred (e.g., conflicting title/date, or they want grammar edits beyond spelling).
