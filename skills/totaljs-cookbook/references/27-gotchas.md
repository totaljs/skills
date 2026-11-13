# 27. Gotchas

| Problem | Solution |
|---|---|
| `$.invalid(404)` / `$.invalidsafe()` returns **JSON** | a custom `notfound($)` with `$.response.status = 404` and a view |
| `@{!section('head')}` in the layout crashes | use `@{section('head')}` |
| `node -e "…Total.run()"` crashes on `process.argv[1]` | always run a file (`node index.js`) |
| `fetch` doesn't exist on the server (older Node.js) | `REQUEST()` / `DOWNLOAD()` |
| `ON('ready')` in a controller didn't fire reliably | `setImmediate(load)` in `exports.install` |
| CSS resets (normalize.css, also bundled in the Total.js UI library `spa.min.css`) give `input[type=search]` `box-sizing: content-box` → the field overflows | `box-sizing: border-box` directly on the input |
| `MEMORIZE().save()` writes only after 10 s | keep it in mind when testing/restarting |
| The Markdown renderer adds `loading="lazy"` and the `hidden` class | don't add it twice, remove `hidden` |
| Images in a readme with `src="URL"` (a syntax example) → 404 | filter out `src` values that aren't addresses |
| Switching the data source to a branch with an old format would break the site | validate before swapping ([section 25](25-security.md)) |
| `CONF.url` applies locally too → canonical points to production | intended; when testing, check paths, not the host |
| The action `$` has no `$.json()` / `$.view()` | finish with `$.success()` / `$.callback()` / `$.invalid()` ([section 8](08-actions.md)) |
| `ROUTE(url, fn, ['upload'], size)` silently registers a wrong route | flags and size go into the URL: `@upload <10MB` ([section 9](09-controllers-routing.md)) |
| `<5m` is a timeout of five minutes, not five megabytes | sizes need the unit `MB` / `kB` |
| `+orders_create` vs `+API` — two different `+` signs | before the method = auth; before the schema string = "has a payload" |
| A module calls `FUNC.x()` at its top level → `undefined` | definitions load after modules; call `FUNC` inside exported functions |
| `TEMP` loses its content every few minutes | runtime state belongs on `MAIN`, expiring values in `CACHE` |
| `PATH.verify()` looks like validation | it only creates the directory; constrain user input yourself |
| `DATA.modify()` / `DATA.remove()` without a filter changes every row | always `.id()` / `.where()` + `.error(404)` |
| `.promise()` without `$` in an action → an unhandled rejection | `.promise($)` in actions; `try/catch` elsewhere |
| A value from `config` arrives as the string `'true'` | declare the type: `allow_register (boolean) : true` |
| WebSocket messages are already parsed JSON | no `JSON.parse` / `JSON.stringify`; send objects |
