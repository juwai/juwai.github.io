---
layout: post
title:  "A simple guide for ZeroMQ"
date:   2015-09-23 18:00:09
categories: post
---

RPC
=========

## Definition

### Subroutine

In different programming languages, a subroutine may be called a `procedure`, a `function`, a `routine`, a `method`, or a `subprogram`.

### Remote Procedure Call

+ Network
+ function(parameters) -> return

-----------------

## Component

+ 1: ZeroMQ
+ 2: MessagePack

-----------------

## Demo

### Prepare

0. make sure everyone connected to the same Wi-Fi

1. Expose a port on your project's vagrant box(update Vagrant file)

```
config.vm.network "forwarded_port", guest: 5555, host: 5555
```

and run `vagrant reload`

2. Prepare a Python `virtual environment`

```
$ vagrant ssh
$ mkdir /vagrant/rpc_demo
$ cd /vagrant/rpc_demo
$ virtualenv .virtualenv
$ ve pip install msgpack-python pyzmq
```

3. install packages(*)

```
$ vagrant ssh
$ sudo yum install strace lsof
```

### Check List

+ `php -m`(make sure `zmq` and `msgpack` exist, should already installed for )
+ `ve pip freeze`

---------------------

### demo 1

Request - Reply(all PHP)

+ client.php

{% highlight php %}
<?php

$client = new ZMQSocket(new ZMQContext(), ZMQ::SOCKET_REQ);
$client->connect("tcp://127.0.0.1:5555");

$client->send("hello there!");
$result = $client->recv();
echo "Get back message: $result\n";
{% endhighlight %}

+ server.php

{% highlight php %}

$server = new ZMQSocket(new ZMQContext(), ZMQ::SOCKET_REP);
$server->bind("tcp://127.0.0.1:5555");

while ($message = $server->recv()) {
    echo "Get message: $message\n";
    /* echo back the message */
    $server->send($message);
}
{% endhighlight %}

### demo 2

Request - Reply(PHP + Python)

+ what would happen if server go alive after request was already sent

+ client.py

{% highlight python %}
import zmq
import msgpack

ctx = zmq.Context()
socket = ctx.socket(zmq.REQ)
socket.connect("tcp://127.0.0.1:5555")

socket.send('hello')
msg = socket.recv()

print(msg)
{% endhighlight %}

+ server.php

{% highlight php %}
<?php

$server = new ZMQSocket(new ZMQContext(), ZMQ::SOCKET_REP);
$server->bind("tcp://127.0.0.1:5555");

while ($message = $server->recv()) {
    echo "Get message: $message\n";
    /* echo back the message */
    $server->send($message);
}
{% endhighlight %}

### demo 3

Pub - Sub / Push - Pull

#### Pub Sub

+ producer.php

{% highlight php %}
<?php
$producer = new ZMQSocket(new ZMQContext(), ZMQ::SOCKET_PUB);
$producer->bind("tcp://*:5555");

while (1) {
    $date = date('m/d/Y h:i:s a', time());
    echo "$date \n";
    $producer->send($date);
    sleep(1);
}
{% endhighlight %}

+ consumer.php

{% highlight php %}
<?php
$consumer = new ZMQSocket(new ZMQContext(), ZMQ::SOCKET_SUB);

$consumer->connect("tcp://127.0.0.1:5555");
$consumer->setSockOpt(ZMQ::SOCKOPT_SUBSCRIBE, '');

while(1) {
    $date = $consumer->recv();

    echo "Get date: $date\n";
}
{% endhighlight %}

#### Push Pull

+ push.php

{% highlight php %}
<?php

$socket = new ZMQSocket(new ZMQContext(), ZMQ::SOCKET_PUSH);
$socket->bind("tcp://*:5555");

while(1) {
    $date = date('m/d/Y h:i:s a', time());

    echo "Now is $date\n";
    $socket->send($date);
    sleep(0.5);
}
{% endhighlight %}

+ pull.php

{% highlight php %}
<?php

$socket = new ZMQSocket(new ZMQContext(), ZMQ::SOCKET_PULL);
$socket->connect('tcp://127.0.0.1:5555');

while($message = $socket->recv()) {
    echo "Get back message $message\n";
    sleep(2);
}
{% endhighlight %}

### demo 4

Request - multiple Reply(make some change to client.php, change server's reponse to name)

everyone can join us.

+ and why we need consul(register and service discover)
+ remember pub/sub? using pub/sub to publish event and consul-template can generate newest rpc config for laravel

#### client.py

{% highlight python %}
import zmq

ctx = zmq.Context()
socket = ctx.socket(zmq.REQ)
socket.connect(b"tcp://10.168.168.145:5555")
socket.connect(b"tcp://10.168.168.129:5555")
socket.connect(b"tcp://10.168.168.157:5555")

for i in range(1000):
    frame = []
    frame.append(b"hello")
    socket.send_multipart(frame)
    from time import sleep
    sleep(1)
    frame = socket.recv_multipart()
    print(frame)
{% endhighlight %}

### demo 5

MessagePack

+ client.py

{% highlight python %}

import zmq
import msgpack

ctx = zmq.Context()
socket = ctx.socket(zmq.REQ)
socket.connect("tcp://127.0.0.1:5555")

socket.send(msgpack.packb(['hello', 'world']))
msg = socket.recv()

print(msgpack.unpackb(msg))
{% endhighlight %}

+ server.php

{% highlight php %}
<?php

$server = new ZMQSocket(new ZMQContext(), ZMQ::SOCKET_REP);
$server->bind("tcp://127.0.0.1:5555");

while ($message = $server->recv()) {
    $data = msgpack_unpack($message);
    var_dump($data);
    $server->send(msgpack_pack(array_reverse($data)));
}
{% endhighlight %}

### demo 6

Make a PHP RPC server

+ client.py

{% highlight python %}
import zmq
import msgpack

ctx = zmq.Context()
socket = ctx.socket(zmq.REQ)
socket.connect("tcp://127.0.0.1:5555")

data = {}
data['function_name'] = 'plus'
data['parameters'] =  [1, 2, 2,3,3,4,2,3,-1]

socket.send(msgpack.packb(data))
answer = socket.recv()

print(msgpack.unpackb(answer))
{% endhighlight %}

+ server.php

{% highlight php %}
<?php
$route = [
    'plus' => function(array $data) {
        return array_sum($data);
    },
];

$server = new ZMQSocket(new ZMQContext(), ZMQ::SOCKET_REP);
$server->bind("tcp://127.0.0.1:5555");

while ($message = $server->recv(100)) {
    $data = msgpack_unpack($message);
    $func_name = $data['function_name'];
    $param = $data['parameters'];

    $answer = $route[$func_name]($param);

    $server->send(msgpack_pack($answer));
}
{% endhighlight %}

+ what's `framework`(laravel framework/Zask framework)
+ what's `IoC`(inversion of control) and `AOP`

-----------

## Reference

+ [http://zguide.zeromq.org/page:all](http://zguide.zeromq.org/page:all)
+ [http://msgpack.org/index.html](http://msgpack.org/index.html)
+ [http://php.net/manual/en/functions.anonymous.php](http://php.net/manual/en/functions.anonymous.php)
