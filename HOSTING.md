# Make Archify Live Guide shareable (GitHub)

Local Live Guide (`http://127.0.0.1:8765`) works only on **your** machine.  
To share a URL with others, publish static files with **GitHub Pages**.

---

## Option A — GitHub Pages (recommended, free)

### 1) This repo already has a Pages workflow

File: `.github/workflows/pages.yml`

It publishes:
- kit docs + Live Guide template
- every project under `projects/<name>/` (diagrams + live-guide HTML)

### 2) Turn on Pages in GitHub

1. Open https://github.com/Sulemanhassan899/Archify-kit  
2. **Settings → Pages**  
3. Source: **GitHub Actions**  
4. Push to `main` (or re-run the workflow)

### 3) Your public URLs

After the workflow is green:

```text
https://sulemanhassan899.github.io/Archify-kit/
https://sulemanhassan899.github.io/Archify-kit/projects/obecno/live-guide/
```

Architecture HTML files are also under:

```text
https://sulemanhassan899.github.io/Archify-kit/projects/obecno/<architecture-folder>/...
```

### 4) What others can see

| Works on Pages | Needs your local server |
|----------------|-------------------------|
| Architecture HTML / diagrams | Live SSE “watching disk” |
| Static Live Guide UI | Excel API / live QA polling against local files |
| Manifest + iframes | Writing new QA results |

For a **demo share link**, Pages is enough (architecture tab).  
For **live QA while testing**, keep using localhost on your Mac.

---

## Option B — Publish only one project

```bash
# From Archify-kit
git add projects/my-app
git commit -m "Add my-app archify"
git push
```

Pages redeploys automatically. Share:

```text
https://sulemanhassan899.github.io/Archify-kit/projects/my-app/live-guide/
```

Update that project’s `manifest.json` so `architecture_entry` is a **relative** path that works on Pages (same as local).

---

## Option C — Custom domain (optional)

In repo **Settings → Pages → Custom domain**, add e.g. `archify.yourdomain.com`,  
then create a DNS CNAME to `sulemanhassan899.github.io`.

---

## Option D — Always-on server (advanced)

If you need real live QA APIs for remote teammates:

1. Run `node live-guide/server.mjs` on a VPS (DigitalOcean, Fly.io, Railway, etc.)
2. Set `ARCHIFY_ROOT` + `PORT`
3. Put HTTPS in front (Caddy / nginx)
4. Share `https://your-server/live-guide/`

GitHub Pages cannot run that Node watch server; use a VPS for true live mode.

---

## Security

- Do **not** commit `credentials.local.yaml` or secrets into Pages.
- This kit gitignores credentials and `registry.local.yaml`.
- Review `projects/*/qa/results` before pushing if they contain sensitive notes.
