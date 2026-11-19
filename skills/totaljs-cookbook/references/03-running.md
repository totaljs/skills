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
| `node index.js` | debug | port **8000**, the watcher recompiles changed JavaScript (controllers, actions, modules, definitions); views are read fresh on every request |
| `node index.js release` | release | views are cached, HTML is minified |

- In debug mode you **don't need to restart** the server after editing files — the watcher does it on its own (see "After a change" below). When `config` changes, the watcher restarts the whole process.
- A numeric argument sets the port: `node index.js 8001`.
- The debug runner creates `index.js.json`, `index.js.map` and `index.pid` in the project root → they belong in `.gitignore`.
- In production the app can listen on a unix socket: `options.unixsocket = '/path/app.socket'; options.unixsocket777 = true;`

### After a change: what is reloaded and how to know

The watcher of the debug mode treats two kinds of files differently:

| What changed | What happens | In the log |
|---|---|---|
| JavaScript that must be recompiled: `controllers/`, `actions/`, `schemas/`, `modules/`, `definitions/`, `plugins/` | the application is restarted with the new code | a line `--- UPD --- [ 2026-01-15 10:20:44 ] --------> /actions/items.js` |
| A view (`views/`) | nothing is restarted, the next request renders the new view | nothing |
| A static file (`public/`) | served as it is on the next request | nothing |
| `config` | the whole process is restarted | the start banner again |

The log is the output of the debug process: the terminal it runs in, or the file the output is written to (for example `logs/debug.log`).

- **After editing JavaScript, wait for its `UPD` line** and for the port to answer again before testing. The restart usually takes a few seconds, but a busy watcher can take much longer, so a fixed pause is a guess: a test that runs too early silently checks the old code and "passes" or "fails" for the wrong reason.
- **After editing only a view or a static file, test at once.** There will be no line in the log to wait for.
- Save a real change: the watcher looks for changed content, and `touch` alone may not be noticed.
- A restart empties everything kept in the memory of the process: `MAIN`, `CACHE`, the counters of `BLOCKED()` and cached sessions. A test of a rate limit has to finish between two restarts.
- `ON('ready')` runs again after every restart, so a handler of it must be safe to repeat.

```bash
# before the edit: how many times the log already mentions the file
count=$(grep -c '/actions/items.js' logs/debug.log)

# ... edit actions/items.js ...

# wait until the watcher picked up this change and the application answers again
until [ "$(grep -c '/actions/items.js' logs/debug.log)" -gt "$count" ]; do sleep 2; done
until curl -s -o /dev/null http://localhost:8000/; do sleep 2; done
```

> **Gotcha:** don't start Total.js via `node -e "require('total5'); Total.run()"`. The `flow-flowstream.js` module reads `process.argv[1]`, which doesn't exist with `-e`, and the app crashes. Always run a file.

Recommended `.gitignore`:

```
index.js.json
index.js.map
index.pid
tmp/
public/download/
```
