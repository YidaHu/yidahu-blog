# 利用Redis做消息队列实现生产消费和发布订阅

## 前言

在工作中，我们经常会使用队列，在Python中也有原生队列，但是原生的队列是存储在内存中，当重启系统后队列中的数据就会丢失，无法进行分布式。

消息队列最常被使用的三种场景：异步处理、流量控制和服务解耦。当然，消息队列的适用范围不仅仅局限于这些场景，还有包括：

- 作为发布 / 订阅系统实现一个微服务级系统间的观察者模式；
- 连接流计算任务和数据；
- 用于将消息广播给大量接收者。

简单的说，我们在单体应用里面需要用队列解决的问题，在分布式系统中大多都可以用消息队列来解决。

Redis可以作为简单的消息队列来用，但是它毕竟不是专业的消息队列，如果对于有很大的消息队列需求的系统还是考虑使用专业的MQ，在这里只讲解一下利用Redis做简单的消息队列，实现生产消费和发布订阅的场景。

## 生产消费模型

生产者生产消息放到队列里，多个消费者同时监听队列，谁先抢到消息谁就会从队列中取走消息；即对于每个消息只能被最多一个消费者拥有。

### 应用场景

在任务的处理时间比较长的情况下：

比如上传文件并处理，那么这个时候可以将用户上传和处理文件分成两个过程，用一个队列暂时存储用户上传文件的消息，然后立刻返回用户上传成功，然后有专门的线程处理队列中的文件。

在比如双十一的时候，会产生大量的订单，那么不可能同时处理那么多的订单，需要将订单放入一个队列里面，然后由专门的线程处理订单。当然这里需要更加专业的MQ去做了。

### 代码示例

**生产者**

```python
import redis
import random
import logging
from flask import Flask, redirect

app = Flask(__name__)

rcon = redis.StrictRedis(host='localhost', db=5)
prodcons_queue = 'task:prodcons:queue'


@app.route('/producer')
def producer():
    elem = random.randrange(10)
    rcon.lpush(prodcons_queue, elem)
    logging.info("lpush {} -- {}".format(prodcons_queue, elem))
    return redirect('/')


if __name__ == '__main__':
    app.run(debug=True)
```

**消费者**

```python
import redis


class Task(object):
    def __init__(self):
        self.rcon = redis.StrictRedis(host='localhost', db=5)
        self.queue = 'task:prodcons:queue'

    def consumer(self):
        while True:
            task = self.rcon.blpop(self.queue, 0)[1]
            print("Task get", task)


if __name__ == '__main__':
    print('listen task queue')
    Task().consumer()

```



## 发布订阅模型

发布者生产消息放到队列里，多个监听队列的消费者都会收到同一份消息；即正常情况下每个消费者收到的消息应该都是一样的。

比如你打开你的微信订阅号，你订阅的作者发布的文章，会广播给每个订阅者。在这个场景里，微信公众号就是一个Pulisher，而你就是一个Subscriber，你收到的文章就是一个Message。

### 应用场景

1. 应用程序需要向大量消费者广播信息。例如微信订阅号就是一个消费者量庞大的广播平台。
2. 应用程序需要与一个或多个独立开发的应用程序或服务通信，这些应用程序或服务可能使用不同的平台、编程语言和通信协议。

### 代码示例

**发布**

```python
import redis
import random
import logging
from flask import Flask, redirect

app = Flask(__name__)

rcon = redis.StrictRedis(host='localhost', db=5)
pubsub_channel = 'task:pubsub:channel'


@app.route('/pubsub')
def pubsub():
    ps = rcon.pubsub()
    ps.subscribe(pubsub_channel)
    elem = random.randrange(10)
    rcon.publish(pubsub_channel, elem)
    return redirect('/')


if __name__ == '__main__':
    app.run(debug=True)
```

**订阅**

```python
import redis


class Task(object):

    def __init__(self):
        self.rcon = redis.StrictRedis(host='localhost', db=5)
        self.ps = self.rcon.pubsub()
        self.ps.subscribe('task:pubsub:channel')

    def listen_task(self):
        for i in self.ps.listen():
            if i['type'] == 'message':
                print("Task get", i['data'])


if __name__ == '__main__':
    print('listen task channel')
    Task().listen_task()
```

## 总结

Redis可以作为简单的消息队列来用，但是它毕竟不是专业的消息队列，如果对于有很大的消息队列需求的系统还是考虑使用专业的MQ等。

