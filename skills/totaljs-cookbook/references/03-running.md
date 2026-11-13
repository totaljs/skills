# 3. Running, debug and release

`index.js`:

```js
require('total5');
const options = {};
options.release = process.argv.includes('--release') || process.argv.includes('release');
Total.run(options);
```

| Command | Mode | Note |
|---|---|---|
| `node index.js` | debug | port **8000**, the watcher reloads changed controllers, actions, views and modules |
| `node index.js release` | release | views are cached, HTML is minified |

- In debug mode you **don't need to restart** the server after editing files — the watcher reloads them on its own (the log shows `--- UPD --- /controllers/…`). When `config` changes, the watcher restarts the whole process.
- A numeric argument sets the port: `node index.js 8001`.
- The debug runner creates `index.js.json`, `index.js.map` and `index.pid` in the project root → they belong in `.gitignore`.
- In production the app can listen on a unix socket: `options.unixsocket = '/path/app.socket'; options.unixsocket777 = true;`

> **Gotcha:** don't start Total.js via `node -e "require('total5'); Total.run()"`. The `flow-flowstream.js` module reads `process.argv[1]`, which doesn't exist with `-e`, and the app crashes. Always run a file.

Recommended `.gitignore`:

```
index.js.json
index.js.map
index.pid
tmp/
public/download/
```
