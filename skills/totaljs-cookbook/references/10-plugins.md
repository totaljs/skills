# 10. Plugins

A plugin is the feature unit of a backend. Total.js auto-loads `/plugins/<id>/index.js` into `PLUGINS.<id>` and also the plugin's own `schemas/`, `actions/`, `controllers/` and `definitions/`. The identifier is the directory name (`plugins/orders` → `PLUGINS.orders`).

```
plugins/
	orders/
		index.js            metadata + ROUTE() in exports.install()
		schemas/
			orders.js       NEWSCHEMA('Orders', ...)  (auto-loaded)
		public/             optional admin/UI assets
			index.html
			form.html
```

Two common shapes:

1. **API feature plugin** (JSON backends, mobile APIs) — `index.js` registers routes, `schemas/` holds the actions. No UI.
2. **Admin / UI plugin** — the same server entry plus `public/` HTML for a Total.js client.

### `index.js`

```js
exports.name = '@(Orders)';
exports.icon = 'ti ti-receipt';
exports.position = 10;
exports.permissions = [{ id: 'orders', name: 'Orders' }, { id: 'orders_remove', name: 'Remove orders' }];
exports.visible = user => user.sa || user.permissions.includes('orders');

exports.install = function() {
	ROUTE('+API ?  -orders_list         --> Orders/list');
	ROUTE('+API ?  -orders_read/{id}    --> Orders/read');
	ROUTE('+API ?  +orders_create       --> Orders/create');
	ROUTE('+API ?  +orders_update/{id}  --> Orders/update');
	ROUTE('+API ?  -orders_remove/{id}  --> Orders/remove');
	// FILE / SOCKET / upload routes of this feature also belong here
};
```

| Export | Type | Meaning |
|---|---|---|
| `name` | String | display name, with localization markup `@(…)` |
| `icon` | String | `ti ti-<icon>` |
| `position` | Number | order in the navigation |
| `permissions` | Array | optional permission catalog `[{ id, name }]` introduced by the plugin |
| `visible` | Function(user) | returns whether the plugin is visible for the session; `user.sa` / `user.su` see everything |
| `hidden` | Boolean | hides the plugin in the menu (it is still evaluated on the client) |
| `import` | String | optional file from the plugin's `public/` imported by the client app while it loads; most plugins don't use it |
| `install` | Function | registers routes |

`name`, `icon`, `position`, `visible` and `permissions` matter for a Total.js admin/UI client. A pure API plugin can keep only `install`.

### Rules

- `index.js` is the route table; the actions live in `schemas/`. A tiny plugin (one or two actions) may declare `NEWACTION` with `route: '+API ?'` directly in `index.js`.
- Do not `require()` a plugin schema from another file. Do not turn a plugin into an npm-style package of classes.
- Plugins talk to each other through `ACTION()`, `FUNC` and events (`EMIT` / `ON`), not through imports.
- Optional product modules can skip registration:

```js
exports.install = function() {
	if (!CONF.allow_orders)
		return;
	ROUTE('+API ?  -orders_list --> Orders/list');
};
```
