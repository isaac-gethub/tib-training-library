# TIB Training Library

Live reading platform for the SAP Controls Mastery career-prep documents
(resume, deep-dive narratives, how-to execution guides, study roadmap,
interview prep, readiness tracker summary, and the offshore SOX proposal).

## How this works

This is a single static file (`index.html`) with no build step. On load it:

1. Fetches the document list (nav) from the Supabase table
   `training_library_docs` (project `blfgwysgekfqhcafofhe`, project name
   `kabo-training-lms`), ordered by `sort_order`.
2. Fetches each document's `body_html` from the same table on demand, when
   a user clicks it in the sidebar.

There is no backend/server code in this repo — it is a plain HTML/CSS/JS
file that talks directly to Supabase's REST API (PostgREST) from the
browser, using the project's public **anon** key (safe to expose client-side;
it only grants read access, enforced by a Row Level Security policy on the
`training_library_docs` table that allows `SELECT` and nothing else).

## Editing document content

Do NOT edit document text in this repo. Content lives in Supabase, not in
this file. To change a document:

1. Open the Supabase SQL editor for project `blfgwysgekfqhcafofhe`.
2. `UPDATE public.training_library_docs SET body_html = $$...$$ WHERE doc_id = '...';`
3. Refresh the live site — no redeploy needed, content updates immediately.

To add a new document, `INSERT` a new row (see table columns: `doc_id`,
`category`, `title`, `subtitle`, `sort_order`, `body_html`) — the nav and
page both render dynamically from whatever rows exist.

Only redeploy this repo if you're changing the page's structure, styling,
or the Supabase connection details themselves (`SUPABASE_URL` /
`SUPABASE_ANON_KEY`, both near the top of the `<script>` block in
`index.html`).

## Deploy — GitHub + Vercel (standard TIB workflow)

```bash
# 1. From this folder, initialize and push to a new GitHub repo
git init
git add .
git commit -m "TIB Training Library — Supabase-driven single-page site"
git branch -M main
git remote add origin https://github.com/isaac-gethub/tib-training-library.git
git push -u origin main
```

```
2. In the Vercel dashboard (team: isaacbabs-2368s-projects):
   - New Project → Import the GitHub repo above
   - Framework preset: Other (static — no build command needed)
   - Root directory: /
   - Deploy
```

```
3. Custom domain (GoDaddy DNS, following the existing TIB pattern):
   - Vercel → Project → Settings → Domains → add the desired subdomain
   - GoDaddy → DNS → add a CNAME record pointing that subdomain to
     the target Vercel gives you (cname.vercel-dns.com or a
     project-specific target)
   - Do NOT route through Cloudflare — prohibited across the TIB stack
```

## Deployment protection

If the Vercel project has Deployment Protection enabled (Vercel login
required to view), turn it off for public trainee access:
Vercel Dashboard → Project → Settings → Deployment Protection → Off.

## Notes

- No environment variables are required — the Supabase URL and anon key
  are intentionally hardcoded in `index.html` since they are safe to expose
  client-side under the current RLS policy.
- No `package.json` / build tooling — this is deliberately a zero-dependency
  static file.
