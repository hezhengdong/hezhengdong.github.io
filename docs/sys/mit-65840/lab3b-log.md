# Lab3B Log

## 前言

一天梳理思路，一天写代码，一天改 bug，测试 500 次后无报错。

仔细阅读 guidance 后写代码，速度果然快了不少。

## 实现思路

**客户端从提交命令到命令被执行的流程**

1. 服务器初始化：
	- 启动 len(rf.peers) 个 replicator 线程，每个线程负责一个节点的日志同步，循环检查 `[nextIndex, len(logs)]` 区间的变动；
	- 启动一个 applier 线程，用于同步 commitIndex 与 lastApplied。
2. 领导者通过 Start 收到新条目，追加至自身日志。
3. replicator 发现区间变动，向跟随者发送 RPC 以复制日志。
4. 跟随者收到 RPC，将新条目追加至自身节点。
5. replicator 接收跟随者的成功回复后，通过排序 matchIndex 检查中位数，判断条目是否成功复制在过半数节点上。如果成功，更新 commitIndex。
6. applier 发现 lastApplied < commitIndex，将条目发送至提交管道，更新 lastApplied。

**论文相关章节**

1. 5.3 节：复制日志基本实现；nextIndex 机制解决日志不一致问题。
2. 5.4.1 节：选举限制机制。
3. 5.4.2 节：只提交当前任期条目，解决已提交日志被覆盖的问题。

**[Students' Guide to Raft](https://thesquareplanet.com/blog/students-guide-to-raft/) 相关重要内容**

1. "The importance of details" 章节：收到 AppendEntries RPC 后，必须检查是否存在冲突。如果存在冲突，截断日志并追加新条目。否则什么都不需要做。
2. "Term confusion" 章节：`matchIndx = prevLogIndex + len(entries[])`，而不是 `nextIndex - 1`。

**优化**

参考[这篇文章](https://github.com/OneSizeFitsQuorum/MIT6.824-2021/blob/master/docs/lab2.md#%E5%A4%8D%E5%88%B6%E6%A8%A1%E5%9E%8B)中提到的[SOFAJRaft 日志复制 - pipeline 实现剖析 | SOFAJRaft 实现原理](https://mp.weixin.qq.com/s/jzqhLptmgcNix6xYWYL01Q)。

很优秀的一篇文章，有趣，且有技术含量。

我实现了“改进 2 - 特点 3 - 批量发送”。每个节点的同步由一个线程负责，线程每次能够传输多条命令。根据节点所处的状态，使用条件变量，挂起或唤醒线程。

文章中提到，但我没有实现的：

- “关注 2”：实现很简单。再添加一个数组，为每个 peer 维护一个 Inflight 即可。
- “关注 3”：实现似乎很复杂。需要控制服务器对 RPC 的发送与接收基于一条通道完成，而不是池化发送与接收，否则会导致频繁触发 Inflight 同步。

## 相关知识点

Go 条件变量的使用细节：

```go
func (rf *Raft) function() {
	rf.mu.Lock()
	for (rf.state != Leader) {
		rf.cond.Wait()
	}
	rf.mu.Unlock()
}
```

1. 挂起 goroutine 后，必须使用 for 循环检查挂起条件。因为存在虚假唤醒问题。AI 说虚假唤醒是系统层面的问题。[Go 官方文档](https://pkg.go.dev/sync#Cond.Wait)也建议循环中使用，暂时没看懂原因。
2. 当 `rf.cond.Wait()` 开始执行后，底层会自动解锁，随后将当前 goroutine 挂起。
3. 当 `rf.cond.Wait()` 结束执行后，底层唤醒 goroutine 后，会自动尝试加锁。