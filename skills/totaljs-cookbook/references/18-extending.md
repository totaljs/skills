# 18. Extending the framework

Rarely needed. When unsure, use `FUNC`.

| Need | Mechanism |
|---|---|
| Pure helper | `FUNC.x = function(...)` |
| Integration | `/modules/x.js` → `MODS.x` |
| Domain operation | `NEWSCHEMA` / `NEWACTION` |
| Boot / auth / db | `definitions/` |
| Request/response sugar where `this` must be `$` | `Total.extend('$', 'x', fn)` |

### `Total.extend()`

Adds a method to a framework prototype:

```js
Total.extend('$', 'mail', function(address, subject) {
	var $ = this;
	return MAIL(address, subject, 'mail/notif', { user: $.user });
});

// later in an action
$.mail(user.email, '@(Welcome)');
```

| First argument | Target |
|---|---|
| `'$'` / `'options'` | the action `$` |
| `'controller'` | HTTP and WebSocket controllers |
| `'error'` / `'errorbuilder'` | `ErrorBuilder` |
| `'restbuilder'` | `RESTBuilder` |
| `'mail'` / `'email'` | mail message |
| `'view'` | view engine |
| `'querybuilder'` / `'database'` | QueryBuilder — verify in the installed version first; prefer a `FUNC` helper that takes a builder |

Use it only when many actions need the same **context** helper. Not for ordinary functions:

```js
// WRONG — not a request concern
Total.extend('$', 'hashPassword', function(pw) { ... });

// RIGHT
FUNC.hash_password = function(pw) {
	return String(pw).sha256(CONF.salt);
};
```

Two or three extensions per app are plenty. Do not invent a utility module of prototype patches.

### `DEF`

Framework defaults; override rarely.

```js
DEF.onSuccess = function(value) {
	return { success: true, value: value };
};
```

Changing `DEF.onSuccess` changes `$.success()` globally. Do not replace it with `{ ok: true, data }` unless the whole product (and every client) agrees.

### `MIDDLEWARE()`

`MIDDLEWARE('name', function($, next) { … })` plus the route flag `#name`. Use it for rare cross-cutting HTTP concerns that `AUTH()` and actions cannot express. Do not rebuild Express middleware chains — auth, permissions and validation already have framework homes.
