# 2. Project structure

```
index.js              start script: require('total5'); Total.run(...)
config                configuration (key : value) → CONF
config-debug          overlay for debug mode (optional)
config-release        overlay for release mode (optional)
definitions/          startup: AUTH, DATA init, FUNC.*, MAIN.*, CRON, ON()
modules/              integrations and shared code → MODS.<filename>
controllers/          HTTP / FILE / SOCKET routes, exports.install()
plugins/<feature>/
  index.js            metadata + ROUTE() registration
  schemas/*.js        NEWSCHEMA() actions (auto-loaded)
  public/             optional UI assets of the plugin
actions/              standalone NEWACTION() files (auto-loaded)
schemas/              shared NEWSCHEMA() files (optional)
workers/              Total.js workers (optional)
views/                templates (.html), layout.html is the default layout
public/               static files
databases/            data (MEMORIZE, downloaded JSON files)
tmp/                  temporary files
```

The framework loads JavaScript from these directories. You do not register them and you do not `require()` them.

**Which shape for which project**

| Project | Uses |
|---|---|
| Website without a database | `actions/`, `controllers/`, `views/`, `modules/`, `databases/` (MEMORIZE) |
| API backend | `definitions/`, `plugins/<feature>/`, `modules/`, a few `controllers/` (upload, health, sockets) |
| Both | all of the above; pages in `controllers/` call the same actions as the API |

**Load order** (verified in the total5 source):

```
.env / config → CONF
extensions → modules → actions → schemas → models → definitions → controllers → middleware → …
plugins/<id>/index.js, then that plugin's controllers, actions, schemas, definitions
```

Each file's `exports.install()` runs when that file is loaded. This means:

- a module from `/modules/` is already available when actions, definitions and controllers load,
- actions are defined before a controller calls them,
- definitions see modules; plugin schemas see the `FUNC.*` assigned in `definitions/`,
- **a module's top-level body must not call `FUNC.*`** — definitions have not run yet. Call `FUNC` from exported functions, which run after boot,
- action bodies run per request, long after boot, so they may use anything.

**Global variable convention:** the framework is available as `Total` and under the older shorthand `F` (the same object). In new code use the more readable `Total` (`Total.run()`, `Total.Fs`, `Total.error()`).

`index.js` stays tiny. Do not build an application container, logger factory or config loader around boot — Total.js already loads `config`, `.env`, definitions, modules, plugins, controllers, schemas and actions.
