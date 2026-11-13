# 8. Actions: NEWACTION and NEWSCHEMA

Business logic belongs in actions. A controller or an API route only calls an action; nothing sits underneath it (no service classes, no repositories).

Total.js 5 has two first-class ways to declare actions. Both run through the same `$` and the same `ACTION()` caller.

| | Use it for | Lives in |
|---|---|---|
| `NEWSCHEMA('Name', fn)` + `schema.action()` | a group of related actions of one domain; routes are registered by the plugin | `plugins/<feature>/schemas/` |
| `NEWACTION('Name\|action', opt)` | one isolated action, optionally with its own `route` | `/actions/` |

### NEWACTION

```js
NEWACTION('Items|browse', {
	name: 'Browse items',
	input: 'search:String, page:Number, limit:Number',   // input schema
	route: '+API ?',                                     // API route (authorized users only)
	action: async function($, model) {
		// model = validated input
		$.callback(output);    // success
		// $.invalid(404);     // error
	}
});
```

### NEWSCHEMA

```js
// plugins/orders/schemas/orders.js
NEWSCHEMA('Orders', function(schema) {

	schema.action('list', {
		name: 'List orders',
		query: 'page:Number, limit:Number, search:String',
		action: async function($) {
			var response = await DATA.list('tbl_order')
				.where('isremoved', false)
				.autoquery($.query, 'id:String, name:String, dtcreated:Date', 'dtcreated_desc', 100)
				.promise($);
			$.callback(response);
		}
	});

	schema.action('read', {
		params: '*id:String',
		action: async function($) {
			var response = await DATA.read('tbl_order').id($.params.id).where('isremoved', false).error(404).promise($);
			$.callback(response);
		}
	});

	schema.action('create', {
		input: '*name:String, body:String',
		action: async function($, model) {
			model.id = UID();
			model.userid = $.user.id;
			model.dtcreated = NOW;
			await DATA.insert('tbl_order', model).promise($);
			$.success(model.id);
		}
	});

	schema.action('update', {
		params: '*id:String',
		input: '*name:String, body:String',
		action: async function($, model) {
			model.dtupdated = NOW;
			await DATA.modify('tbl_order', model).id($.params.id).error(404).promise($);
			$.success($.params.id);
		}
	});

	schema.action('remove', {
		params: '*id:String',
		permissions: 'orders_remove',
		action: async function($) {
			await DATA.modify('tbl_order', { isremoved: true, dtremoved: NOW }).id($.params.id).error(404).promise($);
			$.success($.params.id);
		}
	});
});
```

The routes are registered by the plugin ([section 10](10-plugins.md)):

```js
ROUTE('+API /api/  -orders_list         --> Orders/list');
ROUTE('+API /api/  -orders_read/{id}    --> Orders/read');
ROUTE('+API /api/  +orders_create       --> Orders/create');
ROUTE('+API /api/  +orders_update/{id}  --> Orders/update');
ROUTE('+API /api/  -orders_remove/{id}  --> Orders/remove');
```

Schema files are auto-loaded. Never `require()` them.

### Options

| Option | Meaning |
|---|---|
| `name` | human-readable name (documentation) |
| `input` | schema of the payload → the `model` argument (`$.model`) |
| `query` | schema of query arguments → `$.query` |
| `params` | schema of route params → `$.params` |
| `output` | schema of the response; the output is transformed to match it |
| `route` | `NEWACTION` only: registers a route for the action — `'+API ?'`, `'-API ?'`, `'API ?'` or a classic `'+POST ?/tasks/create/'` |
| `action` | `function($, model)` — required |
| `user` | `true` → `$.user` must exist |
| `permissions` | `'perm1, perm2'` → the framework calls `UNAUTHORIZED($, …)`; `$.user.sa` / `$.user.su` pass everything |
| `sa` | `true` → `$.user.sa` or `$.user.su` must be `true` |
| `partial` | `true` → only the received fields are validated (PATCH-like updates) |
| `publish` | performs `NEWPUBLISH()` with the data: `true` copies input/output, a string is a schema, `+string` extends it |
| `extend` | a property name; extends the current payload with this action's response |

In `route`, `?` is replaced by `CONF.$api` (default `/api/`), so `'+API ?'` means `'+API /api/'`.

### Data schema

`name:Type, name:Type`, separated by commas. `*name` = required. Types are case-insensitive. A field without a type is a string.

| Kind | Types |
|---|---|
| Basic | `string`, `number`, `boolean`, `date`, `json` |
| Numbers | `tinyint`, `smallint`, `number` |
| Identifiers | `uid`, `guid` |
| Validated strings | `email`, `phone`, `zip`, `url`, `base64`, `color` (`#F0F0F0`), `icon` (`ti ti-home`) |
| Normalized strings | `name` (person's name, capitalized), `lowercase`, `uppercase`, `capitalize`, `capitalize2` |
| Data URI | `datauri` → `{ type, buffer }`, `datauri2` → a valid data URI string |
| Enum | `status:{draft\|published\|archived}` |
| Arrays | `tags:[String]`, `ids:[UID]` |
| Nested object | `address:{ street:String, city:String }` |
| Array of objects | `rows:[name:String, age:Number]` |
| Linked schema | `profile:@Profile` — declared as `NEWSCHEMA('@Profile', '*name:String, age:Number')` |

The schema is designed for one level of properties; keep payloads flat.

### The action `$` (Options)

> **The most common mistake:** an action's `$` is an **Options** object, not the HTTP controller. It has no `$.json()`, `$.view()`, `$.file()` or `$.html()`. Finish every action with `$.success()`, `$.callback()` or `$.invalid()`.

| Read | Meaning |
|---|---|
| `model` / `$.model` | validated `input` |
| `$.query`, `$.params` | validated query / route params |
| `$.user` | the session from `AUTH()` |
| `$.headers`, `$.ip`, `$.ua`, `$.language`, `$.url` | request details |
| `$.files` | uploaded files |
| `$.controller` | the HTTP controller when the action was called from a route (may be missing) |

| Respond | Result |
|---|---|
| `$.success()` | `{ success: true }` |
| `$.success(value)` | `{ success: true, value: value }` (built by `DEF.onSuccess`) |
| `$.callback(payload)` | the raw payload — lists, rows, query results |
| `$.invalid('@(Message)')` | validation / business error |
| `$.invalid(404)` | error with an HTTP status |
| `$.done(value)` | returns a Node-style callback: error → `$.invalid`, otherwise `$.success(value)` |
| `$.redirect(url)` | redirect |

Do not rebuild the envelope by hand (`$.callback({ success: true, value: id })`). Respond exactly once, and `return` after `$.invalid()`.

`.promise($)` on `DATA`, `ACTION` and `RESTBuilder` forwards errors to `$.invalid()` automatically and stops the action, so no `try/catch` is needed around them.

### Calling actions

```js
// from another action — errors go to $.invalid
var id = await ACTION('Orders/create', { name: 'X' }).user($.user).promise($);

// from a page controller
ACTION('Items|browse', { search: 'grid', page: 2 }).callback(function(err, response) {
	if (err)
		notfound($);
	else
		$.view('browse', response).repository.func = Func;
});

// fire and forget
ACTION('Items|refresh').callback(ERROR('Source offline'));
```

The id is the schema name plus the action (`Orders/create`) or the `NEWACTION` id (`Items|browse`). Builder methods: `.query({…})`, `.params({…})`, `.user(session)`, `.config({…})` — they supply what a route would otherwise supply.

### Internal actions

`route: '+API ?'` registers the action for signed-in users only. In an app without an `AUTH()` definition nobody can call it from outside → a safe "internal" action that is still callable through `ACTION()`. An action with no `route` and no `ROUTE()` is internal by definition.

### Reusable preconditions

A route may chain actions; `(response)` marks the one whose output is returned:

```js
ROUTE('+API /api/  +orders_create  --> Orders/check Orders/insert (response)');
```

Use it for reusable preconditions (uniqueness, ownership). Do not build hidden pipelines with side effects.
