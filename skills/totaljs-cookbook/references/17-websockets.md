# 17. WebSockets

Use the framework's sockets. Do not add `socket.io`.

### Gateway

A socket route is one controller instance shared by all its clients. Messages are **JSON by default**: the handler receives a parsed object and `send()` takes an object (`@text` / `@binary` on the route change that).

```js
// controllers/realtime.js
exports.install = function() {
	ROUTE('+SOCKET /realtime/', socket);    // AUTH() runs at the handshake
};

function socket($) {

	MAIN.ws = $;

	// runs when the last client leaves; drop process-global references here
	$.autodestroy(() => MAIN.ws = null);

	$.on('open', function(client) {
		FUNC.realtime_bind(client, client.user.id);
	});

	$.on('close', function(client) {
		FUNC.realtime_unbind(client);
	});

	$.on('message', function(client, msg) {
		if (msg && msg.type === 'ping')
			client.send({ type: 'pong', ts: NOW });
	});
}
```

| API | Meaning |
|---|---|
| `$.on('open', fn(client))` | a client connected |
| `$.on('close', fn(client, code, reason))` | a client disconnected |
| `$.on('message', fn(client, message))` | a message arrived |
| `$.send(obj, [filter])` | broadcast to all clients of the route |
| `$.autodestroy(fn)` | destroy the route instance when it is empty |
| `client.send(obj)`, `client.close(code, reason)` | one client |
| `client.user`, `client.query`, `client.ip`, `client.headers` | handshake data |

Dynamic socket routes are just more `ROUTE('SOCKET /path/', handler)` entries.

### Authentication by the first message

A browser cannot send custom headers with a WebSocket. When the token is not in a cookie, register the route without `+` and authenticate in the first message:

```js
function socket($) {

	$.on('open', function(client) {
		client.authtimer = setTimeout(() => client.close(4001, 'auth timeout'), 15000);
	});

	$.on('close', function(client) {
		clearTimeout(client.authtimer);
		FUNC.realtime_unbind(client);
	});

	$.on('message', async function(client, msg) {

		if (!msg || !msg.type)
			return;

		if (msg.type === 'auth') {
			var user = await FUNC.realtime_auth(client, msg.token);
			if (user) {
				clearTimeout(client.authtimer);
				client.send({ type: 'auth_ok', userid: user.id });
			} else {
				client.send({ type: 'auth_error' });
				client.close(4003, 'unauthorized');
			}
			return;
		}

		if (!client.userid)
			return;

		// ... authorized messages
	});
}
```

`FUNC.realtime_auth` decrypts the token the same way `AUTH()` does, calls `FUNC.realtime_bind` and returns the user.

### Sending to a user

Keep the connection registry on `MAIN` and the public API on `FUNC` (`definitions/realtime.js`):

```js
MAIN.ws_users = {};     // userid → Set of clients

FUNC.realtime_bind = function(client, userid) {
	if (!MAIN.ws_users[userid])
		MAIN.ws_users[userid] = new Set();
	MAIN.ws_users[userid].add(client);
	client.userid = userid;
};

FUNC.realtime_unbind = function(client) {
	var set = MAIN.ws_users[client.userid];
	if (set) {
		set.delete(client);
		if (!set.size)
			delete MAIN.ws_users[client.userid];
	}
};

FUNC.realtime_send = function(userid, msg) {
	var set = MAIN.ws_users[userid];
	if (set) {
		for (var client of set)
			client.send(msg);
	}
};
```

Actions then push through `FUNC.realtime_send(userid, { type: 'order', id: id })`, usually from an event handler (`ON('orders.create', …)`).

### More instances

The in-process `MAIN.ws_users` does not cross Node processes. With more than one instance, a small Redis pub/sub bridge inside a definition is a legitimate `require('redis')`. Keep the public API as `FUNC.realtime_send`.

`WEBSOCKETCLIENT` exists when the server itself needs an outbound socket.
