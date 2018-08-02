# elastic-apm-http-client

[![npm](https://img.shields.io/npm/v/elastic-apm-http-client.svg)](https://www.npmjs.com/package/elastic-apm-http-client)
[![Build status](https://travis-ci.org/elastic/apm-nodejs-http-client.svg?branch=master)](https://travis-ci.org/elastic/apm-nodejs-http-client)
[![codecov](https://img.shields.io/codecov/c/github/elastic/apm-nodejs-http-client.svg)](https://codecov.io/gh/elastic/apm-nodejs-http-client)
[![Standard - JavaScript Style Guide](https://img.shields.io/badge/code%20style-standard-brightgreen.svg?style=flat)](https://github.com/feross/standard)

A low-level HTTP client for communicating with the Elastic APM intake
API.

This module is meant for building other modules that needs to
communicate with Elastic APM.

If you are looking to use Elastic APM in your app or website, you'd most
likely want to check out [the official Elastic APM agent for
Node.js](https://github.com/elastic/apm-agent-nodejs) instead.

## Installation

```
npm install elastic-apm-http-client --save
```

## Example Usage

```js
const Client = require('elastic-apm-http-client')

const client = new Client({
  serviceName: 'My App',
  agentName: 'my-nodejs-agent',
  agentVersion: require('./package.json').version,
  userAgent: 'My Custom Elastic APM Agent'
})

const span = {
  name: 'SELECT FROM users',
  duration: 42,
  start: 0,
  type: 'db.mysql.query'
}

client.sendSpan(span)
```

## API

### `new Client(options)`

Construct a new `client` object. Data given to the client will be
converted to ndjson, compressed using gzip, and streamed to the APM
Server.

Arguments:

- `options` - An object containing config options (see below)

Data sent to the APM Server as part of the metadata package:

- `agentName` - (required) The APM agent name
- `agentVersion` - (required) The APM agent version
- `serviceName` - (required) The name of the service being instrumented
- `serviceVersion` - The version of the service being instrumented
- `frameworkName` - If the service being instrumented is running a
  specific framework, use this config option to log its name
- `frameworkVersion` - If the service being instrumented is running a
  specific framework, use this config option to log its version
- `hostname` - Custom hostname (default: OS hostname)

HTTP client configuration:

- `userAgent` - (required) The HTTP user agent that your module should
  identify it self as
- `secretToken` - The Elastic APM intake API secret token
- `serverUrl` - The APM Server URL (default: `http://localhost:8200`)
- `headers` - An object containing extra HTTP headers that should be
  used when making HTTP requests to he APM Server
- `rejectUnauthorized` - Set to `false` if the client shouldn't verify
  the APM Server TLS certificates (default: `true`)
- `serverTimeout` - HTTP request timeout in milliseconds. If no data is
  sent or received on the socket for this amount of time, the request
  will be aborted. It's not recommended to set a `serverTimeout` lower
  than the `time` config option. That might result in healthy requests
  being aborted prematurely (default: `15000` ms)
- `keepAlive` - If set the `false` the client will not reuse sockets
  between requests (default: `true`)
- `keepAliveMsecs` - When using the `keepAlive` option, specifies the
  initial delay for TCP Keep-Alive packets. Ignored when the `keepAlive`
  option is `false` or `undefined` (default: `1000` ms)
- `maxSockets` - Maximum number of sockets to allow per host (default:
  `Infinity`)
- `maxFreeSockets` - Maximum number of sockets to leave open in a free
  state. Only relevant if `keepAlive` is set to `true` (default: `256`)

Streaming configuration:

- `size` - The maxiumum compressed body size (in bytes) of each HTTP
  request to the APM Server. An overshoot of up to the size of the
  internal zlib buffer should be expected as the buffer is flushed after
  this limit is reached. The default zlib buffer size is 16 kb (default:
  `1048576` bytes / 1 MB)
- `time` - The maxiumum number of milliseconds a streaming HTTP request
  to the APM Server can be ongoing before it's ended (default: `10000`
  ms)

Data sanitizing configuration:

- `truncateStringsAt` - Maximum size in bytes for strings stored as
  Elasticsearch keywords. Strings larger than this will be trucated
  (default: `1024` bytes)

### Event: `close`

The `close` event is emitted when the client and any of its underlying
resources have been closed. The event indicates that no more events will
be emitted, and no more data can be sent by the client.

### Event: `error`

Emitted if an error occurs. The listener callback is passed a single
Error argument when called.

The client is not closed when the `error` event is emitted.

### Event: `finish`

The `finish` event is emitted after the `client.end()` method has been
called, and all data has been flushed to the underlying system.

### `client.sent`

An integer indicating the number of events (spans, transactions, or errors)
sent by the client. An event is considered sent when the HTTP request
used to transmit it have ended.

### `client.sendSpan(span[, callback])`

Send a span to the APM Server.

Arguments:

- `span` - A span object that can be serialized to JSON
- `callback` - Callback is called when the `span` have been flushed to
  the underlying system

### `client.sendTransaction(transaction[, callback])`

Send a transaction to the APM Server.

Arguments:

- `transaction` - A transaction object that can be serialized to JSON
- `callback` - Callback is called when the `transaction` have been
  flushed to the underlying system

### `client.sendError(error[, callback])`

Send a error to the APM Server.

Arguments:

- `error` - A error object that can be serialized to JSON
- `callback` - Callback is called when the `error` have been flushed to
  the underlying system

### `client.flush([callback])`

Flush the internal buffer and end the current HTTP request to the APM
Server. If no HTTP request is in process nothing happens.

Arguments:

- `callback` - Callback is called when the internal buffer have been
  flushed and the HTTP request ended. If no HTTP request is in progress
  the callback is called in the next tick.

### `client.end([callback])`

Calling the `client.end()` method signals that no more data will be sent
to the `client`. If the internal buffer contains any data, this is
flushed before ending.

Arguments:

- `callback` - If provided, the optional `callback` function is attached
  as a listener for the 'finish' event

### `client.destroy()`

Destroy the `client`. After this call, the client has ended and
subsequent calls to `sendSpan()`, `sendTransaction()`, `sendError()`,
`flush()`, or `end()` will result in an error.

## License

[MIT](LICENSE)
