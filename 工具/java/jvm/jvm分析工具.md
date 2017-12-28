Java本身提供了多种丰富的方法和工具来帮助开发人员查看和分析GC及JVM内存的状况，同时开源界和商业界也有一些工具可用于查看、分析GC及JVM内存的状况。通过这些分析，可以排查程序中内存泄露的问题及调优程序的性能。下面介绍几种常用的免费工具，其中知名的有JProfiler 等。

　　1. 输出GC日志

　　输出GC日志对于跟踪分析GC的状况来说，无疑是最直接地分析内存回收状况的方法，只是GC日志输出后需要人为地进行分析，以判断GC的状况。

　　JVM支持将日志输出到控制台或指定的文件中，方法有如下几种。

　　输出到控制台

　　在JVM的启动参数中加入-XX:+PrintGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCApplicationStoppedTime，按照参数的顺序分别输出GC的简要信息，GC的详细信息、GC的时间信息及GC造成的应用暂停的时间。

　　输出到指定的文件

　　在1中的jvm启动参数中再增加-Xloggc: gc.log可指定将gc的信息输出到gc.log中。

　　可用于GC跟踪分析的参数还有-verbose:gc、-XX:+PrintTenuringDistribution等。

　　2. GC Portal

　　将GC日志输出固然有一定的作用，但如果要靠人为进行分析，还是相当复杂的。因此Sun提供了一个GC Portal来帮助分析这些GC日志，并生成相关的图形化的报表，GC Portal部署起来会有些麻烦，它需要运行在老版本的Tomcat上，同时需要数据库，部署完毕后通过上传日志文件的方式即可完成GC日志的分析，此GC日志输出的JVM参数为：-verbose:gc -XX:+PrintGCDetails -XX:+PrintGCTimeStamps [-Xloggc:文件名]，在上传日志时GC Portal的选项里只有jdk 1.2或jdk 1.2-1.4的版本。虽然经过测试，JDK 6的日志也是可以分析出来的，但它的限制在于仅支持5MB的gc日志的分析，GC Portal可提供吞吐量的分析、耗费的CPU的时间、造成的应用暂停的时间、每秒从新生代转化到旧生代的数量、minor GC的状况及Full GC的状况等，





　　GC Portal中还有一个很有用的部分是提供调整GC参数的预测，例如可以选择给young size增加20%的空间。GC Portal会根据当前的日志信息来评估在调整参数后的运行效果，不一定很准确，但毕竟能带来一些参考意义。



3. JConsole

　　JConsole可以图形化查看JVM中内存的变化状况，JConsole是JDK 5及以上版本中自带的工具，位于JDK的bin目录下，运行时直接运行JConsole.exe或JConsole.sh（要求支持图形界面）。在本地的Tab页上看到运行了java的pid，双击即可查看相应进程的JVM状况，同时，JConsole也支持查看远程的JVM的运行状况，具体可参见JConsole的User Guide。

　　JConsole中显示了JVM中很多的信息：内存、线程、类和MBean等，在打开JConsole的内存Tab页后，可看到JVM内存部分的运行状况。这对于分析内存是否有溢出及GC的效果更加直接明了，JConsole的运行效果如图2所示。





　　4. JVisualVM

　　JVisualVM是JDK 6 update 7之后推出的一个工具，它类似于JProfiler的工具，基于此工具可查看内存的消耗情况、线程的执行状况及程序中消耗CPU、内存的动作。

　　在内存分析上，JVisualVM的最大好处是可通过安装VisualGC插件来分析GC趋势、内存消耗详细状况。

　





　　基于JVisualVM的Profiler中的Memory还可查看对象占用内存的状况，如图4所示。

4. JMap

　　JMap是JDK中自带的一个用于分析JVM内存状况的工具，位于JDK的bin目录下。使用JMap可查看目前JVM中各个代的内存状况、JVM中对象的内存的占用状况，以及导出整个JVM中的内存信息。



　　查看JVM中各个代的内存状况

　　在linux上执行jmap -heap [pid]，就可查看整个JVM中内存的状况，看到的信息类似如下（和JDK版本、GC策略有关）：

using thread-local object allocation.  Parallel GC with 8 thread(s)  Heap Configuration:     MinHeapFreeRatio = 40    MaxHeapFreeRatio = 70    MaxHeapSize      = 1610612736 (1536.0MB)     NewSize          = 524288000 (500.0MB)     MaxNewSize       = 524288000 (500.0MB)     OldSize          = 4194304 (4.0MB)     NewRatio         = 8    SurvivorRatio    = 8    PermSize         = 100663296 (96.0MB)     MaxPermSize      = 268435456 (256.0MB)  Heap Usage:  PS Young Generation  Eden Space:     capacity = 430702592 (410.75MB)     used     = 324439936 (309.4100341796875MB)     free     = 106262656 (101.3399658203125MB)     75.32806675098904% used  From Space:     capacity = 46333952 (44.1875MB)     used     = 13016424 (12.413429260253906MB)     free     = 33317528 (31.774070739746094MB)     28.092626331550566% used  To Space:     capacity = 46792704 (44.625MB)     used     = 0 (0.0MB)     free     = 46792704 (44.625MB)     0.0% used  PS Old Generation     capacity = 1086324736 (1036.0MB)     used     = 945707880 (901.8973159790039MB)     free     = 140616856 (134.1026840209961MB)     87.05572548059884% used  PS Perm Generation     capacity = 100663296 (96.0MB)     used     = 46349592 (44.202415466308594MB)     free     = 54313704 (51.797584533691406MB)     46.044182777404785% used
从以上信息中可看出JVM堆的配置信息，如NewSize、NewRatio、SurvivorRatio等；JVM堆的使用情况，新生代中的Eden Space、From Space、To Space的使用情况，旧生代和持久代的使用情况。
　　要注意的是在使用CMS GC的情况下，jmap -heap的执行有可能会导致Java进程被挂起。



　　JVM中对象的内存的占用情况

　　在查看JVM内存状况时，除了要知道每个代的占用情况外，很多时候更要知道其中各个对象占用的内存大小，这样便于分析对象的内存占用情况，在分析OutOfMemory的场景中尤其适用。

　　

　　导出整个JVM中的内存信息

　　通过以上方法能查看到JVM中对象内存的占用情况，但很多时候还要知道这个对象到底是谁创建的。例如上面显示出来的[C，只知道它占用了那么多的空间，但不知道是什么对象创建出的[C，于是jmap提供了导出整个jvm中的内存信息的支持。基于一些jvm内存的分析工具，例如sun JDK 6中的jhat、Eclipse Memory Analyzer，可以分析jvm中内存的详细信息，例如[C是哪些对象创建的。

　　执行如下命令即可导出整个jvm中的内存信息：

jmap -dump:format=b,file=文件名 [pid]
　　5. JHat

　　JHat是Sun JDK 6及以上版本中自带的一个用于分析jvm堆dump文件的工具，基于此工具可分析jvm heap中对象的内存占用状况、引用关系等。

　　执行如下命令分析jvm堆的dump文件：

jhat -J-Xmx1024M [file]
　　执行后等待console中输出Started HTTP server on port 7000，看到后就可以通过浏览器访问http://ip:7000了，此页面默认为按package分类显示系统中所有的对象实例。在页面的最下端有Other Queries导航，其中有显示jvm中对象实例个数的链接、有显示jvm中对象大小的链接等，点击显示jvm

　

6. JStat

　　JStat是Sun JDK自带的一个统计分析JVM运行状况的工具，位于JDK的bin目录下，除了可用于分析GC的状况外，还可用于分析编译的状况、class加载的状况等。

　　JStat用于GC分析的参数有：-gc、-gccapacity、-gccause、-gcnew、-gcnewcapacity、-gcold、-gcoldcapacity、-gcpermcapacity、-gcutil。常用的为-gcutil。通过-gcutil可按一定频率查看jvm中各代的空间的占用情况、minor GC的次数、消耗的时间、full GC的次数及消耗的时间的统计，执行jstat -gcutil [pid] [interval]，可看到类似如下的输出信息：

 S0     S1       E      O       P       YGC
YGCT     FGC    FGCT     GCT   0.00   74.24  96.73  73.43  46.05  17808
382.335   208  315.197  697.533  45.37   0.00   28.12  74.97  46.05  17809
382.370   208  315.197  697.568
　　其中S0、S1就是Survivor空间的使用率，E表示Eden空间的使用率，O表示旧生代空间的使用率，P表示持久代的使用率，YGC表示minor GC的执行次数，YGCT表示minor GC执行消耗的时间，FGC表示Full GC的执行次数，FGCT表示Full GC执行消耗的时间，GCT表示Minor GC+Full GC执行消耗的时间。

　　7. Eclipse Memory Analyzer

　　Eclipse Memory Analyzer是Eclipse提供的一个用于分析jvm堆dump文件的插件，借助这个插件可查看对象的内存占用状况、引用关系、分析内存泄露等。

　　Eclipse Memory Analyzer（MAT）的网站为：http://www.eclipse.org/mat/，在eclipse中可以直接远程安装此插件。不过由于此插件在分析堆dump文件时比较耗内存，因此在分析前最好先将eclipse的jvm的内存设置大一点，MAT分析dump文件后的对象占用内存及引用关系如图3所示。

　　相对而言MAT功能比jhat强大很多，分析的速度也快一些，因此，如果要分析jvm堆dumap文件，首选推荐的是MAT。

　　在进行JVM内存状况分析时，通常要关注的主要有GC的趋势、内存的具体消耗状况。

　　GC趋势对于可图形界面连到需查看GC状况的机器的情况而言，VisualVM是常用的选择；对于不能采用图形界面方式的，输出GC日志 及采用jstat命令直接分析是常用的选择。

　　在查找内存是程序中的什么对象占用时，需要分析内存的具体消耗状况，对于有图形界面可用的情况，VisualVM是常用的选择；对于不能采用图形界面方式的，可通过jmap dump生成文件后，再通过MAT进行分析是常用的选择。
