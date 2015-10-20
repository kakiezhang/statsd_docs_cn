Supported Servers
------------------
现支持的服务
------------------

StatsD supports pluggable server modules that listen for incoming
metrics.

StatsD支持插件式的服务模块，用于监听传入的度量单位。

StatsD includes the following built-in servers:

StatsD包含了以下内建的服务：

* UDP (`udp`): Listens for metrics on a UDP port. One UDP packet contains at
  least one StatsD metric. Multiple metrics can be received in a single packet
  if separated by the \n character.
* UDP (`udp`): 通过UDP端口监听度量单位。一个UDP的数据包至少含有一个StatsD的度量单位。如果以换行符号\n来分隔，那可以在一个单独的数据包中接收多个度量单位。
* TCP (`tcp`): Listens for metrics on a TCP port. Metrics can span multiple TCP
  packets, meaning each metric must be terminated by the \n character when
  received over TCP.
* TCP (`tcp`): 通过TCP端口监听度量单位。度量单位可以跨越多个TCP数据包，这意味着当以TCP来接收时，每个单位的结尾必须以换行符号\n来结束。

By default, the `udp` server will be loaded automatically. Currently only a
single server can be run at once. To select which server is loaded, set
the `server` configuration variable to the server module to load.

默认情况下，`udp`服务会自动加载。目前一次只能运行一个单独的服务。要选择加载哪一个服务，可以将`server`这个配置变量设置成对应的服务模块来进行加载。

Servers are just npm modules which implement the interface described in
section [Server Interface](./server_interface.md). In order to be able to load the server,
add the module name into the `server` variable in your config. As the name is also
used in the `require` directive, you can load one of the provided servers by
giving the relative path (e.g. `./servers/udp`).

服务其实只是npm模块，这些模块实现了在[Server Interface](./server_interface.md)这部分章节当中所描述的接口。为了能加载到服务，在你的配置当中增加模块的名称到`server`变量当中。由于在`require`指令中也会使用这个模块名称，所以你可以通过给出相对路径来加载所提供的服务之一（例如`./servers/udp`）。
