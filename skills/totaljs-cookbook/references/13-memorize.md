# 13. Data without a database: MEMORIZE and REPO

Small and medium data sets (hundreds to thousands of records) don't need a database. The whole catalog lives in memory and is persisted to a JSON file.

`actions/items.js`:

```js
// Default values
REPO.db = MEMORIZE('db');          // databases/memorize_db.json

if (!REPO.db.items)
	REPO.db.items = EMPTYARRAY;

if (!REPO.db.pictures)
	REPO.db.pictures = {};
```

- `MEMORIZE(name)` returns an object loaded from `databases/memorize_<name>.json` and adds methods to it:
  - `.save()` — saves with a **10-second delay** (debounce); multiple calls are merged into one write,
  - `.set(key, value)` — sets a value and calls `save()`.
- `REPO` is the global application object, also available in views (`@{REPO.db.items.length}`).
- `EMPTYARRAY` / `EMPTYOBJECT` are shared empty values of Total.js (never mutate them).

**When it's not enough:** when the data doesn't fit in memory, or when you need transactions or writes from multiple processes.

### `REPO` vs `MAIN`

`REPO` is the bag that views can read, which is why the catalog lives there. Process-only runtime state (sessions, socket maps) belongs on `MAIN`, and nothing durable belongs in `TEMP` ([section 6](06-globals.md)).

Move to PostgreSQL ([section 12](12-postgresql.md)) as soon as the app has user accounts, writes coming from requests, or data that does not fit comfortably in memory.
