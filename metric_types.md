StatsD Metric Types
==================

StatsD度量单位的类型
==================


Counting
--------

计数
--------

    gorets:1|c

This is a simple counter. Add 1 to the "gorets" bucket.
At each flush the current count is sent and reset to 0.
If the count at flush is 0 then you can opt to send no metric at all for
this counter, by setting `config.deleteCounters` (applies only to graphite
backend).  Statsd will send both the rate as well as the count at each flush.

这是一个简单的计数器。往"gorets"这个桶内增加1。
在每一次刷新数据到后端的时候，当前的计数都会被发送出去并且被重置为0。
如果在刷新数据的时候计数为0，那么你可以选择通过设置`config.deleteCounters`（仅仅适用于graphite的后端）的值，来做到不发送任何一个度量单位到这个计数器。Statsd将会在每次刷新数据的时候，同时把速率和计数的值发送过去。

### Sampling

### 采样

    gorets:1|c|@0.1

Tells StatsD that this counter is being sent sampled every 1/10th of the time.

告诉StatsD这个计数器以每隔10分之一的时间间隔发送采样。

Timing
------

计时
------

    glork:320|ms|@0.1

The glork took 320ms to complete this time. StatsD figures out percentiles,
average (mean), standard deviation, sum, lower and upper bounds for the flush interval.
The percentile threshold can be tweaked with `config.percentThreshold`.

这次这个glork花了320毫秒完成。StatsD会为刷新时间间隔计算出百分比，平均值，标准差，总和，上下限。
百分比的阈值可以通过设置`config.percentThreshold`来进行调整。

The percentile threshold can be a single value, or a list of values, and will
generate the following list of stats for each threshold:

百分比的阈值可以是单个值，或是一个列表，并且将会为每一个阈值生成以下一列统计：

    stats.timers.$KEY.mean_$PCT
    stats.timers.$KEY.upper_$PCT
    stats.timers.$KEY.sum_$PCT

Where `$KEY` is the stats key you specify when sending to statsd, and `$PCT` is
the percentile threshold.

这里`$KEY`就是那个发送到statsd时候你所指定的统计键，`$PCT`就是百分比阈值。

Note that the `mean` metric is the mean value of all timings recorded during
the flush interval whereas `mean_$PCT` is the mean of all timings which fell
into the `$PCT` percentile for that flush interval. And the same holds for sum
and upper. See [issue #157](https://github.com/etsy/statsd/issues/157) for a
more detailed explanation of the calculation.

需要注意的是，`mean`这个度量单位是在刷新时间间隔期间，所有计时纪录的平均值，因此`mean_$PCT`就是在那个刷新时间间隔中落到百分之`$PCT`中的那些所有的计时的平均值。并且这一套理论同样适用于总和以及上限。一个关于计算更加详细的解释，可见 [issue #157](https://github.com/etsy/statsd/issues/157) 。

If the count at flush is 0 then you can opt to send no metric at all for this timer,
by setting `config.deleteTimers`.

如果在刷新时的计数为0，你可以选择通过设置`config.deleteTimers`，来做到不发送任何一个度量单位到这个计时器。

Use the `config.histogram` setting to instruct statsd to maintain histograms
over time.  Specify which metrics to match and a corresponding list of
ordered non-inclusive upper limits of bins (class intervals).
(use `inf` to denote infinity; a lower limit of 0 is assumed)
Each `flushInterval`, statsd will store how many values (absolute frequency)
fall within each bin (class interval), for all matching metrics.
Examples:

用`config.histogram`这个配置让statsd随时间来维持直方图。指定需要匹配哪种度量单位，并且指定一组对应的有序不包含上限的分区（分类间隔）。
（用`inf`来表示无穷大；假设0为下限）
每个`flushInterval`，statsd都将会为所有匹配的度量单位，存储落在每个分区（分类间隔）内的值。例子：

* no histograms for any timer (default): `[]`
* histogram to only track render durations,
  with unequal class intervals and catchall for outliers:

        [ { metric: 'render', bins: [ 0.01, 0.1, 1, 10, 'inf'] } ]

* histogram for all timers except 'foo' related,
  with equal class interval and catchall for outliers:

        [ { metric: 'foo', bins: [] },
          { metric: '', bins: [ 50, 100, 150, 200, 'inf'] } ]


* 没有直方图供给计时器（默认）： `[]`
* 通过不相等的分类间隔和包罗万象的异常值，只用来追踪render的持续时间的直方图：

        [ { metric: 'render', bins: [ 0.01, 0.1, 1, 10, 'inf'] } ]

* 通过相等的分类间隔和包罗万象的异常值，除了与'foo'相关所有的计时器的直方图：

        [ { metric: 'foo', bins: [] },
          { metric: '', bins: [ 50, 100, 150, 200, 'inf'] } ]

Statsd also maintains a counter for each timer metric. The 3rd field
specifies the sample rate for this counter (in this example @0.1). The field
is optional and defaults to 1.

Statsd也为每个计时器的度量单位维持了一个计数器。第三个字段为这个计数器指明了它的采样率（这个例子中是@0.1）。这个字段是可选项，并且它的默认值是1。

Note:

注：

* first match for a metric wins.
* bin upper limits may contain decimals.
* this is actually more powerful than what's strictly considered
histograms, as you can make each bin arbitrarily wide,
i.e. class intervals of different sizes.

* 第一个需要匹配得到度量单位.
* 分区的上限可能包含小数。
* 这其实是一个经过严格深思熟虑后的更加强大的直方图，因为你可以使你的每个分区都足够大，比如：不同大小的分类间隔。

Gauges
------

计量
------

StatsD now also supports gauges, arbitrary values, which can be recorded.

StatsD现在也支持计量，任意的值都可以被记录。

    gaugor:333|g

If the gauge is not updated at the next flush, it will send the previous value. You can opt to send
no metric at all for this gauge, by setting `config.deleteGauges`

如果这个计量在下一次的刷新中没有被更新，那statsd会发送上一次的值。你可以选择设置`config.deleteGauges`的值，来做到不发送任何一个度量单位到这个计量。

Adding a sign to the gauge value will change the value, rather than setting it.

如果不是直接去设置它，那给计量值增加一个符号，将会改变它的值。

    gaugor:-10|g
    gaugor:+4|g

So if `gaugor` was `333`, those commands would set it to `333 - 10 + 4`, or
`327`.

所以如果`gaugor`的值是`333`，那些命令行可以将它设置为`333 - 10 + 4`，或者`327`。

Note:

注：

This implies you can't explicitly set a gauge to a negative number
without first setting it to zero.

这意味着你需要在第一次设置它的值为0，才可以明确的设置一个计量为负数。

Sets
----

集合
----

StatsD supports counting unique occurences of events between flushes,
using a Set to store all occuring events.

用一个集合存储所有发生的事件，这样StatsD就可以计算出发生在刷新时的不同事件。

    uniques:765|s

If the count at flush is 0 then you can opt to send no metric at all for this set, by
setting `config.deleteSets`.

如果这个计数在刷新时为0，那你可以选择设置`config.deleteSets`的值，来做到不发送任何一个度量单位到这个集合。

Multi-Metric Packets
--------------------

多单位的报文
--------------------

StatsD supports receiving multiple metrics in a single packet by separating them
with a newline.

通过把度量单位之间用换行符分隔开，StatsD支持在一个报文中接受多个度量单位。

    gorets:1|c\nglork:320|ms\ngaugor:333|g\nuniques:765|s

Be careful to keep the total length of the payload within your network's MTU. There
is no single good value to use, but here are some guidelines for common network
scenarios:

小心保持报文有效的荷载总长度在你的网络MTU值内。这里没有一个很好的单独的用例，但是有一些关于普通网络场景的使用指南：

* Fast Ethernet (1432) - This is most likely for Intranets.
* Gigabit Ethernet (8932) - Jumbo frames can make use of this feature much more
  efficient.
* Commodity Internet (512) - If you are routing over the internet a value in this
  range will be reasonable. You might be able to go higher, but you are at the mercy
  of all the hops in your route.

* 高速以太网 (1432) - 极大部分为内部网。
* 千兆以太网 (8932) - 大型的框架可以很有效的利用这个特征。
* 初级网络 (512) - 如果你是通过路由来上网，那么这会是一个合理的区间段。你可能想把这个值调的更高，但是这样你的线路可能会受到跳线的影响。

*(These payload numbers take into account the maximum IP + UDP header sizes)*

*(需要把最大IP + UDP的报头大小给考虑到这些有效荷载的数量当中)*
