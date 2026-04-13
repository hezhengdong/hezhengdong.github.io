# Lab1 MapReduce

## 前言

不知道该学什么才是有用的。没有学过分布式，这门课程的评价很好，所以来学一学。（2025 版）

- 论文我使用[有道速读](https://read.youdao.com/)翻译为中文阅读的，个人认为翻译的质量很不错。
- 视频看大佬的[翻译](https://mit-public-courses-cn-translatio.gitbook.io/mit6-824)
- Go 语言，如果专门去学习的话感觉很无聊。读代码的过程中，哪里看不懂，我就去看[菜鸟教程](https://www.runoob.com/go/go-tutorial.html)相关章节或者问 AI，正反馈更强。

先把论文通读一遍，可以看一些相关的科普视频辅助理解，对 MapReduce 的原理建立一个大致认知。

然后依据论文、讲义、代码梳理实现思路。做 Lab 要先以讲义为准，其次为论文。毕竟 Lab 实现的是简化版，与论文会有些出入。

## 实现思路

### 1、架构设计

**1.1、思考主节点与工作节点间的协作方式**

论文中说"主节点向工作节点分配任务"，Lab 中提示"工作节点向主节点请求任务"（Hints 第二条）。

我建议选择"工作节点向主节点请求任务"。这样实现能够简化代码，主节点不需要维护工作节点的状态，工作节点也不需要部署 RPC 服务器，还能实现自动的负载均衡。

**1.2、主节点维护任务状态**

一种容易想到的思路是：使用 map 数据结构存储任务，在 value 中维护任务的状态（空闲、执行中、已完成）。但这样做会导致全量过滤，例如超时检测需遍历所有元素寻找执行中任务。

所以根据不同状态所需的操作，使用不同的数据结构。

- 对于空闲任务。主节点需要快速获取空闲任务，分配给工作节点。所以选择队列。
	- Go 没有队列数据结构，可以使用 Channel，类似先进先出队列，且原生支持并发。
- 对于执行中任务。超时检测需要遍历；当工作节点执行完任务，主节点需要根据 id 将对应任务移除。所以选择 Map，做到 O(1) 复杂度检索并移除任务。
- 对于已完成任务。只需维护一个 int 变量即可，统计已完成任务的数量。我使用 Set，记录了已完成的任务 id，用于打印日志。

下面是我的 Coordinator 实现。

```go
type Coordinator struct {
	// 任务状态管理
	IdleMapTasks chan MapTask
	InProcessMapTasks map[int]MapTask
	CompletedMapTasks map[int]struct{}
	IdleReduceTasks chan ReduceTask
	InProcessReduceTasks map[int]ReduceTask
	CompletedReduceTasks map[int]struct{}

	// 阶段控制
	Phase Phase

	// 任务数量, 与已完成任务数作对, 判断当前阶段是否完成
	NMap int
	NReduce int

	// 并发控制
	mu sync.Mutex
}
```

**1.3、超时检测**

由于主节点维护任务状态，不维护工作节点状态。所以主节点需要记录任务开始执行的时间，启动后台线程定时检测执行中任务是否超时。如果超时，则将其移入空闲队列。

这种方式也省去了心跳检测。（心跳检测烦了我很久，Coordinator 要维护额外的状态，写了坨臃肿的屎山后感觉思路错了，然后 AI 给了如此简便的方法。思考半天不如 AI 灵机一动😭😭😭）

### 2、Worker 实现

实现 Worker 的重点在于：想明白数据在不同阶段是如何传输的？

原始数据 -> Map 输入 -> Map 输出 -> Reduce 输入 -> Reduce 输出

- 原始数据 -> Map 输入：Lab 中，一个文本文件就是一个 Map 任务的输入。论文中，还需要按照一定规则对数据进行切片。
- Map 输出 -> Reduce 输入（重点）：
	- 思路：
		- 对于 Map 输出，需要根据分区函数（hash(key) mod NReduce）将输出划分为 NReduce 个文件。
		- 这样所有 Map 任务执行完成后，相同 key 的键值对持久化在相同的分区中。
		- Reduce 根据分区获取数据，即可成功聚合相同 key 的键值对。 
	- 根据 Lab 提示：文件名 mr-X-Y 中的 Y 充当分区的作用。
	- 分享一个 bug：Reduce 获取输入数据时，需要使用正则表达式精确匹配。我最开始使用 `mr-*-Y` 匹配，导致最后测试时，把 job count test 的相关文件也给匹配进去了。

图片讲解：

### 3、Coordinator 实现

**3.1、Coordinator 实现**

主要是实现 MakeCoordinator 方法与 RPC 方法。

- MakeCoordinator 方法，进行 Coordinator 实例化，后台启动超时检测方法。
- 依据"工作节点向主节点请求任务"，Coordinator 只需创建两个 RPC 方法：RequestTask 与 ReportTaskDone

**3.2、并发问题**

调用 RPC 方法，本质上相当于在 Coordinator 中创建了一个线程执行 RPC 方法。多个 Worker 并发调用，RPC 方法内部操作共享的 Coordinator 数据结构，会导致并发问题。所以两个 RPC 方法需要添加悲观锁。

**3.3、RPC 实现**

Go RPC 实现需要严格遵守规范。例如必须使用两个结构体指针作为函数参数，表示 RPC 参数与返回值，必须返回 error 类型。

代码框架中提供了 RPC 调用的示例，参考示例即可。

## 建议

- 把整个 Lab 再拆分为小模块实现。每实现一个模块，就编写测试用例检查。
	- 例如实现 Coordinator 结构体后，使用超时检测测试数据结构是否符合预期；实现 Worker 后，手动模拟 Coordinator，检查 Worker 对文件的读写是否正确。
- 做 Lab 要先以讲义为准，其次为论文。毕竟 Lab 实现的是简化版，与论文会有些出入。
- 当自己思考了半天架构，并依据自己思考的架构写了一坨屎山的时候，还是问问 AI 比较好。

## 废话

只算写代码的时间的话，花费了六天。读论文、读代码、梳理思路的时间没有详细统计，花的时间肯定比写代码久。

感觉好难呀，但是看别人说 Lab1 是课程最简单的实验😭
