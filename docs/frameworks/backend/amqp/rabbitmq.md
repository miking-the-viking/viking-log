# [RabbitMQ](https://www.rabbitmq.com/)

- [Getting Started](https://www.rabbitmq.com/getstarted.html)
- [JC's RabbitMQ Sandbox](https://github.com/wonderpanda/tsrabbit)
- [Examples from RabbitMQ Tutorial running JC's RabbitMQ Docker Container](https://github.com/miking-the-viking/learn-rabbitmq)

## Overview

RabbitMQ is a widely used open source message broker based on AMQP. It accepts and forwards messages acting as post box, post office and post man; the difference being that RabbitMQ accepts, stores and forwards binary blobs of data (_messages_) instead of paper.

### `rabbitmqctl`

`rabbitmqctl` is a command line tool for RabbitMQ that can be used for various purposes such as:

- listing exchanges on the RabbitMQ server (`sudo rabbitmqctl list_exchanges`)
- listing queues on the RabbitMQ server (`sudo rabbitmqctl list_queues`)
- listing the unacknowledged messages (`sudo rabbitmqctl list_queues name messages_ready messages_unacknowledged`)
- listing bindings (`rabbitmqctl list_bindings`)

### RabbitMQ Jargon

- _Producing_ means sending. A program that sends messages is a _producer_.

![Producer](./images/producer.png "Producer")

- A _Queue_ is the name for a "post box" that resides inside RabbitMQ. Messages flow through RabbitMQ and applications, but can only be stored inside a _queue_. _Queue's_ are bound by the host's memory & disk limits, acting as a large message buffer. Many _producers_ can send messages that go into one queue, and many _consumers_ can try to receive data from one _queue_.

![Queue](./images/queue.png "Queue")

- _Consuming_ has similar meaning to receiving. A _consumer_ is a program that mostly waits to receive messages.

![Consumer](./images/consumer.png "Consumer")

## RabbitMQ Tutorials

### [Hello World](https://www.rabbitmq.com/tutorials/tutorial-one-javascript.html)

Two components:

- _Publisher_ will connect to RabbitMQ, send a single message then exit.
- _Receiver_(consumer) will continue to listen for messages and print them out.

### [Work Queues](https://www.rabbitmq.com/tutorials/tutorial-two-javascript.html)

![Work Queue](./images/work-queue.png "Work Queue")

In this tutorial, we'll create a _work queue_ (_task queue_) to distribute time-consuming tasks among multiple workers.

The main idea behind _Work Queues_ (_Task Queues_) is to avoid doing a resource-intensive task immediately and having to wait for it's completion. The task is instead scheduled to be done later. The _task_ is encapsulated as a message and sent to a queue which a worker process will eventually execute.

This implementation is a simulation of a resource intensive task. We'll use '.' to signify 1 second of work.

The default approach for mutliple workers implements a round robin queue.

#### Message Acknowledgment

If a worker is not setup properly, the moment RabbitMQ delivers a message to worker it marks the message for deletion: hence if the worker died, then message lost and any subsequent messages that were sent to this particular worker but not yet handled.

In order to not lose any tasks: if a worker dies, deliver the task to another worker.

RabbitMQ supports _message acknowledgments_: an ack(nowledgment) is sent back by the consumer to tell RabbitMQ that a particular message has been received, processed and that RabbitMQ can delete it.

If a consumer dies (channel closed, connection closed, or TCP connection lost) without sending an ack, RabbitMQ will understand that the message wasn't processed fully and will re-queue it. If there are other consumers online at that time, it will quickly redeliver it to another consumer.

*NOTE* There aren't aby nessage timeouts. RabbitMQ will redeliver the message when the consumer dies, even if processing the message takes a very, very long time.

Use the boolean option `noAck` to specify if the should be an acknowledgment, default is false requiring acknowledgment.

#### Message Durability

Using ack(nowledgment) we can ensure that even if a consumer dies, the task isn't lost. However the task will be lost if the RabbitMQ server stops. All queues and messages will be forgotten unless RabbitMQ is confugred to set them as the boolean `durable`.

First, RabbitMQ must be declared as durabl in the `assertQueue` call in both the producer and consumer.

*NOTE:* RabbitMQ wil not let you redefine a queue whose name already exists with a differently configured durability. You'll be required to use a different name in this case.

Last, the messages must be marked as _persistent_ by using the boolean `persistent` option in the `Channel.sendToQueue` method.

*NOTE:* There is a slight chance that the message may be missed (not really written to disk and only saved in cache). For stronger persistence guarantees check out [`publisher confirms`](https://www.rabbitmq.com/confirms.html).

#### Fair Dispatch

![Prefetch](./images/prefetch-count.png "Prefetch")


RabbitMQ blindly dispatches every n-th message to the n-th consumer, this doesn't lead to a fair distribution of work as some workers may get overloaded with heavy tasks and others may not.

In order to overcome this issue use the `prefetch` method with a value of `1`. This tells RabbitMQ not to give more than one message to a worker at a time. Or _don't dispatch a new message to worker until is has processed and acknowledged the previous one_. It will instead dispatch it to the next worker that is not still busy.

```
ch.prefetch(1);
```

Using both message acknowledgments and `prefetch` it is possible to setup a work queue. The durability options let the tasks survive even if RabbitMQ is restarted.

Further `Channel` methods and message properties can be read in the [`amqplib` docs](http://www.squaremobius.net/amqp.node/channel_api.html).


### Publish/Subscribe

In the previous tutorial using the _Work Queue_, the assumption was that each task is delivered to exactly one worker. In this tutorial the message will be delivered to multiple consumers in a pattern known as "publish/subscribe".

An example implementation of this is a logging system: The producer will emit log messages, the consumer(s) will receive and print them. So emphasize multiple receivers, they will consume the same message but one will console log the output, while the other will write it to a file.

#### Exchanges

Previously messages were went and received to and from a _queue_. Now to delve into the full messaging model of Rabbit.

We previously established the following conventions: 

 - A _producer_ is a user application that sends messages.
 - A _queue_ is a buffer that stores messages.
 - A _consumer_ is a user application that receives messages.

The core idea in the messaging model of RabbitMQ is that the producer never sends any messages directly to the queue. Quite often the producer doesn't event know if a message will be delivered to any queue at all.

Instead the producer can only send messages to an _exchange_.

![Exchange](./images/exchanges.png "Exchange")

An _exchange_ is simply defined in that on one end it **receives messages from _producers_** and on the other end it **pushes them into _queues_**. The _exchange_ muyst know exactly what to do with a message it receives. (_Append to a particular queue? Appended to many queues? Discard the message?_) These rules are defined by the _exchange type_.

There are several exchange types available: `direct`, `topic`, `headers` and `fanout`.

##### Exchange Types

###### `fanout`

A `fanout` exchange is simple: it broadcasts all the messages it receives to all the queues it knows (perfect for a logger).

###### `headers`

###### `topic`

###### `direct`

A `direct` exchange takes a route `binding key` as a parameter in order to send the messages directly to the proper queue. Any `binding keys` not registered to a queue will be discarded.

#### Temporary Queues

Previously we used queues which had a specified name (`hello` and `task_queue`), it was important to name the queue in order to share that particular queue between the producer(s) and consumer(s).

This is not the case for the logger. The logger is to log **all** messages, as well as only currently flowing messages, not the old ones. This requires two things:

1. Whenever we connect to RabbitMQ, we need a fresh, empty queue. This could be done by creating a queue with a random name, or preferably: let the server choose a random queue name for use.

2. Once we disconnect the consumer, the queue should be automatically deleted.

In the `amqp.node` client, when the supplied queue name is an empty string a non-durable queue is created with a generated name.

```javascript
ch.assertQueue('', {exclusive: true});
```

When the method returns, the queue instance contains a random queue-name generated by RabbitMQ. When the connection that declared it closes, the queue will be deleted because it is declared as `exclusive`.

[More on `exclusive` flag and other qwueue properties](https://www.rabbitmq.com/queues.html)

#### Bindings

![Bindings](./images/bindings.png "Bindings")

The relationship between an exchange and a queue is called a binding.

```javascript
ch.bindQueue(queue_name, 'logs', '');
```

#### All together

![Complete Diagram](./images/3-overall.png "Complete Diagram")

Messages are still produced by the producer, however how they're handled will now publish messages to the `logs` exchange. A routing key needs to be provided when sending the message (but for a `fanout` exchange the value is ignored).

### Routing

In the previous tutorial a simple logging system was built, messages could be broadcast to many receivers.

Here we will add a feature to it: **subscribing to only a subset of the messages**

For instance, only saving the critical error messages to the log file (to conserve disk space), while still printing all of the log messages to the console.

#### Bindings (part 2)

A Binding is a relationship between an exchange and a queue (_the queue is interested in messages from **this** exchange_). They were used in the previous broadcasting example.

```javascript
ch.bindQueue(q.queue, ex, '');
```

Bindings can take an extra binding key as a parameter to create a binding with a key.

```javascript
ch.bindQueue(queue_name, exchange_name, 'key')
```

The meaning of the binding key varies by exchange type. For `fanout` exchanges the value is ignored.

#### Direct Exchange

A `direct` exchange is simple: a message goes to the queues whose `binding key` exactly matches the `routing key` of the message.

![Direct Exchange](./images/direct-exchange.png "Direct Exchange")

In this example, the `direct` exchange `x` has two queues bound to it: one with the binding key `orange` and the other with binding keys `black` and `green`.

In this example setup a message published to the exchange with the routing key `orange` wil lbe rouoted to queue `Q1`, the messages with the routing key `black` or `green` will go to `Q2` and all others will be discarded.

##### Multiple Direct Exchange Bindings

![Multiple Direct Exchange Bindings](./images/direct-exchange-multiple.png "Multiple Direct Exchange Bindings")

It is permitted to bind multiple queues with the same binding key.

#### Emitting Logs

By using Multiple Direct Exchange Bindings we can duplicate critical logs to be stored to a log file. By supplying the log severity as a `routing key` the message will be directed to any necessary additional destinations.

#### Subscribing

Subscribing works the same, however we now need to create a binding for each severity we're interested in.

### All Together

![All together](./images/4-overall.png "All together")


### Topics

Direct exchange binding was used in the previous example and demonstrated a feasibility for some complex tasks, however it has limitations: it cannot do routing based on **multiple criteria**. Introducting `topic` echange!

#### Topic Exchange

Messages sent to a `topic` exchange can't have an arbitrary `routing_key`, it **must be a list of words, delimited by dots**. Examples include "`stock.usd.nyse`","`nyse.vmw`", "`quick.orange.rabbit`" up to the limit of 255 bytes.

The binding key must also be in the same form. `topic` exchange logic is similar to `direct` however with the added benefit that the "key" can include:

- `*` to substitute exactly for one word
- `#` to substitute for 0 or more words

**NOTE** When wildcards `#` and `*` not used, a `topic` exchange behaves like a `direct` exchange.

![Topic Wildcard Routing](./images/topic-wildcards.png "Topic Wildcard Routing")


### Remote Procedure Call (RPC)

_Work Queues` have already been used to distribute time-consuming tasks amongst multiple workers. However in the scenario where the computer must wait for a remotely computed result a different pattern must be employed: _Remote Procedure Call_.

#### Callback Queue

In order to receive a response a "callback" queue address must be sent with the RPC request, such as the default queue.

```javascript
ch.assertQueue('', {exclusive: true});

ch.sendToQueue('rpc_queue',new Buffer('10'), { replyTo: queue_name });

# ... then code to read a response message from the callback queue ...
```

> ##### Message properties
> The AMQP 0-9-1 protocol predefines a set of 14 properties that go with a message. Most of the properties are rarely used, with the exception of the following:
> - `persistent`: Marks a message as persistent (with a value of true) or transient (false). You may remember this property from the second tutorial.
> - `content_type`: Used to describe the mime-type of the encoding. For example for the often used JSON encoding it is a good practice to set this property to: application/json.
> - `reply_to`: Commonly used to name a callback queue.
> - `correlation_id`: Useful to correlate RPC responses with requests.

#### Correlation Id

It's rather inefficient to create a callback queue for every RPC request (as in the previous example). It's more efficient to create a single callback queue per client (or less?).

Hence a new issue: having received a response in that queue it's not clear to which request the response belongs to. Behold the `correlation_id` property. By assigning it a unique value for every request this property can be checked against any received message in the callback queue to match it's corresponding response. If an unknown `correlation_id` is identified, its message can be safely discarded.

![Finally...](./images/finally.png "Finally...")

#### Finally

In summary:

1. When the client starts up, it creates an anonymous exclusive callback queue. \
(If an RPC request) the Client sends a message with two properties: 
   - `reply_to` which is set to the callback queue
   - `correlation_id` which is to set a unique value for every request.
2. The request is sent to an `rpc_queue` queue.
3. The RPC worker (server) is waiting for requests on that queue, when a request is received it executes the job and returns a message to the Client using the queue from the `reply_to` field.
4. The Client waits for the data on the callback queue. When a message is received it compares the `correlation_id` peroperty. If it matches the value from the request it returns the response to the application.

