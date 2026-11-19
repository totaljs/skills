# 1. Architecture and principles

Total.js is not Express and not Nest. Do not redesign it around routers, DI containers or `require()` graphs. **If a pattern would make sense in Nest or Fastify, it is probably the wrong Total.js pattern.**

The running application is a **single global framework instance**. Files are auto-loaded by directory. Application code shares behavior through globals, not imports.

### The seven rules

1. **Globals, not imports.** `FUNC`, `MAIN`, `MODS`, `CONF`, `DATA`, `PATH`… already exist ([section 6](06-globals.md)). There is no service locator to invent — they are the locator.
2. **Directories, not registration.** Put a file in the right directory and the framework loads it ([section 2](02-project-structure.md)).
3. **Actions own the domain.** Business rules live in `NEWACTION` / `NEWSCHEMA` ([section 8](08-actions.md)). Controllers own HTTP, files and sockets only ([section 9](09-controllers-routing.md)).
4. **A feature is a plugin.** `plugins/<feature>/index.js` + `schemas/` ([section 10](10-plugins.md)).
5. **One of each.** One `AUTH()` delegate, one `CORS()` call, one API gateway (`POST /api/`), one response envelope (`$.success()`).
6. **Code is written for a person.** Someone who opens the file in a year must understand it without running it: plain names, short functions and a comment wherever the reason is not obvious from the code ([section 5](05-coding-style.md), "Comments").
7. **Code is written to be copied.** An action, a module or a plugin should work in the next project after copying its file or folder: what differs between projects is in `config`, and the file says at its top what it needs ([section 5](05-coding-style.md), "Reusable code").

### `require()` policy

`require()` is exceptional. Legitimate:

```js
require('total5');                                                   // index.js only
require('querybuilderpg').init('', CONF.database, 1, ERROR('DB'));   // definitions/db.js
const { S3Client } = require('@aws-sdk/client-s3');                  // a real external SDK, inside a module
```

Not legitimate inside controllers, schemas, actions, definitions or modules:

```js
// WRONG — internal sharing and Node built-ins
const storage = require('../modules/storage');
const helper = require('./lib/db');
const Path = require('path');
const Fs = require('fs');
const https = require('https');
```

Use instead:

```js
MODS.storage.putFile(...);
FUNC.hash_password(password);
PATH.join(PATH.public(), 'uploads');
PATH.fs.readFileSync(filename);
RESTBuilder.POST(url, payload).promise($);
```

CLI scripts, one-off migrations and tests that run *outside* `Total.run()` (`node scripts/migrate.js`) may `require()` Node built-ins and `pg`. Application runtime code must not.

### Where each kind of code goes

| Need | Put it here |
|---|---|
| Domain operation (create order, list items) | `NEWSCHEMA` / `NEWACTION` in `plugins/<feature>/schemas/` or `/actions/` |
| Feature HTTP API | `plugins/<feature>/index.js` (routes) + `schemas/` (actions) |
| Web page, health, webhook, OAuth redirect, upload, download | `controllers/` + `ROUTE('GET|POST|FILE …')` |
| WebSocket gateway | `controllers/` + `ROUTE('SOCKET …')` |
| Shared helper used by several features | `FUNC.*` assigned in `definitions/` |
| Helper used only by the views of one controller | a local `Func` object → `repository.func` ([section 20](20-view-helpers.md)) |
| Runtime map, cache, socket registry | `MAIN.*` |
| External integration with its own API (S3, payments, push, AI) | `/modules/<name>.js`, called as `MODS.<name>` |
| Database driver init | `definitions/db.js` |
| Auth | `AUTH()` in `definitions/auth.js` |
| Periodic work | `CRON()` or `ON('service')` in `definitions/` |
| Heavy isolated process | `/workers/<name>.js` |
| Config value | `CONF.key` from `config` |
| Request/response sugar needed by many actions | `Total.extend('$', …)` — rarely ([section 18](18-extending.md)) |

When unsure between `FUNC`, a module and an extension, use `FUNC`.

Do not add `src/`, `lib/`, `services/`, `repositories/`, `utils/` or `middleware/` directories.

### What this architecture is not

- not REST-resource controllers,
- not dependency injection,
- not one `require()` per file,
- not repository / unit-of-work classes,
- not Express middleware chains,
- not a generic Node.js folder layout with Total.js bolted on.
