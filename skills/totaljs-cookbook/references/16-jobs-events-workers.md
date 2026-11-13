# 16. Periodic tasks, cron, events and workers

```js
ON('ready + service', function(counter) {
	// Every hour (service counter increments every minute)
	if (!counter || counter % 60 === 0) {
		badges();
		ACTION('Items|refresh').callback(ERROR('Source offline'));
	}
});
```

- `ready` fires once after startup (`counter` is `undefined` then), `service` fires **every minute** with an increasing `counter`.
- `ON('ready + service', …)` = one handler for both events.
- Run time-dependent tasks (e.g. expiring the "NEW" badge) **even when the data source fails** — otherwise stale values stay in place.

### `ON('ready')`

Fires when the app is loaded — a safe place to warm caches or start optional connections. When it is registered after boot, the handler is called immediately.

### `ON('service')` for light maintenance

```js
ON('service', function(counter) {
	if (counter % 5 === 0)
		FUNC.refresh_storage_usage();
	if (counter % 30 === 0)
		FUNC.remove_expired_sessions();
});
```

Good uses: expiring sessions, refreshing counters, light cleanup. Keep it fast.

### `CRON()`

For wall-clock schedules. Standard 5-field cron; the returned object has `.remove()`.

```js
CRON('*/5 * * * *', function() {
	FUNC.run_escalations();
});

CRON('0 3 * * *', async function() {
	await FUNC.purge_tmp_files();
});
```

**Never use `setInterval` job runners or `node-cron`.** `ON('service')` = "every N minutes since start", `CRON()` = "at this time". Both belong in `definitions/jobs.js`.

### A job queue without workers

Most backends do not need workers. A queue table + `CRON()` is enough:

```js
FUNC.process_queue = async function(limit) {
	var jobs = await DATA.find('tbl_job').where('status', 'queued').sort('dtcreated').take(limit || 10).promise();
	for (var i = 0; i < jobs.length; i++) {
		try {
			await FUNC.process_job(jobs[i]);
		} catch (e) {
			Total.error(e, 'job ' + jobs[i].id);
		}
	}
};

CRON('* * * * *', function() {
	FUNC.process_queue(15);
});
```

### Workers

Only for isolated heavy work (image processing, big imports). The code lives in `/workers/<name>.js`:

```js
var worker = NEWTHREAD('resize');        // worker_threads, file: workers/resize.js
var fork = NEWFORK('import');            // child_process.fork
var pool = NEWTHREADPOOL('resize', 4);
```

`NEWTHREAD()` / `NEWFORK()` without a name attach to the parent from inside the worker script.

### Events

```js
ON('orders.create', function(order) { ... });
EMIT('orders.create', order);
```

Use events for decoupled side effects (notify, realtime push, audit). Do not replace actions with an event bus.
