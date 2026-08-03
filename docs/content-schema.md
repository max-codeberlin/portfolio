# Content schema — projects

The single source of truth is [`src/content.config.ts`](../src/content.config.ts).
The Decap CMS form in [`public/admin/config.yml`](../public/admin/config.yml) must
mirror it. This document explains what each field is _for_, which the code cannot.

Entries live in `src/content/projects/<slug>.md`. The filename becomes the URL
slug (`src/content/projects/lesekiste.md` → `/work/lesekiste`).

## Fields

| Field      | Type                         | Required    | Purpose                                                                                                       |
| ---------- | ---------------------------- | ----------- | ------------------------------------------------------------------------------------------------------------- |
| `title`    | string                       | yes         | Project name. Used as the page `<h1>` and in listings.                                                        |
| `status`   | `in-progress` \| `completed` | yes         | Drives the status pill. Lets the site show current work, not just a finished archive.                         |
| `hmw`      | string                       | yes         | The "How might we …" framing, **without** the prefix — the UI adds it. Keep it as a question.                 |
| `summary`  | string (≤ 280 chars)         | yes         | Card copy and meta description. One or two sentences.                                                         |
| `year`     | number                       | yes         | Year the work was done, or started for ongoing work.                                                          |
| `tags`     | string[]                     | no          | Discipline and domain tags, e.g. `["UX Research", "EdTech"]`. Defaults to `[]`.                               |
| `cover`    | image path                   | no          | Cover image, path relative to the markdown file. Astro optimises it at build time.                            |
| `coverAlt` | string                       | conditional | **Required whenever `cover` is set** — the build fails otherwise. Describe the image, don't repeat the title. |

The markdown body below the frontmatter is the case study itself.

**That's the whole schema, on purpose.** Every field costs three files to keep
in sync, so the set stays at what a case study genuinely can't be written
without. Things like `role`, `duration`, `featured` or an external `link` are
easy to add — see below — but they should be added when a design calls for
them, not in advance.

## Conventions

- **`hmw` is the hook.** Every project opens with the same framing, so the
  portfolio reads as one point of view rather than a list of deliverables.
- **`status: in-progress` is a feature, not an apology.** Showing current work is
  an explicit goal of the site — half-finished projects belong here.
- **Tags are a controlled vocabulary.** Reuse existing tags before inventing new
  ones; the CMS offers the existing set as suggestions.
- **Images live next to the entry.** Cover images go in
  `src/content/projects/_media/` and are referenced as `./_media/cover.jpg`, so
  Astro can optimise and hash them.

## Example

```markdown
---
title: Lesekiste
status: in-progress
hmw: make reading practice feel like play for six-year-olds?
summary: A tangible reading companion that turns daily practice into a shared
  ritual between child and parent.
year: 2026
tags:
  - UX Research
  - EdTech
cover: ./_media/lesekiste-cover.jpg
coverAlt: A wooden box with illustrated cards spread out on a kitchen table
---

## The starting point

…
```

## Adding a field

1. Add it to the zod schema in `src/content.config.ts` (with a doc comment).
2. Add the matching widget to `public/admin/config.yml`.
3. Add a row to the table above.
4. Render it wherever it belongs.

Skipping step 2 is the usual cause of "the CMS saved it but the build fails".
