# 19. Views and layout

### Syntax

| Notation | Meaning |
|---|---|
| `@{model.name}` | output **with HTML escaping** |
| `@{!html}` | raw output (no escaping) |
| `@{if cond}…@{else}…@{fi}` | condition |
| `@{foreach m in items}…@{end}` | loop; `index` = position |
| `@{section head}…@{end}` | section content (in a view) |
| `@{section('head')}` | section output (in the layout) |
| `@{body}` | view content in the layout |

Available in views: `model`, `query`, `controller` (`controller.url`), `repository`, `REPO`, `CONF`.

### A layout with a section for `<head>`

`views/layout.html`:

```html
<head>
	<meta charset="utf-8" />
	@{section('head')}
	…
</head>
<body>
	<main id="content">
		@{body}
	</main>
</body>
```

A view fills its own `<head>` (title, meta, JSON-LD):

```html
@{section head}@{!repository.func.head(controller, 'detail', model)}@{end}
```

> **Gotcha:** in the layout it's `@{section('head')}`, not `@{!section('head')}` — the latter fails with `ReferenceError: section is not defined`.

### Documenting the model at the top of a view

A comment at the start of a view describes the shape of the model — it helps both people and AI:

```html
<!--
model.page {Number}
model.items {Array Object}
model.pagination {Object}
-->
```

### Styles directly in the layout

All CSS lives in a `<style>` in the layout: no extra request, design tokens in one place.

```css
:root {
	--bg: #F5F4F0; --ink: #15171B; --accent: #FF7B00; --accent-ink: #AD5000;
}
@media (prefers-color-scheme: dark) {
	:root { --bg: #0F1013; --ink: #EEEDE8; --accent-ink: #FF7B00; }
}
```
