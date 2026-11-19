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

The code must be readable by a person, not only correct. Write a comment where a reader would otherwise have to guess; leave it out where the code already says the same thing.

Comment these places:

- **The top of a file**: what the file does, what it needs (other actions, tables, `config` keys) and how to call it, with one example of a call and of its result.
- **A constant with a meaning**: a limit, a timeout, a threshold, and why it has this value (`// The privacy policy promises 48 hours and the cleaning runs once an hour, so the limit is an hour shorter.`).
- **A decision that looks wrong or could be "simplified" into a bug**: a workaround, an order of steps that matters, a check that seems redundant, a deliberate omission.
- **A rule of the business or of the law** that the code enforces, in the words of that rule.
- **A regular expression, a long SQL query or a formula**: what it matches or computes, in one sentence.
- **A view**: a comment describing its model at the top ([section 19](19-views.md)).

Do not comment what the next line does (`// increments the counter`), do not paste the same `$` field dump into every action, and do not keep commented-out code. A comment that no longer matches the code is worse than none: change it together with the code.

```js
// avoid: repeats the code
// Checks the limit
if (BLOCKED({ ip: 'insert_' + $.ip }, LIMITS.insert)) {

// preferred: says why
// The offer waits for the confirmation of the e-mail: until then it is not processed and visitors do not see it.
model.isconfirmed = false;
```

Readable code needs fewer comments in the first place: name a value instead of explaining a number, return early instead of nesting, and split a function that does two things.

### Reusable code

Write every action, module and plugin so that it can be copied into another project and work there. Reuse by copying a file is the normal way of sharing code between Total.js projects, so a unit that cannot be copied is a unit that will be rewritten.

- **One unit, one file or folder.** An action file, a module or a `plugins/<feature>/` folder carries everything it needs: its constants, its small helpers, and next to it its SQL (`init.sql`), its data and its README.
- **Nothing of the project inside.** A URL, a token, a limit, a name of the product or an e-mail address is read from `config` (`CONF.<key>`) with a safe default, never written into the code ([section 4](04-configuration.md)).
- **State the dependencies at the top.** The first comment lists the actions, tables and `config` keys the file relies on, so a person who copies it knows what else to bring.
- **Talk through public contracts.** Call other code with `ACTION()` / `$.action()`, `MODS.<name>` or an event, with declared `input` and a described output. Do not reach into the internals of another file.
- **Generic names.** `items`, `id`, `name`, `dtcreated` ([Names](#names) above) and generic mechanisms (a queue of e-mails, a cache table, a rate limit) instead of names of one product.
- **A small helper may be repeated.** A three-line function copied into two self-contained files is better than a shared file that both must then travel with. Move a helper to `FUNC` or `MODS` only when it is real shared behavior ([section 7](07-sharing-code.md)).

Before finishing a unit, ask: *could this file be dropped into an empty Total.js project and work after setting its `config` keys?* If not, find what ties it to this project and move that to `config` or to the caller.

### Errors and localization

User-facing messages use Total.js localization markup `@(…)`; transport failures use status codes:

```js
$.invalid('@(Invalid credentials)');
$.invalid(401);
$.invalid(403);
$.invalid(404);
```
