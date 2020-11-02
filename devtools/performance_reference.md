# Chrome 开发者工具 —— Performance 使用参考

> 原文链接：<https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference?hl=zh-cn>
> 相关文章：<https://github.com/hewq/blog/blob/master/devtools/performance_start.md>

<!-- TOC -->

- [Chrome 开发者工具 —— Performance 使用参考](#chrome-开发者工具--performance-使用参考)
  - [Disable JavaScript Samples](#disable-javascript-samples)
  - [查看主线程](#查看主线程)
  - [Call Tree](#call-tree)
  - [Bottom-Up](#bottom-up)
  - [Event Log](#event-log)
  - [Interactions](#interactions)
  - [Memory](#memory)
  - [Enable advanced paint instrumentation](#enable-advanced-paint-instrumentation)

<!-- /TOC -->

## Disable JavaScript Samples

- 默认情况下，**Main** 部分会详细记录 JavaScript 的调用堆栈，使用 **Disable JavaScript Samples** 可以隐藏这些调用。像一些自定义的函数调用会被隐藏。下图为隐藏前和隐藏后的比较![jssampleenabble](images/js-samples-enabled.png)![jssampledisable](images/js-samples-disabled.png)

## 查看主线程

- **Main** 部分记录了主线程的活动![main1](images/main1.svg)点击其中一个事件，在 **Summary** 面板中查看更详细的信息![maineventsummary](images/maineventsummary.png)DevTools 用火焰图表示主线程的活动。x 轴表示随时间的记录，y 轴表示调用的事件。上层的事件调用（触发）了下层的事件（`anonymous`代表匿名函数）![flamechart](images/flamechart.png)

## Call Tree

- 使用 **Call Tree** 来查看哪个根事件花费的时间最多。**Call Tree** 只显示选中部分的记录。![calltree](images/call-tree.png)
- **Self Time** 表示直接花费在该事件上的时间，**Total Time** 表示花费在该事件及其所有后代（子孙）事件的总时间。
- 默认情况下，分组菜单设置为无分组，使用分组菜单可以根据各种标准对活动表进行排序。
- 点击 **Show Heaviest Stack** ![show-heaviest-stack](images/show-heaviest-stack.png) 会显示所选事件中哪些子事件执行时间最长。

## Bottom-Up

- 使用 **Bottom-Up** 可以查看哪些活动在总体上占用了最多的时间。**Bottom-Up** 只显示选中部分的记录。![bottom-up](images/bottom-up.png)在上图的火焰图中可以看出几乎所有的时间都花在三个对`wait`的调用上，所以 **Bottom-Up** 中，最顶部的是`wait`；`wait`调用下面的黄色部分其实是数以千计的 GC（垃圾回收） 调用，所以下一个开销最大的是`Minor GC`。
- **Self Time** 表示直接花费在该事件上的时间，**Total Time** 表示花费在该事件及其所有后代（子孙）事件的总时间。

## Event Log

- 使用 **Event Log** 可以按时间顺序查看事件的执行![event-log](images/event-log.png)
- **Start Time** 表示相对于记录的开始时间，例如图中的`1573.0ms` 表示该事件在记录开始后的 1573 毫秒后被执行。
- **Self Time** 表示直接花费在该事件上的时间，**Total Time** 表示花费在该事件及其所有后代（子孙）事件的总时间。

## Interactions

- 使用 **Interactions** 来分析记录过程中发生的用户交互。![interactions](images/interactions.svg)底部的红线表示等待主线程所花费的时间。

## Memory

- 启用 **Memory** 可以查看记录过程中的内存使用情况。![memory](images/memory.svg)

## Enable advanced paint instrumentation

- 启用 **Enable advanced paint instrumentation**，在 **Main** 中点击 **Paint**，**Paint Profiler** 面板会显示有关绘制事件的高级信息![paint-profiler](images/paint-profiler.png)
