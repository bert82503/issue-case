
# 关于 CMS GC remark 时间长的case

【现象】有一个`应用出现了CMS GC remark通常要消耗4–5s`的现象，由于remark是STW(stop-the-world)的，
因此造成了`应用暂停时间过长`的现象。

CMS GC 由以下几个主要步骤构成：
* `initial-mark`: 这步是 STW 的；
* concurrent-mark: 这步是和应用并发同时做的；
* preclean: 和应用并发，忘记是1.5哪个版本后增加的一个优化；
* `remark`: 这步是 STW 的；
* concurrent-sweep: 这步是和应用并发做的；

可见，**对于CMS GC而言，在暂停时间这点上最重要的关注点是`initial-mark`和`remark`两个阶段**
（在一些JDK版本上，`jstat`看到一次CMS GC会被统计成两次FGC，原因就是它其实是按STW次数来统计的，这个小bug由当时还在淘宝的撒迦同学fix了）。

【知识】`remark如果耗时较长，通常原因是在CMS GC已经结束了concurrent-mark步骤后，旧生代的引用关系仍然发生了很多的变化`。
旧生代的引用关系发生变化的原因主要是：
* 在这个间隔时间段内，`新生代晋升到旧生代的对象比较多`；
* 在这个间隔时间段内，`新生代没怎么发生YGC，但创建出来的对象又比较多`，这种通常就只能是`新生代比较大的原因`；

【根源】这个**应用当时的启动参数的-Xmn为16g，但每次YGC后存活的其实不多**，因此基本可以确定是`由于新生代太大导致的`，
于是`建议应用方将-Xmn降到了4g，remark时间很快就缩短到了几百ms`，基本符合预期。


2013年7月16日

原文：[关于CMS GC remark时间长的case - 毕玄-林昊](http://hellojava.info/?p=147)
