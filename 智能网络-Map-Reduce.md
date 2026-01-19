## 一、MapReduce 要解决什么问题？

在 MapReduce 出现之前：

- 需要在**上千台机器**上处理 **TB / PB 级数据**
- 程序员不得不手写：
  - 任务拆分
  - 节点通信
  - 容错（机器宕机）
  - 负载均衡
- **分布式系统复杂性极高**

👉 **核心问题**：

> 能否让程序员只关心“数据怎么变换”，而不用关心“怎么在集群上跑”？

**MapReduce 的答案**：

> 把大规模数据处理抽象成两个函数：**Map** 和 **Reduce**

------

## 二、MapReduce 的核心思想（一句话版）

> 将一个大规模数据处理任务，分解为
>  **Map（并行产生中间键值对） → Shuffle（自动分组） → Reduce（按键聚合）**

程序员只需写：

```
map(key, value) → list(key', value')
reduce(key', list(value')) → list(value'')
```

其余一切（并行、调度、失败恢复）由系统负责。

------

## 三、MapReduce 的计算模型（原理核心）

### 1️⃣ 输入与输出的统一抽象：键值对（Key-Value）

- **输入**：`(key, value)`
- **Map 输出**：`(key', value')`
- **Reduce 输出**：`(key'', value'')`

👉 好处：

- 通用（文本、日志、网页、索引都能表示）
- 易于分区与并行

------

### 2️⃣ Map 阶段（数据“拆分 + 局部处理”）

**作用**：

- 独立处理输入数据的每一条记录
- 生成中间键值对

特点：

- **完全并行**
- Map 任务之间互不通信
- 输入数据通常来自 GFS（Google File System）

示例（词频）：

```
map(doc_id, doc_content):
  for word in doc_content:
    emit(word, 1)
```

------

### 3️⃣ Shuffle 阶段（系统自动完成，论文重点）

**Shuffle 是 MapReduce 的灵魂**

系统自动完成：

1. 按 `key'` 对 Map 输出进行分区
2. 网络传输到对应 Reduce 节点
3. 按 key 排序、合并

👉 程序员 **完全不用写代码**

这是 MapReduce 相比早期并行系统的**关键创新点**。

------

### 4️⃣ Reduce 阶段（全局聚合）

**作用**：

- 对同一个 key 的所有 value 进行聚合处理

示例（词频）：

```
reduce(word, [1,1,1,1]):
  emit(word, sum)
```

特点：

- 每个 key 只由 **一个 Reduce 处理**
- 保证语义正确性

------

## 四、系统架构（论文中的执行机制）

### 1️⃣ Master / Worker 架构

- **Master 节点**
  - 任务切分
  - 调度 Map / Reduce
  - 监控 Worker 状态
- **Worker 节点**
  - 执行 Map 或 Reduce
  - 处理本地数据块

------

### 2️⃣ 任务切分（Task Granularity）

- 输入数据切成 **M 个 Map task**
- Reduce 阶段分成 **R 个 Reduce task**

论文建议：

- M ≫ 机器数（提高负载均衡）
- R 由用户指定（通常几百到几千）

------

## 五、容错机制（MapReduce 的工程亮点）

论文非常强调：**机器一定会失败**

### 1️⃣ Worker 失败

- Map Worker 挂了 → 重新执行 Map
- Reduce Worker 挂了 → 重新执行 Reduce

**关键原因**：
 Map 和 Reduce 都是**确定性函数**（同样输入 → 同样输出）

------

### 2️⃣ Master 失败

- OSDI 2004 版本中：
  - Master 是单点
  - 实际工程中用 checkpoint 和重启解决

------

### 3️⃣ 备份任务（Speculative Execution）

- 慢节点 ≠ 挂节点
- Master 会：
  - 对“拖后腿”的任务启动 **备份执行**
  - 谁先完成用谁的结果

👉 大幅降低 **长尾延迟**

------

## 六、为什么 MapReduce 可扩展？

论文中的三个关键设计理由：

1. **函数式思想**
   - 无共享状态
   - 易于并行和容错
2. **数据本地性**
   - 尽量在数据所在节点执行 Map
   - 减少网络 I/O
3. **粗粒度任务**
   - 减少调度和通信成本

------

## 七、经典应用场景（论文实例）

论文中列举的真实 Google 任务：

- 倒排索引（Web 搜索）
- URL 访问日志分析
- PageRank 中的某些阶段
- 机器学习数据预处理

👉 MapReduce 不是万能，但**非常擅长批处理**

------

## 八、MapReduce 的局限（论文已隐含）

虽然强大，但论文也承认：

- 不适合：
  - 低延迟在线查询
  - 迭代型算法（如机器学习反复迭代）
- Shuffle 成本高
- 每一轮 MapReduce 都要落盘

👉 这直接催生了：

- **Spark**
- **Flink**
- **Dryad**