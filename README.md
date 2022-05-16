# wrappitmq – it wraps amqplib for us lazy people!

[![Build Status](https://travis-ci.com/patrickd-/wrappitmq.node.svg?branch=master)](https://travis-ci.com/patrickd-/wrappitmq.node) [![Coverage Status](https://coveralls.io/repos/github/patrickd-/wrappitmq.node/badge.svg)](https://coveralls.io/github/patrickd-/wrappitmq.node)

```
npm install wrappitmq
```

This is a Node.JS library for Inter-Process Communication with RabbitMQ based on [amqplib](https://github.com/squaremo/amqp.node) (AMQP 0-9-1). It intends to take care of all the complicated stuff and offers you a simple API with sensible defaults.


## WorkQueue

Producer enqueues a task to be worked on. You may block execution until the Broker confirms that it has received the message. That task will be picked up by *one* Worker and it will be acknowledged if the consuming function resolves. If an error occurs within the consuming function the error event will be emitted on the queue and the task will be re-enqueued for another try.

```javascript
const WorkQueue = require('wrappitmq').WorkQueue;

// Set-up.
const queue = new WorkQueue({
  // All these options are optional.
  prefetch: 1, // How many messages to work on asynchronously (Default: 1)
  queue: 'workqueue', // Name of the queue to use. (Default: workqueue)
  url: 'amqp://localhost:5672' // Can also be specified on connect()
});

// URL here is optional, can also be set as above.
await queue.connect('amqp://localhost:5672');

queue.on('error', (err) => {
  // Something went wrong and you should log it.
});
queue.on('close', (err) => {
  // The connection was closed and the queue is no longer usable.
  // An err is given if the connection was closed due to an error.
});

// Enqueue a task.
await queue.enqueue({ do: 'it' });

// Consume tasks.
const cancel = await queue.consume(async (task) => {
  await work(task);
  // Will be acknowledged when work() is done.
});

// Cancel consumer to stop receiving tasks.
await cancel();

// Close connection.
await queue.close();
```

## PubSub

Publisher publishes a message on a certain topic. You may block execution until the Broker confirms that it has received the message. The message will be broadcast to *all* Subscribers listening on that topic. There is no acknowledgement and persistence handling for these exchanges. Messages will be lost if nobody has subscribed to the topic.

```javascript
const PubSub = require('wrappitmq').PubSub;

// Set-up.
const pubsub = new PubSub({
  // All these options are optional.
  exchange: 'pubsub', // Name of the queue to use. (Default: pubsub)
  url: 'amqp://localhost:5672' // Can also be specified on connect()
});

// URL here is optional, can also be set as above.
await pubsub.connect('amqp://localhost:5672');

pubsub.on('error', (err) => {
  // Something went wrong and you should log it.
});
pubsub.on('close', (err) => {
  // The connection was closed and the queue is no longer usable.
  // An err is given if the connection was closed due to an error.
});

// Publish a message.
await pubsub.publish('login', { userId: 1 });

// Subscribe to a topic.
const unsub = await pubsub.subscribe('login', async (message) => {
  await loggedIn(message.userId);
});

// Cancel subscription to stop receiving messages.
await unsub();

// Close connection.
await pubsub.close();
```

## Tests

Run the libraries integration tests with:
```
$ AMQP_URL="amqp://user:pass@localhost:5672" npm test
```

To determine test coverage run:
```
$ AMQP_URL="amqp://user:pass@localhost:5672" npm run coverage
```

## Changelog

### v1.1.0

This version updated amqplib from [0.5 to 0.8](https://github.com/squaremo/amqp.node/blob/main/CHANGELOG.md), which came with some possibly breaking changes. Please make sure to test for these before upgrading wrappitmq to v1.1.0:

* amqplib v0.8.0: Support for NodeJS prior to v10 is dropped ([PR 615](https://github.com/squaremo/amqp.node/pull/615))
* amqplib v0.5.5: Minor but possibly breaking change: after [PR 498](https://github.com/squaremo/amqp.node/pull/498), all confirmation promises still unresolved will be rejected when their associated channel is closed.

## Contributions

Are always welcome!

* Initial development by @patrickd-
* Name suggested by @leschekfm :D
* Updated by @KevinBierende
