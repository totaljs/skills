---
name: totaljs-cookbook
description: Conventions and recipes for building Total.js 5 (total5) applications in Node.js - websites with views and SEO, and API backends with plugins, NEWACTION / NEWSCHEMA actions, ROUTE(), AUTH(), PostgreSQL through DATA / QueryBuilder, uploads, WebSockets and CRON jobs. Use it whenever you create, edit, review or debug code in a Total.js project (files in controllers/, definitions/, modules/, plugins/, actions/, schemas/, views/, a `config` file, or code that calls require('total5')), and before answering questions about Total.js.
---

# Total.js 5 cookbook

Total.js is a global/singleton framework. It is not Express, Nest or Fastify. Most mistakes come from applying patterns of those frameworks, so read the matching reference file **before** writing code. Do not rely on memory of Total.js 3/4 or of other frameworks.

## How to use this skill

1. Read the rules below. They apply to every file.
2. Open the reference files for the task (table at the end). Each is self-contained and short.
3. Before finishing, check the work against [references/26-anti-patterns.md](references/26-anti-patterns.md) and [references/27-gotchas.md](references/27-gotchas.md).

The whole cookbook in one file is `COOKBOOK.md` in the repository root (generated from `references/`).

## Rules that always apply

**Architecture**

- Globals, not imports: `CONF`, `FUNC`, `MAIN`, `MODS`, `DATA`, `PATH`, `REPO`, `CACHE`, `NOW` already exist. Never import them.
- Files are auto-loaded by directory (`definitions/`, `modules/`, `controllers/`, `actions/`, `schemas/`, `plugins/<id>/`). Nothing is registered and nothing internal is `require()`d.
- `require()` is allowed only for `total5` in `index.js`, `querybuilderpg` in `definitions/db.js`, and a real external SDK inside a module. Never `require('fs')`, `path`, `https`, `crypto` or another project file. Use `PATH.*`, `PATH.fs`, `Total.Crypto`, `HASH()`, `RESTBuilder`, `MODS.<name>`, `FUNC.<name>`.
- Business logic lives in actions (`NEWSCHEMA` / `NEWACTION`). Controllers only handle pages, files, uploads, webhooks and sockets.
- A backend feature is a plugin: `plugins/<feature>/index.js` (routes) + `plugins/<feature>/schemas/<feature>.js` (actions).
- No `src/`, `lib/`, `services/`, `repositories/`, `utils/`, `middleware/`. No service classes, repositories, DI or Express middleware.

**Actions and routes**

- An action's `$` is not the HTTP controller. Finish with `$.success(value)`, `$.callback(payload)` or `$.invalid(error)`. Never `$.json()` / `$.view()` in an action, never a hand-built `{ success: true }`.
- JSON APIs go through one gateway: `ROUTE('+API /api/  +orders_create  --> Orders/create')`. The sign before the method is auth (`+` authorized, `-` unauthorized). The sign before the schema string is the payload (`+` with data, `-` without data, `%` partial).
- Flags, size and timeout are written inside the route string: `ROUTE('+POST /upload/ @upload <10MB', upload)`. The Total.js 4 form with an array of flags does not work.
- One `AUTH()` delegate in `definitions/auth.js`. Tokens are `ENCRYPTREQ` / `DECRYPTREQ`, not JWT. Identity comes from `$.user`, never from the payload.
- One `CORS()` call for the app.

**Data**

- SQL goes through `DATA` (`DATA.find/list/read/insert/modify/remove/query`), with `.promise($)` in actions. No ORM, no `require('pg')`, no string concatenation in SQL.
- Builder first, raw SQL last: `DATA.read('tbl_user').id(id)`, `DATA.find()`, `DATA.list()` are preferred; `DATA.query()` only where the builder cannot express the query.
- Filter in the query (`.where()`, `.in()`), never load a table to filter it in JavaScript.
- Tables `tbl_*`, views `view_*`, lowercase names, `id` from `UID()`, dates `dt*`, booleans `is*`.
- Runtime state on `MAIN`, expiring values in `CACHE`, nothing durable in `TEMP`.

**Background work and realtime**

- `CRON()` or `ON('service')`, never `setInterval` or `node-cron`.
- `ROUTE('SOCKET /path/', handler)`, never `socket.io`.

**Style**

- Tabs, semicolons, single quotes, `var`-first (no `const` inside functions), classic `for` loops.
- User-facing errors use localization markup: `$.invalid('@(Invalid credentials)')`.
- Code is written for a person: plain names, short functions and a comment wherever the reason is not obvious from the code.
- Code is written to be copied: an action, a module or a plugin works in the next project after copying its file or folder; what differs between projects is in `config`.

## Reference files

| Task | Read |
|---|---|
| Starting a project, deciding where code goes | [01-architecture](references/01-architecture.md), [02-project-structure](references/02-project-structure.md), [28-checklist](references/28-checklist.md) |
| `index.js`, debug/release, ports, what restarts after a change | [03-running](references/03-running.md) |
| `config`, `CONF`, environment variables | [04-configuration](references/04-configuration.md) |
| Formatting and naming | [05-coding-style](references/05-coding-style.md) |
| Which global does what (`UID`, `NOW`, `CACHE`, `BLOCKED`, `MAIL`…) | [06-globals](references/06-globals.md) |
| Modules, definitions, `FUNC`, `MAIN`, `MODS` | [07-sharing-code](references/07-sharing-code.md) |
| `NEWACTION`, `NEWSCHEMA`, input schemas, `ACTION()` | [08-actions](references/08-actions.md) |
| `ROUTE()`, controllers, responses, API routing, CORS | [09-controllers-routing](references/09-controllers-routing.md) |
| Creating a feature / plugin | [10-plugins](references/10-plugins.md) |
| Login, sessions, tokens, permissions | [11-auth](references/11-auth.md) |
| PostgreSQL, QueryBuilder, `autoquery`, SQL style | [12-postgresql](references/12-postgresql.md) |
| Small data without a database (`MEMORIZE`, `REPO`) | [13-memorize](references/13-memorize.md) |
| Files, `PATH`, uploads, downloads, `FILESTORAGE` | [14-filesystem-uploads](references/14-filesystem-uploads.md) |
| Calling other services (`RESTBuilder`, `REQUEST`, `DOWNLOAD`) | [15-http-client](references/15-http-client.md) |
| Cron, periodic tasks, events, workers | [16-jobs-events-workers](references/16-jobs-events-workers.md) |
| WebSockets, realtime | [17-websockets](references/17-websockets.md) |
| `Total.extend()`, `DEF`, `MIDDLEWARE()` | [18-extending](references/18-extending.md) |
| Views, layout, sections | [19-views](references/19-views.md), [20-view-helpers](references/20-view-helpers.md) |
| String/Array/Date prototypes, Markdown, utilities | [21-utilities](references/21-utilities.md) |
| SEO, JSON-LD, sitemap, robots, llms.txt | [22-seo](references/22-seo.md) |
| Frontend without a framework | [23-frontend](references/23-frontend.md) |
| Performance | [24-performance](references/24-performance.md) |
| Security, robustness | [25-security](references/25-security.md) |
| Reviewing code, final check | [26-anti-patterns](references/26-anti-patterns.md), [27-gotchas](references/27-gotchas.md) |

Typical combinations:

- **New API backend:** 01, 02, 04, 07, 08, 09, 10, 11, 12, 28.
- **New feature in an existing backend:** 10, 08, 12, then 26.
- **New website without a database:** 02, 03, 08, 09, 13, 19, 20, 22, 28.
- **Uploads or files:** 14, 09.
- **Realtime:** 17, 11, 16.
