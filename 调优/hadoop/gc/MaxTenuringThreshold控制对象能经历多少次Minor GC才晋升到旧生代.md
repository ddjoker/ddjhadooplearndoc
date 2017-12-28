
MaxTenuringThreshold这个参数用于控制对象能经历多少次Minor GC才晋升到旧生代，默认值是15，那是不是意味着对象要经历15次minor gc才晋升到旧生代呢，来看下面的一个例子。
```java
public class GCTenuringThreshold{
public static void main(String[] args) throws Exception{
GCMemoryObject object1=new GCMemoryObject(2);
GCMemoryObject object2=new GCMemoryObject(8);
GCMemoryObject object3=new GCMemoryObject(8);
GCMemoryObject object4=new GCMemoryObject(8);
object2=null;
object3=null;
GCMemoryObject object5=new GCMemoryObject(8);
Thread.sleep(4000);
object2=new GCMemoryObject(8);
object3=new GCMemoryObject(8);
object2=null;
object3=null;
object5=null;
GCMemoryObject object6=new GCMemoryObject(8);
Thread.sleep(5000);
}
}
class GCMemoryObject{
private byte[] bytes=null;
public GCMemoryObject(int multi){
bytes=new byte[1024*256*multi];
}
}
```

以-Xms20M –Xmx20M –Xmn10M –XX:+UseSerialGC参数执行以上代码，通过jstat -gcutil [pid] 1000 10的方式查看执行效果，很惊讶执行结果竟然是在第二次minor GC的时候object1就被晋升到old中了，而可以肯定的是这个时候to space空间是充足的，也就是说并不是在to space空间充足的情况下，对象一定要经历MaxTenuringThreshold次才会晋升到old，那具体规则到底是怎么样的呢，翻看Hotspot 6 update 21中SerialGC的实现，可以看到在每次minor GC后，会对这个存活周期的阈值做计算，计算的代码如下：
```c
size_t desired_survivor_size = (size_t)((((double) survivor_capacity)*TargetSurvivorRatio)/100);
size_t total = 0;
int age = 1;
assert(sizes[0] == 0, "no objects with age zero should be recorded");
while (age < table_size) { total += sizes[age]; // check if including objects of age 'age' made us pass the desired // size, if so 'age' is the new threshold if (total > desired_survivor_size) break;
age++;
}

int result = age < MaxTenuringThreshold ? age : MaxTenuringThreshold;
```
其中`desired_survivor_size`是指`survivor space/2`，从上面的代码可看出，在计算存活周期这个阈值时，`hotspot`会遍历所有`age`的`table`，并对其所占用的大小进行累积，当累积的大小超过了`survivor space`的一半时，则以这个`age`作为新的存活周期阈值，最后取`age`和M`axTenuringThreshold`中更小的一个值。

按照这样的规则，上面的运行效果就可验证了，第一次minor gc的时候存活周期的阈值为MaxTenuringThreshold，minor gc结束后计算出新的阈值为1，在第二次minor gc时object 1的age已经是1了，因此object1被晋升到了旧生代。

这个规则对于Serial GC以及ParNew GC（但对于开启了UseAdaptiveSizePolicy的ParNew GC而言也无效，默认是不开启的）均有效，对于PS（Parallel Scavenge） GC而言，在默认的情况下第一次以InitialTenuringThreshold（默认为7）为准，之后在每次minor GC后均会动态计算，规则比上面的复杂，后续再另写一篇blog来说，在设置-XX:-UseAdaptiveSizePolicy后，则以MaxTenuringThrehsold为准，并且不会重新计算，会是恒定值。

如希望跟踪每次minor GC后新的存活周期的阈值，可在启动参数上增加：`-XX:+PrintTenuringDistribution`，输出的信息中的：
Desired survivor size 1048576 bytes, new threshold 7 (max 15)
new threshold 7即标识新的存活周期的阈值为7。
