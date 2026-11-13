# 11. Authentication and authorization

Authorization is **one delegate**: `AUTH()`. It runs for every request except static files, and its result drives the `+` / `-` route flags. `AUTH()` answers *who*; actions answer *whether*.

### `AUTH()`

```js
// definitions/auth.js
AUTH(async function($) {

	var token = FUNC.read_header_token($);
	if (!token || token.length < 20) {
		$.invalid();
		return;
	}

	var data = DECRYPTREQ($, token, CONF.salt);
	if (!data || !data.id) {
		$.invalid();
		return;
	}

	var session = await MAIN.sessions.get(data.id);
	if (!session)
		session = await DATA.read('tbl_session').id(data.id).promise();

	if (!session || (session.dtexpire && session.dtexpire < NOW)) {
		$.invalid();
		return;
	}

	var user = await FUNC.refresh_session(session);
	if (user)
		$.success(user);
	else
		$.invalid();
});
```

- `$.success(user)` → `$.user` is set; `+` routes match.
- `$.invalid()` → unauthorized; `-` routes match.
- Unprefixed routes (`API`, `GET`) run either way and may read `$.user`.
- `$` here is an Options object (no `$.json`). Inside `AUTH()` a bare `$.invalid()` is enough; in an action use `$.invalid(401)` for an explicit status.

The simplest possible delegate:

```js
AUTH(function($) {
	if ($.headers['x-token'] === CONF.token)
		$.success({ id: 'admin', name: 'Admin', sa: true });
	else
		$.invalid();
});
```

### Token headers

Accept a small compatibility set; prefer `x-token`:

```js
FUNC.read_header_token = function($) {
	var headers = $.headers || EMPTYOBJECT;
	var token = headers['x-token'] || headers.token || '';
	var auth = headers.authorization || '';
	if (!token && auth && auth.substring(0, 7).toLowerCase() === 'bearer ')
		token = auth.substring(7).trim();
	return token || '';
};
```

Tokens are opaque: the encrypted session id (`ENCRYPTREQ`). Do not put roles into a JWT that the client can read and the server trusts blindly. For a browser app the same token may travel in a cookie (`$.cookie('name', token, '14 days')`).

### Sessions

Create in `FUNC`, cache on `MAIN`, persist through `DATA`:

```js
FUNC.create_session = async function($, userid) {

	var session = {};
	session.id = UID();
	session.userid = userid;
	session.ip = $.ip;
	session.dtexpire = NOW.add(CONF.cookie_expires || '14 days');
	session.dtcreated = NOW;

	await DATA.insert('tbl_session', session).promise();

	var user = await FUNC.refresh_session(session);
	user && await MAIN.sessions.set(session.id, user);

	return { token: ENCRYPTREQ($, { id: session.id }, CONF.salt), user: FUNC.user_safe(user) };
};
```

- `FUNC.refresh_session(session)` loads the current user (id, name, `sa`, `permissions`) and returns `null` when the user is gone or removed.
- `FUNC.user_safe(user)` must strip password hashes, reset tokens and secrets.
- Logout removes the row, the `MAIN.sessions` entry and any `CACHE` snapshot.
- Expire old sessions in `ON('service')` or `CRON()` ([section 16](16-jobs-events-workers.md)).

### Login

```js
// plugins/auth/schemas/auth.js
NEWSCHEMA('Auth', function(schema) {

	schema.action('login', {
		input: '*email:Email, *password:String',
		action: async function($, model) {

			if (BLOCKED($, 5, '15 minutes')) {
				$.invalid('@(Too many attempts)');
				return;
			}

			let user = await DATA.read('tbl_user')
				.fields('id')
				.where('email', model.email)
				.where('password', FUNC.hash_password(model.password))
				.where('isremoved', false)
				.promise($);

			if (!user) {
				$.invalid('@(Invalid credentials)');
				return;
			}

			BLOCKED($, -1);
			$.success(await FUNC.create_session($, user.id));
		}
	});

	schema.action('me', {
		action: function($) {
			$.callback(FUNC.user_safe($.user));
		}
	});
});
```

```js
// plugins/auth/index.js
exports.install = function() {
	ROUTE('-API ?  +auth_login   --> Auth/login');    // public
	ROUTE('+API ?  -auth_me      --> Auth/me');       // session required
	ROUTE('+API ?  -auth_logout  --> Auth/logout');
};
```

Public login/register stay on `-API`. Protected work stays on `+API`. A public list that can personalize when a user is present uses an unprefixed `API` route and reads `$.user`.

### Permissions

Declarative, on the action:

```js
schema.action('remove', {
	permissions: 'orders_remove',   // UNAUTHORIZED($, 'orders_remove'); sa/su pass
	// sa: true,                    // super admin only
	// user: true,                  // any signed-in user
	action: async function($) { ... }
});
```

`UNAUTHORIZED` reads `$.user.permissions` (an array of strings) and lets `$.user.sa` / `$.user.su` through, so keep that shape on the session. Row ownership belongs in the query, not in JavaScript: `.where('userid', $.user.id)`.

### API keys and sockets

- If you accept `x-api-key`, resolve it **inside** `AUTH()` and map it to the same session shape (`id`, `permissions`, `auth_source: 'api'`). No second middleware.
- `AUTH()` also runs at the WebSocket handshake, so `+SOCKET` works when the token travels in a cookie or header. A browser cannot set custom headers on a WebSocket; then authenticate with the first message ([section 17](17-websockets.md)).

### Do not

- implement Passport / `jsonwebtoken` middleware,
- decode tokens in every controller,
- store the password hash on `$.user`,
- trust a `role` or `userid` field sent by the client,
- split identity by URL path unless you truly have separate apps; prefer one user with roles/permissions.
