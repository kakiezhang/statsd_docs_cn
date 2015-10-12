TCP Stats Interface
-------------------

TCP统计接口
-------------------

A really simple TCP management interface is available by default on port 8126
or overriden in the configuration file. Inspired by the memcache stats approach
this can be used to monitor a live statsd server.  You can interact with the
management server by telnetting to port 8126, the following commands are
available:

这是一个非常简单的TCP管理接口，可以通过8126端口或者在配置文件当中自行修改来使用。受memcache统计方法的启发，这个接口可用于实时监控statsd服务器。你可以通过telnet 8126端口来进行管理服务的操作，以下是可用的telnet命令：

* stats - some stats about the running server
* counters - a dump of all the current counters
* gauges - a dump of all the current gauges
* timers - a dump of the current timers
* delcounters - delete a counter or folder of counters
* delgauges - delete a gauge or folder of gauges 
* deltimers - delete a timer or folder of timers
* health - a way to set the health status of statsd

* stats - 一些关于正在运行的服务器的统计数据
* counters - 所有当前计数器的堆存值
* gauges - 所有当前压力计数器的堆存值
* timers - 当前计时器的堆存值
* delcounters - 删除一个计数器或者删除计算器的文件夹
* delgauges - 删除一个压力计数器或者删除压力计数器的文件夹
* deltimers - 删除一个计时器或者删除计时器的文件夹
* health - 用于设置statsd健康状态的方法

The stats output currently will give you:

就现在而言，stats将会给你输出：

* uptime: the number of seconds elapsed since statsd started
* messages.last_msg_seen: the number of elapsed seconds since statsd received a message
* messages.bad_lines_seen: the number of bad lines seen since startup

* uptime: 自statsd启动以来所经过的秒数
* messages.last_msg_seen: 从statsd接收到上一个消息后所经过的秒数
* messages.bad_lines_seen: 从statsd启动以来所见的错误行数

You can use the del commands to delete an individual metric like this :

你可以像这样用`del commands`去删除单个度量单位：

    #to delete counter sandbox.test.temporary
    echo "delcounters sandbox.test.temporary" | nc 127.0.0.1 8126

    #删除计数器sandbox.test.temporary
    echo "delcounters sandbox.test.temporary" | nc 127.0.0.1 8126

Or you can use the del command to delete a folder of metrics like this :

或者你可以像这样用`del command`去删除一个度量单位的文件夹：

    #to delete counters sandbox.test.*
    echo "delcounters sandbox.test.*" | nc 127.0.0.1 8126

    #删除计数器sandbox.test.*
    echo "delcounters sandbox.test.*" | nc 127.0.0.1 8126

Each backend will also publish a set of statistics, prefixed by its module name.

每个后端还将公布一组统计数据，以它的模块名为前缀。

Graphite:

* graphite.last_flush: unix timestamp of last successful flush to graphite
* graphite.last_exception: unix timestamp of last exception thrown whilst flushing to graphite
* graphite.flush_length: the length of the string sent to graphite
* graphite.flush_time: the time it took to send the data to graphite

* graphite.last_flush: 上一次将数据成功刷新到graphite的unix时间戳
* graphite.last_exception: 上一次在刷新数据到graphite抛出异常的unix时间戳
* graphite.flush_length: 发送到graphite的字符串长度
* graphite.flush_time: 将数据发送到graphite的时间

Those statistics will also be sent to graphite under the namespaces
`stats.statsd.graphiteStats.last_exception` and
`stats.statsd.graphiteStats.last_flush`.

以上那些统计也可通过以`stats.statsd.graphiteStats.last_exception` 和 `stats.statsd.graphiteStats.last_flush`的命名方式传递到graphite。

A simple nagios check can be found in the utils/ directory that can be used to
check metric thresholds, for example the number of seconds since the last
successful flush to graphite.

在`utils/`目录下可以找到一个简单的nagios检测脚本，它被用来检查度量单位的阈值。比如上一次将数据成功刷新到graphite的秒数。

The health output:

健康状态的输出：

* the health command alone allows you to see the current health status.
* using health up or health down, you can change the current health status.
* the healthStatus configuration option allows you to set the default health status at start.

* 单独一个`health`命令可以让你看到当前的健康状态。
* 使用`health up`或者`health down`，你可以改变当前的健康状态。
* 你被允许可以在开始的时候，就在健康状态的配置选项中设置默认的健康状态。
