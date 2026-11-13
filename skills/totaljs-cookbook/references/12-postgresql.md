# 12. PostgreSQL: DATA and QueryBuilder

Application code talks to PostgreSQL through **`DATA`** (QueryBuilder). Do not add repositories, ORMs or `require('pg')` inside actions. Views (`view_user`) belong in SQL, not in a JS repository.

### Init

```js
// definitions/db.js — a legitimate require of an npm package
require('querybuilderpg').init('', CONF.database, 1, ERROR('DB'));
```

The empty name registers the default connection used by `DATA`; `1` is the pool size. `config`: `database : postgresql://user:pass@127.0.0.1:5432/app`.

`DATA` is the shared singleton; each call executes immediately. `DB()` creates a **new** controller — older docs use it everywhere; in Total.js 5 code prefer `DATA`.

### Operations

Each returns a QueryBuilder:

```js
DATA.find('tbl_user');                          // array of rows
DATA.list('tbl_user');                          // paginated { items, count, page, pages, limit }
DATA.read('tbl_user');                          // one row or null
DATA.insert('tbl_user', { id: UID(), name: 'Created', dtcreated: NOW });
DATA.modify('tbl_user', { name: 'Updated', dtupdated: NOW });   // alias: DATA.update
DATA.remove('tbl_user');
DATA.check('tbl_user');                         // exists? → Boolean
DATA.count('tbl_user');
DATA.scalar('tbl_user', 'avg', 'age');
DATA.query('SELECT id, name FROM tbl_user WHERE email=$1', [email]);
```

`modify` and `remove` without a filter touch the **whole table** — always add `.id()` / `.where()`.

### Builder methods

Filters combine with **AND** unless you use `.or()`.

```js
builder.fields('id,name,dtcreated');
builder.id(value);                         // id = value
builder.userid(value);                     // userid = value
builder.where(column, value);
builder.where(column, '>', value);         // = > < >= <= <>
builder.between(column, a, b);
builder.in(column, array);
builder.search(column, value, [operator]); // ILIKE; operator: '*' (default), 'beg', 'end'
builder.query('isremoved=FALSE');          // raw fragment — trusted SQL only
builder.or(function() {
	this.where('name', 'Peter');
	this.where('name', 'Anna');
});
builder.sort('dtcreated', true);           // true / 'desc' → DESC
builder.take(50);
builder.skip(50);
builder.paginate(page, limit, maxlimit);
builder.error(404);                        // error when nothing was found / changed
builder.error('@(Already exists)', true);  // reverse: error when a row exists
builder.autoquery(query, schema, default_sort, default_maxlimit);
builder.callback(function(err, response) { ... });
builder.promise($);                        // Promise; errors go to $.invalid
```

### Recipes

```js
// read or 404
let user = await DATA.read('tbl_user').id($.params.id).where('isremoved', false).error(404).promise($);

// insert
model.id = UID();
model.dtcreated = NOW;
await DATA.insert('tbl_order', model).promise($);

// update or 404
await DATA.modify('tbl_order', { name: model.name, dtupdated: NOW }).id(id).error(404).promise($);

// soft delete
await DATA.modify('tbl_order', { isremoved: true, dtremoved: NOW }).id(id).error(404).promise($);

// uniqueness
await DATA.check('tbl_user').where('email', model.email).error('@(E-mail is already used)', true).promise($);

// many rows by ids — never a read in a loop
let rows = await DATA.find('tbl_user').fields('id,name').in('id', ids).promise($);
```

### Lists with `autoquery()`

`autoquery()` reads `$.query` and builds filters, sorting, paging and the field list from a schema string. Only the listed fields can be filtered, sorted or returned.

```js
var response = await DATA.list('view_order')
	.where('isremoved', false)
	.autoquery($.query, 'id:String, name:String, email:String, dtcreated:Date', 'dtcreated_desc', 100)
	.promise($);

$.callback(response);   // { items, count, page, pages, limit }
```

The client then controls the list through the query string: `page`, `limit`, `sort=name_asc`, `fields=id,name`, and one argument per filtered field (`name=…`). Return the `DATA.list` result as it is; do not invent another list envelope. Use views for denormalized lists.

### `.promise($)` vs `.promise()`

- `.promise($)` — an error (including `.error(404)`) is sent through `$.invalid()` and the action stops. Use it in actions.
- `.promise()` — rejects; you must `try/catch`. Use it outside a request (`AUTH`, cron, `FUNC` helpers without `$`).

### Raw SQL

```js
var rows = await DATA.query('SELECT id, name FROM tbl_user WHERE email=$1', [email]).promise($);
```

**Never concatenate request values into SQL.** `builder.query('…')` is for trusted fragments only.

### SQL style

- Table and field names all lowercase; tabs; `;` at the end of every command; no needless type casts (`name<>'x'`, not `name<>'x'::text`).
- Divide scripts: TABLES, VIEWS, STORED PROCEDURES/FUNCTIONS, INDEXES.
- Prefixes, singular: `tbl_user`, `cl_type` (codelists), `view_user`, `sp_user` (procedures), `fn_user` (functions).
- Dates start with `dt` and are stored in UTC: `dtcreated`, `dtupdated`, `dtpaid`.
- Booleans start with `is`: `ispaid`, `isremoved`, `ispublished`.
- Identifiers: `id` (primary key, `text` from `UID()`), foreign keys `userid`, `productid`.
- `text` instead of `varchar`.
- The same names in different tables (`name`, `body`); short names; whenever possible every table has `id`, `name` and `dtcreated`.

Keep this order of fields:

```sql
CREATE TABLE "public"."tbl_channel_message" (

	-- IDENTIFIERS FIRST
	"id" text NOT NULL,
	"userid" text,
	"channelid" text,

	-- MAIN FIELDS
	"body" text,

	-- NUMBERS
	"countupdate" INT2 DEFAULT 0,
	"countviews" INT2 DEFAULT 0,

	-- BOOLEANS
	"ispinned" bool DEFAULT FALSE,
	"isremoved" bool DEFAULT FALSE,

	-- DATES LAST
	"dtupdated" timestamp,
	"dtremoved" timestamp,
	"dtcreated" timestamp DEFAULT timezone('utc'::text, now()),

	-- FOREIGN KEYS
	CONSTRAINT "tbl_channel_message_userid_fkey" FOREIGN KEY ("userid") REFERENCES "public"."tbl_user"("id") ON DELETE CASCADE,

	PRIMARY KEY ("id")
);
```
