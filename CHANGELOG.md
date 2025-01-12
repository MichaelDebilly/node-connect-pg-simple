# Changelog

## 7.0.0 (2021-09-06)

* **Breaking change:** Now requires at least Node.js 12.x
* **Internal:** Updated some developer dependencies and test targets
* As well as all changes in `7.0.0-0`

## 7.0.0-0 (2021-01-18)

* **Possibly breaking change:** Align session expiration logic with [connect-redis](https://github.com/tj/connect-redis/blob/30efd159103ace270c844a5967428b43e7b8ba4a/migration-to-v4.md#changes-to-ttl-management) which in turned aligned with connect-mongo. Fixes #54.
* **Minor breaking change:** The `.close()` method is now async and returns a `Promise` that will be resolved when this session store is fully shut down. Fixes #183.
* **Minor breaking change:** Now requires Node version `^10.17.0 || >=11.14.0`
* **Feature:** New option: `disableTouch`. Disables updating of TTL in database on touch. Fixes #55.
* **Feature:** New option: `createTableIfMissing`. When set, the session table will be automatically created if missing. Fixes #158 and #173. Thanks @aadeshmisra!
* **Tweak**: Slightly tweaked the pg-promise integration. Fixes #153.
* **Tweak**: Introduced a new internal `_asyncQuery()` function in a move to modernize internal code on top of Promise / async / await.

## 6.2.1 (2020-08-19)

* **Fix:** Regression, query errors wasn't properly forwarded. Fixes #180 and #179. Thanks @alxndrsn! (5c324ac)
* **Test:** Added test for above regression (fd36978)
* **Change:** Improved types + error return values (f73ea0d 68a2242)
* **Change:** Updated SECURITY.md to delegate security reports to Tidelift, and thus ensure quicker responses (7683d40 59c7fbc)

## 6.2.0 (2020-08-06)

* **Important fix**: Bump pg to 8.2.1 to support node 14+
* **Change**: Change default prine interval to 15 mins
* **Test**: Add Node 14 to GitHub CI
* **Test**: Added more types and type linting

## 6.1.0 (2019-12-29)

* **Feature**: Prune intervals are now by default randomized between 50% and 150% of the set prune value, making the average prune interval be the same as before, but makes database load more even by making it unlikely for eg. many instances to all prune at once.
* **Feature**: New option `pruneSessionRandomizedInterval` enables deactivation + customization of the new random prune interval feature.
* **Change**: Default prune interval is now `5` minutes, rather than `1` minute. No need to clean extremely often. Will probably make even longer eventually, but a more drastic change could be kind of a breaking change. Please comment in #162 with feedback on future default.
* **Performance**: The database schema definition now specifies an index on the `expire` column. You have to **add this yourself** if you have already set up this module. The change is purely for enhancing performance and can be skipped if no performance issues have been experiences. It is recommended to apply it though.

## 6.0.1 (2019-08-21)

* **Very minor security fix**: `schemaName` and `tableName` wasn't escaped. If any of the two contained a string with a double quote in it, then that would enable an SQL injection. This was previously a feature of `tableName`, before the introduction of a separate `schemaName`, as that allowed a schema to be defined as part of `tableName`. Defining schema name through `tableName` is still supported after this fix, but is now *deprecated*.
* **Fix**: Errors wasn't propagated properly. Fixed in #150. Thanks @bobnil!

## 6.0.0 (2019-07-28)

* **Breaking change**: Now requires at least Node.js 10.x, this as Node.js 8.x [only have a short time left in its LTS](https://github.com/nodejs/Release)
* **Breaking change**: This project now uses [`INSERT ... ON CONFLICT`](https://www.postgresql.org/docs/current/sql-insert.html#SQL-ON-CONFLICT), more popularly known as `UPSERT`. This is only supported on PostgreSQL version 9.5 and above.
* Listen on pool errors. Fixes #29

## 5.0.0 (2018-06-06)

* **Breaking change**: Now requires at least Node.js 8.x (this as Node.js 6.x [only have a short time left in its LTS](https://github.com/nodejs/Release) and I rather don't bump the major version more often than I have to)
* **Breaking change**: Now expects [pg](https://www.npmjs.com/package/pg) 7.x to be used
* **Fix**: Connection string is now handled by [pg](https://www.npmjs.com/package/pg) instead of by this module. Should improve support for things like ssl.

## 4.2.1 (2017-08-20)

* **Fix**: The pruning timer will no longer keep Node alive, it's been given the [`unref()`](https://nodejs.org/api/timers.html#timers_timeout_unref) treatment

## 4.2.0 (2017-05-20)

* **Feature**: New option `pgPromise` enables the library to re-use an existing connection from [pg-promise](https://github.com/vitaly-t/pg-promise). This is a mutually-exclusive alternative to specifying `pool`, `conObject`, or `conString` (only one of these can be provided).

## 4.1.0 (2017-05-19)

* **Feature**: New option `conObject` enables connection details to be set through an object
* **Improvement**: Hardening of `conString` parsing + some added tests of it

## 4.0.0 (2017-05-19)

* **Breaking change + improved support**: When the [pg](https://www.npmjs.com/package/pg) module is provided to this module, then a pool from the new `6.x` version of that module is now required rather than providing the module itself

## 3.1.2

* **Bug fix**: Previous timestamp fix failed critically, fixing it again. Thanks @G3z and @eemeli

## 3.1.1

* **Bug fix**: The internal query helper was treating params() wrong when called with two argument. Thanks for reporting @colideum!
* **Bug fix**: If the database and the node instances had different clocks, then things wouldn't work that well due to mixed timestamp sources. Now node handles all timestamps. Thanks for reporting @sverkoye!

## 3.1.0

* **Feature**: Support the `store.touch()` method to allow for extending the life time of a session without changing the data of it. This enables setting the `resave` option to `false`, which is recommended to avoid a session extender save overwriting another save that adds new data to the session. More info in the [express-session readme](https://github.com/expressjs/session#resave).
* **Fix**: Relax the engine requirements – accept newer versions of Node.js/iojs as well

## 3.0.2

* **Fix**: Added support for [sails](http://sailsjs.org/) by supporting sending the full Express 3 object into the plugin

## 3.0.1

* **Fix**: If the `pg` instance used is created by this module, then this module should also close it on `close()`

## 3.0.0

* **Improvement**: Rather than randomly cleaning up expired sessions that will now happen at the `options.pruneSessionInterval` defined interval.
* **Breaking change**: Clients now need to close the session store to gracefully shut down their app as the pruning of sessions can't know when the rest of the app has stopped running and thus can't know when to stop pruning sessions if it itsn't told so explicitly through thew new `close()` method – or by deactivating the automatic pruning by settinging `options.pruneSessionInterval` to `false`. If automatic pruning is disabled the client needs to call `pruneSessions()` manually or otherwise ensure that old sessions are pruned.

## 2.3.0

* **Fix regression**: No longer default to `public` schema, as added in `2.2.0`, but rather default to the pre-`2.2.0` behavior of no defined schema. This to ensure backwards compatibility with the `2.x` branch, per semantic versioning best practise.

## 2.2.1

* **Hotfix**: Update `require('pg')` to match package.json, thanks for reporting @dmitriiabramov

## 2.2.0

* **New**: Now possibly to set another schema than the default
* **Change**: Now using the `pg` dependency again rather than `pg.js` as the latter will be discontinued as `pg` now fills its role

## 2.1.1

* Fix bug with creating new sessions that was caused by 2.1.0

## 2.1.0

* Enable the table name to be configured through new `tableName` option

## 2.0.0

* **Backwards incompatible change**: Support for Express 4 means that Express 3 apps (and similar for Connect apps) should send `express.session` to the module rather than just `express`.
* **Dependency change**: The database module is now [pg.js](https://www.npmjs.org/package/pg.js) rather than [pg](https://www.npmjs.org/package/pg) – same library, but without compilation of any native bindings and thus less delay when eg. installing the application from scratch.

## 1.0.2

* Support for PostgreSQL versions older than 9.2

## 1.0.1

* Fix for sometimes not expiring sessions correctly

## 1.0.0

* First NPM-version of the script originally published as a Gist here: https://gist.github.com/voxpelli/6447728
