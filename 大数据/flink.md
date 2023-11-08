## flink
> Flink 是一个框架和分布式处理引擎，用于对无界和有界数据流进行有状态计算。并且 Flink 提供了数据分布、容错机制以及资源管理等核心功能。Flink提供了诸多高抽象层的API以便用户编写分布式任务

### 流处理

![image](https://github.com/jsjchai/interview/assets/13389058/9485fb00-49f3-43c6-8b20-725b11eb104e)

### flink的API
> Flink 为开发流/批处理应用程序提供了不同级别的抽象

![image](https://github.com/jsjchai/interview/assets/13389058/c0565232-1f43-45c3-b463-490e4c89dcf5)

* 最低级别的抽象仅提供有状态且及时的流处理
   * 提供基础的核心接口完成流、状态、事件、时间等复杂操作，功能灵活但使用成本较高，一般面向源码研发人员
* DataStream API （有界/无界流）。这些流畅的 API 提供了数据处理的通用构建块，例如各种形式的用户指定的转换、联接、聚合、窗口、状态等。在这些 API 中处理的数据类型在相应的编程语言中表示为类
* Table API是一种以table为中心的声明式 DSL ，它可以动态更改表（当表示流时）
   * 通过Table操作和注册表完成数据计算，支持与DataStream/Dataset相互转换
* Flink 提供的最高级别抽象是SQL。这种抽象在语义和表达上都类似于Table API，但将程序表示为 SQL 查询表达式。SQL抽象与 Table API 紧密交互，并且可以对Table API中定义的表执行 SQL 查询
  
### Flink计算模型
* 输入端（source）: Flink程序的输入端，支持多个数据源对接
* 转换（Transform）: Flink程序的转换过程，实现DataStream/Dataset的计算和转换
* 输出端（sink）: Flink的输出端，支持内部和外部输出源

### Flink 架构 
> Flink 运行时由两种类型的进程组成：一个JobManager和一个或多个TaskManager

![image](https://github.com/jsjchai/interview/assets/13389058/4c3cefc3-1c07-477a-9cba-47c92781ed2e)

* Client不是运行时和程序执行的一部分，而是用于准备数据流并将其发送到 JobManager
* 总是至少有一个 JobManager。高可用性设置可能有多个 JobManager，其中一个始终是领导者，其他是 备用
#### JobManager
> JobManager具有与协调 Flink 应用程序的分布式执行相关的许多职责：它决定何时安排下一个任务（或一组任务）、对已完成的任务或执行失败做出反应、协调检查点以及协调故障恢复等其他的
* ResourceManager
    *  ResourceManager负责 Flink 集群中的资源取消/分配和配置——它管理任务槽，任务槽是 Flink 集群中资源调度的单位
    *  Flink 针对不同的环境和资源提供者（例如 YARN、Kubernetes 和独立部署）实现了多个 ResourceManager
    *  在独立设置中，ResourceManager 只能分配可用 TaskManager 的插槽，而无法自行启动新的 TaskManager
* Dispatcher
  * Dispatcher提供 REST 接口来提交 Flink 应用程序执行，并为每个提交的作业启动一个新的 JobMaster 。它还运行 Flink WebUI 以提供有关作业执行的信息
* JobMaster
  * JobMaster负责管理单个 JobGraph的执行。Flink 集群中可以同时运行多个作业，每个作业都有自己的 JobMaster
#### TaskManager
> TaskManager 执行数据流的任务，并缓冲和交换数据流
* 必须始终至少有一个 TaskManager。TaskManager 中资源调度的最小单位是任务槽。TaskManager 中的任务槽数表示并发处理任务的数量。请注意，多个运算符可以在一个任务槽中执行
#### Operator Chains(算子链)
> 对于分布式执行，Flink将算子子任务链接到 任务中。每个任务由一个线程执行。将运算符链接到任务中是一种有用的优化：它减少了线程间切换和缓冲的开销，并提高了总体吞吐量，同时减少了延迟
#### Task Slots
> 每个worker（TaskManager）都是一个JVM进程，并且可以在单独的线程中执行一个或多个子任务。为了控制 TaskManager 接受的任务数量，它具有所谓的任务槽（至少一个）
* 每个任务槽代表任务管理器的固定资源子集
* 对资源进行分槽意味着子任务不会与其他作业的子任务竞争托管内存，而是拥有一定量的保留托管内存。请注意，这里没有发生 CPU 隔离；目前插槽仅分隔任务的托管内存。
* 通过调整任务槽的数量，用户可以定义子任务如何相互隔离
* 每个 TaskManager 有一个槽意味着每个任务组在单独的 JVM 中运行（例如，可以在单独的容器中启动）
* 拥有多个槽意味着更多的子任务共享同一个 JVM
* 同一 JVM 中的任务共享 TCP 连接（通过多路复用）和心跳消息。它们还可以共享数据集和数据结构，从而减少每个任务的开销

### Flink 应用程序执行
#### Flink Application Cluster 
* 集群生命周期
  * Flink 应用程序集群是一个专用的 Flink 集群，仅执行来自一个 Flink 应用程序的作业，并且方法 main()在集群上运行，而不是在客户端上运行。作业提交是一个一步过程：您不需要先启动 Flink 集群，然后将作业提交到现有集群会话；相反，您将应用程序逻辑和依赖项打包到可执行作业 JAR 中，集群入口点 ( ApplicationClusterEntryPoint) 负责调用该main()方法来提取 JobGraph。例如，这允许您像 Kubernetes 上的任何其他应用程序一样部署 Flink 应用程序。因此，Flink 应用程序集群的生命周期与 Flink 应用程序的生命周期绑定在一起
* 资源隔离
  * 在 Flink 应用程序集群中，ResourceManager 和 Dispatcher 的范围仅限于单个 Flink 应用程序，这比 Flink 会话集群提供了更好的关注点分离
#### Flink Session Cluster
* 集群生命周期
  * 在 Flink 会话集群中，客户端连接到一个预先存在的、长期运行的集群，该集群可以接受多个作业提交。即使所有作业完成后，集群（和 JobManager）仍将继续运行，直到手动停止会话。因此，Flink 会话集群的生命周期不受任何 Flink 作业的生命周期的约束。
* 资源隔离
  * TaskManager 槽位由 ResourceManager 在作业提交时分配，并在作业完成后释放。由于所有作业都共享同一个集群，因此对集群资源存在一些竞争，例如提交作业阶段的网络带宽。这种共享设置的一个限制是，如果一个 TaskManager 崩溃，那么在此 TaskManager 上运行任务的所有作业都将失败；类似地，如果JobManager发生致命错误，也会影响集群中运行的所有作业
* 其他注意事项
  * 有预先存在的集群可以节省大量申请资源和启动 TaskManager 的时间。这在作业执行时间非常短且启动时间较长会对端到端用户体验产生负面影响的场景中非常重要 - 就像短查询的交互式分析一样，希望作业能够快速执行使用现有资源执行计算

### Flink分布式快照-检查点（Checkpoints）
#### Checkpoints
> 分布式快照即所谓的一致性检查点。定义为某个时间点上所有任务状态的一份拷贝(快照)，该时间点也是所有任务刚好处理完一个相同数据的时间
* 检查点通过允许恢复状态和相应的流位置来使 Flink 中的状态具有容错能力，从而为应用程序提供与无故障执行相同的语义
* 检查点的主要目的是在意外作业失败时提供恢复机制。
* 检查点的生命周期由 Flink 管理，即检查点由 Flink 创建、拥有和释放 - 无需用户交互。
* 由于检查点经常被触发，并且依赖于故障恢复，因此检查点实现的两个主要设计目标是
    * 创建时尽可能轻量级
    * 恢复尽可能快
* 针对这些目标的优化可以利用某些属性，例如作业代码在执行尝试之间不会改变
* 如果用户终止应用程序，检查点将自动删除（除非检查点明确配置为保留
* 检查点以状态后端特定的（本机）数据格式存储（可能是增量的，具体取决于特定的后端）
#### 分布式快照机制
> flink采用基于 checkpoint 的分布式快照机制，能够保证作业出现 fail-over 后可以从最新的快照进行恢复，即分布式快照机制可以保证 Flink 系统内部的“精确一次”处理
* Flink 分布式快照的核心元素
    * Barrier（数据栅栏）
       * 可以把 Barrier 简单地理解成一个标记，该标记是严格有序的，并且随着数据流往下流动。每个 Barrier 都带有自己的 ID，Barrier 极其轻量，并不会干扰正常的数据处理
    * 异步
       * 每次在把快照存储到我们的状态后端时，如果是同步进行就会阻塞正常任务，从而引入延迟。因此 Flink 在做快照存储时，采用异步方式
    * 增量保存
* Flink的分布式快照是根据Chandy-Lamport算法量身定做的。简单来说就是持续创建分布式数据流及其状态的一致快照
* 核心思想是在 input source 端插入 barrier，控制 barrier 的同步来实现 snapshot 的备份和 exactly-once 语义
### 保存点-Savepoints
> Savepoint 是流作业执行状态的一致映像，通过 Flink 的检查点机制创建。您可以使用 Savepoints 来停止和恢复、分叉或更新您的 Flink 作业。保存点由两部分组成：稳定存储（例如 HDFS、S3 等）上包含（通常较大）二进制文件的目录和（相对较小）元数据文件。稳定存储上的文件代表作业执行状态映像的网络数据。保存点的元数据文件（主要）包含指向作为保存点一部分的稳定存储上的所有文件的指针，以相对路径的形式
* 保存点仅由用户创建、拥有和删除。这意味着，Flink 在作业终止后和恢复后都不会删除保存点
* 保存点以状态后端独立（规范）格式存储（注意：从 Flink 1.15 开始，保存点也可以以后端特定的本机格式存储，这种格式创建和恢复速度更快，但有一些限制)。
### State Backends
> 检查点的持久化
#### HashMapStateBackend
* HashMapStateBackend在内部将数据作为对象保存在 Java 堆上。键/值状态和窗口运算符保存存储值、触发器等的哈希表
* HashMapStateBackend将数据作为对象存储在堆上，因此重用对象是不安全的
* 使用场景
    * 具有大状态、长窗口、大键/值状态的作业
    * 所有高可用性设置
#### EmbeddedRocksDBStateBackend
> EmbeddedRocksDBStateBackend 将运行中的数据保存在RocksDB数据库中，该数据库（默认情况下）存储在 TaskManager 本地数据目录中
* 数据存储为序列化字节数组，这些数组主要由类型序列化器定义，从而导致键比较是按字节进行的，而不是使用 Java 的hashCode()和equals()方法
* EmbeddedRocksDBStateBackend 始终执行异步快照
* EmbeddedRocksDBStateBackend 的限制
  * 由于 RocksDB 的 JNI 桥接 API 基于 byte[]，因此每个键和每个值支持的最大大小均为 2^31 字节。在 RocksDB 中使用合并操作的状态（例如 ListState）可以默默地累积值大小 > 2^31 字节，然后在下次检索时失败。这是目前 RocksDB JNI 的限制
* 使用场景
  * 具有非常大的状态、长窗口、大键/值状态的作业
  * 所有高可用性设置
* 局限
  * 保留的状态量仅受可用磁盘空间量的限制
  * 与将状态保存在内存中的 HashMapStateBackend 相比，这允许保留非常大的状态。然而，这也意味着该状态后端可实现的最大吞吐量将会较低
  * 从后端进行的所有读/写都必须经过反/序列化来检索/存储状态对象
* EmbeddedRocksDBStateBackend 重用对象是安全的    
### flink内存管理
#### 进程内存
> Flink JVM进程的总进程内存由Flink应用程序消耗的内存（Flink 总内存）和 JVM 运行进程所消耗的内存组成。Flink 内存消耗总量包括JVM堆内存和堆外 （直接或本机）内存的使用量

![image](https://github.com/jsjchai/interview/assets/13389058/2d75d32d-dab1-4d19-8898-9eb0a8a1598e)

#### TaskManager内存

![image](https://github.com/jsjchai/interview/assets/13389058/45180209-e892-4e1b-b925-fc5b534da8f9)

#### JobManager 内存

![image](https://github.com/jsjchai/interview/assets/13389058/36fc4aff-f879-4cf4-b7ed-a91b96283902)

