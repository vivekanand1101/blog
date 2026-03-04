---
title: "Realtime Events using Tornado and Rabbitmq"
date: 2017-09-09
categories: 
  - "tech"
---

At my day job, i needed a way to send real time events to clients that would in turn trigger some action on their side. The clients could ask for some computation from server which may take time.

To tackle this situation, i ended up making [Tornado](https://github.com/tornadoweb/tornado) as a websocket server which will be different from our web app server (and both behind [nginx](https://nginx.org/en/)). There are a couple of other services which the client may ask for indirectly. Since, those computations won't have normal request - response cycle, the results from the computations will have to pushed to the clients. Since, the communication between the client and server is two-way so, websocket seemed fitting. For routing of messages internally, i decided to use [Rabbitmq](https://www.rabbitmq.com/) and [Celery](http://www.celeryproject.org/) for actual execution of tasks.

The problem with this is: Rabbitmq consumer and Tornado both run their own I/O loop. That confused me a little because i had heard this combo worked for [zulip](https://github.com/zulip/zulip) when i was randomly reading about their architecture. So, i duckduckgoed(:D) and found this article: [https://reminiscential.wordpress.com/2012/04/07/realtime-notification-delivery-using-rabbitmq-tornado-and-websocket/](https://reminiscential.wordpress.com/2012/04/07/realtime-notification-delivery-using-rabbitmq-tornado-and-websocket/)   . It turns out he also had a similar doubt and he got a solution.

[Pika](https://github.com/pika/pika) library comes with a tornado adapter named [TornadoConnection](https://github.com/pika/pika/blob/master/pika/adapters/tornado_connection.py). This makes running the rabbitmq consumer loop inside the tornado IOloop itself. The code for tornado connection is fairly simple. As the code given in the blog wasn't fully functional, i had to contact the source code of pika a couple of times.

Each websocket connection in tornado gets a unique [WebSocketHandler](https://github.com/tornadoweb/tornado/blob/master/tornado/websocket.py#L60) object and these are not directly accessible from the tornado application object. But, the reverse is true. Each websocket handler has access to the application object. So, using TorandoConnection, we tie up one pika consumer to the tornado application object.

**server.py**

\[code language="python"\]

def main(): ''' The main method to run the tornado application '''

io\_loop = tornado.ioloop.IOLoop.instance()

pc = PikaConsumer(io\_loop)

application.pc = pc application.pc.connect() application.listen(8080) io\_loop.start() \[/code\]

**consumer.py**

\[code language="python"\]

class PikaConsumer(object): ''' The pika client the tornado will be part of '''

def \_\_init\_\_(self, io\_loop): print 'PikaClient: \_\_init\_\_' self.io\_loop = io\_loop self.connected = False self.connecting = False self.connection = None self.channel = None self.event\_listeners = {}

def connect(self): ''' Connect to the broker ''' if self.connecting: print 'PikaClient: Already connecting to RabbitMQ' return

print 'PikaClient: Connecting to RabbitMQ' self.connecting = True

cred = pika.PlainCredentials('someuser', 'somepass') param = pika.ConnectionParameters( host='localhost', port=5672, virtual\_host='somevhost', credentials=cred) self.connection = TornadoConnection( param, on\_open\_callback=self.on\_connected) self.connection.add\_on\_close\_callback(self.on\_closed)

def on\_connected(self, connection): print 'PikaClient: connected to RabbitMQ' self.connected = True self.connection = connection self.connection.channel(self.on\_channel\_open)

def on\_channel\_open(self, channel): print 'PikaClient: Channel open, Declaring exchange' self.channel = channel # declare exchanges, which in turn, declare # queues, and bind exchange to queues self.channel.exchange\_declare( exchange='someexchange', type='topic') self.channel.queue\_declare(self.on\_queue\_declare, exclusive=True)

def on\_queue\_declare(self, result): queue\_name = result.method.queue self.channel.queue\_bind( self.on\_queue\_bind, exchange='someexchange', queue=queue\_name, routing\_key='commands.\*') self.channel.basic\_consume(self.on\_message)

def on\_queue\_bind(self, is\_ok): print 'PikaClient: Exchanges and queue created/joined'

def on\_closed(self, connection): print 'PikaClient: rabbit connection closed' self.io\_loop.stop()

def on\_message(self, channel, method, header, body): print 'PikaClient: message received: %s' % body self.notify\_listeners(body) # important, since rmq needs to know that this msg is received by the # consumer. Otherwise, it will be overwhelmed channel.basic\_ack(delivery\_tag=method.delivery\_tag)

def notify\_listeners(self, event\_obj): # do whatever you wish pass

def add\_event\_listener(self, listener): # listener.id is the box id now self.event\_listeners\[listener.id\] = { 'id': listener.id, 'obj': listener} print 'PikaClient: listener %s added' % repr(listener)

def remove\_event\_listener(self, listener): try: del self.event\_listeners\[listener.id\] print 'PikaClient: listener %s removed' % repr(listener) except KeyError: pass

def event\_listener(self, some\_id): ''' Gives the socket object with the given some\_id '''

tmp\_obj = self.event\_listeners.get(some\_id) if tmp\_obj is not None: return tmp\_obj\['obj'\] return None

\[/code\]

That's it. In your _WebSocketHandler_ objects, you can access the consumer via: _self.application.pc_

Although, this is working fine for me right now but, i am not fully satisfied with this. At present each connection is listening to a single queue because in rabbitmq one consumer cannot listen to multiple queues.
