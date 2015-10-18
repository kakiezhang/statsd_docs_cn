Backend Interface
-----------------

后端借口
-----------------

Backend modules are Node.js [modules][nodemods] that listen for a
number of events emitted from StatsD. Each backend module should
export the following initialization function:

后端模块是Node.js的[模块][节点]，可以用来监听从StatsD发出的一系列事件。每个后端模块都应该输出以下初始化的函数：

* `init(startup_time, config, events, logger)`: This method is invoked
  from StatsD to initialize the backend module. It accepts four
  parameters: `startup_time` is the startup time of StatsD in epoch
  seconds, `config` is the parsed config file hash, `events` is the
  event emitter that backends can use to listen for events and
  `logger` is StatsD's configured logger for backends to use.

* `init(startup_time, config, events, logger)`: StatsD在初始化后端模块的时候会调用这个方法。它接受4个参数：`startup_time`是StatsD在启动时的新纪元时间，`config`是解析后的配置文件的哈希值，`events`是事件发生器，它被后端用于监听事件，`logger`是为后端所使用的StatsD的配置日志记录器。

  The backend module should return `true` from init() to indicate
  success. A return of `false` indicates a failure to load the module
  (missing configuration?) and will cause StatsD to exit.

  后端模块应该从init()函数中返回`true`来表示成功。返回`false`表示加载模块失败（遗漏配置？）并且将会造成StatsD退出。

Backends can listen for the following events emitted by StatsD from
the `events` object:

后端可以监听由StatsD从`events`对象中发出的以下事件：

* Event: **'flush'**

* 事件: **'flush'**

  Parameters: `(time_stamp, metrics)`

  参数: `(time_stamp, metrics)`

  Emitted on each flush interval so that backends can push aggregate
  metrics to their respective backend services. The event is passed
  two parameters: `time_stamp` is the current time in epoch seconds
  and `metrics` is a hash representing the StatsD statistics:

  在每次刷新间隔时发出该事件，使得后端可以把汇集的度量单位推送到他们各自的后端服务。这个事件需要传两个参数：`time_stamp`是新纪元的当前时间戳，`metrics`是一个代表了StatsD统计信息的哈希值：

  ```
metrics: {
    counters: counters,
    gauges: gauges,
    timers: timers,
    sets: sets,
    counter_rates: counter_rates,
    timer_data: timer_data,
    statsd_metrics: statsd_metrics,
    pctThreshold: pctThreshold
}
  ```

  The counter_rates and timer_data are precalculated statistics to simplify
  the creation of backends, the statsd_metrics hash contains metrics generated
  by statsd itself. Each backend module is passed the same set of
  statistics, so a backend module should treat the metrics as immutable
  structures. StatsD will reset timers and counters after each
  listener has handled the event.

  这里的counter_rates和timer_data值是为了简化后端的创建而预先计算好的统计数据，statsd_metrics这个哈希值包含了由statsd本身生成的度量单位。每个后端模块都会接收到相同一组统计，所以后端模块应该把这个metrics参数当作是一种不可变的结构。在每个监听者处理了其事件之后，StatsD将会重置timers和counters的值。

* Event: **'status'**

* 事件: **'status'**

  Parameters: `(writeCb)`

  参数: `(writeCb)`

  Emitted when a user invokes a *stats* command on the management
  server port. It allows each backend module to dump backend-specific
  status statistics to the management port.

  当用户在管理服务的端口中调用了*stats*命令时，StatsD会发出这个事件。它会使得每个后端模块来把后端特有的状态统计输出到管理服务的端口。

  The `writeCb` callback function has a signature of `f(error,
  backend_name, stat_name, stat_value)`. The backend module should
  invoke this method with each stat_name and stat_value that should be
  sent to the management port. StatsD will prefix each stat name with
  the `backend_name`. The backend should set `error` to *null*, or, in
  the case of a failure, an appropriate error.

  `writeCb`这个回调函数有着`f(error, backend_name, stat_name, stat_value)`这样的特征。后端模块在调用这个方法的时候，需要带上每个应被发送到管理端口的stat_name和stat_value的值。StatsD将会用`backend_name`作为每个统计名称的前缀。后端应该把`error`的值设为*null*，或者在万一碰到失败的时候，给出一个合适的错误。

* Event: **'packet'**

* 事件: **'packet'**

  Parameters: `(packet, rinfo)`

  参数: `(packet, rinfo)`

  This is emitted for every incoming packet. The `packet` parameter contains
  the raw received message string and the `rinfo` parameter contains remote
  address information from the UDP socket.

  这是在每一个包进入的时候发出的事件。`packet`参数包含了原始的收到的消息字符串，`rinfo`参数包含了从UDP套接字过来的远程地址信息。


