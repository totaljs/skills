# 5. Coding style

Match these conventions in all application JavaScript.

### Formatting

- Tabs, not spaces (tab width 4).
- Semicolons at the end of statements.
- Short strings in single quotes `'...'`.
- No unnecessary whitespace; no braces around a single-statement `if` / `for` body.
- Classic `for` loops over `forEach()` where possible.
- **`var`-first.** Do not declare `const` inside function scopes; use `var` (or `let` for block bindings that must rebind). `const` at the top of a file is acceptable for a real npm import or a file-level constant object.

```js
// preferred
var user = await DATA.read('tbl_user').id($.user.id).promise($);
for (var i = 0; i < items.length; i++)
	FUNC.normalize(items[i]);

// avoid in application files
const user = await DATA.read('tbl_user').id($.user.id).promise($);
items.forEach((item) => FUNC.normalize(item));
```

### Names

Reuse the same names across projects, and use the same names in SQL and JS:

- `id`, `name`, `value`, `email`, `body`
- `items` or `arr` for collections; `item`, `tmp` for locals
- `dtcreated`, `dtupdated` (dates start with `dt`), `isremoved`, `ispublished` (booleans start with `is`)
- foreign keys: `userid`, `orderid`

| Thing | Convention | Example |
|---|---|---|
| File names | lowercase; kebab-case only for several words | `orders.js`, `ai-engine.js` |
| Schema names | PascalCase | `NEWSCHEMA('Orders', …)` |
| Standalone action ids | `Namespace\|action` | `NEWACTION('Orders\|create', …)` |
| Public API schema strings | snake_case | `orders_list`, `orders_create` |
| `FUNC` helpers | snake_case | `FUNC.hash_password` |
| Tables / views | `tbl_`, `cl_`, `view_` + singular | `tbl_order`, `view_user` |

### Comments

Explain non-obvious constraints, not what the next line does. Do not paste the same `$` field dump into every action. A view starts with a comment describing its model ([section 19](19-views.md)).

### Errors and localization

User-facing messages use Total.js localization markup `@(…)`; transport failures use status codes:

```js
$.invalid('@(Invalid credentials)');
$.invalid(401);
$.invalid(403);
$.invalid(404);
```
