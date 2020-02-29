# spring boot + spring cloud 配置

## 1.spring boot 配置

### 1.1 Rabbitmq配置

```yml
spring: 
  # rabbitmq
  rabbitmq:
    addresses: 192.168.0.250:5672,192.168.0.251:5672
    virtual-host: /
    username: admin
    password: Rabbitmq-401
    connection-timeout: 1000
    cache:
      connection:
        size: 2
        mode: connection    
      channel:
        checkout-timeout: 1000
        size: 2

```

[Spring boot Rabbitmq配置好文章](https://www.jianshu.com/p/2c2a7cfdd38a)

***setAddresses\***：设置了rabbitmq的地址、端口，集群部署的情况下可填写多个，“,”分隔。

***setUsername\***：设置rabbitmq的用户名。

***setPassword\***：设置rabbitmq的用户密码。

***setVirtualHost\***：设置virtualHost。

***setCacheMode\***：设置缓存模式，共有两种，*CHANNEL*和*CONNECTION*模式。

*CHANNEL*模式，程序运行期间ConnectionFactory会维护着一个Connection，所有的操作都会使用这个Connection，但一个Connection中可以有多个Channel，操作rabbitmq之前都必须先获取到一个Channel，否则就会阻塞（可以通过setChannelCheckoutTimeout()设置等待时间），这些Channel会被缓存（缓存的数量可以通过setChannelCacheSize()设置）；

*CONNECTION*模式，这个模式下允许创建多个Connection，会缓存一定数量的Connection，每个Connection中同样会缓存一些Channel，除了可以有多个Connection，其它都跟CHANNEL模式一样。

这里的Connection和Channel是spring-amqp中的概念，并非rabbitmq中的概念，官方文档对Connection和Channel有这样的描述：

> Sharing of the connection is possible since the "unit of work" for messaging with AMQP is actually a "channel" (in some ways, this is similar to the relationship between a Connection and a Session in JMS).

关于*CONNECTION*模式中，可以存在多个Connection的使用场景，官方文档的描述：

> The use of separate connections might be useful in some environments, such as consuming from an HA cluster, in conjunction with a load balancer, to connect to different cluster members.

***setChannelCacheSize\***：设置每个Connection中（注意是每个Connection）可以缓存的Channel数量，注意只是缓存的Channel数量，不是Channel的数量上限，操作rabbitmq之前（send/receive message等）要先获取到一个Channel，获取Channel时会先从缓存中找闲置的Channel，如果没有则创建新的Channel，当Channel数量大于缓存数量时，多出来没法放进缓存的会被关闭。

注意，改变这个值不会影响已经存在的Connection，只影响之后创建的Connection。

***setChannelCheckoutTimeout\***：当这个值大于0时，*channelCacheSize*不仅是缓存数量，同时也会变成数量上限，从缓存获取不到可用的Channel时（也就是说，目前缓冲中所有的Channel都已经被借出去，也就是正在被使用，还没有被还回来），不会创建新的Channel，会等待这个值设置的毫秒数，到时间仍然获取不到可用的Channel会抛出AmqpTimeoutException异常。

同时，在*CONNECTION*模式，这个值也会影响获取Connection的等待时间，超时获取不到Connection也会抛出AmqpTimeoutException异常。



