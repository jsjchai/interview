## redission
> Redisson 是一个具有内存数据网格功能的 Redis Java 客户端。它提供了更方便、最简单的使用 Redis 的方法。Redisson 对象提供了关注点分离，使您能够将注意力集中在数据建模和应用程序逻辑上
## 功能
* Redisson是一个在Redis的基础上实现的Java驻内存数据网格（In-Memory Data Grid）。它不仅提供了一系列的分布式的Java常用对象，还提供了许多分布式服务
* 基于NIO的Netty框架，通过吞吐量
* 使用Lua脚本
  * 原子操作
  * 减少网络开销
* Redisson 使用布隆过滤器

### 锁
> 如果获取锁的 Redisson实例崩溃，那么该锁可能会永远挂在获取状态。为了避免这种情况，Redisson维护锁看门狗，它会在锁持有者 Redisson 实例处于活动状态时延长锁过期时间。默认情况下，锁定看门狗超时为30秒,可通过配置文件修改
