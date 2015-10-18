Statsd Cluster Proxy
==============

Statsd集群代理
==============

Statsd Cluster Proxy is a udp proxy that sits infront of multiple statsd instances.

Statsd集群代理是一个udp的代理，其位置在多个statsd的实例之前。


Create a proxyConfig.js file:

创建一个proxyConfig.js文件：

  `cp exampleProxyConfig.js proxyConfig.js`

Once you have modified your config file run:

一旦修改好了你的配置文件后，运行：
  
  `node proxy.js proxyConfig.js`


It uses a consistent hashring to send the unique metric names to the same statsd instances so that
the aggregation works properly.

它采用了一致性的哈希环来发送各自唯一的度量单位名称到相同statsd实例，以次来保证能够进行正常的聚合。

It handles a simple health check that dynamically recalculates the hashring if a statsd instance goes offline.

如果有一个statsd实例下线了，那么会有一个简单的健康检查去动态的重新计算这个哈希环。

Config Options are documented in the [exampleProxyConfig.js][exampleProxyConfig.js]

这里[exampleProxyConfig.js][exampleProxyConfig.js]有记录配置文件的选项

Notes
--------------
注
--------------
In your statsd configuration make sure to have the following configuration set: `deleteIdleStats: true`

需要确保在你的statsd配置文件当中有一下的配置项：`deleteIdleStats: true`

We plan to remove this restriction in the near future: [#pull/348][pull_348]

我们计划在不久的将来删除这个限制：[#pull/348][pull_348]

[exampleProxyConfig.js]: https://github.com/etsy/statsd/blob/master/exampleProxyConfig.js
[pull_348]: https://github.com/etsy/statsd/pull/348
