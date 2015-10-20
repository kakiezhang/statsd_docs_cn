Server Interface
-----------------
服务接口
-----------------

Server modules are Node.js [modules][nodemods] that receive metrics for StatsD.
Each server module should export the following initialization function:

服务模块是Node.js[模块][节点]，用于获取StatsD的度量单位。每个服务模块都应该输出以下的初始化函数：

* `start(config, callback)`: This method is invoked from StatsD to initialize
  and start the server module listening for metrics. It accepts two
  parameters: `config` is the parsed config file hash and `callback` is a
  function to call with metrics data, when it's available.

* `start(config, callback)`: StatsD调用这个方法来进行初始化，并且启动服务模块来监听度量单位。它接收两个参数：`config`是经过解析的配置文件的哈希值，`callback`是一个回调函数，当它可用的时候同度量单位数据一起来调用。

  The callback function accepts two parameters: `packet` contains one or more
  metrics separated by the \n character, and `rinfo` contains remote address
  information.

  这个回调函数接收两个参数：`packet`包含了一个或者更多以换行符\n来分隔的度量单位，`rinfo`包含了远程地址信息。

  The server module should return `true` from start() to indicate
  success. A return of `false` indicates a failure to load the module
  (missing configuration?) and will cause StatsD to exit.

  服务模块应该在start()函数中返回`true`来表示成功。返回`false`表示加载模块失败（缺少配置？）并且将会造成StatsD退出。
