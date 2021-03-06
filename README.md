# IndexedDB Polyfill

[![Build Status](https://img.shields.io/travis/axemclion/IndexedDBShim.svg)](https://travis-ci.org/axemclion/IndexedDBShim)
[![Dependencies](https://img.shields.io/david/axemclion/indexeddbshim.svg)](https://david-dm.org/axemclion/indexeddbshim)
[![devDependencies](https://img.shields.io/david/dev/axemclion/indexeddbshim.svg)](https://david-dm.org/axemclion/indexeddbshim?type=dev)
[![npm](http://img.shields.io/npm/v/indexeddbshim.svg)](https://www.npmjs.com/package/indexeddbshim)
[![Bower](http://img.shields.io/bower/v/IndexedDBShim.svg)](http://bower.io/search/?q=IndexedDBShim)
[![License](https://img.shields.io/npm/l/indexeddbshim.svg)](LICENSE-APACHE)

|[Live Demo!](http://nparashuram.com/IndexedDBShim/tests/index.html?useShim=true)
|------------------------------------------------------------

__Use a single, indexable, offline storage API across all desktop and mobile
browsers and Node.js.__

Even if a browser natively supports IndexedDB, you may still want to use this
shim.  Some native IndexedDB implemenatations are [very buggy](http://www.raymondcamden.com/2014/9/25/IndexedDB-on-iOS-8--Broken-Bad).
Others are [missing certain features](http://codepen.io/cemerick/pen/Itymi).
There are also many minor inconsistencies between different browser
implementations of IndexedDB, such as how errors are handled, how transaction
timing works, how records are sorted, how cursors behave, etc.  Using this
shim will ensure consistent behavior across all browsers.

## Features

- Adds full IndexedDB support to any web browser that [supports WebSQL](http://caniuse.com/#search=websql)
- Does nothing if the browser already [natively supports IndexedDB](http://caniuse.com/#search=indexeddb)
- Can _optionally replace_ native IndexedDB on browsers with [buggy implementations](http://www.raymondcamden.com/2014/09/25/IndexedDB-on-iOS-8-Broken-Bad/)
- Can _optionally enhance_ native IndexedDB on browsers that are [missing certain features](http://codepen.io/cemerick/pen/Itymi)
- Works on __desktop__ and __mobile__ devices as well as __Node.js__
- Works on __Cordova__ and __PhoneGap__ via the [IndexedDB plug-in](http://plugins.cordova.io/#/package/com.msopentech.websql)
- This shim is basically an IndexedDB-to-WebSQL adapter.
- Can be used in Node (courtesy of [websql](https://www.npmjs.com/package/websql)
    which sits on top of SQLite3)
- More details about the project at [gh-pages](http://nparashuram.com/IndexedDBShim)

## Installation

You can download the [development](https://raw.githubusercontent.com/axemclion/IndexedDBShim/master/dist/indexeddbshim.js)
or [production (minified)](https://raw.githubusercontent.com/axemclion/IndexedDBShim/master/dist/indexeddbshim.min.js) script, or install it using [NPM](https://docs.npmjs.com/getting-started/what-is-npm)
or [Bower](http://bower.io/).

Please note that the version currently in `master` is the only one which
supports Node.js (and has a number of fixes), but we are not yet ready
for release.

### Bower

````bash
bower install IndexedDBShim
````

### npm

```bash
npm install indexeddbshim
```

## Browser set-up

Add the following script to your page:

```html
<script src="dist/indexeddbshim.min.js"></script>
```

If you need full Unicode compliance (handling special
non-alphanumeric identifiers in store and index names),
use the following instead:

```html
<script src="dist/indexeddbshim-UnicodeIdentifiers.min.js"></script>
```
## Node set-up

```js
const setGlobalVars = require('indexeddbshim');
GLOBAL.window = GLOBAL; // We'll allow ourselves to use `window.indexedDB` or `indexedDB` as a global
setGlobalVars(); // See signature below
```

## Usage/API

For the browser scripts, if the browser already natively supports IndexedDB
and is not known to be buggy, then the script won't do anything.

Otherwise, assuming WebSQL is available, the script will add the
[IndexedDB API](https://developer.mozilla.org/en-US/docs/Web/API/IndexedDB_API)
to the browser (unless you use one of the non-invasive files, in which case
`setGlobalVars` can be used to optionally add the API to an object of your
choosing; if you also wish Unicode support, you will need to add it yourself).

Either way, you can use IndexedDB just like normal.
[Here's an example](https://gist.github.com/BigstickCarpet/a0d6389a5d0e3a24814b).

### setGlobalVars(\<winObj or `null`\>, initialConfig)

In the non-invasive builds (and Node.js), globals are not automatically set.
You have the choice to set globals when you wish as well as to set the API
on an object of your choosing in place of setting globals.

This is done through `setGlobalVars()` (which is otherwise called in the
browser builds automatically with no arguments).

This function defines `shimIndexedDB`, `indexedDB`, `IDBFactory`, etc. on
one of the following objects in order of precedence:

1. The passed in `winObj` object if defined
2. `window` (for Node, define `GLOBAL.window = GLOBAL;`)
3. `self` (for web workers)
4. A new object

The `initialConfig` argument, if present, should be an object whose keys
are the config properties to set and its values are the config values (see
`shimIndexedDB.__setConfig` below).

If you are adding your own `window.openDatabase` implementation, supplying
it within `initialConfig` (keyed as `openDatabase`) will ensure that
`shimIndexedDB.__useShim()` is auto-invoked for you if poor IndexedDB
support is detected.

### shimIndexedDB.\__useShim();

To force IndexedDBShim to shim the browser's native IndexedDB (if our code
is not already auto-shimming your browser when detecting poor browser
support), add this method call to your script.

On browsers that support WebSQL, this line will _completely replace_ the
native IndexedDB implementation with the IndexedDBShim-to-WebSQL
implementation.

On browsers that _don't_ support WebSQL, but _do_ support IndexedDB, this
line will patch many known problems and add missing features. For example,
on Internet Explorer, this will add support for compound keys.

### shimIndexedDB.\__forceClose(connIdx, msg)

The spec anticipates the [closing of a database connection with a forced flag](http://w3c.github.io/IndexedDB/#steps-for-closing-a-database-connection).

The spec also mentions [some circumstances](http://w3c.github.io/IndexedDB/#database-connection)
where this may occur:

> A connection may be closed by a user agent in exceptional circumstances,
> for example due to loss of access to the file system, a permission change,
> or clearing of the origin’s storage.

Since the latter examples are under the browser's control, this method may
be more useful on the server or for unit-testing.

If the first argument, `connIdx` is missing (or `null` or `undefined`),
all connections will be force-closed. It can alternatively be an integer
representing a 0-based index to indicate a specific connection to close.

The second argument `msg` will be appended to the `AbortError` that will be
triggered on the transactions of the connection.

Individual `IDBDatabase` database connections can also be force-closed
with a particular message:

```js
db.__forceClose(msg);
```

### shimIndexedDB.\__debug(boolean)

The IndexedDB polyfill has sourcemaps enabled, so the polyfill can be debugged
even if the minified file is included.

To print out detailed debug messages, add this line to your script:

```js
shimIndexedDB.__debug(true);
```

### shimIndexedDB.\__setConfig()

Rather than using globals, a method has been provided to share state across
IndexedDBShim modules.

Configuration can be set early in the non-invasive browser and Node builds
via the second argument to `setGlobalVars()` (see its definition above).

Its signature (for setting configuration after `shimIndexedDB` is created) is:

```js
shimIndexedDB.__setConfig({property: value, property2: value2, ...});
```

or:

```js
shimIndexedDB.__setConfig(property, value);
```

#### Configuration options

The available properties are:

- __DEBUG__ - Boolean (equivalent to calling `shimIndexedDB.__debug(val)`)
- __cursorPreloadPackSize__ - Number indicating how many records to preload for
    caching of (non-multiEntry) `IDBCursor.continue` calls. Defaults to 100.
- __UnicodeIDStart__ and __UnicodeIDContinue__ - Invocation of
    `createObjectStore` and `createIndex` calls for validation of key paths.
    The specification technically allows all
    `IdentifierName`](https://tc39.github.io/ecma262/#prod-IdentifierName)
    strings, but as this requires a [very large regular expression](https://gist.github.com/brettz9/b4cd6821d990daa023b2e604de371407),
    it is replaced by default with `[$A-Z_a-z]` and `[$0-9A-Z_a-z]`,
    respectively. Note that these are and must be expressed as strings,
    not `RegExp` objects. You can use this configuration to change the default
    to match the spec or as you see fit. In the future we may allow the spec
    behavior via optional dynamic loading of an internal module.
- __win__,  Object on which there may be an `openDatabase` method (if any)
    for WebSQL; Defaults to `window` or `self` in the browser and for Node,
    it is set by default to [`node-websql`](https://github.com/nolanlawson/node-websql).
- __DEFAULT_DB_SIZE__ - Used as estimated size argument (in bytes) to
    underlying WebSQL `openDatabase` calls. Defaults to `4 * 1024 * 1024` or
    `25 * 1024 * 1024` in Safari (apparently necessary due to Safari creating
    larger files and possibly also due to Safari not completing the storage
    of all records even after permission is given). Has no effect in Node (using [`node-websql`](https://github.com/nolanlawson/node-websql)),
    and its use in WebSQL-compliant browsers is implementation dependent (the
    browser may use this information to suggest the use of this quota to the
    user rather than prompting the user regularly for say incremental 5MB
    permissions).
- __sqlBusyTimeout__ - Integer used by Node WebSQL for
    [SQLite config](https://github.com/mapbox/node-sqlite3/wiki/API#databaseconfigureoption-value)
    to set the [busy timeout](https://www.sqlite.org/c3ref/busy_timeout.html)
    (Defaults to 1000 ms)
- __sqlTrace__ - Callback used by Node WebSQL for
    [SQLite config](https://github.com/mapbox/node-sqlite3/wiki/API#databaseconfigureoption-value)
    (Invoked when an SQL statement executes, with a rendering of the statement text)
- __sqlProfile__ - Callback used by Node WebSQL for
    [SQLite config](https://github.com/mapbox/node-sqlite3/wiki/API#databaseconfigureoption-value)
    (Invoked every time an SQL statement executes)

### shimIndexedDB.\__getConfig()

For retrieving a config value:

```js
shimIndexedDB.__getConfig(property);
```

## Known Issues

All code has bugs, and this project is no exception.  If you find a bug,
please [let us know about it](https://github.com/axemclion/IndexedDBShim/issues).
Or better yet, [send us a fix](https://github.com/axemclion/IndexedDBShim/pulls)!
Please make sure someone else hasn't already reported the same bug though.

Here is a summary of known issues to resolve:

1. `blocked` and `versionchange` `IDBVersionChangeEvent` event support
2. Structured Cloning Algorithm
    1. `eval` is currently in use (in `src/Sca.js`)
    1. Support cyclic objects
3. Add new Binary/`ArrayBuffer`/Views on buffers (`TypedArray` or `DataView`)
    support

There are a few bugs that are outside of our power to fix.  Namely:

### Browser rollback

While we do try to rollback the database version in the browser when
called for, we are not able to prolong WebSQL transactions so as to
be able to benefit from the auto-rollback they perform upon encountering
an error nor does WebSQL permit manual ROLLBACK commands so that we
could undo the various WebSQL calls we need to make up IndexedDB
transactions.

The special build of `websql` that we use does allow such spec-compliant
(and data-integrity-friendly!) rollback behavior.

### iOS

Due to a [bug in WebKit](https://bugs.webkit.org/show_bug.cgi?id=137034), the
`window.indexedDB` property is read-only and cannot be overridden by
IndexedDBShim.  There are two possible workarounds for this:

1. Use `window.shimIndexedDB` instead of `window.indexedDB`
2. Create an `indexedDB` variable in your closure

By creating a variable named `indexedDB`, all the code within that closure
will use the variable instead of the `window.indexedDB` property.  For example:

```js
(function() {
    // This works on all browsers, and only uses IndexedDBShim as a final fallback
    var indexedDB = window.indexedDB || window.mozIndexedDB || window.webkitIndexedDB || window.msIndexedDB || window.shimIndexedDB;

    // This code will use the native IndexedDB, if it exists, or the shim otherwise
    indexedDB.open("MyDatabase", 1);
})();
```

### Windows Phone

IndexedDBShim works on Windows Phone via a Cordova/PhoneGap plug-in.  There
are two plugins available: [cordova-plugin-indexedDB](https://github.com/MSOpenTech/cordova-plugin-indexedDB)
and [cordova-plugin-indexeddb-async](https://github.com/ABB-Austin/cordova-plugin-indexeddb-async).
Both plug-ins rely on a WebSQL-to-SQLite adapter, but there are differences
in their implementations.  Try them both and see which one works best for
your app.

## Building

To build the project locally on your computer:

1. __Clone this repo__
If you clone the repository to work against an unstable version, you only
need to clone the repository recursively (via
`git clone https://github.com/axemclion/IndexedDBShim.git --recursive`)
if you wish to have the W3C tests available for testing (which
unfortunately loads all W3C tests into the "web-platform-tests"
subdirectory rather than just the IndexedDB ones). Otherwise, just use
`git clone https://github.com/axemclion/IndexedDBShim.git`

2. __Install dev dependencies (and websql for Node)__
`npm install`

3. __Run the build script__
`npm start`

4. __Done__

The output files will be generated in the `dist` directory

## Testing

There are currently three folders for tests, `tests-qunit`,
`tests-mocha` and `tests-polyfill` (the latter are also Mocha-based
tests, but at present its W3C tests
[only work in Node](https://github.com/axemclion/IndexedDBShim/issues/249)).

They can be run through a variety of means as described below.

To properly build the files (lint, browserify, and minify), use `npm start`
or to also keep a web server, run `npm run dev` (or `grunt dev`). If
you wish to do testing which only rebuilds the browser files, run
`npm run dev-browser` and if only testing Node, run `npm run dev-node`.
But before release, one should run `npm run build` (or `npm run dev`).

The tests produce various database files. These are avoided in
`.gitignore` and should be cleaned up if the tests pass, but if
you wish to delete them all manually, run `npm run clean`.

### Browser testing

All QUnit-based tests should pass in modern browsers.

All Mocha-based browser tests should pass except for one test having
a [problem in Firefox](https://github.com/axemclion/IndexedDBShim/issues/250).

#### Automated browser unit testing

Follow all of the steps above to build the project, then run `npm test`
or `npm run sauce-qunit` (or `npm run phantom-qunit` or
`grunt phantom-qunit` to avoid using Saucelabs when you have
credentials set up as environmental variables) to run the unit tests.

Note that when not running Saucelabs, the tests are run in
[PhantomJS](http://phantomjs.org/), which is a headless WebKit browser.

The older PhantomJS version has problems with two tests, however:
`index.openCursor(range)` and
`IDBObjectStore.openKeyCursor` due apparently to
a bug with the WebKit browser used in the older PhantomJS implementation
(but the tests themselves report as having such problems).

#### Manual browser testing

If you want to run the tests in a normal web browser, you'll need to
spin-up a local web server and then open
[`tests-qunit/index.html?noglobals`](https://github.com/axemclion/IndexedDBShim/blob/master/tests-qunit/index.html?noglobals)
and/or [`tests-mocha/index.html`](https://github.com/axemclion/IndexedDBShim/blob/master/tests-mocha/index.html)
in your browser. You can also run `npm run dev` and point your
browser to `http://localhost:9999/tests-qunit/index.html` or
`http://localhost:9999/tests-mocha/index.html`.

Note that, for the Mocha tests, you probably wish to
"Switch to IndexedDBShim" when doing
the testing since otherwise, it will only test the native implementation.

### Node Testing

To run the Node tests, run the following:

1. `npm run node-qunit` - The full test suite sometimes
    [does not complete execution](https://github.com/axemclion/IndexedDBShim/issues/251).
2. `npm run mocha`
3. `npm run tests-polyfill` (or its components `npm run fake`,
    `npm run mock`, `npm run w3c-old`). Note that only `fake` is
    currently passing in full, however.
4. `npm run w3c` (you must first run
    `git submodule update --init --recursive` if you have not already
    recursively cloned the repository). Note that some of these tests
    may not be passing because of the test environment not being
    completely configured for Node. We are working on fixing this.
    There are some older and less complete W3C tests that can be run
    with `npm run w3c-old`, but the goal is to remove these once
    the new ones are configured properly.

To run a specific Mocha test (which includes the `tests-polyfill`
tests), run `npm --test=... run mocha`.

### Testing in a Cordova/PhoneGap app

If you want to run the tests in a Cordova or PhoneGap app, then you'll need
to create a new Cordova/PhoneGap project, and add the
[IndexedDB plug-in](http://plugins.cordova.io/#/package/com.msopentech.indexeddb).
Then copy the contents of our
[`tests`](https://github.com/axemclion/IndexedDBShim/tree/master/tests)
directory into your project's `www` directory.   Delete our
[`index.html`](https://github.com/axemclion/IndexedDBShim/blob/master/tests/index.html)
file and rename
[`cordova.html`](https://github.com/axemclion/IndexedDBShim/blob/master/tests/cordova.html)
to `index.html`.

## Contributing

Pull requests or Bug reports welcome!!
