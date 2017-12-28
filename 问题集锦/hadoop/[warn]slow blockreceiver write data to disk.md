# org.apache.hadoop.hdfs.server.datanode.DataNode: Slow BlockReceiver write data to disk cost
Warnning抛出说明当前datanode往磁盘写数据时，延迟超过阈值。
出现的原因可能是磁盘压力大，在写数据时出现高延迟状况，可以通过iostat辅助判断一下。
也可能是磁盘出现某些故障（比如缺电等），半死不死，能写进数据，但性能不好。

于I/O-bond类型的进程，我们经常用iostat工具查看进程IO请求下发的数量、系统处理IO请求的耗时，进而分析进程与操作系统的交互过程中IO方面是否存在瓶颈。



下面通过iostat命令使用实例，说明使用iostat查看IO请求下发情况、系统IO处理能力的方法，以及命令执行结果中各字段的含义。

 

1.不加选项执行iostat

我们先来看直接执行iostat的输出结果：


```
 iostat
```
    Linux 2.6.16.60-0.21-smp (linux)     06/12/12

    avg-cpu:  %user   %nice %system %iowait  %steal   %idle
               0.07    0.00    0.05    0.06    0.00   99.81

    Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
    sda               0.58         9.95        37.47    6737006   25377400
    sdb               0.00         0.00         0.00        824          0

单独执行`iostat`，显示的结果为从系统开机到当前执行时刻的统计信息。以上输出中，除最上面指示系统版本、主机名和日期的一行外，另有两部分：

    `avg-cpu`: 总体cpu使用情况统计信息，对于多核cpu，这里为所有cpu的平均值

    `Device`: 各磁盘设备的IO统计信息



对于cpu统计信息一行，我们主要看`iowait`的值，它指示cpu用于等待io请求完成的时间。`Device`中各列含义如下：

    Device: 以sdX形式显示的设备名称
    tps: 每秒进程下发的IO读、写请求数量
    Blk_read/s: 每秒读扇区数量(一扇区为512bytes)
    Blk_wrtn/s: 每秒写扇区数量
    Blk_read: 取样时间间隔内读扇区总数量
    Blk_wrtn: 取样时间间隔内写扇区总数量

我们可以使用-c选项单独显示avg-cpu部分的结果，使用-d选项单独显示Device部分的信息。



2.指定采样时间间隔与采样次数

与sar命令一样，我们可以以`iostat interval [count]`形式指定iostat命令的采样间隔和采样次数：

复制代码
```bash
iostat -d 1 2
```
    Linux 2.6.16.60-0.21-smp (linux)     06/13/12

    Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
    sda               0.55         8.93        36.27    6737086   27367728
    sdb               0.00         0.00         0.00        928          0

    Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
    sda               2.00         0.00        72.00          0         72
    sdb               0.00         0.00         0.00          0          0

以上命令输出`Device`的信息，采样时间为1秒，采样2次，若不指定采样次数，则iostat会一直输出采样信息，直到按”ctrl+c”退出命令。注意，第1次采样信息与单独执行iostat的效果一样，为从系统开机到当前执行时刻的统计信息。



3.以kB为单位显示读写信息(-k选项)

我们可以使用-k选项，指定iostat的部分输出结果以kB为单位，而不是以扇区数为单位：

复制代码
```
 iostat -d -k
```
      Linux 2.6.16.60-0.21-smp (linux)     06/13/12

      Device:            tps    kB_read/s    kB_wrtn/s    kB_read    kB_wrtn
      sda               0.55         4.46        18.12    3368543   13686096
      sdb               0.00         0.00         0.00        464          0

以上输出中，kB_read/s、kB_wrtn/s、kB_read和kB_wrtn的值均以kB为单位，相比以扇区数为单位，这里的值为原值的一半(1kB=512bytes*2)



4.更详细的io统计信息(-x选项)

为显示更详细的io设备统计信息，我们可以使用-x选项，在分析io瓶颈时，一般都会开启-x选项：

复制代码
```
iostat -x -k -d 1
```
    Linux 2.6.16.60-0.21-smp (linux)     06/13/12

    ……
    Device:         rrqm/s   wrqm/s     r/s     w/s    rkB/s    wkB/s avgrq-sz avgqu-sz   await  svctm  %util
    sda               0.00  9915.00    1.00   90.00     4.00 34360.00   755.25    11.79  120.57   6.33  57.60

以上各列的含义如下：

    rrqm/s: 每秒对该设备的读请求被合并次数，文件系统会对读取同块(block)的请求进行合并
    wrqm/s: 每秒对该设备的写请求被合并次数
    r/s: 每秒完成的读次数
    w/s: 每秒完成的写次数
    rkB/s: 每秒读数据量(kB为单位)
    wkB/s: 每秒写数据量(kB为单位)
    avgrq-sz:平均每次IO操作的数据量(扇区数为单位)
    avgqu-sz: 平均等待处理的IO请求队列长度
    await: 平均每次IO请求等待时间(包括等待时间和处理时间，毫秒为单位)
    svctm: 平均每次IO请求的处理时间(毫秒为单位)
    %util: 采用周期内用于IO操作的时间比率，即IO队列非空的时间比率


对于以上示例输出，我们可以获取到以下信息：

每秒向磁盘上写30M左右数据(wkB/s值)
每秒有91次IO操作(r/s+w/s)，其中以写操作为主体
平均每次IO请求等待处理的时间为120.57毫秒，处理耗时为6.33毫秒
等待处理的IO请求队列中，平均有11.79个请求驻留


以上各值之间也存在联系，我们可以由一些值计算出其他数值，例如：

`util = (r/s+w/s) * (svctm/1000)`

对于上面的例子有：`util = (1+90)*(6.33/1000) = 0.57603`
