JVM 参数设置详细说明

1: heap size
a: -Xmx<n>                       指定 jvm 的最大 heap 大小 , 如 :-Xmx=2g

b: -Xms<n>                       指定 jvm 的最小 heap 大小 , 如 :-Xms=2g ， 高并发应用， 建议和-Xmx一样， 防止因为内存收缩／突然增大带来的性能影响。

c: -Xmn<n>                       指定 jvm 中 New Generation 的大小 , 如 :-Xmn256m。 这个参数很影响性能， 如果你的程序需要比较多的临时内存， 建议设置到512M， 如果用的少， 尽量降低这个数值， 一般来说128／256足以使用了。

d: -XX:PermSize=<n>         指定 jvm 中 Perm Generation 的最小值 , 如 :-XX:PermSize=32m。 这个参数需要看你的实际情况，。 可以通过jmap 命令看看到底需要多少。

e: -XX:MaxPermSize=<n>   指定 Perm Generation 的最大值 , 如 :-XX:MaxPermSize=64m

f: -Xss<n>                        指定线程桟大小 , 如 :-Xss128k， 一般来说，webx框架下的应用需要256K。 如果你的程序有大规模的递归行为， 请考虑设置到512K／1M。 这个需要全面的测试才能知道。 不过， 256K已经很大了。 这个参数对性能的影响比较大的。

g: -XX:NewRatio=<n>        指定 jvm 中 Old Generation heap size 与 New Generation 的比例 , 在使用 CMS GC 的情况下此参数失效 , 如 :-XX:NewRatio=2

h: -XX:SurvivorRatio=<n>   指定 New Generation 中 Eden Space 与一个 Survivor Space 的 heap size 比例 ,-XX:SurvivorRatio=8, 那么在总共 New Generation 为 10m 的情况下 ,Eden Space 为 8m

i: -XX:MinHeapFreeRatio=<n> 指定 jvm heap 在使用率小于 n 的情况下 ,heap 进行收缩 ,Xmx==Xms 的情况下无效 , 如 :-XX:MinHeapFreeRatio=30

j: -XX:MaxHeapFreeRatio=<n> 指定 jvm heap 在使用率大于 n 的情况下 ,heap 进行扩张 ,Xmx==Xms 的情况下无效 , 如 :-XX:MaxHeapFreeRatio=70

k: -XX:LargePageSizeInBytes=<n> 指定 Java heap 的分页页面大小 , 如 :-XX:LargePageSizeInBytes=128m

2: garbage collector
a: -XX:+UseParallelGC 指定在 New Generation 使用 parallel collector, 并行收集 , 暂停 app threads, 同时启动多个垃圾回收 thread, 不能和 CMS gc 一起使用 . 系统吨吐量优先 , 但是会有较长长时间的 app pause, 后台系统任务可以使用此 gc

b: -XX:ParallelGCThreads=<n> 指定 parallel collection 时启动的 thread 个数 , 默认是物理 processor 的个数 ,

c: -XX:+UseParallelOldGC 指定在 Old Generation 使用 parallel collector

d: -XX:+UseParNewGC 指定在 New Generation 使用 parallel collector, 是 UseParallelGC 的 gc 的升级版本 , 有更好的性能或者优点 , 可以和 CMS gc 一起使用

e: -XX:+CMSParallelRemarkEnabled 在使用 UseParNewGC 的情况下 , 尽量减少 mark 的时间

f: -XX:+UseConcMarkSweepGC 指定在 Old Generation 使用 concurrent cmark sweep gc,gc thread 和 app thread 并行 ( 在 init-mark 和 remark 时 pause app thread). app pause 时间较短 , 适合交互性强的系统 , 如 web server

g: -XX:+UseCMSCompactAtFullCollection 在使用 concurrent gc 的情况下 , 防止 memory fragmention, 对 live object 进行整理 , 使 memory 碎片减少

h: -XX:CMSInitiatingOccupancyFraction=<n> 指示在 old generation 在使用了 n% 的比例后 , 启动 concurrent collector, 默认值是 68, 如 :-XX:CMSInitiatingOccupancyFraction=70 有个 bug, 在低版本(1.5.09 and early)的 jvm 上出现 , http://bugs.sun.com/bugdatabase/view_bug.do?bug_id=6486089

i: -XX:+UseCMSInitiatingOccupancyOnly 指示只有在 old generation 在使用了初始化的比例后 concurrent collector 启动收集

3:others
a: -XX:MaxTenuringThreshold=<n> 指定一个 object 在经历了 n 次 young gc 后转移到 old generation 区 , 在 linux64 的 java6 下默认值是 15, 此参数对于 throughput collector 无效 , 如 :-XX:MaxTenuringThreshold=31

b: -XX:+DisableExplicitGC 禁止 java 程序中的 full gc, 如 System.gc() 的调用. 最好加上么， 防止程序在代码里误用了。对性能造成冲击。

c: -XX:+UseFastAccessorMethods get,set 方法转成本地代码

d: -XX:+PrintGCDetails 打应垃圾收集的情况如 : [GC 15610.466: [ParNew: 229689K->20221K(235968K), 0.0194460 secs] 1159829K->953935K(2070976K), 0.0196420 secs]   e: -XX:+PrintGCTimeStamps 打应垃圾收集的时间情况 , 如 : [Times: user=0.09 sys=0.00, real=0.02 secs]

f: -XX:+PrintGCApplicationStoppedTime 打应垃圾收集时 , 系统的停顿时间 , 如 : Total time for which application threads were stopped: 0.0225920 seconds

4: a web server product sample and process
JAVA_OPTS=” -server -Xmx2g -Xms2g -Xmn256m -XX:PermSize=128m -Xss256k -XX:+DisableExplicitGC -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+CMSParallelRemarkEnabled -XX:+UseCMSCompactAtFullCollection -XX:LargePageSizeInBytes=128m -XX:+UseFastAccessorMethods -XX:+UseCMSInitiatingOccupancyOnly -XX:CMSInitiatingOccupancyFraction=70 ”   最初的时候我们用 UseParallelGC 和 UseParallelOldGC,heap 开了 3G,NewRatio 设成 1. 这样的配置下 young gc 发生频率约 12,3 秒一次 , 平均每次花费 80ms 左右 ,full gc 发生的频率极低 , 每次消耗 1s 左右 . 从所有 gc 消耗系统时间看 , 系统使用率还是满高的 , 但是不论是 young gc 还是 old gc,applicaton thread pause 的时间比较长 , 不合适 web 应用 . 我们也调小 New Generation 的 , 但是这样会使 full gc 时间加长 .   后来我们就用 CMS gc(-XX:+UseConcMarkSweepGC), 当时的总 heap 还是 3g, 新生代 1.5g 后 , 观察不是很理想 , 改为 jvm heap 为 2g 新生代设置 -Xmn1g, 在这样的情况下 young gc 发生的频率变成 ,7,8 妙一次 , 平均每次时间 40~50 毫秒左右 ,CMS gc 很少发生 , 每次时间在 init-mark 和 remark(two steps stop all app thread) 总共平均花费 80~90ms 左右 .   在这里我们曾经 New Generation 调大到 1400m, 总共 2g 的 jvm heap, 平均每次 ygc 花费时间 60~70ms 左右 ,CMS gc 的 init-mark 和 remark 之和平均在 50ms 左右 , 这里我们意识到错误的方向 , 或者说 CMS 的作用 , 所以进行了修改   最后我们调小 New Generation 为 256m,young gc 2,3 秒发生一次 , 平均停顿时间在 25 毫秒左右 ,CMS gc 的 init-mark 和 remark 之和平均在 50ms 左右 , 这样使系统比较平滑 , 经压力测试 , 这个配置下系统性能是比较高的   在使用 CMS gc 的时候他有两种触发 gc 的方式 :gc 估算触发和 heap 占用触发 . 我们的 1.5.0.09 环境下有次 old 区 heap 占用再 30% 左右 , 她就频繁 gc, 个人感觉系统估算触发这种方式不靠谱 , 还是用 heap 使用比率触发比较稳妥 .   这些数据都来自 64 位测试机 , 过程中的数据都是我在 jboss log 找的 , 当时没有记下来 , 可能存在一点点偏差 , 但不会很大 , 基本过程就是这样 .

5: 总结 web server 作为交互性要求较高的应用 , 我们应该使用 Parallel+CMS,UseParNewGC 这个在 jdk6 -server 上是默认的 ,new generation gc, 新生代不能太大 , 这样每次 pause 会短一些 .CMS mark-sweep generation 可以大一些 , 可以根据 pause time 实际情况控制
