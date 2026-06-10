# Streaming Systems 读书笔记与中文精设解析

本项目是关于流计算领域经典著作 **《Streaming Systems: The What, Where, When, and How of Large-Scale Data Processing》**（由 Tyler Akidau, Slava Chernyak, Reuven Lax 著，O'Reilly 出版）的完整读书笔记、核心概念拆解与深度中文翻译解析。

流数据处理已经成为现代大数据架构的基座。本项目旨在通过系统、详实的中文精设笔记，帮助每一位对分布式流计算感兴趣的开发者，跨越从基础概念到端到端 Exactly-Once 语义、流表相对性、流式 SQL 乃至流式 Join 的技术天堑。

---

## 📖 目录（Chapters Outline）

本项目已全面完成，涵盖原书全部 10 个章节：

### 第一部分：Beam 模型基础 (Part I: The Beam Model)
*   **[Chapter 1. Streaming 101](./chapters/Chapter%201.Streaming101.md)**：流处理术语定义（Bounded/Unbounded, Tables/Streams）、事件时间（Event Time）与处理时间（Processing Time）的核心区别，以及 Lambda 架构的局限。
*   **[Chapter 2. The What, Where, When, and How of Data Processing](./chapters/Chapter%202.The%20What,Where,When,and%20How%20of%20data%20processing.md)**：深入探讨有状态乱序数据处理的四个终极哲学问题：
    *   **What** is being computed? (Transformations 算子转换)
    *   **Where** in event time? (Windowing 窗口机制)
    *   **When** in processing time? (Triggers & Watermarks 触发器与水印)
    *   **How** do results relate? (Accumulation 累积模式)
*   **[Chapter 3. Watermarks](./chapters/Chapter%203.watermarks.md)**：时间进度衡量工具——水印（Watermarks）的定义、完美与启发式水印的构建、水印在计算管道中的传播机制，以及主流框架（Dataflow, Flink）的案例分析。
*   **[Chapter 4. Advanced Windowing](./chapters/Chapter%204.Advanced%20Windowing.md)**：处理时间窗口的实现路径、会话窗口（Session Windows）的原型分配与动态合并机制，以及三大自定义窗口（非对齐固定窗口、每元素固定窗口、有界会话窗口）的实战应用。
*   **[Chapter 5. Exactly-Once and Side Effects](./chapters/Chapter%205.Exactly-Once%20and%20Side%20Effects.md)**：分布式环境下精确一次（Exactly-Once）语义的物理实现：Shuffle 去重、计算确定性、布隆过滤器与 GC 优化，以及 Sources（可重放）与 Sinks（两阶段提交事务/幂等）的端到端配合。

### 第二部分：流与表相对性与高级演进 (Part II: Streams and Tables)
*   **[Chapter 6. Streams and Tables](./chapters/Chapter%206.Streams%20and%20Tables.md)**：流计算的终极理论——**流与表的相对性理论 ($\text{Streams} \rightleftharpoons \text{Tables}$)**。用流表的双重性透镜解构经典的 MapReduce 批处理模型，实现批流合一的底层逻辑。
*   **[Chapter 7. The Practicalities of Persistent State](./chapters/Chapter%207.The%20Practicalities%20of%20Persistent%20State.md)**：流计算状态的物理底座。隐式状态（原始分组与增量合并）与显式泛化状态（Value/Bag/Map State）的设计，结合**定时器（Timers）**攻克复杂工业场景：广告转化归因（Conversion Attribution）。
*   **[Chapter 8. Streaming SQL](./chapters/Chapter%208.Streaming%20SQL.md)**：声明式流处理的未来。基于**时变关系（Time-Varying Relations）**的关系代数闭包完备性。探讨流表显式选择（TABLE/STREAM）、系统虚拟列及核心的**撤回机制（Sys.Undo / Retraction）**。
*   **[Chapter 9. Streaming Joins](./chapters/Chapter%209.Streaming%20Joins.md)**：流式环境下的各种无窗口关联（FULL OUTER, INNER, ANTI, SEMI Join）中的状态开销与撤回语义，利用窗口和水印进行**状态修剪（State Pruning）**，以及攻克变动维度关联的终极武器：**时间有效性窗口关联（Temporal Validity Joins）**。
*   **[Chapter 10. The Evolution of Large-Scale Data Processing](./chapters/Chapter%2010.The%20Evolution%20of%20Large-Scale%20Data%20Processing.md)**：大规模数据处理技术十五年波澜壮阔的演进史。解构 MapReduce, Hadoop, Flume, Storm, Spark, MillWheel, Kafka, Dataflow, Flink, Beam 等 10 大标志性技术项目的兴衰、妥协、继承与涅槃。

---

## 🎯 本书核心思想方法论

在阅读和学习本项目提供的笔记时，请牢记 Apache Beam 模型的灵魂架构：

1.  **What** results are calculated? $\rightarrow$ **Transformations**（计算和、机器学习模型、过滤等）
2.  **Where** in event time are results calculated? $\rightarrow$ **Windowing**（固定、滑动、会话窗口）
3.  **When** in processing time are results materialized? $\rightarrow$ **Triggers + Watermarks**（何时输出）
4.  **How** do refinements of results relate? $\rightarrow$ **Accumulation**（Discarding, Accumulating, Retracting）

流与表不是孤立的：**流（Streams）是运动的数据（Data in motion），代表 Actions；表（Tables）是静止的数据（Data at rest），代表 State**。它们相互流转、互为表里，共同构成了现代实时计算宇宙的物理真理。

---

## 🤝 参与贡献与交流

本书笔记对流计算的概念做了极为详尽、透彻的中文阐释。如果你在阅读过程中发现任何笔误、翻译不准确或有更好的业务实践，欢迎提交 Pull Request 或 Issue！

流数据处理的未来已经到来，让我们在无界的数据浪潮中并肩破浪前行！

*“It is the what, where, when, and how that define the system.”*
