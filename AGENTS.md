# Repository Guide

## Structure

- This is a collection for the `skills` CLI, not an application package. Each distributable skill lives at `skills/<skill-name>/SKILL.md`.
- A skill's YAML front matter supplies its `name` and trigger `description`; keep `name` aligned with its directory name and make the description precise enough to drive activation.
- The skill body is the operative workflow. Keep examples and commands applicable to downstream repositories, not this collection.

## Verification

- No repository-local build, lint, test, formatter, or CI configuration exists. Review `SKILL.md` front matter and Markdown examples directly after edits.
- Test collection installation from the repository root with `npx skills add .`; install only the Rails skill with `npx skills add . --skill rails-i18n`.
