# 28. Checklist for a new project

**Basics**
- [ ] `index.js` with a `release` switch, locally `node index.js` (debug, port 8000)
- [ ] `config` with `url : https://example.com`
- [ ] `.gitignore` (`index.js.json`, `index.js.map`, `index.pid`, `tmp/`)
- [ ] logic in `actions/`, the controller only calls actions and renders
- [ ] view helpers in a local `Func` + `view.repository.func = Func`; shared helpers on `FUNC`
- [ ] integrations and shared libraries in `modules/` (`MODS.<name>`), never `require()`
- [ ] every file starts with a comment (what it does, what it needs, how to call it); constants and non-obvious decisions say why
- [ ] every action, module and plugin can be copied to another project: project values in `config`, dependencies listed at the top

**Layout and views**
- [ ] `@{section('head')}` in the layout, `@{section head}…@{end}` in views
- [ ] CSS tokens + dark mode, mobile menu, skip link, `lang`
- [ ] HTML 404 via `ROUTE('#404')`
- [ ] a comment with the model shape at the top of every view

**SEO**
- [ ] title, description, canonical, robots, OG, Twitter, JSON-LD on every page
- [ ] one `<h1>`, no skipped heading levels
- [ ] `sitemap.xml`, `robots.txt`, `llms.txt`
- [ ] 301 for non-canonical URLs, 404 for invalid parameters, `noindex` for search
- [ ] favicons (SVG + ICO + apple-touch + manifest)

**Data and operations**
- [ ] downloads via `REQUEST`/`DOWNLOAD`, validation before swapping
- [ ] periodic tasks via `ON('ready + service')`; recalculate time-dependent values even when the source is down
- [ ] after deployment: Google Search Console + Bing Webmaster Tools, submit the sitemap

**API backend**
- [ ] `config`: `database`, `salt`, typed keys; no secrets in code
- [ ] `definitions/db.js` (`querybuilderpg`), `definitions/auth.js` (one `AUTH()`), `definitions/func.js`, one `CORS()`
- [ ] every feature is `plugins/<feature>/index.js` (routes) + `schemas/<feature>.js` (`NEWSCHEMA`)
- [ ] all JSON work goes through `+API /api/`; plain routes only for files, health, webhooks, sockets
- [ ] every action declares `input` / `query` / `params` and ends with `$.success()` / `$.callback()` / `$.invalid()`
- [ ] permissions via `permissions:` / `FUNC.require_perm`; ownership in the query
- [ ] tables `tbl_*` with `id`, `dt*`, `is*`; lists via `DATA.list().autoquery()`
- [ ] uploads: `@upload <size>`, an extension whitelist, `FILESTORAGE` or `MODS.storage`, signed download ids
- [ ] jobs via `CRON()` / `ON('service')`; realtime via `ROUTE('SOCKET …')` + `FUNC.realtime_send`
- [ ] no internal `require()`, no `fs` / `path` / `https` imports, no `setInterval`, no repositories

**New feature (plugin) in an existing backend**
- [ ] SQL: table (+ view for lists) following the naming and field order
- [ ] `plugins/<feature>/schemas/<feature>.js` with `list`, `read`, `create`, `update`, `remove`
- [ ] `plugins/<feature>/index.js` with the routes and `exports.permissions`
- [ ] shared helpers to `FUNC`, integrations to `MODS`, side effects via `EMIT`
