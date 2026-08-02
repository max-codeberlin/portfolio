# portfolio

Website to display my work, myself and current projects.

---

## What this project is

The personal portfolio of **Max Pinkert** — a designer's site, built to do three
things: show the work, show the person behind it, and show what is currently
being worked on. That third one matters: work in progress is meant to be visible
here, not hidden until it's polished.

The focus is **UX and product design**, moving towards **children's and education
technology**. That direction should be legible from the site without being
announced — through what the case studies are about and how they're framed.

**The tone to aim for: professional, but not plain.** Restrained is not the same
as boring. The site should feel considered and deliberate, with one or two
confident gestures rather than a page competing with itself for attention.

### How work flows through this repo

```
Figma (design)  ──MCP──▶  this repo (implementation)  ──Actions──▶  GitHub Pages
                              ▲
                   Decap CMS ─┘  (case studies, written in the browser or locally)
```

All visual design is done by Max in Figma and handed to an agent as a Figma URL
for implementation. **Agents implement designs; they do not invent them.** When
there is no design yet, build restrained and token-driven so the real design
drops in cleanly later. Case-study content comes from Max through the CMS — never
write case studies, client names, metrics, or bio copy on his behalf.

## Stack

| Concern    | Choice | Why |
| ---------- | ------ | --- |
| Framework  | [Astro](https://astro.build) 7, static output | Ships HTML, no client JS by default. A portfolio doesn't need a runtime. |
| Styling    | Plain CSS + custom properties | Tokens map 1:1 to Figma Variables. No build-tool indirection between design and code. |
| Content    | Astro content collections | Markdown + a zod schema that fails the build on bad data. |
| CMS        | [Decap CMS](https://decapcms.org) | Writes markdown back into the repo. No database, no vendor. |
| Fonts      | Inter Variable + Instrument Serif, self-hosted | No third-party requests. |
| Hosting    | GitHub Pages via GitHub Actions | Free, static, already where the code lives. |
| Design I/O | Figma MCP server | Lets an agent read the actual design instead of guessing from a screenshot. |

## Quick start

```bash
npm install
npm run dev        # http://localhost:4321
```

| Command | What it does |
| ------- | ------------ |
| `npm run dev` | Dev server with hot reload |
| `npm run check` | `astro check` — **must be 0 errors, 0 warnings before you commit** |
| `npm run build` | Static build into `dist/` |
| `npm run preview` | Serve the built output |
| `npx decap-server` | Local CMS backend so `/admin` works without OAuth |

Both `check` and `build` run in CI on every pull request.

## Repository layout

```
.github/workflows/
  deploy.yml            Build + deploy to GitHub Pages on push to main
  ci.yml                Type-check + build on every PR
design/
  tokens.json           DTCG token export — the Figma ⇄ code exchange format
docs/
  content-schema.md     Every project field, what it's for, how to add one
  deployment.md         Pages setup, custom domain, DNS, rollback
  design-to-code.md     Figma MCP workflow and the token sync rules
  cms.md                Decap CMS: local editing, and the OAuth caveat
public/
  admin/                Decap CMS shell + config.yml
  og/default.png        Social share image
src/
  config/site.ts        Name, slogan, links — site-wide constants
  content.config.ts     Project schema (zod). The enforcer.
  content/projects/     Case studies as markdown; images in _media/
  layouts/BaseLayout.astro   <head>, meta, OG tags, skip link
  pages/                File-based routes
  styles/
    tokens.css          Design tokens — primitives, then semantics. Start here.
    global.css          Reset, type defaults, a11y helpers, .container
CLAUDE.md               Working rules for agents. Read before changing anything.
.mcp.json               Figma MCP server declaration
```

## The design system in one screen

Tokens live in [`src/styles/tokens.css`](src/styles/tokens.css) in two tiers:

- **Primitives** — raw values, named after the thing: `--color-yuzu-400`,
  `--space-md`. Never used directly in a component.
- **Semantics** — what components actually consume: `--text-secondary`,
  `--surface-raised`, `--status-in-progress`. These are what get remapped for
  dark mode.

The palette is nature-inspired and named in German: **Yuzu** (citrus yellow, the
signature accent), **Rote Beete** (beetroot, the deep counterweight), **Tomaten**
(tomato, used sparingly as a signal), **Lieblingsort** (a grounded green), plus
warm-tinted neutrals so nothing sits coldly against them.

**The one rule that matters:** a component that reaches past the semantic layer
to a primitive breaks dark mode silently — no error, no failed build, just a
screen that goes wrong for half your visitors. Use semantics.

The hex values currently in the repo are the working palette and are the values
to reconcile against Figma Variables once that collection is published. **The
token *names* are the stable contract, not the hexes.**

## Adding a case study

Either through the CMS (`/admin`, see [docs/cms.md](docs/cms.md)) or by hand:
create `src/content/projects/<slug>.md` with the frontmatter documented in
[docs/content-schema.md](docs/content-schema.md).

Every project opens with a **"How might we …"** framing (the `hmw` field) so the
portfolio reads as one point of view rather than a pile of deliverables. And
`status: in-progress` is a feature — showing current work is the point.

A field lives in three places and all three must agree, or the CMS will save
frontmatter that the build then rejects:

1. `src/content.config.ts` — the zod schema
2. `public/admin/config.yml` — the CMS form
3. `docs/content-schema.md` — the explanation

## Current state

Infrastructure is in place; the site is a v0.1 placeholder with no case studies
yet.

**Done**
- Astro project, static output, type-checking clean
- Design tokens (CSS custom properties + DTCG JSON), light and dark
- Project content schema + documentation
- Decap CMS wired to the schema, editorial workflow on
- v0.1 homepage — name, role, slogan, one brand gesture
- GitHub Actions: deploy on `main`, CI on PRs
- Figma MCP connection verified, handoff workflow documented

**Needs a human (can't be done from code)**
- **Settings → Pages → Source: GitHub Actions** — the first deploy fails without
  it ([docs/deployment.md](docs/deployment.md))
- Custom domain: DNS records, then `public/CNAME` and `site` in `astro.config.mjs`
- An OAuth relay if you want to edit at `/admin` on the live site
  ([docs/cms.md](docs/cms.md)) — local editing already works
- Publishing the Figma Variable collections so the token hexes can be reconciled

**Next**
- Real case studies
- `/work` index and project detail pages, built from the Figma designs
- An about page

## For agents

Read [CLAUDE.md](CLAUDE.md) before making changes. The short version:

1. **Never hardcode a value that has a token.** Semantics, not primitives.
2. **Accessibility is part of "done"** — semantic HTML, visible focus states, alt
   text, reduced-motion, and both colour schemes working.
3. **Don't invent content.** Placeholder copy must read as placeholder.
4. **Static and fast.** Adding client-side JS needs a reason and an ask.
5. **Run `npm run check` before committing.** Zero warnings.

For Figma work, read [docs/design-to-code.md](docs/design-to-code.md) first:
`get_metadata` to orient → `get_screenshot` to actually look at it →
`get_variable_defs` for tokens → `get_design_context` last, one frame at a time.
Never skip the screenshot; the XML gives you structure, the image gives you
intent.
