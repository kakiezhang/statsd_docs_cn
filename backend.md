Supported Backends
------------------

支持的后端
------------------

StatsD supports pluggable backend modules that can publish
statistics from the local StatsD daemon to a backend service or data
store. Backend services can retain statistics in a time series data store,
visualize statistics in graphs or tables, or generate alerts based on
defined thresholds. A backend can also correlate statistics sent from StatsD
daemons running across multiple hosts in an infrastructure.

StatsD支持插件式的后端模块，这些模块可以将统计数据从本地的StatsD的守护进程发布到一个后端服务或者数据存储。后端服务可以把统计保留在以时间序列的数据存储中，在图形或表格的可视化统计中，或者根据定义的阈值生成报警。后端也可以关联到被发送至在基础架构的多个主机上面运行的StatsD守护进程的统计数据。

StatsD includes the following built-in backends:

StatsD包含以下内建的后端：

* [Graphite][graphite] (`graphite`): An open-source
  time-series data store that provides visualization through a web-browser.
* Console (`console`): Outputs the received
  metrics to stdout (see what's going on during development).
* Repeater (`repeater`): Utilizes the `packet` emit API to
  forward raw packets retrieved by StatsD to multiple backend StatsD instances.

* [Graphite][graphite] (`graphite`): 一个开源的，通过web游览器可进行可视化访问的，时间序列的数据存储。
* Console (`console`): 把接收到的度量单位输出到标准输出（看看在开发过程当中发生了什么）。
* Repeater (`repeater`): 利用`packet`来发送API请求，从而转发被StatsD获取到的原始的报文到多个后端StatsD实例。


By default, the `graphite` backend will be loaded automatically. Multiple
backends can be run at once. To select which backends are loaded, set
the `backends` configuration variable to the list of backend modules to load.

默认情况下，`graphite`后端将会被自动加载。多个后端可以同时运行。为了选择加载哪些后端，设置`backends`这个配置变量到后端模块的列表中来进行加载。


Backends are just npm modules which implement the interface described in
section [Backend Interface](./backend_interface.md). In order to be able to load the backend, add the
module name into the `backends` variable in your config. As the name is also
used in the `require` directive, you can load one of the provided backends by
giving the relative path (e.g. `./backends/graphite`).

后端其实就是npm的模块，实现了在[Backend Interface](./backend_interface.md)章节中所描述的接口。为了能够加载后端，在你的配置当中，增加模块名称到`backends`变量。因为这个模块名也被用在`require`指令当中，你可以通过给出相对路径（比如：`./backends/graphite`）来加载所提供的后端之一。

A robust set of are also available as plugins to allow easy reporting into databases,
queues and third-party services.

这里也有一套强大的集合，可以作为插件那样，以便于把数据汇集到数据库，队列和第三方的服务。

## Available Third-party backends
## 现有的第三方后端

- [amqp-backend](https://github.com/mrtazz/statsd-amqp-backend)
- [atsd-backend](https://github.com/axibase/atsd-statsd-backend)
- [node-bell](https://github.com/eleme/node-bell)
- [couchdb-backend](https://github.com/sysadminmike/couch-statsd-backend)
- [datadog-backend](https://github.com/DataDog/statsd-datadog-backend)
- [elasticsearch-backend](https://github.com/markkimsal/statsd-elasticsearch-backend)
- [ganglia-backend](https://github.com/jbuchbinder/statsd-ganglia-backend)
- [hosted graphite backend](https://github.com/hostedgraphite/statsdplugin)
- [influxdb backend](https://github.com/bernd/statsd-influxdb-backend)
- [instrumental backend](https://github.com/collectiveidea/statsd-instrumental-backend)
- [leftronic backend](https://github.com/sreuter/statsd-leftronic-backend)
- [librato-backend](https://github.com/librato/statsd-librato-backend)
- [mongo-backend](https://github.com/dynmeth/mongo-statsd-backend)
- [monitis backend](https://github.com/jeremiahshirk/statsd-monitis-backend)
- [netuitive backend](https://github.com/Netuitive/statsd-netuitive-backend)
- [opentsdb backend](https://github.com/emurphy/statsd-opentsdb-backend)
- [socket.io-backend](https://github.com/Chatham/statsd-socket.io)
- [stackdriver backend](https://github.com/Stackdriver/stackdriver-statsd-backend)
- [statsd-backend](https://github.com/dynmeth/statsd-backend)
- [statsd http backend](https://github.com/bmhatfield/statsd-http-backend)
- [statsd aggregation backend](https://github.com/wanelo/gossip_girl)
- [zabbix-backend](https://github.com/parkerd/statsd-zabbix-backend)

[graphite]: http://graphite.wikidot.com
