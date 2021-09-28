# node-std-browser

[![Build Status][ci-img]][ci]

[Node standard library](https://nodejs.org/docs/latest/api/) for browser.

Features:

-   Based on [`node-libs-browser`](https://github.com/webpack/node-libs-browser)
    for Webpack
-   Maintained with newer versions and modern implementations
-   Works with Webpack and Rollup
-   Exports implementation with
    [`node:` protocol](https://nodejs.org/api/esm.html#esm_node_imports) which
    allows for builtin modules to be referenced by valid absolute URL strings

## Install

```sh
npm install node-std-browser --save-dev
```

## Usage

### Webpack

<details>
	
<summary>Show me</summary>

As of Webpack 5, aliases and globals provider need to be explicitly configured.

```js
// webpack.config.js
const stdBrowser = require('node-std-browser');
const webpack = require('webpack');

module.exports = {
	// ...
	resolve: {
		alias: stdBrowser
	},
	plugins: [
		new webpack.ProvidePlugin({
			process: stdBrowser.process,
			Buffer: stdBrowser.buffer
		})
	]
};
```

Some packages such as `native-url` expose ESM file through `.mjs` extension.
Additional Webpack configuration could be needed to properly handle those
packages.

For example, to make `native-url` use ESM version of `native-querystring`, apply
following configuration:

```js
// webpack.config.js

module.exports = {
	// ...
	module: {
		rules: [
			{
				type: 'javascript/auto',
				test: /\.mjs$/,
				include: /\/native-url\//,
				resolve: {
					mainFields: ['module']
				}
			}
		];
	}
}
```

</details>

### Rollup

<details>
	
<summary>Show me</summary>

Since many packages expose only CommonJS implementation, you need to apply
plugins to handle CommonJS exports. Those packages could have dependencies
installed with npm so they need to be properly resolved (taking into account
browser-specific implementations). Additionally, it’s recommended to handle Node
globals automatically.

Some dependencies can have circular dependencies and Rollup will warn you about
that. You can
[ignore these warnings with `onwarn` function](https://github.com/rollup/rollup/issues/1089#issuecomment-635564942).

```js
// rollup.config.js
const stdBrowser = require('node-std-browser');
const globals = require('rollup-plugin-node-globals');
const { default: resolve } = require('@rollup/plugin-node-resolve');
const commonjs = require('@rollup/plugin-commonjs');
const json = require('@rollup/plugin-json');
const alias = require('@rollup/plugin-alias');

module.exports = {
	// ...
	plugins: [
		alias({
			entries: stdBrowser
		}),
		resolve({
			browser: true
		}),
		commonjs(),
		json(),
		globals()
	],
	onwarn: (warning, rollupWarn) => {
		const packagesWithCircularDependencies = [
			'util/',
			'assert/',
			'readable-stream/',
			'crypto-browserify/'
		];
		if (
			!(
				warning.code === 'CIRCULAR_DEPENDENCY' &&
				packagesWithCircularDependencies.some((modulePath) =>
					warning.importer.includes(modulePath)
				)
			)
		) {
			rollupWarn(warning);
		}
	}
};
```

</details>

## Package contents

| Module                | Browser implementation                                                            | Mock implementation        | Notes                                                |
| --------------------- | --------------------------------------------------------------------------------- | -------------------------- | ---------------------------------------------------- |
| `assert`              | [assert](https://github.com/browserify/commonjs-assert)                           |                            |
| `buffer`              | [buffer](https://github.com/feross/buffer)                                        | [buffer](mock/buffer.js)   |
| `child_process`       |                                                                                   |                            |
| `cluster`             |                                                                                   |                            |
| `console`             | [console-browserify](https://github.com/browserify/console-browserify)            | [console](mock/console.js) |
| `constants`           | [constants-browserify](https://github.com/juliangruber/constants-browserify)      |                            |
| `crypto`              | [crypto-browserify](https://github.com/crypto-browserify/crypto-browserify)       |                            |
| `dgram`               |                                                                                   |                            |
| `dns`                 |                                                                                   | [dns](mock/dns.js)         |
| `domain`              | [domain-browser](https://github.com/bevry/domain-browser)                         |                            |
| `events`              | [events](https://github.com/browserify/events)                                    |                            |
| `fs`                  |                                                                                   |                            |
| `http`                | [stream-http](https://github.com/jhiesey/stream-http)                             |                            |
| `https`               | [https-browserify](https://github.com/substack/https-browserify)                  |                            |
| `module`              |                                                                                   |                            |
| `net`                 |                                                                                   | [net](mock/net.js)         |
| `os`                  | [os-browserify](https://github.com/CoderPuppy/os-browserify)                      |                            |
| `path`                | [path-browserify](https://github.com/browserify/path-browserify)                  |                            |
| `process`             | [process](https://github.com/defunctzombie/node-process)                          | [process](mock/process.js) |
| `punycode`            | [punycode](https://github.com/bestiejs/punycode.js)                               |                            |
| `querystring`         | [native-querystring](https://github.com/niksy/native-querystring)                 |                            |
| `readline`            |                                                                                   |                            |
| `repl`                |                                                                                   |                            |
| `stream`              | [stream-browserify](https://github.com/browserify/stream-browserify)              |                            |
| `string_decoder`      | [string_decoder](https://github.com/nodejs/string_decoder)                        |                            |
| `sys`                 | [util](https://github.com/browserify/node-util)                                   |                            |
| `timers`              | [timers-browserify](https://github.com/browserify/timers-browserify)              |                            |
| `timers/promises`     | [isomorphic-timers-promises](https://github.com/niksy/isomorphic-timers-promises) |                            |
| `tls`                 |                                                                                   | [tls](mock/tls.js)         |
| `tty`                 | [tty-browserify](https://github.com/browserify/tty-browserify)                    | [tty](mock/tty.js)         |
| `url`                 | [native-url](https://github.com/GoogleChromeLabs/native-url)                      |                            | Contains additional exports from newer Node versions |
| `util`                | [util](https://github.com/browserify/node-util)                                   |                            |
| `vm`                  | [vm-browserify](https://github.com/browserify/vm-browserify)                      |                            |
| `zlib`                | [browserify-zlib](https://github.com/browserify/browserify-zlib)                  |                            |
| `_stream_duplex`      | [readable-stream](https://github.com/nodejs/readable-stream)                      |                            |
| `_stream_passthrough` | [readable-stream](https://github.com/nodejs/readable-stream)                      |                            |
| `_stream_readable`    | [readable-stream](https://github.com/nodejs/readable-stream)                      |                            |
| `_stream_transform`   | [readable-stream](https://github.com/nodejs/readable-stream)                      |                            |
| `_stream_writable`    | [readable-stream](https://github.com/nodejs/readable-stream)                      |                            |

## API

### packages

Returns: `object`

Exports absolute paths to each module directory (where `package.json` is
located), keyed by module names. Modules without browser replacements return
module with default export `null`.

Some modules have mocks in the mock directory. These are replacements with
minimal functionality.

## Node support

Minimum supported version should be Node 10.

If you’re using ESM in Node < 12.20, note that
[subpath patterns](https://nodejs.org/api/packages.html#packages_subpath_patterns)
are not supported so mocks can’t be handled. In that case, it’s recommended to
use CommonJS implementation.

## Browser support

Minimum supported version should be Internet Explorer 11, but most modules
support even Internet Explorer 9.

## License

MIT © [Ivan Nikolić](http://ivannikolic.com)

<!-- prettier-ignore-start -->

[ci]: https://github.com/niksy/node-std-browser/actions?query=workflow%3ACI
[ci-img]: https://github.com/niksy/node-std-browser/workflows/CI/badge.svg?branch=master

<!-- prettier-ignore-end -->
