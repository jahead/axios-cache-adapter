# :rocket: axios-cache-adapter [![Build Status](https://travis-ci.org/RasCarlito/axios-cache-adapter.svg?branch=master)](https://travis-ci.org/RasCarlito/axios-cache-adapter) [![codecov](https://codecov.io/gh/RasCarlito/axios-cache-adapter/branch/master/graph/badge.svg)](https://codecov.io/gh/RasCarlito/axios-cache-adapter)

> Caching adapter for axios. Store request results in a configurable store to prevent unneeded network requests.

Adapted from [superapi-cache](https://github.com/stephanebachelier/superapi-cache)
by [@stephanebachelier](https://github.com/stephanebachelier)

## Install

Using npm

```sh
npm install --save axios-cache-adapter
```

Or bower

```sh
bower install --save axios-cache-adapter
```

Or from a CDN like unpkg.com

```html
<script type="text/javascript" src="https://unpkg.com/axios-cache-adapter"></script>
```

## Usage

You can use the adapter directly

```js
import axios from 'axios'
import { setupCache } from 'axios-cache-adapter'

const cache = setupCache({
  maxAge: 15 * 60 * 1000
})

const api = axios.create({
  adapter: cache.adapter
})

api({
  url: 'http://some-rest.api/url',
  method: 'get'
}).then(async (response) => {
  // Do something fantastic with response.data \o/
  console.log('Request response:', response)

  // Interacting with the store, see `localForage` API.
  const length = await cache.store.length()

  console.log('Cache store length:', length)
})
```

Or you can get an instance of axios pre-configured with the cache adapter

```js
import { setup } from 'axios-cache-adapter'

const api = setup({
  cache: {
    maxAge: 15 * 60 * 1000
  }
})

api({
  url: 'http://some-reset.api/url',
  method: 'get'
}).then(async (response) => {
  // Do something awesome with response.data \o/
  console.log('Request response:', response)

  // Interacting with the store, see `localForage` API.
  const length = await api.cache.length()

  console.log('Cache store length:', length)
})
```

Custom `localForage` store example:

```js
import localforage from 'localforage'
import memoryDriver from 'localforage-memoryStorageDriver'
import { setup } from 'axios-cache-adapter'

async function configure () {
  await localforage.defineDriver(memoryDriver)

  const store = localforage.createInstance({
    // List of drivers used
    driver: [
      localforage.INDEXEDDB,
      localforage.LOCALSTORAGE,
      memoryDriver._driver
    ],
    // Prefix all storage keys to prevent conflicts
    name: 'my-cache'
  })

  // Create axios instance with cache adapter configured using localforage store
  return setup({
    cache: {
      maxAge: 15 * 60 * 1000,
      store
    }
  })
}

configure().then(async (api) => {
  const response = await api.get('http://some-rest.api/url')

  // Display something beautiful with response.data ;)
})
```

## API

### setupCache(options)

Create a cache adapter instance. Takes an `options` object to configure how the cached requests will be handled,
where they will be stored, etc.

#### Options

* `maxAge {Number}`: Maximum time for storing each request in milliseconds, defaults to 15 minutes (`15 * 60 * 1000`)
* `limit {Number}`: Maximum number of cached request (last in, first out queue system), no limit by default
* `store {Object}`: An instance of [`localForage`](https://github.com/localForage/localForage), defaults to a custom in memory store
* `key {Mixed}`: Can either be a `String` or a `Function` which receives the `request` object as first parameter to return a unique cache key for each request. Defaults to `req => req.url`
* `exclude {Object}`: Object defining which kind of requests should be excluded from cache
  * `filter {Function}`: A method which receives the request and returns `true` to exclude request from cache, defaults to `null`
  * `query {Boolean}`: If `true` all requests containing a query will be excluded, defaults to `true`
  * `paths {Array}`: An `Array` of regular expressions to match against request URLs, if a match is found it will be excluded, defaults to `[]`
* `clearOnStale {Boolean}`: Clear cached item when it is stale, defaults to `true`
* `clearOnError {Boolean}`: Clear all cache when a write error occurs (prevents size quota problems with `localStorage`)
* `debug {Boolean}`: Print some logs to console, defaults to `false`

#### Returns

`setupCache()` returns an object containing the configured `adapter`, the cache `store` and the `config` that is applied to this instance.

### setup(options)

Create an `axios` instance pre-configured with the cache adapter. Takes an `options` object to configure the cache and
axios at the same time.

#### Options

* `cache {Object}`: Same options as the `setupCache` method

All the other parameters passed in the `options` will be passed directly to the [`axios.create`](https://github.com/mzabriskie/axios#creating-an-instance) method.

#### Returns

`setup()` returns an instance of `axios` pre-configured with the cache adapter.
The cache `store` is conveniently attached to the `axios` instance as `instance.cache` for easy access.

## Building

```sh
npm run build
```

Webpack is used to build [umd](https://github.com/umdjs/umd) versions of the library that are placed in the `dist` folder.

* `cache.js`
* `cache.min.js`
* `cache.node.js`
* `cache.node.min.js`

A different version of `axios-cache-adapter` is generated for node and the browser due to how Webpack 4 uses a `target` to change how the UMD wrapper is generated using `global` or `window`. If you are using the library in node or in your front-end code while using a module bundler (Webpack, rollup, etc) the correct version will be picked up automatically thanks to the `"main"` and `"browser"` fields in the `package.json`.

`axios-cache-adapter` is developped in ES6+ and uses async/await syntax. It is transpiled to ES5 using `babel` with `preset-env`.

## Testing

Tests are executed using [karma](https://github.com/karma-runner/karma).

To launch a single run tests using ChromeHeadless:

```sh
npm test
```

To launch tests in watch mode in Chrome for easier debugging with devtools:

```sh
npm run watch
```

## Browser vs Node.js

`axios-cache-adapter` was designed to run in the browser. It does work in nodejs using the [in memory store](https://github.com/RasCarlito/axios-cache-adapter/blob/master/src/memory.js). But storing data in memory is not the greatests idea ever.

You can give a `store` to override the in memory store but it has to comply with the [`localForage`](https://github.com/localForage/localForage) API and `localForage` does not work in nodejs for very good reasons that are better explained in [this issue](https://github.com/localForage/localForage/issues/57).

Maybe it should be possible to connect `axios-cache-adapter` to a [redis](https://redis.io/) store or something equivalent.

If you have any suggestions for a better way to work in nodejs please open an issue or submit a pull request.

## License

MIT © [Carl Ogren](https://github.com/RasCarlito)
