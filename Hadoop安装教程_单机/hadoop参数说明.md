
I am going through several blogs and tutorials and everywhere I found the JVM heap size should be set to lower than the Map and Reduce memory defined. For example, suppose I have defined the following configuration in my mapred-site.xml file.

<name>mapreduce.map.memory.mb</name>
<value>4096</value>
<name>mapreduce.reduce.memory.mb</name>
<value>8192</value>
Then Each Container will run JVMs for the Map and Reduce tasks. The JVM heap size should be set to lower than the Map and Reduce memory defined above, so that they are within the bounds of the Container memory allocated by YARN. Therefore It should be something like this

mapreduce.map.java.opts=-Xmx3072m
mapreduce.reduce.java.opts=-Xmx6144m
Why cant be define the JVM heap size same as Mapper and Reducer memory size. Is there any pros and cons of this? What will happen if I define the JVM heap size same as Mapper and Reduce memory size.
