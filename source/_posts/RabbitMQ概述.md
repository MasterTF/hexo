---
title: RabbitMQ概述
date: 2017-02-05 22:27:19
tags:
    - 消息队列
    - RabbitMQ
categories:
    - 技术
---
## 安装
mac上安装rabbitmq比较简单，参考官网教程 http://www.rabbitmq.com/install-standalone-mac.html 即可。遇到了下载很慢问题，找了其他网站上下。

其他系统安装方式在官网也有详细描述，不再累述。

<!-- more -->

## 配置
1. 查找默认配置位置：find / -name "rabbitmq.config.example"，我这边搜索结果是：/usr/share/doc/rabbitmq-server-3.6.1/rabbitmq.config.example
2. 复制默认配置：cp /usr/share/doc/rabbitmq-server-3.6.1/rabbitmq.config.example /etc/rabbitmq/
3. 修改配置文件名：cd /etc/rabbitmq ; mv rabbitmq.config.example rabbitmq.config
4. 开启 Web 界面管理：rabbitmq-plugins enable rabbitmq_management
5. 重启 RabbitMQ 服务：service rabbitmq-server restart
6. 开放防火墙端口：
    - sudo iptables -I INPUT -p tcp -m tcp --dport 15672 -j ACCEPT
    - sudo iptables -I INPUT -p tcp -m tcp --dport 5672 -j ACCEPT
    - sudo service iptables save
    - sudo service iptables restart
7. 浏览器访问：http://localhost:15672 默认管理员账号：guest 默认管理员密码：guest
    
参考https://github.com/judasn/Linux-Tutorial/blob/master/RabbitMQ-Install-And-Settings.md

## 核心概念
### Broker 
简单来说就是消息队列服务器

### Exchange 
消息交换机，服务器中的实体，用来接收生产者发送的消息并将这些消息路由给服务器中的队列

### Queue 
消息队列载体，用来保存消息直到发送给消费者，每个消息都会被投入到一个或多个队列

### Binding 
绑定器，它的作用就是把exchange和queue按照路由规则绑定起来

### Binding Key 
绑定关键字，Exchange视自身类型来决定Binding的路由行为

### Routing Key 
消息的路由关键字，Exchange根据这个关键字决定如何路由某条消息

### vhost 
虚拟主机，一批交换器、消息队列和相关对象，虚拟主机是共享相同的身份认证和加密环境的独立服务器域，一个broker里可以开设多个vhost，用作不同用户的权限分离

### Channel 
多路复用连接中的一条独立的双向数据流通道，为会话提供物理传输介质，在客户端的每个连接里，可建立多个channel

### Consumer 
消费者，一个从消息队列中请求消息的客户端应用程序

### Producer 
生产者，一个向交换器发布消息的客户端应用程序

## 工作流程
![](/img/15255306886088.jpg)

1. 消息生产者将消息发布(Public)到 Exchange 中.
2. Exchange 根据队列的绑定关系将消息分发到不同的 Queue 中.
3. AMQP broker 根据订阅规则将消息发送给消费者 或 消费者自行根据需要从消息队列中获取消息.

Message是当前模型中所操纵的基本单位，它由Producer产生，经过Broker被Consumer所消费。它的基本结构有两部分: Header和Body。Header是由Producer添加上的各种属性的集合，这些属性有控制Message是否可被缓存，接收的queue是哪个，优先级是多少等。Body是真正需要传送的数据，它是对Broker不可见的二进制数据流，在传输过程中不应该受到影响。

## Exchange 和 Exchange 类型


| 类型|默认预定义的名字|
| --------------| ----------------- |
|Direct Exchange|空字符串和 amq.direct|
|Fanout Exchange|amq.fanout|
|Topic Exchange |amq.topic|
|Headers Exchange|amq.match(在 RabbitMQ 中, 额外提供amq.headers)|

### 关于默认 Exchange
默认的 exchange 是一个由 broker 预创建的匿名的(即名字是空字符串) direct exchagne. 对于简单的程序来说, 默认的 exchange 有一个实用的属性: 如果没有显示地绑定 Exchnge, 那么创建的每个 queue 都会自动绑定到这个默认的 exchagne 中, 并且此时这个 queue 的 route key 就是这个queue 的名字.

例如当我们声明了一个名为 "search-indexing-online" 的 queue, 那么 AMQP broker 会以 "search-indexing-online" 作为 route key 将此 queue 绑定到默认的 exchange 中. 因此当一个消息以 route key 为 "search-indexing-online" 投递到默认的 exchange 中时, 此消息就会被路由到这个 queue 中去. 换句话说, 由于有默认的 exchagne 的存在, 我们就好像可以直接将消息投递到指定的 queue 中去而不需要经过 exchange 一样.

## RabbitMQ应用场景
官网中有很详细的样例，各个语言都有，http://www.rabbitmq.com/getstarted.html 此处不再说明。

## 持久化
服务重启时, 是否能恢复队列中的数据.

考虑这样的场景, 当消息被暂存到队列后, 在没有被提取的情况下, RabbitMQ 服务停掉了怎么办.

```python
# -*- coding: utf-8 -*-
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.exchange_declare(exchange='first', type='fanout')
channel.queue_declare(queue='hello')
channel.queue_bind(exchange='first', queue='hello')
channel.basic_publish(exchange='first', routing_key='', body='Hello World!')
```
上面的代码, 我们创建了一条内容为 Hello World! 的消息, 通过命令行工具:

```shell
$  ./rabbitmqctl list_queues
Listing queues ...
hello    1
...done.

$ ./rabbitmqctl list_exchanges
Listing exchanges ...
    direct
amq.direct    direct
amq.fanout    fanout
amq.headers    headers
amq.match    headers
amq.rabbitmq.log    topic
amq.rabbitmq.trace    topic
amq.topic    topic
first    fanout
...done.
```

可以查到, 在名为 hello 的队列中, 有 1 条消息. 有一个类型为 fanout , 名为 first 的交换器.

此时通过 Ctrl-C 或 ./rabbitmqctl stop 把 RabbitMQ 服务停掉, 再重启. 交换器, 队列, 消息都是不会恢复的.

所以, 默认情况下, **消息**, **队列**, **交换器** 都不具有持久化的性质. 如果我们需要持久化功能, 那么在声明的时候就需要配置好.

交换器和队列的持久化性质, 在声明时通过一个 durable 参数即可实现:

```python
channel.exchange_declare(exchange='first', type='fanout', durable=True)
channel.queue_declare(queue='hello', durable=True)
```

这样, 在服务重启之后, first 和 hello 都会恢复. 但是 hello 中的消息不会, 还需要额外配置. 这是 **消息** 的属性的相关内容:

```python
channel.basic_publish(exchange='first',
                      routing_key='',
                      body='Hello World!',
                      properties=pika.BasicProperties(
                         delivery_mode = 2,# make message persistent
                      ))
```



这里注意一下, 消息的持久化并不是一个很强的约束, 涉及数据落地的时机, 及系统层面的 **fsync** 等问题, 不要认为消息完全不会丢. 如果要尽可能高地提高消息的持久化的有效性, 还需要配置其它的一些机制, 比如后面会谈到的 **状态反馈** 中的 **confirm mode**.

**交换器 , 队列, 消息** 这三者的持久化问题都介绍过了. 前两者是一经声明, 则其性质无法再被更改, 即你不能先声明一个非持久化的队列, 再声明一个持久化的同名队列, 企图修改它, 这是不行的. 你重复声明时, 相关参数需要一致. 当然, 你可以删除它们再重新声明:

```python
channel.queue_delete(queue='hello')
channel.exchange_delete(exchange='first')
```

## 调度策略
交换器如何把消息给到哪些队列, 是每个队列给一条, 或者把一条消息给多个队列.

我们考虑交换器 Exchange 和队列 Queue 的关系. Exchange 在得到消息后会依据规则把消息投到一个或多个队列当中.

在调度策略方面, 有两个需要了解的地方, 一是交换器的类型(前面我们用的是 fanout), 二是交换器和队列的绑定关系. 在绑定了的前提下, 我们再谈不同类型的交换器的规则. 绑定动作本身也会影响交换器的行为.

交换器的类型, 内置的有四种, 分别是:

- fanout
- direct
- topic
- headers

下面一一介绍.

### fanout
故名思义, fanout 类型的交换器, 其行为是把消息转发给所有绑定的队列上, 就是一个"广播"行为.

```pyton
# -*- coding: utf-8 -*-

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='first', type='fanout')
channel.queue_declare(queue='A')
channel.queue_declare(queue='B')
channel.queue_declare(queue='C')

channel.queue_bind(exchange='first', queue='A')
channel.queue_bind(exchange='first', queue='B')

channel.basic_publish(exchange='first',
                      routing_key='',
                      body='Hello World!')
```

运行 N 次, 通过 rabbitmqctl 可以看到 A 和 B 中就有 N 条消息, 而 C 中没有消息. 因为只有 A 和 B 是绑定到了 first 上的.

### direct
direct 类型的行为是"先匹配, 再投送". 即在绑定时设定一个 routing_key , 消息的 routing_key 匹配时, 才会被交换器投送到绑定的队列中去

```python
# -*- coding: utf-8 -*-

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='first', type='direct')
channel.queue_declare(queue='A')
channel.queue_declare(queue='B')

channel.queue_bind(exchange='first', queue='A', routing_key='a')
channel.queue_bind(exchange='first', queue='B', routing_key='b')

channel.basic_publish(exchange='first',
                      routing_key='a',
                      body='Hello World!')
```

A 和 B 虽然都绑定在了类型为 direct 的 first 上, 但是绑定时的 routing_key 不同.

当一个 routing_key 为 a 的消息出来时, 只会被 first 投送到 A 里.

### topic
topic 和 direct 类似, 只是匹配上支持了"模式", 在"点分"的 routing_key 形式中, 可以使用两个通配符:

- \* 表示一个词.
- \# 表示零个或多个词.

```python
# -*- coding: utf-8 -*-

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='first', type='topic')
channel.queue_declare(queue='A')
channel.queue_declare(queue='B')

channel.queue_bind(exchange='first', queue='A', routing_key='a.*.*')
channel.queue_bind(exchange='first', queue='B', routing_key='a.#')

channel.basic_publish(exchange='first',
                      routing_key='a',
                      body='Hello World!')

channel.basic_publish(exchange='first',
                      routing_key='a.b.c',
                      body='Hello World!')
```

在发出的两条消息当中, a 只会被 a.# 匹配到. 而 a.b.c 会被两个都匹配到.

所以, 最终的结果会是 A 中有一条消息, B 中有两条消息.

### headers
headers 也是根据规则匹配, 相较于 direct 和 topic 固定地使用 routing_key , headers 则是一个自定义匹配规则的类型.

在队列与交换器绑定时, 会设定一组键值对规则, 消息中也包括一组键值对( headers 属性), 当这些键值对有一对, 或全部匹配时, 消息被投送到对应队列.

```python
# -*- coding: utf-8 -*-

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='first', type='headers')
channel.queue_declare(queue='A')
channel.queue_declare(queue='B')

channel.queue_bind(exchange='first', queue='A', arguments={'a': '1'})
channel.queue_bind(exchange='first', queue='B', arguments={'b': '2', 'c': 3, 'x-match': 'all'})

channel.basic_publish(exchange='first',
                      routing_key='',
                      properties=pika.BasicProperties(
                          headers = {'a': '2'},
                      ),
                      body='Hello World!')

channel.basic_publish(exchange='first',
                      routing_key='',
                      properties=pika.BasicProperties(
                          headers = {'a': '1', 'b': '2'},
                      ),
                      body='Hello World!')
 ```

绑定时, 通过 arguments 参数设定匹配规则, x-match 是一个特殊的规则, 表示需要全部匹配上, 还是只匹配一条:

- all , 全部匹配.
- any , 只匹配一个.

消息的 headers 属性会用于规则的匹配.

上面的代码中, 第一条消息不会匹配任何规则. 第二条消息, 匹配到 A , 但是不会匹配到 B (虽然有一条 b:2 ).

最终的结果是, A 中有一条消息, B 中没有消息.

## 分配策略
队列面对消费者时, 如何把消息吐出去, 来一个消费者就把消息全给它, 还是只给一条.

调度策略是影响 Exchange 是不是要把消息给 Queue , 而分配策略影响队列如何把消息给 Consuming .

考虑这样的场景: 队列中有多条消息, 每一个消费者取出消息后, 都要花 10 秒来处理它, 处理完一条消息之后才可能再取出一条继续处理. 刚开始只有一个消费者, 过了 2 秒后来了第二个消费者, 此时, 这两个消费者获取消息的行为是一个什么状态?

我们的需求可能是, 当一个消费者来时, 只给它一条消息, 等它再"请求"时, 再给. 或者也可能是, 当有消费者时, 就把目前有的消息全给它(因为不知道是否还有其它的消费者, 所以既然来了一个就让它尽量多处理一些消息).

先产生一些等待处理的消息:

```python
# -*- coding: utf-8 -*-

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='first', type='fanout')
channel.queue_declare(queue='A')

channel.queue_bind(exchange='first', queue='A')

for i in range(10):
    channel.basic_publish(exchange='first',
                          routing_key='',
                          body=str(i))
``` 

然后是消费者的实现:

```python
# -*- coding: utf-8 -*-

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='A')

def callback(ch, method, properties, body):
    import time
    time.sleep(10)
    print body

channel.basic_consume(callback, queue='A', no_ack=True)
channel.start_consuming()
``` 

上面的代码, 是假设处理一条消息需要 10 秒的时间. 但是事实上, 你只要一执行代码, 马上再使用 rabbitmqctl 查看队列状态时, 会发现队列已经空了. 因为在关闭 ack 的情况下, Queue 的行为是, 一旦有消费者请求, 那么当前队列中的消息它都会一次性吐很多出去.

ack 机制在后面 状态反馈 会介绍到, 简单来说是一种确认消息被正确处理的机制.

如果我们想一次只吐一条消息, 当其它消费者连上来时, 还可以并行处理, 简单地把 ack 打开就可以了(默认就是打开的).

再考虑一下细节. 当有多个消费者连上时, 它是从队列一次取一条消息, 还是一次取多条消息(这样至少可以改善性能). 这可以通过配置 channel 的 qos 相关参数实现:

```python
# -*- coding: utf-8 -*-importpika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='A')
channel.basic_qos(prefetch_count=2)

defcallback(ch, method, properties, body):
    importtime
    time.sleep(10)
    print body
    ch.basic_ack(delivery_tag = method.delivery_tag)

channel.basic_consume(callback, queue='A', no_ack=False)
channel.start_consuming()
```

通过配置 prefetch_count 参数, 来设置一次从队列中取多少条消息. 要看到效果, 至少需要启 2 个消费者.

之前是 10 个数字按顺序入了队列, channel 的配置是一次取 2 个, 那么启 2 个消费者的话, 过 10 秒, 在两个消费者的输出中分别能看到 0 , 2 . 这时把两个消费者都 Ctrl-C , 通过 rabbitmqctl 能看到 A 队列中还有 8 条消息.

## 状态反馈
当消息从某一个队列中被提出后, 这个消息的生命周期就此结束, 还是说需要一个具体的信号以明确标识消息已被正确处理.

状态反馈 的功能目的是为了确认行为的结果. 比如, 当你向 Exchange 提交一个消息时, 这个消息是否提交成功, 是否送达到了队列中. 当你从队列中提取消息之后, RabbitMQ 的 Server 如何处理, 因为在提取消息之后, Consuming 可能判断消息有问题, 可能在处理的过程中出现了异常.

在一些关键的节点上, 要保证消息的正确处理, 安全处理, 是需要很多细节上的控制的. AMQP 协议本身也为此作了相关设计, 甚至是事务机制. 事实上在 AMQP 中要确保消息的业务可靠性只能使用事务, 不过在 RabbitMQ 中有一些相应的简便的扩展机制来达到同样目的.

### 信息发布的确认
回看一下之前的一段代码:

```python
channel.basic_publish(exchange='first', routing_key='', body='Hello World!')
```

这段代码要做的事, 是把一条消息发给名为 first 的交换器. 这个过程中可能出现意外:

- exchange 的名字写错了.
- exchange 得到消息后, 发现没有对应的 queue 可以投送.
- 投送到 queue 后当前没有消费者来提取它.

上面的三种情况, 第一种, 会直接引发一个调用错误. 第三种, 通常不是问题, 反正消息会在 queue 中暂存. 但是第二种情况很多时候是需要避免的, 否则消息就丢失了, 更严重的是 Producing 对此浑然不知.

在这个地方, 我们就需要确认消息发出之后, 是否成功地被投送到 queue 中去了(或者知道它不能被投送到任何 queue 中去).

要确认这些状态信息, 首先需要把 channel 设置到 confirm mode , 也称之为 Publisher Acknowledgements 机制 (和消息的 ack 机制区分开). 它的目的就是为了确认 Producing 发出的信息的状态.

打开 confirm mode 的方法是:

```python
channel.confirm_delivery()
```

之后的 publish 行为就可以收到服务器的反馈. 比如在 basic_publish 函数中, 通过 mandatory=True 参数来确认发出的消息是否有 queue 接收, 并且所有 queue 都成功接收.

```python
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='first', type='fanout')
channel.queue_declare(queue='A')

#channel.queue_bind(exchange='first', queue='A')

channel.confirm_delivery()

r = channel.basic_publish(exchange='first',
                          routing_key='',
                          body='Hello',
                          mandatory=True,
                         )
print r
```

上面的代码中, 因为名为 first 的 Exchange 没有绑定任何的 queue , 在 mandatory 参数的作用下, basic_publish 会返回 False .

对于持久化性质, confirm mode 的确认结果是表示, 一条 persisting 的消息, 投送给一个 durable 的队列成功, 并且数据已经成功写到磁盘. 当然, 因为系统缓存的问题, 为确保数据成功落地, 得到确认信息有时可能需要长达几百毫秒的时间, 应用对此应该有所准备, 而不至于在性能上受此影响.

### 消息提取的确认
在未关闭消息的 ack 机制的情况下, 当消息被 Consuming 从队列中提取后, 在未明确获取确认信息之前, 队列中的消息是不会被删除的. 这样, 流程上就变成, 当消息被提取之后, 队列中的这条消息处于"等待确认"的状态. 如果 Consuming 反馈"成功"给队列, 则消息可以安全地被删除了. 如果反馈"拒绝"给队列, 则消息可能还需要再次被其它 Consuming 提取.

看下面的例子, 我们先创建顺序的 10 个数字为内容的 10 条消息:

```python
# -*- coding: utf-8 -*-

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

channel.exchange_declare(exchange='first', type='fanout')
channel.queue_declare(queue='A')

channel.queue_bind(exchange='first', queue='A')

for i in range(10):
    channel.basic_publish(exchange='first', routing_key='', body=str(i))
```

提取消息的逻辑:

```python
# -*- coding: utf-8 -*-

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
r = channel.basic_get(queue='A', no_ack=False) #0
print r[-1], r[0].delivery_tag
```

上面的代码会提取第一条消息, 但是并没有向 Queue 反馈此消息是否被正确处理, 所以这条消息在队列中仍然存在, 直到 Connection 被释放后, 被提取过但是未被确认的消息的状态被重置, 它就可以被重新提取.

要确认消息, 或者拒绝消息, 使用对应的 basic_ack 和 baskc_reject 方法:

```python
# -*- coding: utf-8 -*-

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
r = channel.basic_get(queue='A', no_ack=False) #0
print r[-1], r[0].delivery_tag
#channel.basic_ack(delivery_tag=r[0].delivery_tag)
channel.basic_reject(delivery_tag=r[0].delivery_tag) 
```
AMQP 协议中, 只提供了 reject 方法, 它只能处理一条消息. 因为 Consuming 是可以一次性提取多条消息的, 所以 RabbitMQ 为此做了扩展, 提供了 basic_nack 方法, 它和 basic_reject 的唯一区别就是支持一次性拒绝多条消息.

```python
# -*- coding: utf-8 -*-

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
r = channel.basic_get(queue='A', no_ack=False) #0
r = channel.basic_get(queue='A', no_ack=False) #1
r = channel.basic_get(queue='A', no_ack=False) #2
channel.basic_nack(delivery_tag=r[0].delivery_tag, multiple=True)
```

delivery_tag 是在 channel 中的一个消息计数, 每次消息提取行为都对应一个数字. nack 的 multiple 机制会自动把不大于指定 delivery_tag 的消息提取都 reject 掉.

在 reject 和 nack 中还有一个 requeue 参数, 表示被拒绝的消息是否可以被重新分配. 默认是 True . 如果消息被 reject 之后, 不希望再被其它的 Consuming 得到, 可以把 requeue 参数设置成 False :

```python
# -*- coding: utf-8 -*-

import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
r = channel.basic_get(queue='A', no_ack=False) #0
channel.basic_nack(delivery_tag=r[0].delivery_tag, multiple=False, requeue=False)
```
basic_consume 和 basic_get 都是从指定 queue 中提取消息, 前者是一个更高层的方法, 还支持 qos 等.



