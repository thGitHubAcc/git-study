有一个50万PV的资料类网站（从磁盘提取文档到内存）原服务器32位，1.5G
的堆，用户反馈网站比较缓慢，因此公司决定升级，新的服务器为64位，16G
的堆内存，结果用户反馈卡顿十分严重，反而比以前效率更低了。原因是啥？内存太大了
为什么原网站慢?
很多用户浏览数据，很多数据load到内存，内存不足，频繁GC，STW长，响应时间变慢
为什么会更卡顿？
内存越大，FGC时间越长
咋办？
PS -> PN + CMS 或者 G1
系统CPU经常100%，如何调优？(面试高频)
CPU100%那么一定有线程在占用系统资源，
找出哪个进程cpu高（top）
该进程中的哪个线程cpu高（top -Hp）
导出该线程的堆栈 (jstack)
查找哪个方法（栈帧）消耗时间 (jstack)
工作线程占比高 | 垃圾回收线程占比高
系统内存飙高，如何查找问题？（面试高频）
导出堆内存 (jmap)
分析 (jhat jvisualvm mat jprofiler … )
如何监控JVM
jstat jvisualvm jprofiler arthas top…


案例1：系统CPU经常100%，如何调优？
推理过程是：CPU100%，那么一定有线程在占用系统资源，所以

找出哪个进程cpu高（top命令）
该进程中的哪个线程cpu高（top -Hp）
如果是java程序，导出该线程的堆栈 （jstack命令）
查找哪个方法（栈帧）消耗时间，哪个方法调用的哪个方法 (jstack)，然后去看这个方法的代码
工作线程占比高 / 垃圾回收线程占比高？
案例2：系统内存飙高，如何查找问题？
导出堆内存 (jmap)
分析 (jhat jvisualvm mat jprofiler … )


# OOM 问题排查案例汇总
OOM产生的原因多种多样，有些程序未必产生OOM，不断FGC(CPU飙高，但内存回收特别少) （上面案例）

## 硬件升级系统反而卡顿的问题（见上）

## 线程池不当运用产生OOM问题（见上）
不断的往List里加对象（实在太LOW）

## jira问题
实际系统不断重启
解决问题 加内存 + 更换垃圾回收器 G1
真正问题在哪儿？不知道

## tomcat http-header-size过大问题（Hector）

## lambda表达式导致方法区溢出问题(MethodArea / Perm Metaspace)
LambdaGC.java -XX:MaxMetaspaceSize=9M -XX:+PrintGCDetails


## 直接内存溢出问题（少见）
《深入理解Java虚拟机》P59，使用Unsafe分配直接内存，或者使用NIO的问题

## 栈溢出问题
-Xss设定太小

## 比较一下这两段程序的异同，分析哪一个是更优的写法：

```java
Object o = null;
for(int i=0; i<100; i++) {
    o = new Object();
    //业务处理
    //这种写法更好，每一次进入循环，都是只有一个引用指向它，其他的没有指向的引用，就有可能被回收啦
}
```

```java
for(int i=0; i<100; i++) {
    Object o = new Object();
    // 循环结束才释放，才会被回收
}
```

## 重写finalize引发频繁GC
小米云，HBase同步系统，系统通过nginx访问超时报警（系统CPU在频繁GC，CPU飚高），最后排查，C++程序员重写finalize引发频繁GC问题
为什么C++程序员会重写finalize？（new对象需要手动回收内存，调用析构函数delete，以为finalize就是析构函数，所以就把它重写了）
finalize耗时比较长，里面写了一些复杂的逻辑，耗时（200ms），回收不过来了

## 如果有一个系统，堆内存一直消耗不超过10%，但是观察GC日志，发现FGC总是频繁产生，会是什么引起的？
因为有人显式调用了System.gc() (这个比较Low)

##Distuptor有个可以设置链的长度，如果过大，然后对象大，消费完不主动释放，会溢出

## 用jvm都会溢出，mycat用崩过，1.6.5某个临时版本解析sql子查询算法有问题，9个exists的联合sql就导致生成几百万的对象

## new 大量线程，会产生 native thread OOM，（low）应该用线程池，
解决方案：减少堆空间（太TMlow了）,预留更多内存产生native thread
JVM内存占物理内存比例 50% - 80%