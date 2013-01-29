In the previous blog [Scaling Real-time Apps on Cloud Foundry Using Node.js and Redis](http://blog.cloudfoundry.com/2013/01/24/scaling-real-time-apps-on-cloud-foundry-using-node-js-and-redis/), we used Redis as Session Store and also as a pub-sub service for chat messages. But in many enterprise grade real-time apps, you may want to use RabbitMQ instead of Redis to do pub-sub. This is especially true for financial or Bank apps like Stock Quote apps where it is critical to *protect* and deliver each-and-every message AND do it as quickly as possible.

So, in this blog, we will start from [Scaling Real-time Apps on Cloud Foundry Using Node.js and Redis](http://blog.cloudfoundry.com/2013/01/24/scaling-real-time-apps-on-cloud-foundry-using-node-js-and-redis/) and simply replace Redis with RabbitMQ pubsub. 

The app architecture (before):

<p align='center'>
  < src="https://github.com/rajaraodv/redispubsub/raw/master/pics/redisAsSSAndPS.png" height="300px" width="500px" />
</p>



The app architecture w/ RabbitMQ (after):

<p align='center'>
  < src="https://raw.github.com/rajaraodv/rabbitpubsub/master/pics/finalArchitecture.png" height="300px" width="500px" />
</p>


***
## Intro to RabbitMQ
Node.js community may not be familiar with RabbitMQ. So here are some of the high-level intro of RabbitMQ.

RabbitMQ is a message broker. It simply accepts message from one or more producers and sends it to one or more consumers. 

RabbitMQ is more sophisticated and flexible than just that. Depending on the configuration, it can also figure out what needs to be done when a consumer crashes(store and re-deliver message), when consumer is slow (queue messages), when there are multiple consumers (distribute work load), or even when RabbitMQ itself crashes (durable). For more please see: [RabbitMQ tutorials](http://www.rabbitmq.com/tutorials/tutorial-one-python.html).

RabbitMQ is also very fast. It uses AMQP protocol [built by and for Wall Street firms like JP Morgan Chase, Goldman Sachs etc](http://en.wikipedia.org/wiki/Advanced_Message_Queuing_Protocol#History) for trading stocks and related activities. RabbitMQ is an Erlang (also well-known for concurrency & speed) implementation of that protocol.
 

For more please go through [RabbitMQ's website](http://www.rabbitmq.com).

***
## RabbitMQ Basics (Creating Producers)

 
 
 
