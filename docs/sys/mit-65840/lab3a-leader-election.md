# Lab3A Leader Election

## 前言

本来想着 Lab3 全部做完后再发布笔记，没想到 Lab3A 就花了这么久时间。趁着记忆还在，梳理重点，希望能帮助到他人。

时间线

- 三天时间阅读 [Time, Clocks, and the Ordering of Events in a Distributed System](https://lamport.azurewebsites.net/pubs/time-clocks.pdf)。
- 三天时间阅读 Raft 论文的前五章。
- 六天时间完成 Lab3A，测试 500 次没问题。

在做 Lab3A 前，一定要阅读课程指导中的 [Raft Locking Advice](https://pdos.csail.mit.edu/6.824/labs/raft-locking.txt)、[Students' Guide to Raft](https://thesquareplanet.com/blog/students-guide-to-raft/)、[Debugging by Pretty Printing](https://blog.josejg.com/debugging-pretty/) 三篇文章。第一篇文章讲述如何使用锁；第二篇文章讲述一些常见的问题，尤其是其中的 Term confusion 问题；第三篇文章讲述如何更好的打印日志。

## 重点记录

### 重点1：状态转换

论文图 4 很清晰的展示了状态的转换。

状态、任期、选票，三者相关程度很高。每次状态转换，都需要考虑任期和选票的变化。

1. Follower -> Candidate：任期+1、选票重置
2. Candidate -> Leader：任期不变、选票不变
3. Leader -> Follower：任期同步、选票重置
4. Candidate -> Follower
	- 收到新任期：任期同步、选票重置
	- 未收到新任期 && 收到领导者心跳：任期不变、选票不变
5. Candidate -> Candidate：任期+1、选票重置



### 重点2：触发器

我一开始走了弯路：每次非领导者变为领导者，开启心跳线程，关闭选举线程；每次领导者变为非领导者，开启选举线程，关闭心跳线程。最终被 bug 包围。参考了[别人的定时器](https://github.com/OneSizeFitsQuorum/MIT6.824-2021/blob/master/docs/lab2.md#sender)是如何实现的：所有定时逻辑都在 ticker() 中控制，简洁方便。

讲义不推荐使用 Go 自带的定时器。自己看了看使用教程，确实麻烦。老实使用 for sleep 加锁并循环，实现定时器功能。

我的实现：

```go
func (rf *Raft) ticker() {
	for !rf.killed() {
		rf.mu.Lock()

		if (rf.state != Leader && time.Since(rf.lastHeartbeat) > rf.electionTimeout) {
			Debugf(dVote, "S%d 检测到选举超时 %v, 启动选举", rf.me, rf.electionTimeout)
			go rf.startElection()
			rf.lastHeartbeat = time.Now() // 重置选举超时时间
			rf.electionTimeout = BaseElectionTimeout + time.Duration(rand.Int63() % RandomCount) * time.Millisecond // 为选举超时时间赋予新的随机值
			rf.mu.Unlock()
			time.Sleep(10 * time.Millisecond) // AI 说 10ms 对现代 CPU 来说负载很小
			continue
		}

		if (rf.state == Leader) {
			go rf.heartbeat()
			rf.mu.Unlock()
			time.Sleep(HeartbeatInterval)
			continue
		}

		rf.mu.Unlock()
	}
}
```



### 重点3：如何统计票数

一种方法是在 Raft 结构体中维护投票数。缺点是会引入额外的复杂度，每次任期变化还需要重置投票数，容易出错。

另一种方法是使用局部变量。由于投票 RPC 异步并发进行，因此需要 channel 传输投票结果。startElection() 通过 for-range 获取结果，判断选票是否过半。还需要使用 WaitGroup 异步等待所有投票线程结束，关闭 channel 释放资源。

伪代码如下：

```go
func (rf *Raft) startElection() {
	var grantedVotes int // 统计获得的票数
	var wg sync.WaitGroup // 等待一组线程完成执行
	results := make(chan bool, len(rf.peers) - 1) // 传输投票结果

	// ...

	for server := range rf.peers {
		// 跳过自己；构造参数...

		// 异步发送投票 RPC
		wg.Add(1)
		go func(server int) {
			defer wg.Done()

			// 发送 RPC...

			// 将结果发送至 channel
			if reply.success {
				results <- true
			} else {
				results <- false
			}
		}(i)
	}

	// 异步等待线程执行完毕，关闭 channel
	go func() {
		wg.Wait()
		close(results)
	}()

	for success := range results {
		// 接收选票
		if success {
			grantedVotes++
		}
		// 阻塞，直到 channel 关闭
	}
}
```

### 重点4：选举超时问题

依据讲义提示和论文内容，我设置心跳间隔为 100ms，选举超时 1500-3000ms。随后发现平均每 150 次测试，就会出现一个“限制时间内未选出领导者”的错误。

经排查，原因是超时时间过长，将范围减少为 750-1500ms，能够显著降低失败概率。[详细解释](https://zhuanlan.zhihu.com/p/1978241677593437256)

### 其他建议

如何编写日志：

1. 记录论文图 4 中的状态转换，例如 "S0 Candidate -> Leader"。状态转换必然是连续的，如果不连续，那就说明出现了问题。
2. 记录谁为谁在哪个任期投票，例如 "S0 -> S1 投票成功 Term=1"。

常见测试失败原因：

1. 选出多个领导者：[任期混淆问题](https://thesquareplanet.com/blog/students-guide-to-raft/#term-confusion)
2. 限制时间内未能选出领导者：选举超时时间过长
