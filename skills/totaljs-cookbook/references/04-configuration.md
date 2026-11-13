# 4. Configuration

The `config` file uses the `key : value` format; lines starting with `//` are comments. Custom keys are available as `CONF.<key>`.

```
// Canonical address of the site
url                  : https://example.com
```

```js
// Canonical host: from config, otherwise from the request
function hostname($) {
	return CONF.url ? CONF.url.replace(/\/+$/, '') : $.hostname();
}
```

**Why `CONF.url`:** `$.hostname()` builds the address from the `Host` header and `X-Forwarded-Proto`. Behind a reverse proxy this easily produces `http://` instead of `https://`, or both `www.` and non-`www` → duplicate canonical URLs. A fixed domain in the config solves it once and for all (it also applies locally).

Built-in keys worth knowing: `CONF.$api` (the default API endpoint, `/api/`), `CONF.version`, `CONF.name`.

### More about `config`

`config` is not JSON. Keys become `CONF.<key>`; `config-debug` and `config-release` overlay it by mode.

```
name                     : My App
database                 : postgresql://user:pass@127.0.0.1:5432/app
salt                     : change-me
cookie_expires           : 14 days
allow_register (boolean) : true
api_key (env)            : MY_API_KEY
```

- A type in parentheses converts the value: `(boolean)`, `(number)`, `(array)`, `(json)` / `(object)`, `(date)`, `(random)` (generates a random string), `(hash)`. `(env)` copies `process.env.MY_API_KEY` into `CONF.api_key`.
- Declare the type of booleans and numbers. An untyped flag may arrive as the string `'true'`.
- Read config directly: `CONF.database`, `CONF.salt`. **Do not wrap it** in a settings module and do not load a parallel `config.json`.
- **`process.env` is for boot and deployment overrides only.** `index.js` may read `process.env.PORT` / `NODE_ENV`. Feature code reads `CONF`. If a value must come from the environment, use an `(env)` key, or assign it once in a definition:

```js
if (process.env.FEATURE_X != null)
	CONF.feature_x = process.env.FEATURE_X;
```
