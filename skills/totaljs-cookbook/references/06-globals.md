# 6. Globals

These names exist after `require('total5')`. Never import them.

| Global | Purpose |
|---|---|
| `Total` / `F` | Framework instance. Prefer `Total`. |
| `CONF` | Config from `config` / `config-debug` / `config-release` |
| `FUNC` | Application functions you assign ([section 7](07-sharing-code.md)) |
| `MAIN` | Application runtime data you assign ([section 7](07-sharing-code.md)) |
| `MODS` | Auto-loaded `/modules/*.js` |
| `PLUGINS` | Auto-loaded `/plugins/<id>` |
| `REPO` | Global bag that is also visible in views. Not a session store. |
| `TEMP` | Scratch object. **Wiped about every 5 minutes.** Never for sessions. |
| `CACHE` | In-memory TTL cache |
| `DATA` | Shared QueryBuilder. **Use this for SQL.** ([section 12](12-postgresql.md)) |
| `MEMORIZE` | A persisted JSON object ([section 13](13-memorize.md)) |
| `PATH` | Application directories and fs helpers ([section 14](14-filesystem-uploads.md)) |
| `DEF` | Framework defaults and hooks (`DEF.onSuccess`, …) |
| `DEBUG` | `process.env.NODE_ENV !== 'production'` |
| `NOW` | Current `Date`, refreshed by the framework timer |
| `U` / `Utils` | Total.js utility helpers. Not `Total.Util` (that is `node:util`). |
| `EMPTYOBJECT` / `EMPTYARRAY` | Shared frozen empties — never mutate |
| `ROUTE` | Register HTTP / API / FILE / SOCKET routes |
| `CORS` | CORS origins — call once |
| `AUTH` | The authorization delegate ([section 11](11-auth.md)) |
| `NEWACTION` / `NEWSCHEMA` / `ACTION` | Declare and call actions ([section 8](08-actions.md)) |
| `MIDDLEWARE` | Named middleware (rare) |
| `UNAUTHORIZED` | Permission check used by the action option `permissions` |
| `BLOCKED` | Simple IP rate limit |
| `RESTBuilder` / `REQUEST` / `DOWNLOAD` | Outbound HTTP ([section 15](15-http-client.md)) |
| `WEBSOCKETCLIENT` | Outbound WebSocket |
| `CRON` | Cron jobs ([section 16](16-jobs-events-workers.md)) |
| `ON` / `ONCE` / `OFF` / `EMIT` | Events |
| `NEWTHREAD` / `NEWFORK` / `NEWTHREADPOOL` | Workers under `/workers/<name>.js` |
| `UID` / `GUID` / `HASH` | Identifiers and hashes |
| `ENCRYPT` / `DECRYPT` / `ENCRYPTREQ` / `DECRYPTREQ` | Crypto helpers |
| `MAIL` / `HTMLMAIL` / `LOGMAIL` | Mail |
| `FILESTORAGE` | Native file storage ([section 14](14-filesystem-uploads.md)) |
| `ERROR` | Error-logging callback factory (`ERROR('DB')`) |
| `ErrorBuilder` | Structured errors |
| `SUCCESS` | `DEF.onSuccess(value)` |
| `PROMISIFY` | Callback API → Promise |
| `TRANSFORM` / `NEWTRANSFORM` | Value transforms |
| `PROXY` | HTTP proxy routes |

### Identifiers and hashes

```js
UID();                        // unique id (timestamp + counter + checksum) — use for primary keys
GUID();                       // a real 128-bit GUID
GUID(40);                     // random string of a given length — API keys, tokens

HASH('value', 'sha256');      // sha1, sha256, sha512, md5, crc32
String(password).sha256(CONF.salt);
id.sign(CONF.salt);           // 'id-signature', see signed downloads
```

### `NOW`

A shared `Date` refreshed by the framework timer so the app does not create `new Date()` all the time. It is the convention for `dtcreated` / `dtupdated`. For a precise "right now" in a security comparison, `new Date()` is still acceptable.

```js
NOW.add('14 days');
NOW.add('15 minutes');
```

### `ENCRYPTREQ` / `DECRYPTREQ`

Encrypts a payload and binds it to the request's user-agent (and to the IP when the 4th argument is `true`). This is the usual session token — do not introduce `jsonwebtoken` for the same job.

```js
var token = ENCRYPTREQ($, { id: session.id }, CONF.salt);
var data = DECRYPTREQ($, token, CONF.salt);     // null when it does not match
```

### `CACHE`

```js
CACHE.set('user_' + id, snapshot, '10 minutes');
var snapshot = CACHE.get('user_' + id);
CACHE.remove('user_' + id);
CACHE.reset('user_');   // removes keys containing this string
```

Invalidate on update/logout. Do not authorize from a stale cache when the database says the user is gone.

### `BLOCKED` and `ERROR`

```js
// max. 5 attempts per IP in 15 minutes
if (BLOCKED($, 5, '15 minutes')) {
	$.invalid('@(Too many attempts)');
	return;
}

BLOCKED($, -1);   // reset the counter, e.g. after a successful login

// ERROR('label') returns a callback that only logs the error
ACTION('Items|refresh').callback(ERROR('Source offline'));
```

### Mail

```js
MAIL(address, '@(Welcome)', 'mail/welcome', model);   // a view from /views/mail/welcome.html
HTMLMAIL(address, subject, html);
```

### Which bag for which data

| Data | Where |
|---|---|
| Configuration | `CONF` |
| Business records | PostgreSQL via `DATA`, or `MEMORIZE` for small data sets |
| Sessions, socket maps, registries, boot flags | `MAIN` |
| Values with an expiration | `CACHE` |
| Data that views read directly | `REPO` |
| A few minutes of scratch | `TEMP` |
