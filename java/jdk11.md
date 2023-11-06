## jdk11新增内容
* 模块化
* 引入了var关键字
* CMS，即并发标记扫描垃圾收集器，现已弃用且不支持
* 新增String方法，如 isBlank()、lines()、repeat(n)、stripLeading()、stripTrailing() 和 strip()
* HTTP Client 标准化
* ZGC
  
## [jvm](https://docs.oracle.com/javase/specs/jvms/se11/jvms11.pdf)
### ZGC
* 停顿时间不超过10ms
* 停顿时间不会随着堆的大小，或者活跃对象的大小而增加
* 支持8MB~4TB级别的堆（未来支持16TB）

### GC之痛
> 很多低延迟高可用Java服务的系统可用性经常受GC停顿的困扰。GC停顿指垃圾回收期间STW（Stop The World），当STW时，所有应用线程停止活动，等待GC停顿结束。以美团风控服务为例，部分上游业务要求风控服务65ms内返回结果，并且可用性要达到99.99%。但因为GC停顿，我们未能达到上述可用性目标。当时使用的是CMS垃圾回收器，单次Young GC 40ms，一分钟10次，接口平均响应时间30ms。通过计算可知，有（40ms + 30ms) * 10次 / 60000ms = 1.12%的请求的响应时间会增加0 ~ 40ms不等，其中30ms * 10次 / 60000ms = 0.5%的请求响应时间会增加40ms。可见，GC停顿对响应时间的影响较大。为了降低GC停顿对系统可用性的影响，我们从降低单次GC时间和降低GC频率两个角度出发进行了调优，还测试过G1垃圾回收器，但这三项措施均未能降低GC对服务可用性的影响

### ZGC原理
> 与CMS中的ParNew和G1类似，ZGC也采用标记-复制算法，不过ZGC对该算法做了重大改进：**ZGC在标记、转移和重定位阶段几乎都是并发的**，这是ZGC实现停顿时间小于10ms目标的最关键原因

![image](https://github.com/jsjchai/interview/assets/13389058/a03aa555-65ab-449d-949d-63cd547b6c52)


### 参考文档
* [新一代垃圾回收器ZGC的探索与实践](https://tech.meituan.com/2020/08/06/new-zgc-practice-in-meituan.html)
