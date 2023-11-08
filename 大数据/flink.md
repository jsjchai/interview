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


