Configuring Graphite for StatsD 
-------------------------------

配置StatsD的Graphite 
-------------------------------

Many users have been confused to see their hit counts averaged, gone missing when
the data is intermittent, or never stored when statsd is sending at a different
interval than graphite expects. Careful setup of Graphite as suggested below should help to alleviate all these issues. When configuring Graphite, two main factors you need to consider are:

很多用户一直很迷惑，当数据出现间歇性，或在statsd以一个不同于graphite所期望的时间间隔发送数据的时候，如何去看他们命中的平均次数。按如下建议的Graphite的仔细的设置应该可以帮助你缓解这些问题。当配置Graphite时，你需要考虑两个主要因素：

1. What is the highest resolution of data points kept by Graphite, and at which points in time is data downsampled to lower resolutions. This decision is by nature directly related to your functional requirements: how far back should you keep data? what is the data resolution you actually need? However, the retention rules you set must also be in sync with statsd.

1. Graphite所保持的数据点当中的最高分辨率是什么，并且在哪一个点以较低的分辨率去向下取样数据。这个决定本质上直接关系到了你的功能性需求：你应该保留多久的数据？你真正需要的数据分辨率是多少？然而，你设置的保留规则也必须与statsd进行同步。

2. How should data be aggregated when downsampled, in order to correctly preserve its meaning? Graphite of course knows nothing of the 'meaning' of your data, so let's explore the correct setup for the various metrics sent by statsd.

2. 为了正确的保留其含义，当向下采样时数据该如何聚合？Graphite当然不知道你数据的“意义”，所以让我们来探索一下由statsd发送的各种指标的正确的设置说明。

### Storage Schemas
### 存储方式

To define retention and downsampling which match your needs, edit Graphite's conf/storage-schemas.conf file. Here is a simple example file that would handle all metrics sent by statsd:

要定义保留和向下采样哪个是你需要的，编辑Graphite的conf/storage-schemas.conf文件。这儿有一个简单的示例文件，它将会处理statsd发送的所有指标项：

    [stats]
    pattern = ^stats.*
    retentions = 10s:6h,1min:6d,10min:1800d

This translates to: for all metrics starting with 'stats' (i.e. all metrics sent by statsd), capture:

这就是说：对于所有以“stats”开头的指标项（即由statsd发送的所有指标项），捕捉：

* 6 hours of 10 second data (what we consider "near-realtime")
* 6 days of 1 minute data
* 5 years of 10 minute data

* 10秒钟的数据，存储6小时（就是我们所认为的“近似实时”）
* 1分钟的数据，存储6天
* 10分钟的数据，存储5年

These settings have been a good tradeoff so far between size-of-file (database files are fixed size) and data we care about. Each "stats" database file is about 3.2 megs with these retentions.

到目前为止，这些配置在有大小的文件（数据库文件是固定大小的）和我们所关心的数据当中，起到了权衡的作用。每个带有这些保留值的“stats”数据库文件大约有3.2兆。

Retentions are read from the file in order and the first pattern that matches is used. 

保留值都是按顺序从文件中读取，然后应用匹配得到的第一个模式。

Graphite stores each metric in its own database file, and the retentions take effect when a metric file is first created. This means that changing this config file would not affect any files already created. To view or alter the settings on existing files, use whisper-info.py and whisper-resize.py included with the Whisper package.

Graphite把每个指标项存储在它自己的数据库文件当中，在第一次创建指标项文件的时候，保留值才会生效。这意味着修改这个配置文件将不会影响任何已经被创建的文件。要查看或更改现有文件的设置，使用附带在Whisper包中的whisper-info.py和whisper-resize.py。

#### Correlation with statsd's flush interval:
#### 与statsd的刷新时间间隔有关：

In the case of the above example, what would happen if you flush from statsd any faster than every 10 seconds? in that case, multiple values for the same metric may reach Graphite at any given 10-second timespan, and only the last value would take hold and be persisted - so your data would immediately be partially lost. 

按上面例子的情况，如果你以比每10秒都快的速度从statsd进行刷新，会发生什么？这种情况下，对同一指标项将会有多个值可以在任意给定的10秒的时间跨度中达到Graphite，并且将获取最后一个值进行持久化 － 所以你的数据会有一部分被立即丢失。

To fix that, simply ensure your flush interval is at least as long as the highest-resolution retention. However, a long interval may cause other unfortunate mishaps, so keep reading - it pays to understand what's really going on.

要解决这个问题，只需简单的确保你的刷新间隔时间至少要和最高分辨率的保留值一样长。但是，一个长的间隔又可能会造成其他不幸的事故，所以继续往下看 － 这是值得我们去理解其中真正发生了些什么的。

(Note: Older versions of Graphite do not support the human-readable time format shown above)

（注：老版本的Graphite不支持上面所显示的人类可读的时间格式）

### Storage Aggregation
### 聚合存储

The next step is ensuring your data isn't corrupted or discarded when downsampled. Continuing with the example above, take for instance the downsampling of .mean values calculated for all statsd timers: 

下一步，在向下取样的时候，确保你的数据没有被损坏或者丢弃。继续上个例子，就拿经过计算的向下取样的.mean值来用于所有statsd的计时器：

Graphite should downsample up to 6 samples representing 10-second mean values into a single value signfying the mean for a 1-minute timespan. This is simple: just average all samples to get the new value, and this is exactly the default method applied by Graphite. However, what about the .count metric also sent for timers? Each sample contains the count of occurences per flush interval, so you want these samples summed-up, not averaged! 

Graphite应该向下取样6个采样本，一个样本为10秒钟的平均值，这样表示成以1分钟的时间跨度为单位的平均值。这里有个例子：仅仅取所有样本的平均值，这正好是Graphite提供的默认方法。然而，假设将.count度量也发送到计时器会如何呢？每个样本包含每个刷新间隔的出现次数，所以你想要的是这些样本的总和，而不是平均值。

You would not even notice any problem till you look at a graph for data older than 6 hours ago, since Graphite would need only the high-res 10-second samples to render the first 6 hours, but would have to switch to lower resolution data for rendering a longer timespan.

在看到6个小时前的数据图之前，你甚至不会注意到任何问题，由于Graphite只需要10秒钟频度的样本来渲染第一个6小时的图，但是必须切换到较低频率的数据以呈现较长的时间间隔。

Two other metric kinds also deserve a note: 

还有另外两种度量类型也值得注意：

* Counts which are normalized by statsd to signify a per-second count should not be summed, since their meaning does not change when downsampling. 

* 通过statsd标准化的Counts度量是用来表示每秒钟的计数而不应该是求和，因为他们求平均的值在向下采样时不会改变。

* Metrics for minimum/maximum values should not be averaged but rather preserve the lowest/highest point, respectively.

* 最小 / 最大值的指标不应取平均值，而应该分别保留最低 / 最高点。

Let's see now how to configure downsampling in Graphite's conf/storage-aggregation.conf:

现在让我们来看看在Graphite的配置（conf/storage-aggregation.conf）当中如何配置向下取样：

    [min]
    pattern = \.lower$
    xFilesFactor = 0.1
    aggregationMethod = min

    [max]
    pattern = \.upper(_\d+)?$
    xFilesFactor = 0.1
    aggregationMethod = max

    [sum]
    pattern = \.sum$
    xFilesFactor = 0
    aggregationMethod = sum

    [count]
    pattern = \.count$
    xFilesFactor = 0
    aggregationMethod = sum

    [count_legacy]
    pattern = ^stats_counts.*
    xFilesFactor = 0
    aggregationMethod = sum

    [default_average]
    pattern = .*
    xFilesFactor = 0.3
    aggregationMethod = average

This means:

这儿的意思是：

* For metrics ending with '.lower' or '.upper' (these are sent for all timers), keep only the minimum and maximum value when rolling up data and store a None if less than 10% of the datapoints were received.

* 对于以“.lower”或“.upper”结尾的指标单位（这些是为所有计时器发送的），在收集数据时，除非接收到少于10％的数据点不存储，其他情况下只保留最小和最大值。

* For metrics ending with 'count' or 'sum' in the name, or those under 'stats_counts', add all the values together, and store a None only if none of the datapoints were received. This would capture all non-normalized counters, but ignore the per-second ones.

* 对于在名称中以“count”或“sum”结尾的指标单位，或者那些在“stats_counts”下的，将所有值添加在一起，并仅在未接收到任何数据点时不存储。这将捕获所有非标准化的计数器，但会忽略每秒钟的那些。

* For all other databases, average the values (mean) when rolling up data, and
  store a None if less than 30% of the datapoints were received

* 对于所有其他的数据来说，在收集数据时，除非接收到少于30％的数据点不存储，其他情况下求平均值。

Pay close attention to xFilesFactor: if your flush interval is not long enough so there are not enough samples to satisfy this minimum factor, your data would simply be lost in the first downsampling cycle. However, setting a very low factor would also produce a misleading result, since you would probably agree that if you only have a single 10-second mean value sample reported in a 10-minute timeframe, this single sample alone should not normally be downsampled into a 10-minute mean value. For counts, however, every count should count ;-), hence the zero factor.

密切关注一下xFilesFactor：如果你的刷新间隔不够长而导致没有足够的样本来满足最小因子，你的数据将会在第一个降采样周期中丢失。然而，设置一个非常低的因子也将会产生误导结果，因为你可能会同意在10分钟的时间范围内报告只有一个10秒钟的平均值样本，该单个样本通常不应该单独被下采样为10分钟的平均值。然而对counts来说，每个count都应该统计 ;-)，因此因子为0。

**Notes:**

**注意事项：**

1. a '.count' metric is calculated for all timers, but up to and including v0.5.0, non-normalized counters are written under stats_counts - not under stats.counters as you might expect. Post-0.5.0, if you set legacyNamespace=false in the config then counters would indeed be written under stats.counters, in two variations: per-second counts under stats.counters.\<name\>.*rate*, and non-normalized per-flush counts under stats.counters.\<name\>.*count*. Hence, the rules above would handle counts for both timers and legacy/non-legacy counters.

1. “.count”指标单位会为所有的计时器进行计算，但直到并且包括v0.5.0在内的，写在stats_counts下的非标准化计数器 － 正如你所期望的不在stats.counters下的。0.5.0之后的版本，如果你在配置当中设置了legacyNamespace=false，那么计数器实际上会被写在stats.counters下的两个变量当中：stats.counters.\<name\>.*rate*下面的每秒钟的统计，和在stats.counters.\<name\>.*count*下面的非规范化的每次刷新的统计。因此，上述规则将处理计时器和传统/非传统计数器的统计。

2. upper and lower values are also calculated for the n-percentile value defined for timers. The above example does not include rules for these, for brevity and performance.

2. 上限值和下限值还被用在为定时器计算百分点值。为了尽可能简洁，上面的例子不包含这些规则。

Similar to retentions, the aggregations in effect for any metric are set once the metric is first received, so a change to these settings would not affect existing metrics.

与保留类似，一旦首次接收到指标单位，就会设置任何指标的有效汇总，因此对这些设置的更改将不会影响现有指标。

### In conclusion

### 总结

Graphite's handling of your statsd metrics should be verified at least once: is data mysteriously lost at any point? is data downsampled properly? are you defining graphs for counter metrics without knowing what timespan does each y-value actually represent? (admittedly, in some cases you may not even care about the y-values in the graph, as only the trend is of any interest. The coolest graphs seem to always lack y-values...)

Graphite对你的statsd的指标单位的处理应该至少需要验证一次：数据在某个时间点神秘丢失？适当的对数据进行向下采样？您是否在不知道每个y值实际代表什么时间跨度的情况下定义计数器指标的图表？（诚然，在某些情况下，你甚至根本不需要关心图表中的y值，因为感兴趣的仅仅是趋势。最酷的图表似乎总是缺少y值...）

For more information, see: http://graphite.readthedocs.org/en/latest/config-carbon.html

更多详情，参见：http://graphite.readthedocs.org/en/latest/config-carbon.html
