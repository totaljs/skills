# 20. Local methods for views (`repository.func`)

Instead of a global `FUNC.*`, each controller has its own object of methods and passes it to the view. It's clear where a method comes from, and nothing pollutes the global scope.

```js
// Local methods for the views of this controller
// Controller: view.repository.func = Func;
// View: @{repository.func.name()}
const Func = {};

Func.pageurl = function(search, page) { … };
Func.date = function(value, format) { … };
Func.badge = function(m, large) { … };

function browse($) {
	ACTION('Items|browse', input).callback(function(err, response) {
		$.view('browse', response).repository.func = Func;
	});
}
```

In the view:

```html
<a href="@{repository.func.pageurl(query.search, p)}">@{p}</a>
<h3>@{!repository.func.badge(m)}@{m.name}</h3>
```

A function that returns HTML is called via `@{!…}` and **must escape** all of its inputs itself (`.encode()`).

### `Func` vs `FUNC`

They do not compete:

| | Local `Func` | Global `FUNC` |
|---|---|---|
| Purpose | presentation helpers for the views of one controller (URLs, badges, `<head>`) | application helpers shared by features (permissions, sessions, hashing, audit) |
| Declared in | the controller file | `definitions/` |
| Used from | views via `repository.func` | actions, definitions, modules, controllers |
