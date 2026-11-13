# 21. Useful prototypes and utilities

| API | Example / purpose |
|---|---|
| `String.slug()` | `'DataGrid'.slug()` → `datagrid` (ID for URLs) |
| `String.toSearch()` | normalization for search (lowercase, no diacritics, `y`→`i`, no doubled letters) |
| `String.parseJSON(true)` | safe parsing; `true` = convert dates |
| `String.parseDataURI()` | `{ buffer, type }` from a data URI (images in JSON) |
| `String.encode()` / `.safehtml()` | HTML escaping |
| `String.markdown(opt)` | Markdown → HTML on the server (`{ wrap: false }`) |
| `String.makeid()` | short hash of a string (cache key) |
| `String.format(…)` | `'/items/{0}/'.format(id)` |
| `Number.filesize()` | `41161 .filesize()` → `40.19 KB` |
| `Date.format(f)` | `dt.format('d MMM yyyy')` |
| `Array.quicksort('updated_desc')` | fast sorting by a property |
| `Array.take(n)` | the first `n` items |
| `Array.findItem('id', value)` | find an object |
| `Array.wait(fn, done, threads)` | asynchronous loop with parallelism |
| `Utils.paginate(page, pages, max)` | `{ page, pages[], prev, next, visible, … }` for pagination |
| `Tangular.compile(tpl)` | fast template (`{{ name }}`, filters) |
| `PROMISIFY(fn, a, b)` | callback API → Promise (`PROMISIFY(Total.Fs.rename, a, b)`) |
| `Total.readfile(path, 'utf8')` | reading a file as a Promise |
| `Total.Fs`, `Total.Crypto`, `Total.Zlib` | Node.js `fs`, `crypto`, `zlib` |
| `Total.error(err, 'label')` | logging an error |

### Markdown on the server

```js
var html = readme
	.replace(/^\s*#\s+[^\n]*\n*/, '')     // the first headline duplicates <h1> of the page
	.markdown({ wrap: false })
	.replace(/<(\/?)h1/g, '<$1h2');       // the page can have only one <h1>
```

The renderer itself adds `loading="lazy"` to images and the `hidden` class to code blocks — account for both (don't add `loading` a second time, remove `hidden`).
