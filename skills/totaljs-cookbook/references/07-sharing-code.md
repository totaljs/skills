# 7. Sharing code: modules, definitions, FUNC and MAIN

This is how Total.js shares application code. Internal `require()` is the wrong tool.

### Modules → `MODS`

Every `/modules/*.js` file is loaded by the framework and stored on `MODS`. The key is the file name without `.js` (override with `exports.id`).

`modules/storage.js`:

```js
// A module is the right place to require() an SDK the framework does not provide
const { S3Client, PutObjectCommand } = require('@aws-sdk/client-s3');

exports.install = function() {
	// optional startup; keep it short, long work belongs in ON('ready')
};

exports.putFile = async function(relpath, body, mime) {
	// talk to S3 / disk
};

exports.getUrl = async function(relpath) {
	// ...
};
```

Anywhere after boot — controllers, schemas, definitions, other modules:

```js
await MODS.storage.putFile(path, body, mime);
```

Callers use `MODS.storage`, never the SDK and never `require('../modules/storage')`. Modules load before controllers, so an alias at the top of a controller is safe:

```js
// Syntax highlighter (/modules/highlight.js)
const Highlight = MODS.highlight;

html = Highlight.highlight(html);
```

**What belongs in a module:** a reusable integration or library with its own surface — object storage, a mail provider wrapper that is more than `MAIL()`, a search / AI engine, a payment SDK, a push gateway, a syntax highlighter. A module exposes functions on `exports`, may read `CONF`, and may use `DATA` and `FUNC` from functions that run after boot.

**What does not:** `paginate()`, permission checks or SQL. Those are `FUNC` or actions.

### Definitions

`/definitions/*.js` is cross-cutting startup. Definitions are auto-loaded and need no `exports.install()`.

| File (typical) | Assigns |
|---|---|
| `db.js` | `querybuilderpg` → `DATA` |
| `auth.js` | `AUTH()` |
| `func.js` | generic `FUNC.*` (paginate, can, audit) |
| `func-mail.js`, … | more `FUNC.*`, split by concern |
| `sessions.js` | `MAIN.sessions` |
| `jobs.js` | `CRON()` / `ON('service')` |
| `realtime.js` | `FUNC.realtime_*`, `MAIN.ws_users` |
| `cors.js` | the single `CORS()` call |

Several definitions may assign `FUNC.*`. Do not grow one 2,000-line `func.js` of unrelated domain math.

### `FUNC`

Reusable application functions, assigned in definitions:

```js
FUNC.hash_password = function(password) {
	return String(password).sha256(CONF.salt);
};

FUNC.can = function($, perm) {
	var user = $.user;
	if (!user)
		return false;
	if (user.sa)
		return true;
	return (user.permissions || EMPTYARRAY).includes(perm);
};

FUNC.require_perm = function($, perm) {
	if (FUNC.can($, perm))
		return true;
	$.invalid(403);
	return false;
};

FUNC.audit = async function($, action, entity, entityid, payload) {
	try {
		await DATA.insert('tbl_audit', {
			id: UID(),
			userid: $.user ? $.user.id : null,
			entity: entity || null,
			entityid: entityid || null,
			action: action,
			payload: payload ? JSON.stringify(payload) : null,
			ip: $.ip,
			dtcreated: NOW
		}).promise();
	} catch (err) {
		Total.error(err, 'audit');
	}
};
```

Called from actions:

```js
if (!FUNC.require_perm($, 'users.manage'))
	return;
```

Good `FUNC` candidates: password hashing, permission checks, session create/refresh, safe user snapshots, audit/notify, small domain calculators used by more than one plugin.

Bad `FUNC` candidates: one plugin's private formatter (keep it local in the file), view helpers of one controller (local `Func`, [section 20](20-view-helpers.md)), HTTP handlers, raw SQL dumps of a whole feature.

### `MAIN`

Shared **runtime** data. Not configuration, not the database.

```js
var map = new Map();

MAIN.sessions = {
	get: async key => map.get(key) || null,
	set: async (key, value) => map.set(key, value),
	delete: async key => map.delete(key)
};

MAIN.ws_users = {};     // userid → Set of websocket clients
```

Use `MAIN` for the session cache, websocket connection maps, short-lived boot flags and process-local registries. Do not keep business records in `MAIN` when they belong in the database.
