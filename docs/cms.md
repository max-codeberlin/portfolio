# Decap CMS

The CMS lives at `/admin` and is configured in
[`public/admin/config.yml`](../public/admin/config.yml). It edits the markdown in
`src/content/projects/` and commits straight back to this repo — there is no
database and no separate content API.

`publish_mode: editorial_workflow` is on, so saving a project opens a **draft pull
request** rather than committing to `main`. Nothing goes live until that PR is
merged, at which point the deploy workflow rebuilds the site.

## Editing locally (no setup needed)

```bash
npx decap-server      # terminal 1 — proxy that writes to your local files
npm run dev           # terminal 2
```

Then open <http://localhost:4321/admin/>. `local_backend: true` makes the CMS talk
to the proxy instead of GitHub, so there is no login step and no OAuth. Changes
land in your working tree as ordinary file edits for you to commit.

This is the fastest path and the one to use while the schema is still moving.

## Editing in the browser on the live site

The GitHub backend needs an OAuth handshake, and **GitHub Pages cannot provide
it** — Pages serves static files and the handshake needs a server that holds a
client secret. So one small piece has to live somewhere else. Two options:

### Option A — a hosted OAuth relay (recommended)

Deploy one of the ready-made Decap OAuth clients (Netlify Function, Vercel
serverless function, Cloudflare Worker — all are a few lines and free at this
scale), then:

1. GitHub → **Settings → Developer settings → OAuth Apps → New OAuth App**
   - Homepage URL: your site's URL
   - Authorization callback URL: `https://<your-relay>/callback`
2. Put the client ID and secret in the relay's environment. **Never commit the
   secret** — it is not a build-time value and does not belong in this repo.
3. Uncomment `base_url` and `auth_endpoint` in `config.yml` and point `base_url`
   at the relay.

### Option B — move the site to a host with built-in auth

Netlify and Cloudflare Pages both provide the OAuth endpoint themselves, which
removes the relay entirely. This is a bigger change (it drops GitHub Pages and
[`docs/deployment.md`](./deployment.md) stops applying), but it is worth
considering if browser editing turns out to matter more than staying on Pages.

Until one of these is done, `/admin` loads on the live site but cannot log in.
Local editing works either way.

## Keeping the CMS and the schema in step

`config.yml` and `src/content.config.ts` describe the same fields, and nothing
enforces that automatically. When they drift, the CMS saves happily and the build
fails afterwards — the error surfaces in Actions, not in the CMS.

Field reference and the change checklist: [`docs/content-schema.md`](./content-schema.md).

## Media

Cover images upload to `src/content/projects/_media/` and are referenced as
`./_media/<name>`, which keeps them inside `src/` where Astro can optimise and
hash them. Images in `public/` are served as-is with no optimisation, which is why
they don't go there.
