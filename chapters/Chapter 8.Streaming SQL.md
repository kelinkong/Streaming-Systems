## Chapter 8. Streaming SQL (流式 SQL)

SQL（结构化查询语言）是数据处理领域无可争议的声明式行业标准。长久以来，SQL 被打上了“静态、批处理、离线分析”的标签。然而，随着流计算的蓬勃发展，如何将 SQL 的声明式表达能力与流计算的实时性、强一致性无缝结合，成为了流处理技术普及时至关重要的命题。本章我们将探讨**流式 SQL（Streaming SQL）**的理论基石——**时变关系（Time-Varying Relations）**，分析现有系统的设计偏好，并探讨为了支持健壮流处理所需的 SQL 语言扩展及优雅默认行为。

---

### What Is Streaming SQL? (什么是流式 SQL)

要理解流式 SQL，必须首先回归到关系代数（Relational Algebra）的本质。

#### 1. 关系（Relations）
在传统数据库中，SQL 作用的对象是**关系（Relation）**，在物理上表现为一张表（Table）。关系是**点在时间轴上的快照**（Point-in-time snapshot）——它是静态的。

#### 2. 时变关系 (Time-Varying Relations - TVR)
在流式世界中，数据集是随时间不断变化和演进的。为了让 SQL 能够作用于流，我们引入了**时变关系（TVR）**的概念。
*   **定义**：一个时变关系（TVR）代表了关系在时间轴上的完整演化轨迹。你可以将其想象成一个连续的、由无数个经典关系（Snapshot Relations）在不同时间戳上组成的序列。
*   **闭包属性（Closure Property）**：
    时变关系最关键的特性是它**完美地保持了关系代数的闭包属性**。
    > **如果在静态关系上应用 SQL 算子（如 Projection 投影、Selection 选择、Join 关联、Aggregation 聚合）是合法的，那么将这些算子作用于时变关系（TVR）在数学上同样是完全合法且定义完备的。**

这意味着：我们不需要为了流计算去重新发明一套全新的 SQL 语法，经典的 SQL 算子在时变关系（TVR）的世界中依然完全有效。

---

### Looking Backward: Stream and Table Biases (流与表的偏好)

虽然流和表在理论上是对等的，但由于历史演进路线的不同，现有的系统往往带有一种天生的“偏好（Biases）”：

#### 1. Beam 模型：流偏好 (Stream-Biased Approach)
Apache Beam 的设计初衷是处理事件流。
*   在 Beam 中，核心的数据容器是 `PCollection`，它在概念上是一个**流（Stream）**。
*   当我们在 Beam 中执行 `GroupByKey` 或 `Combine` 时，虽然在系统内部物理上产生了一个**表（状态表）**，但 Beam 并不直接把这个表暴露给用户。用户必须对其应用窗口（Windowing）和触发器（Triggers），将其重新转化为另一个 `PCollection` 流，才能在下游算子中观察到结果。

#### 2. SQL 模型：表偏好 (Table-Biased Approach)
传统的 SQL 数据库和物化视图（Materialized Views）的设计初衷是处理表。
*   在 SQL 中，所有的查询、更新、连接都是围绕着**表（Table）**进行的。
*   即使我们使用流式的 SQL 引擎，系统默认也是在更新和维护一张内存中的“物化视图”（表）。如果用户想要得到一个管道输出流，必须显式地进行特殊声明。

---

### Looking Forward: Toward Robust Streaming SQL (构建健壮的流式 SQL)

为了在 SQL 中完美支持 Beam 模型中强大的时间处理能力（Windowing, Watermarks, Triggers, Accumulation），我们需要对 SQL 进行合理的语义扩展，并设计明智的**默认行为**。

#### 1. 表与流的选择 (Table/Stream Selection)
因为时变关系（TVR）在物理上既可以渲染为一张**表（静态累积状态）**，也可以渲染为一个**流（变化事件日志）**，所以 SQL 应该提供明确的关键字来允许用户选择输出的形式：
*   `TABLE`：将 TVR 物化为一张表（如一个持续更新的关系型数据库表/物化视图）。
*   `STREAM`：将 TVR 的变化过程作为更新事件流发射（如 CDC 格式的流）。
*   `TVR`：保留其完整的时变关系形态。

**黄金默认行为（Defaults）**：
为了不增加普通用户的认知负担，流式 SQL 引擎应具备以下智能默认推导：
*   如果所有输入都是 `TABLE`，那么输出默认为 `TABLE`（退化为经典 SQL 语义）。
*   如果输入中包含任何 `STREAM`，那么输出默认为 `STREAM`（自动转换为流式管道）。

#### 2. 显式窗口操作符 (Windowing Operators)
虽然用声明式的 SQL（如基于时间戳的 `GROUP BY`）可以实现简单的固定时间窗口，但流式 SQL 仍极具需要显式的窗口操作符：
*   **简化数学计算**：显式函数（如 `TUMBLE(ts, INTERVAL '2' MINUTE)`，`HOP`，`SESSION`）封装了窗口边界计算的复杂边界数学逻辑。
*   **支持复杂的动态分组**：例如会话窗口（Sessions），这类需要根据数据间隙（Gap）动态合并的窗口，在传统 SQL 的 `GROUP BY` 中几乎是无法用纯声明式表达的。

#### 3. 触发器与水印的集成 (Triggers and Watermarks)
触发器（Triggers）定义了时变关系（TVR）在何时以及以何种频次被拍照物化并转化为流。
*   **默认触发器**：每当有新数据到达（Per-record）就立即触发，这与传统的数据库物化视图语义完美契合。
*   **水印触发器（Watermark Triggers）**：当水印通过窗口截止线（即该窗口的输入被认为完整）时触发。这对于追求“单窗口单输出”和高吞吐的传统批处理作业或强一致通知系统（如防欺诈报警）至关重要。

#### 4. 系统虚拟列 (Special System Columns)
当我们在 SQL 中将一个时变关系（TVR）渲染为流时，流中的更新行数据需要携带一些至关重要的元数据信息。流式 SQL 引擎应提供以下系统虚拟列：
*   `Sys.MTime`：该行记录在 TVR 中最后一次被修改的系统处理时间。
*   `Sys.EmitTiming`：本次输出相对于水印进度的时机状态（`early` 早期推测值、`on-time` 按时完整值、`late` 迟到修订值）。
*   `Sys.EmitIndex`：针对当前 key 和窗口，本次输出是第几次触发的版本号（从 0 开始递增）。
*   `Sys.Undo`：标识当前行是否是一条**撤回记录（Retraction）**。

#### 5. 撤回机制：Sys.Undo (Undo/Redo 语义)
撤回机制是多级流式 SQL 聚合能够产生正确结果的灵魂。
假设我们对数据进行了两次连续的 `GROUP BY`。
1.  第一级 Group By 输出了：`[UserA, Sum=10]`。
2.  随后一条新数据使得 UserA 的 Sum 变成了 12。
3.  如果不使用撤回，直接输出 `[UserA, Sum=12]`，那么第二级聚合就会同时收到 10 和 12，导致累加结果错误（得到 22 而非正确的 12）。
4.  **撤回（Retraction）的工作机制**：系统会向流中发射两条记录：
    *   一条 `Sys.Undo = true` 的撤回行：`[UserA, Sum=10]`（表示“撤回之前的 10”）。
    *   一条 `Sys.Undo = false` 的正常行：`[UserA, Sum=12]`（表示“写入最新的 12”）。
    通过这种 `Undo/Redo` 的精密配对，下游的多级聚合和 Join 算子得以实时修正自己的状态，保证全局数据的绝对正确。

---

### Summary (总结)

流处理与 SQL 的结合是数据工程演进的必然趋势。
*   **时变关系（TVR）** 奠定了流式 SQL 的数学根基，证明了关系代数在流动的数据上依旧完美适用。
*   在融合了 **Beam 模型的流式时间语义** 之后，通过引入**表与流的显式选择（TABLE/STREAM）**、**显式窗口操作符**、**系统虚拟列**以及基于 **`Sys.Undo` 的撤回机制**，流式 SQL 得以兼具声明式编程的极简优雅与强一致、低延迟的强大流计算处理能力。

掌握了流式 SQL 的理论后，在第九章中，我们将面临关系代数中最复杂、最有趣的算子在流式环境下的挑战：**流式关联（Streaming Joins）**。