# 24. Performance

- **Compute when data changes, not when rendering.** For example, a "NEW/UPDATED" badge is stored in `m.badge` by the action (after startup, after a refresh, once an hour); the view only renders it.
- **Cache expensive renders** with a key based on the content:

  ```js
  var stamp = isodate(model.updated) + (model.readme || '').length;
  if (cache && cache.stamp === stamp)
  	return cache.html;
  ```

- **Images** from data URIs are written to disk only when their SHA1 checksum changes.
- **Library scripts** in `<head>` get `defer` so they don't block rendering.
- **The list action** returns only the needed properties (`keys`), not whole objects with readme and code.
- **Search:** matches in the name come first, matches in the description follow.

### With a database

- **Filter in the query, never in JavaScript.** Do not load a table to `.filter()` it:

  ```js
  // WRONG
  var all = await DATA.find('tbl_order').promise();
  var mine = all.filter(m => m.userid === id);

  // RIGHT
  var mine = await DATA.find('tbl_order').where('userid', id).where('isremoved', false).promise($);
  ```

- **No N+1 loops.** One `DATA.find().in('id', ids)` instead of a `DATA.read()` per id.
- **Select only what you return:** `.fields('id,name')`, or the field list of `autoquery()`.
- **Lists are always paginated** (`DATA.list` + a max. limit).
- **Denormalized lists** are SQL views (`view_order`), not joins assembled in JavaScript.
- **Cache hot, rarely changing rows** (session snapshots) in `MAIN` / `CACHE`, and invalidate on update.
