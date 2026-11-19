# 25. Robustness and security

### Atomic swap of downloaded data

Never write a downloaded file directly over production data. Download it aside, validate the format, and only then swap:

```js
await DOWNLOAD(url, tmpfile);

items = (await Total.readfile(tmpfile, 'utf8')).parseJSON(true);

// e.g. an old format of the file would break the whole site
if (!(items instanceof Array) || !items.length || !items.every(m => m && typeof(m) === 'object' && m.name && m.preview)) {
	$.invalid('Invalid format of items.json');
	return;
}

await PROMISIFY(Total.Fs.rename, tmpfile, filename);
```

On an error the site stays on the last valid data.

### Cache with a fallback

Remote content (e.g. documentation pulled from a repository) is loaded at startup from a local copy in `databases/` (available immediately) and only then refreshed from the network. If the network fails, the last version stays.

### More principles

- Every value going into HTML/attributes goes through `.encode()`; HTML returned by functions used in `@{!…}` must be escaped inside the function.
- `X-Robots-Tag` for non-public/helper responses.
- Protect public endpoints that trigger expensive operations (e.g. a manual data refresh) with a token, or remove them.
- Isolate third-party code in previews (sandbox, ideally a different origin).

### Backend security checklist

- **Routes:** everything is `+API` / `+POST` unless it is deliberately public; public writes (login, register, forms) are rate-limited with `BLOCKED()`.
- **Input:** every action declares `input` / `query` / `params`; never read undeclared fields from `$.body`.
- **Identity:** `userid`, roles and permissions come from `$.user`, never from the payload.
- **Ownership:** part of the query (`.where('userid', $.user.id)`), with `.error(404)`.
- **SQL:** QueryBuilder first, parameterized `DATA.query(sql, [args])` only where the builder cannot express the query; no string concatenation.
- **Paths:** generated names on known roots; `PATH.verify()` is not a sanitizer ([section 14](14-filesystem-uploads.md)).
- **Uploads:** a size limit on the route (`<10MB`), an extension whitelist, signed download ids.
- **Output:** `FUNC.user_safe()` strips password hashes, reset tokens and secrets; never return raw rows of `tbl_user`.
- **Passwords:** hashed with a salt from `CONF`; tokens are opaque (`ENCRYPTREQ`).
- **Secrets:** in `config` / `(env)` keys, never in code or in responses.
- **Errors:** user-facing errors are localized messages (`@(…)`); details go to the log (`Total.error(err, 'label')`), not to the client.
