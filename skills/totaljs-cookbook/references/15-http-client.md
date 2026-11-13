# 15. HTTP client: RESTBuilder, REQUEST and DOWNLOAD

Never `require('http')` / `require('https')` and never build a custom client.

| Need | Use |
|---|---|
| A JSON API (send a payload, get a parsed object) | `RESTBuilder` |
| Raw text / HTML / Markdown, custom handling of the status | `REQUEST()` |
| Save a response to a file | `DOWNLOAD()` |

### RESTBuilder

```js
RESTBuilder.GET(url);
RESTBuilder.POST(url, [payload]);
RESTBuilder.PUT(url, [payload]);
RESTBuilder.PATCH(url, [payload]);
RESTBuilder.DELETE(url, [payload]);
RESTBuilder.API(url, action, [payload]);   // a Total.js API endpoint: { schema: action, data: payload }
```

```js
builder.header(name, value);
builder.auth(user_or_token, [password]);
builder.cookie(name, value);
builder.timeout(10000);            // ms, default 5000
builder.urlencoded([payload]);
builder.file(name, filename, [buffer]);
builder.xhr();
builder.keepalive();
builder.insecure();                // skip TLS verification
builder.noparse();                 // return the raw body
builder.callback(function(err, response) { ... });
builder.promise($);                // errors go to $.invalid
builder.stream(callback);
```

```js
var profile = await RESTBuilder.GET('https://api.example.com/v1/profile')
	.header('Authorization', 'Bearer ' + CONF.api_token)
	.promise($);

var id = await RESTBuilder.API('https://other.example.com/api/', 'orders_create', model).header('x-token', CONF.token).promise($);
```

**Don't use `fetch` on the server** — older versions of Node.js don't have it. Total.js has its own client.

### REQUEST

```js
REQUEST({ url: 'https://example.com/file.md', method: 'GET', timeout: 20000 }, function(err, response) {
	// response.status, response.body (string), response.headers
});

// without a callback it returns a Promise
var response = await REQUEST({ url: url });
```

Parallel downloads with a limit (max. 4 at a time) via `Array.wait`:

```js
FILES.wait(function(path, next) {
	REQUEST({ url: SOURCE + path, timeout: 20000 }, function(err, response) {
		if (err || response.status !== 200)
			error = err || new Error(path + ' ' + response.status);
		else
			files[path] = response.body;
		next();
	});
}, function() {
	// all done
}, 4);
```

### DOWNLOAD

Saves the response directly to a file (Promise):

```js
await DOWNLOAD('https://example.com/data/items.json', PATH.tmp('items.json'));
```

Paths: `PATH.databases(name)`, `PATH.public(name)`, `PATH.tmp(name)`.
