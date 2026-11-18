---
name: totaljs-website
description: How to build and maintain a server-rendered website on Total.js 5 (total5) with the structure actions/ + controllers/ + modules/ + views/ + public/ - content catalogs with search and pagination, detail pages, static pages, server-rendered SEO, data kept in memory and synchronized from a remote source, no database and no frontend framework. Use this skill whenever you add, change, review or debug anything in such a project - a page, route, action, view, the layout CSS, SEO tags, the sitemap, a data refresh, an embedded preview or the mobile menu - even when the request doesn't mention Total.js by name. Read it before touching files in actions/, controllers/, modules/, views/ or public/ of a project that calls require('total5').
---

# Total.js 5 website

A guide for websites built on **Total.js 5** that are rendered on the server, keep their data in memory (optionally synchronized from a remote source such as a Git repository or an API) and use plain CSS and JavaScript on the client.

General Total.js rules live in the `totaljs-cookbook` skill and, when the project has one, in `COOKBOOK.md` in the repository root. This skill covers how a website of this shape is organized and what tends to go wrong. When a rule here disagrees with the existing code of the project, follow the project and mention the difference.

## First steps in an unfamiliar project

1. Read `index.js` and `config` (start options, the canonical `url`).
2. List `actions/`, `controllers/`, `modules/`, `definitions/` and `views/`; read the routes in every `exports.install`.
3. Open `views/layout.html`: it shows the design tokens, the global script and how the head section is printed.
4. Find the smallest complete page (controller + view) and use it as the template for new work.

## Run and verify

```bash
node index.js
```

- Start the app exactly like this: debug mode, port **8000**. Don't create other start scripts and don't pass `release` when developing.
- In debug mode the watcher reloads changed controllers, actions, views and modules on its own; a changed `config` restarts the process. You rarely need to restart.
- If the port is taken by another project, don't kill it. A numeric argument sets the port: `node index.js 8001`. Stop a temporary instance afterwards.
- Never start Total.js with `node -e "..."`; it crashes on `process.argv[1]`.
- If `config` sets a canonical `url`, canonical links, the sitemap and Open Graph point to production **even locally**. When verifying, compare paths, not hosts.

Verify a change by requesting the affected pages (the list, a paged list, a search, a detail, static pages, `/sitemap.xml`, an unknown URL for 404) and by looking at them in a browser at desktop and mobile width, in the light and the dark theme. A page must never scroll horizontally.

## Structure

| Path | Responsibility |
|---|---|
| `index.js` | start script: `require('total5'); Total.run(options);` |
| `config` | `key : value` settings, read through `CONF.<key>` |
| `actions/<entity>.js` | business logic: `Entity\|refresh`, `Entity\|browse`, `Entity\|read`; periodic refresh |
| `controllers/<area>.js` | routes, responses and the view helpers (`Func`) of one area |
| `modules/<name>.js` | code shared by several controllers, used as `MODS.<name>` |
| `definitions/` | initialization that must run before controllers |
| `views/layout.html` | the layout, **all CSS** and the global script |
| `views/<page>.html` | one view per page type, plus `404.html` |
| `databases/` | persisted data (`MEMORIZE`, cached downloads) |
| `public/` | static files: favicons, `llms.txt`, generated images |

Load order: `modules → actions → definitions → controllers`. A module is therefore available at the top of a controller, and actions exist before a controller calls them.

## Data flow

```
remote source (Git repository, API, files)
   -> actions (Entity|refresh, periodically via ON('ready + service'))
   -> REPO.db.<entity> (MEMORIZE('db'))
   -> controllers call ACTION('...') and render views
```

- `MEMORIZE('db')` returns an object persisted to `databases/memorize_db.json`; `.save()` writes with a 10-second delay. `REPO` is global and readable in views.
- A refresh downloads **aside** (for example into `tmp/`), validates the format and only then replaces the data. A broken or old-format file must never replace working data, because every page renders from it.
- Data that isn't persisted is missing after a restart until the first download finishes. Decide deliberately: persist it, or answer 503 for that page.
- Compute values that depend on time or on new data in the action, not while rendering. Recalculate time-dependent values on every tick even when the remote source is offline, otherwise stale values stay forever.
- `ON('service')` fires every minute with an increasing counter; `counter % 60 === 0` means hourly.
- On the server use `REQUEST`, `DOWNLOAD` or `RESTBuilder`, not `fetch` - it is missing in older Node.js versions that production may run.

## Conventions

**Server**

- Business logic belongs in actions; controllers call `ACTION(name, data).callback(...)` and render. A list action returns only the fields the list needs.
- Each controller has its own `const Func = {}` with helpers for its views and passes it with `$.view(name, model).repository.func = Func;`. Views call `@{repository.func.name()}`. Keep the global `FUNC` for application-wide helpers, not for presentation.
- Use `Total` (not the `F` shorthand) and `MODS.<name>` for modules.
- Prefer what the framework already has (`slug`, `toSearch`, `markdown`, `parseJSON`, `quicksort`, `Utils.paginate`, `Tangular`, `Total.Fs`, `Total.Zlib`) before adding an npm dependency.
- A missing page is rendered by a `notfound($)` function: `$.response.status = 404; $.view('404')`, also registered as `ROUTE('#404', notfound)`. `$.invalid(404)` returns JSON and is not suitable for pages.
- Generated files use file routes: `ROUTE('FILE /sitemap.xml', sitemap)`.

**Views**

- `@{value}` escapes HTML, `@{!value}` prints raw HTML. A `Func` method that returns HTML must escape its inputs itself with `.encode()`.
- Every view starts with a comment describing the model and fills its own head: `@{section head}@{!repository.func.head(...)}@{end}`. In the layout the section is printed with `@{section('head')}` (not `@{!section(...)}`).
- Available in views: `model`, `query`, `controller`, `repository`, `REPO`, `CONF`.

**CSS and JavaScript**

- All CSS is inline in `views/layout.html`, grouped under `/* Section */` comments. Add styles for a new page as a new commented group; don't create CSS files.
- Colors, radii and fonts are design tokens on `:root`. Both themes are defined through `prefers-color-scheme`; every new color needs a dark value.
- A bright accent color rarely has enough contrast for small text on a light background. Keep a darker token for text and links (contrast at least 4.5 : 1) and use the bright one for large elements, icons and borders.
- Plain JavaScript only. Global behavior (search, mobile menu, copy buttons) is in the script at the end of the layout; page-specific behavior is a `<script>` at the end of that view.
- Global handlers are usually delegated through `data-*` attributes. Before you add a `data-*` attribute to a page, check that the layout script doesn't already listen for the same name - otherwise one click triggers two behaviors.
- CSS resets (also the one bundled in the Total.js UI library) set `input[type=search]` to `box-sizing: content-box`; set `box-sizing: border-box` on search inputs.
- An element with the `hidden` attribute that also has a `display` rule needs `[hidden] { display: none }` in its scope.
- Enhance progressively: a list search is a normal GET form that also works without JavaScript; a script may then swap the server-rendered result block in place.

## Adding a page

1. A controller with the route, its own `Func` (`hostname`, `head`, ...) and a model containing at least `title`, `description` and `url`.
2. A view with the model comment, the head section, a breadcrumb, exactly one `<h1>` and `<h2>` for sections.
3. Styles as a new group in the layout.
4. Links in the main navigation (check its mobile form) and in the footer.
5. An entry in the sitemap.
6. Verify as described above.

## SEO rules

For a content website SEO is a main goal; treat regressions as bugs.

- Every indexable page has a unique `<title>` (about 60 characters), a description (up to 160), `robots`, a canonical URL, Open Graph, Twitter tags and JSON-LD. Build them in one `Func.head()` per controller and pass every value through `.encode()`.
- Build absolute URLs from `CONF.url` when it is set and fall back to `$.hostname()`. Behind a proxy the request host and protocol are unreliable and produce duplicate canonical URLs.
- JSON-LD by page: `WebSite` and `CollectionPage` for lists, a type matching the content plus `BreadcrumbList` for a detail, `TechArticle` or `CollectionPage` plus `BreadcrumbList` for other pages. Escape `<` in the JSON (`<`).
- Content that matters is rendered on the server (lists, documentation via `.markdown()`). Don't move it to client-side rendering.
- Internal search results are `noindex,follow`. Invalid parameters (`?page=99`, `?page=abc`) and unknown items are real 404s. A non-canonical id redirects with 301.
- Helper URLs (previews, downloads, raw files) send `X-Robots-Tag: noindex` and are linked with `rel="nofollow"`.
- Images have `width`, `height` and `alt`; only the first one loads eagerly.
- One `<h1>` per page and no skipped heading levels.
- `sitemap.xml` and `robots.txt` are generated by code; `llms.txt` describes the site for AI crawlers.

## Embedding runnable or user-edited HTML

When a page previews an HTML example or lets the visitor edit and run code:

- Render it in an `<iframe>`; for edited code use `srcdoc` and create a **new** iframe for every run, so scripts and timers of the previous version die with the old one.
- Run edited code in a sandbox **without `allow-same-origin`**. With it the code could reach the page, cookies and storage of the site.
- A sandboxed frame has no origin, so `localStorage`, `sessionStorage` and `document.cookie` throw. Libraries that touch them at startup need a small in-memory shim injected before their code.
- Edited code lives only in the visitor's browser. Never take code from the URL, a form post or storage shared between users - that would let one person run a script in another person's session.
- Load heavy sources on demand (after a click), not with the page.
- After changing this part, test several different items in the sandboxed mode and compare the reported errors with the unsandboxed original.

## Gotchas

| Situation | What to do |
|---|---|
| A view change doesn't show up | The app runs in release mode; run `node index.js` (debug) |
| `ReferenceError: section is not defined` | Use `@{section('head')}` in the layout |
| A page returns empty JSON instead of HTML | `$.invalid()` was used for a page; use `notfound($)` |
| Something happens twice after a click | A `data-*` attribute collides with a global handler in the layout |
| A search field overflows its container | Missing `box-sizing: border-box` |
| `MEMORIZE` data isn't on disk yet | `.save()` is delayed by 10 seconds |
| `ON('ready')` in a controller doesn't fire | Initialize with `setImmediate(load)` inside `exports.install` |
| The Markdown output has `hidden` classes or a duplicated `loading` attribute | The renderer adds them itself; strip or don't add them again |
| A refresh from a new branch or source breaks the site | Validate the downloaded data before replacing the current data |
| Canonical points to production while testing locally | Intended when `config` sets `url` |

## Before you finish

- Pages return the expected status codes and the server log shows no new errors.
- Checked in a browser: desktop and mobile width, light and dark theme, no horizontal scrolling.
- The page has one `<h1>`, a title, a description and a canonical URL; a new page is in the sitemap and in the navigation.
- No CSS file, no presentation helper in the global `FUNC`, no `fetch` on the server, no dependency the framework already covers.
