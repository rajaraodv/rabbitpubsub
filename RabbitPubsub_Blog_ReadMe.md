In the previous blog [Scaling Real-time Apps on Cloud Foundry Using Node.js and Redis](http://blog.cloudfoundry.com/2013/01/24/scaling-real-time-apps-on-cloud-foundry-using-node-js-and-redis/), we used Redis as Session Store and also as a pub-sub service for chat messages. But in many enterprise grade real-time apps, you may want to use RabbitMQ instead of Redis to do pub-sub. This is especially true for financial or Bank apps like Stock Quote apps where it is critical to *protect* and deliver each-and-every message AND do it as quickly as possible.

So, in this blog, we will start from [Scaling Real-time Apps on Cloud Foundry Using Node.js and Redis](http://blog.cloudfoundry.com/2013/01/24/scaling-real-time-apps-on-cloud-foundry-using-node-js-and-redis/) and simply replace Redis with RabbitMQ pubsub. 

The app architecture (before):

<p align='center'>
  <img src="https://github.com/rajaraodv/redispubsub/raw/master/pics/redisAsSSAndPS.png" height="300px" width="500px" />
</p>



The app architecture w/ RabbitMQ (after):

<p align='center'>
  <img src="https://raw.github.com/rajaraodv/rabbitpubsub/master/pics/finalArchitecture.png" height="300px" width="500px" />
</p>


***
## Intro to RabbitMQ
Node.js community may not be familiar with RabbitMQ. So here are some of the high-level intro of RabbitMQ.

RabbitMQ is a message broker. It simply accepts message from one or more producers and sends it to one or more consumers. 

RabbitMQ is more sophisticated and flexible than just that. Depending on the configuration, it can also figure out what needs to be done when a consumer crashes(store and re-deliver message), when consumer is slow (queue messages), when there are multiple consumers (distribute work load), or even when RabbitMQ itself crashes (durable). For more please see: [RabbitMQ tutorials](http://www.rabbitmq.com/tutorials/tutorial-one-python.html).

RabbitMQ is also very fast. It uses AMQP protocol [built by and for Wall Street firms like J.P. Morgan Chase, Goldman Sachs etc](http://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol#History) for trading stocks and related activities. RabbitMQ is an Erlang (also well-known for concurrency & speed) implementation of that protocol.
 

For more please go through [RabbitMQ's website](http://www.rabbitmq.com).

***
## RabbitMQ Basics 
<p align="center">
<img src="https://raw.github.com/rajaraodv/rabbitpubsub/master/pics/rabbitmq.png" />
</p>

RabbitMQ has 4 pieces.

1. Producer ("P") - Sends messages to an exhange along with "Routing key" indicating how to route the message.
2. Exchange ("X") - Recieves message and Routing key from Producers and figures out what to do with the message.
3. Queues("Q") - A temporary place where the messages are stored based on Queue's "binding key" until a consumer is ready to recieve the message. Note: While a Queue physically resides inside RabbitMQ, A consumer is the one that actually creates it by providing a "Binding Key".
4. Consumer("C") - Subscribes to a Queue to recieve messages.

***
## Routing Key, Binding Key and types of Exchanges
To allow various work-flows like pub-sub, work queues, topics, RPC etc., RabbitMQ allows us to independently configure the type of the Exchange, Routing Key and Binding Key.

#### Routing Key:
A string/constraint from Producer instructing Exchange how to route the message. A Routing key looks like: "logs", "errors.logs", "warnings.logs" "tweets" etc. 

#### Binding Key:

Another string/constraint added by a Consumer to a queue to which it is binding/listening to. A Binding key looks like: "logs", "*.logs", "#.logs" etc. 

Note: In RabbitMQ, Binding keys can have "patterns" (but not Routing keys).   

#### Types of Exchange:

Exchanges can be of 4 types:

1. Direct - Sends messages from producer to consumer if Routing Key and Binding key match exactly.
2. Fanout - Sends any message from a producer to ALL consumers (i.e ignores both routing key & binding key)
3. Topic - Sends a message from producer to consumer based on pattern-matching.
4. Headers - If more complicated routing is required beyond simple Routing key string, you can use headers exchange.

As you can see the combination of `the type of Exchange`, `Routing Key` and `Binding Key` makes RabbitMQ behaves completely differently. For example: A `Fanout` Exchange ignores `Routing Key` and `Binding Key` and sends messages to all consumers. A `Topic` Exchange sends a copy of a message to zero, one or more consumers. 

Going into more details is beyond the scope of this blog, but here is another good blog that goes into more details: [AMQP 0-9-1 Model Explained](http://www.rabbitmq.com/tutorials/amqp-concepts.html) 

***
## Creating Exchanges, Producers & Consumers in Node.js

 
 
