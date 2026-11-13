# 23. Frontend without a framework

### Live search on top of server-rendered HTML

The server renders results into `<div id="results">`. The script downloads the whole page and swaps only this block — a single source of truth, and it also works without JS (a plain GET form).

```js
var load = function(url) {
	ctrl && ctrl.abort();
	ctrl = new AbortController();
	fetch(url, { signal: ctrl.signal }).then(r => r.text()).then(function(html) {
		var parsed = new DOMParser().parseFromString(html, 'text/html');
		var results = parsed.getElementById('results');
		if (results) {
			doc.getElementById('results').replaceWith(results);
			doc.title = parsed.title;
			history.replaceState(null, '', url);
		}
	});
};
```

160 ms debounce; the `/` key focuses the search field.

### Copy to clipboard (Safari included)

Safari allows an asynchronous clipboard write only via `ClipboardItem` with a Promise (otherwise it loses the "user gesture"):

```js
if (clipboard.write && window.ClipboardItem)
	copy = clipboard.write([new ClipboardItem({ 'text/plain': source().then(t => new Blob([t], { type: 'text/plain' })) })]);
else
	copy = source().then(t => clipboard.writeText(t));
```

The content is downloaded only after the click — the page doesn't carry tens of kB of code for nothing.

### Live preview in an `<iframe>`

```html
<iframe src="/items/my-item/preview/" loading="lazy"
	sandbox="allow-scripts allow-same-origin allow-forms allow-modals allow-popups allow-downloads"></iframe>
```

Useful for showing a runnable HTML example next to its documentation. The server can insert a small `<style>` (e.g. padding) before `</head>` of the embedded HTML. For full isolation serve previews from a different subdomain.

### Mobile menu

A `☰` button with `aria-expanded`/`aria-controls`, a panel under the header, closing via Escape, a click outside, a click on a link, and `matchMedia` when switching to desktop. Links are 48 px tall (touch targets).

### Favicons from the SVG logo

- `favicon.svg` with a style for `prefers-color-scheme: dark`,
- `favicon.ico` (16/32/48), `apple-touch-icon.png` (180, **opaque background** — iOS fills transparency with black), `icon-192/512.png` + `site.webmanifest`,
- rasterized with Pillow using supersampling (draw 16× larger, then `LANCZOS`).

```html
<link rel="icon" href="/favicon.ico" sizes="32x32" />
<link rel="icon" href="/favicon.svg" type="image/svg+xml" />
<link rel="apple-touch-icon" href="/apple-touch-icon.png" />
<link rel="manifest" href="/site.webmanifest" />
```

### Color contrast

A light accent (`#FF7B00`) on a light background has a contrast of ~2.4 : 1 → introduce a darker token for small text (`--accent-ink: #AD5000`, ≥ 4.5 : 1) and keep the light one for large elements, icons and graphics.
