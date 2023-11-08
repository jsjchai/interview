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
