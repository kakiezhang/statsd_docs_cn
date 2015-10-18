Metric namespacing
-------------------
度量单位的命名空间
-------------------
The metric namespacing in the Graphite backend is configurable with regard to
the prefixes. Per default all stats are put under `stats` in Graphite, which
makes it easier to consolidate them all under one schema. However it is
possible to change these namespaces in the backend configuration options.
The available configuration options (living under the `graphite` key) are:

在Graphite后端中，有关度量单位的命名空间的前缀是可以配置的。默认情况下，所有的统计都会被放在Graphite下的`stats`当中，这使得将他们都整合在同一个架构当中变得更加容易。但是，我们可以在后端的配置选项当中改变这些命名空间。这是现有的配置选项（处在`graphite`中的键）：

```
legacyNamespace:  use the legacy namespace [default: true]
globalPrefix:     global prefix to use for sending stats to graphite [default: "stats"]
prefixCounter:    graphite prefix for counter metrics [default: "counters"]
prefixTimer:      graphite prefix for timer metrics [default: "timers"]
prefixGauge:      graphite prefix for gauge metrics [default: "gauges"]
prefixSet:        graphite prefix for set metrics [default: "sets"]
```

```
legacyNamespace:  使用传统的命名空间 [默认值: true]
globalPrefix:     用来发送统计到graphite的全局前缀 [默认值: "stats"]
prefixCounter:    计数器型度量单位的全局前缀 [默认值: "counters"]
prefixTimer:      计时器型度量单位的全局前缀 [默认值: "timers"]
prefixGauge:      仪表型度量单位的全局前缀 [默认值: "gauges"]
prefixSet:        组合型度量单位的全局前缀 [默认值: "sets"]
```

If you decide not to use the legacy namespacing, besides the obvious changes
in the prefixing, there will also be a breaking change in the way counters are
submitted. So far counters didn't live under any namespace and were also a bit
confusing due to the way they record rate and absolute counts. In the legacy
setting rates were recorded under `stats.counter_name` directly, whereas the
absolute count could be found under `stats_counts.counter_name`. When legacy namespacing
is disabled those values can be found (with default prefixing)
under `stats.counters.counter_name.rate` and
`stats.counters.counter_name.count` now.

如果你决定不用传统的命名规则，那么除了在前缀当中的有明显的修改之外，还将会存在一个在计数器被提交方面的重大改变。到目前为止，计数器还并没有处在任何的命名空间下，并且还会由于它们记录比率和绝对计数的方式而变得有点混乱。在传统设置中，比率是被直接记录在`stats.counter_name`当中，而绝对计数可以在`stats_counts.counter_name`当中找到。当传统命名空间被禁用时，就可以在`stats.counters.counter_name.rate`和`stats.counters.counter_name.count`当中找到那些值（使用默认前缀）。

The number of elements in sets will be recorded under the metric
`stats.sets.set_name.count` (where "sets" is the prefixSet).

在`stats.sets.set_name.count`（其中的“sets”是带前缀的组合）这个单位中记录着组合当中的元素的数量。

