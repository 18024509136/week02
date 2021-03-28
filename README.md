项目说明：
(1)服务端代码在netty-server-demo模块中  
(2)客户端代理在client-demo模块中  

本周第4道关于GC总结的必做题：  
 (1)衡量指标：  
 a.分配速率  
    分配速率表征单位时间内年轻代新增对象大小  
 b.晋升速率  
     晋升速率表征单位时间内从年轻代晋升到年老代的对象大小  
 c.gc频率  
     jvm调优最终目的是在额定内存资源下保证晋升速率持续小于分配速率，且将young gc（尽可能> 1次/15s）和full gc（尽可能> 1次/1天）的频率保持在合理的范围内  
 d.full gc 停顿时间  
     cms的初始标记和重新标记阶段总耗时不要超过200ms（g1在初始标记阶段伴随一次young gc，在最终标记阶段新增的无效引用会少很多，最终标记耗时少）  
 
 (2)通用优化手段：  
 a.计算密集型服务启用parallel gc，非计算密集型服务且要求低延迟响应的web服务启用cms gc，较大堆内存可考虑使用g1 gc  
 b.条件允许，增加堆内存  
 c.非计算密集型且要求低延迟响应的web服务，调整年轻代占比为50%  
 d.若晋升速率远小于分配速率，可考虑提高触发full gc的年老代使用占比阈值。若晋升速率接近分配速率，则考虑降低该阈值（cms是-XX:CMSInitiatingOccupancyFraction、g1是-XX+InitiatingHeapOccupancyPercent）  
 e.增加gc线程数（parallele: -XX:ParallelGCThreads cms/g1：-XX：ConcGCThreads）  
 
 (3)特殊情景识别：  
 a.promotion failed  
     当新生代发生垃圾回收，老年代有足够的空间可以容纳晋升的对象，但是由于空闲空间的碎片化，导致晋升失败，此时会触发单线程且带压缩动作的Full GC。优化两个方向一个是增大年轻代减少晋升速率，
 二是老年代UseCMSCompactAtFullCollection配合CMSFullGCsBeforeCompaction  
 b.concurrent mode failure  
     当CMS在执行回收时，新生代发生垃圾回收，同时老年代又没有足够的空间容纳晋升的对象时，CMS 垃圾回收就会退化成单线程的Full GC。所有的应用线程都会被暂停，老年代中所有的无效对象都被回收。这种情况最好增加堆内存或增加gc线程数  
 c.G1 Humongous Allocation  
     大型对象分配失败，大型对象（Humongous ）是大于G1中region大小50％的对象，增大-XX:G1HeapRegionSize配置
