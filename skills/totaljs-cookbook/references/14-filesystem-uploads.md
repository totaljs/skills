# 14. Filesystem, PATH and uploads

Total.js already wraps Node's filesystem and path APIs. Application code does not `require('fs')` or `require('path')`.

### `PATH`

Each helper returns the directory, or joins a relative piece onto it:

```js
PATH.root()                 PATH.root('package.json')
PATH.public()               PATH.public('uploads/' + id + '.jpg')
PATH.private()
PATH.databases()            PATH.databases('items.json')
PATH.logs()                 PATH.logs('audit.log')
PATH.temp()                 PATH.temp('export.csv')      // alias: PATH.tmp()
PATH.modules()   PATH.plugins()   PATH.templates()
```

```js
PATH.join(a, b, c);
PATH.exists(filename, function(exists, size, isfile) { ... });   // without a callback → Promise
PATH.mkdir(directory);           // recursive, sync
PATH.unlink(file_or_array);
PATH.rmdir(dir_or_array);
PATH.verify(PATH.temp('exports'));   // creates the directory when missing (remembered)

PATH.fs.readFileSync(PATH.temp(name));   // PATH.fs === Total.Fs === node:fs
await Total.readfile(filename, 'utf8');
```

> **`PATH.verify()` is not a security check.** It only ensures a directory exists. It does not sanitize `../` or user-controlled segments.

### Paths from request data

1. Generate the file name yourself (`UID()` + an allowed extension).
2. Join it onto a known root (`PATH.public('uploads')`, `PATH.temp()`).
3. Reject absolute input and `..`.
4. Then write.

```js
var ext = U.getExtension(file.filename).toLowerCase();
if (!['png', 'jpg', 'jpeg', 'webp', 'pdf'].includes(ext)) {
	$.invalid('@(Unsupported file type)');
	return;
}
var filename = PATH.public('uploads/' + UID() + '.' + ext);
```

### Node built-ins on `Total`

```js
Total.Fs        // node:fs        also PATH.fs
Total.Path      // node:path      also PATH.join
Total.Http      Total.Https
Total.Crypto    Total.Zlib      Total.Stream
Total.Url       Total.Os        Total.Util      // node:util — Total.js helpers are U / Utils
Total.Net       Total.Dns       Total.Tls
Total.Child     // child_process
Total.Worker    // worker_threads
```

Prefer `PATH.*`, `HASH()` and `RESTBuilder` over reaching for these. Use `Total.Https` only when `RESTBuilder` / `REQUEST` cannot express the call.

### Upload route

A multipart upload is a normal HTTP route, not an API envelope:

```js
exports.install = function() {
	ROUTE('+POST /upload/ @upload <10MB', upload);
	ROUTE('FILE /download/*.*', download);
};

async function upload($) {

	var file = $.files[0];
	if (!file) {
		$.invalid('@(No file uploaded)');
		return;
	}

	if (!FUNC.require_perm($, 'files_upload'))
		return;

	var id = UID();
	var meta = await file.fs('files', id);     // stores the bytes in FILESTORAGE('files')

	await DATA.insert('tbl_file', {
		id: id,
		userid: $.user.id,
		name: file.filename,
		type: file.type,
		size: file.size,
		dtcreated: NOW
	}).promise($);

	$.success({ id: id, name: file.filename, url: '/download/' + id.sign(CONF.salt) + '.' + meta.ext });
}
```

An uploaded file has `file.filename`, `file.type`, `file.size`, `file.path` (a temporary path), `file.width` / `file.height` for images, and:

```js
await file.read();                // Buffer
await file.fs('files', id);       // → FILESTORAGE, returns the metadata
await file.move(filename);        // move to a path
await file.copy(filename);
```

The route already requires a session (`+POST`); do not decrypt the token again in the handler.

### FileStorage

`FILESTORAGE(name)` is the framework-native blob storage on disk (`databases/fs-<name>/`). Use it whenever files stay on the server; use a `/modules/storage.js` module (called through `MODS.storage`) for S3 or another object store.

```js
var meta = await file.fs('files', id);    // store an upload
$.filefs('files', id);                    // send it back (controller)
FILESTORAGE('files').remove(id);
```

### Signed download ids

When files are served from a public `FILE` route, sign the id so clients cannot guess keys:

```js
// url: /download/{id.sign(CONF.salt)}.jpg
function download($) {
	var name = $.split[1] || '';
	var index = name.lastIndexOf('.');
	var hash = index === -1 ? name : name.substring(0, index);
	var id = hash.substring(0, hash.indexOf('-', 10));
	if (id && hash === id.sign(CONF.salt))
		$.filefs('files', id);
	else
		$.invalid(404);
}
```

### File rules

- Metadata in the database, bytes in storage.
- Return ids and URLs, never raw server paths.
