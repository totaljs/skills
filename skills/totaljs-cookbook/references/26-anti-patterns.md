# 26. Anti-patterns

The mistakes that agents and newcomers produce most often. Each has a framework-native replacement.

| Don't | Do |
|---|---|
| `require('../modules/storage')`, `require('./lib/…')` | `MODS.storage`, `FUNC.*` |
| `require('fs')`, `require('path')`, `require('crypto')`, `require('https')` | `PATH.*`, `PATH.fs`, `Total.Crypto` / `HASH()`, `RESTBuilder` |
| `require('../config.json')`, a settings/env wrapper, `process.env.X` in feature code | `CONF.x` (+ `(env)` keys) |
| `class UserRepository`, service classes, an ORM, `require('pg')` in actions | `DATA` in actions; views in SQL |
| `function sendOk(res, data)`, `authMiddleware(req, res, next)` | `$.success()`, `$.callback()`, `$.invalid()`, `AUTH()` |
| `$.json({ ok: true, data: item })` or a hand-rolled `{ success: true, value }` | `$.success(item)` / `$.callback(item)` |
| `$.json()` / `$.view()` inside an action | `$.success()` / `$.callback()` — the action `$` is not a controller |
| Decrypting the token again in a handler | `+` route flag and `$.user` |
| Passport, `jsonwebtoken` | `AUTH()` + `ENCRYPTREQ` / `DECRYPTREQ` |
| `setInterval(tick, 300000)`, `node-cron` | `ON('service')`, `CRON()` |
| `socket.io` | `ROUTE('SOCKET …')` |
| `https.request({...})`, `fetch` on the server | `RESTBuilder`, `REQUEST()`, `DOWNLOAD()` |
| `DB()` everywhere (older docs) | `DATA`; `DB()` only for a fresh controller |
| Loading a table and filtering in JS; a `DATA.read` per id in a loop | `.where()`; `.in('id', ids)` |
| `DATA.query('SELECT … WHERE id=$1 LIMIT 1')` followed by `if (!rows.length)` for a plain read, insert, update or delete | `DATA.read('tbl').id(id).error(404)`, `DATA.insert()`, `DATA.modify()`, `DATA.remove()`; `DATA.query()` only for joins, aggregates and expressions ([section 12](12-postgresql.md)) |
| `ROUTE('POST /x/', fn, ['upload'], 1024)` (Total.js 4) | `ROUTE('POST /x/ @upload <1MB', fn)` |
| `CORS()` in every plugin | one `CORS()` in a definition |
| One 2,000-line `definitions/func.js` | several definitions split by concern |
| `Total.extend()` for ordinary helpers | `FUNC` |
| Business records in `MAIN`; sessions in `TEMP` or `REPO` | database; `MAIN` |
| REST-resource controllers with business rules | actions + the `/api/` gateway |
| `src/`, `lib/`, `services/`, `repositories/`, `utils/`, `middleware/` | the framework directories ([section 2](02-project-structure.md)) |
| `const` for every local, `forEach` with arrows, spaces, double quotes | `var`, classic `for`, tabs, single quotes ([section 5](05-coding-style.md)) |
| Copying another product's tables and schema names | copy the mechanism: plugin + `NEWSCHEMA` + `DATA` + `FUNC` |
| A file without a comment at its top; `// sets the value` above `x = value`; a "magic" number without a reason | A header that says what the file does and needs; comments that say why ([section 5](05-coding-style.md), "Comments") |
| A product name, URL, token or limit written into an action; a helper that only works with this project's tables | Values in `config`, dependencies listed at the top, a unit that can be copied ([section 5](05-coding-style.md), "Reusable code") |
