# 论文阅读 LRU-K

> 想学一下 LRU-K 算法，但是互联网上没有什么好资料，只好看看论文了qwq。

[论文链接](https://www.cs.cmu.edu/~natassa/courses/15-721/papers/p297-o_neil.pdf)

[参考博客](https://blog.csdn.net/AntiO2/article/details/128439155)

需要先了解什么是 LRU，可以做做这道题：[LRU缓存 - 力扣](https://leetcode.cn/problems/lru-cache/description/?envType=study-plan-v2&envId=top-100-liked)

## LRU问题

论文 “1.1 问题陈述” 描述了 LRU 存在的问题。

LRU 仅根据“数据的最后一次访问时间”决定从缓冲池中删除哪个页面，无法区分数据引用频率的高低。

随后举了两个例子论证：

- 例 1.1：对于 B 树而言，其内部节点的访问频率明显要高于叶子节点。缓冲池在空间不足的情况下，应当优先存储内部节点。但是 LRU 保存的最近被引用的页面，意味着有相当一部分叶子节点被保存，这显然不合适。
- 例 1.2：数据库中，对大量数据的顺序扫描会淹没原本在缓冲池中的热点数据。

后面讲了些已有的优化算法，不是很重要。

## LRU-K概念

开头讲了算法的数学证明。

$I_p = b_p^{-1}$：访问间隔越大，访问频率越小，淘汰优先级更高。

原理：贝叶斯定理，LRU-K 记录“最近 K 次访问的时间间隔”，结合“历史数据”与“短期波动”，更准确的判断事件发生概率。LRU 仅依据“短期波动”。

> 贝叶斯定理好有趣，感兴趣的可以多研究一下。推荐看看3b1b的科普视频。

---

随后给出了两个定义，这两个定义是 LRU-K 的关键。

**定义 2.1 Backward K-distance $b_t(p,K)$**

Backward K-distance $\Rightarrow b_t(p,K) = \begin{cases}
当前时间 - 倒数第K次访问时间 & \text{如果 } p \text{ 的访问次数} \geq K\\
\infty & \text{如果 } p \text{ 的访问次数} < K
\end{cases}$

**定义 2.2 LRU-K**

LRU-K 的淘汰策略：替换缓冲池中 $b_t(p,K)$ 最大的页面。

歧义处理：如果多个页面 $b_t(p,K) = \infty$，则通过**辅助策略**选择应当被淘汰的页面。（例如使用 LRU 作为辅助策略）

特殊说明：LRU-1 等价于经典 LRU 算法。

## 问题

随后论文的 2.1.1 与 2.1.2 讲了两个 LRU-K 实际使用中可能会出现的问题，与相应的解决办法。

**2.1.1 过早丢弃页面与相关引用问题**

问题引入：

- 新页面加载到缓存时，LRU-K 可能会因为其 $b_t(p,K)=\infty$ 而立即丢弃该页面。但现实中，该页面可能很快会被**关联操作**再次访问。

关联操作典型场景：

- 事务内访问：同一事务内，先读后改数据，多次访问同一页面。
- 事务重试
- 进程内访问

解决方法：

- 页面第一次被访问时，增加一个 5s 的“**观察窗口**”，5s 内，该页会保存在缓存中，5s 内对页面的多次操作仅算作是一次访问。

我认为这样的方法还解决了[小林coding 中提到的缓存污染问题](https://www.xiaolincoding.com/os/3_memory/cache_lru.html#%E7%BC%93%E5%AD%98%E6%B1%A1%E6%9F%93%E4%BC%9A%E5%B8%A6%E6%9D%A5%E4%BB%80%E4%B9%88%E9%97%AE%E9%A2%98)，一个不受欢迎的页面短期内被高频访问，同样会造成热点误判，污染缓存，而将“观察窗口”内的多次操作算作一次，避免了该问题。

**2.1.2 页引用保留信息问题**

问题引入：

- 如果一个页面被移除缓存，那么该页面之前的历史记录都会丢失，再次访问页面时，需要重新为该页构建历史记录。高负载场景下，可能会导致高频数据被反复加载和丢弃。

解决方法：

- 即使页面被移除缓存，系统依旧保留 200s 的该页面的访问记录。

## 伪代码

论文 2.1.3 中给出了 LRU-K 的伪代码实现。

核心数据结构

- `HIST(p,n)`：记录页面 p 倒数第 n 次的访问时间。（排除关联周期内的连续访问。）
- `LAST(p)`：记录页面 p 的最后一次访问时间。（无论是否是关联访问。）

伪代码：（论文中的伪代码有些难理解，我给大致翻译了一遍。）

```java
// 在时间 t 引用页面 p 时，应调用的过程：
if (p 已经在缓冲池中) {
    if (t - LAST(P) > 相关引用期) { // 该访问此前没有关联访问
        // 更新页面的一系列访问记录
        long correlatedPeriod = LAST(p) - HIST(p, 1);
        for (int i = 2; i <= K; i++) {
            HIST(p, i) = HIST(p, i - 1) + correlatedPeriod;
        }
        HIST(p, 1) = t;
        LAST(p) = t;
    } else { // 该访问为关联访问
        // 更新最后访问时间
        LAST(P) = t;
    }
} else {
    // 淘汰旧页
    for (Page p : buffer) {
        if (p 的 b_t(p,K) 最大 && p 不处于相关引用期内) {
            // 淘汰该页
            if (victim 是脏页) {
                // 将 victim 写回磁盘
            }
        }
    }
    // 从磁盘中获取新页面 p
    if (HIST(p) 不存在) { // 缓存中不存在页面 p 先前的访问记录
        // 初始化访问记录
        allocate HIST(p);
        for (i = 2; i <= K; i++) {
            HIST(p, i) = 0;
        }
    } else { // 缓存中存在页面 p 先前的访问记录
        // 更新访问记录
        for (i = 2; i <= K; i++) {
            HIST(p, i) = HIST(p, i-1);
        }
    }
    HIST(p, 1) = t;
    LAST(p) = t;
}
```

论文后续证明了 LRU-K 相比其他算法的优越性，做了些性能测试的实验，不是很重要。

## 代码实现

已实现

- 优先淘汰 Backward K-distance 最大的页面。
- 若 Backward K-distance 相同，则优先淘汰 `lastAccessTime` 最小的页面。（实现 LRU 辅助策略。）

未实现：

- “观察窗口”功能。
- “页面访问历史保留”功能。

重点：

- 优先级队列
    - 优先级队列的原理是什么？推荐一个[优先级队列科普视频](https://www.bilibili.com/video/BV1AF411G7cA)。
    - 自定义比较器：主排序与次级排序，详细看注释。（其实是一个很常见的功能实现，但是太容易忘记了。）

```java
import java.util.Comparator;
import java.util.PriorityQueue;
import java.util.concurrent.ConcurrentHashMap;

public class BufferPool {
    private final int K;
    private final int numPages;
    // 存储具体页面
    private final ConcurrentHashMap<PageId, Page> pageStore = new ConcurrentHashMap<>();
    // 存储页ID到元数据的映射
    private final ConcurrentHashMap<PageId, PageMetadata> metaStore = new ConcurrentHashMap<>();
    // 优先级队列
    private final PriorityQueue<PageId> priorityQueue = new PriorityQueue<>(
            // 比较器，定义队列中元素的排序规则
            new Comparator<PageId>() {
                // compare 方法，返回值 < 0，表示 id1 在 id2 之前。
                @Override
                public int compare(PageId id1, PageId id2) {
                    // 获取元数据
                    PageMetadata meta1 = metaStore.get(id1);
                    PageMetadata meta2 = metaStore.get(id2);
                    /* 队列数据结构，尾部添加，头部淘汰。
                       - 主排序：降序排序，优先淘汰“反向K距离”最大的页（“反向K距离”指的就是 Backward K-distance）
                       - 次级排序：升序排序，优先淘汰时间戳最小的页
                     */
                    // 主排序：比较两个页面的“反向K距离”
                    int cmp = Long.compare(meta2.getBackwardKDistance(), meta1.getBackwardKDistance());
                    // 如果二者不相等，返回主排序结果
                    if (cmp != 0) {
                        return cmp;
                    } else {
                        // 如果二者相等，进入次级排序
                        // 次级排序：比较两个页面的时间戳
                        return Long.compare(meta1.getLastAccessTime(), meta2.getLastAccessTime());
                    }
                }
            }
    );

    public BufferPool(int k, int numPages) {
        this.K = k;
        this.numPages = numPages;
    }

    public Page getPage(PageId pid) {
        long currentTime = System.currentTimeMillis();

        // 如果页面存在
        if (pageStore.containsKey(pid)) {
            PageMetadata meta = metaStore.get(pid);
            meta.updateAccess(currentTime);
            return pageStore.get(pid);
        } else {
            // 如果页面不存在

            // 若空间不足，淘汰页面
            if (pageStore.size() >= numPages) {
                evictPage();
            }

            // 从磁盘中读取页面
            Page page = null; // 此处调用从磁盘中读取页面的方法
            pageStore.put(pid, page);

            // 添加元数据相关信息
            PageMetadata meta = new PageMetadata(K, currentTime);
            metaStore.put(pid, meta);
            priorityQueue.add(pid);

            return page;
        }
    }

    public void evictPage() {
        PageId removePid = priorityQueue.poll();
        if (removePid == null) {
            return;
        }

        // 刷新脏页
        if (pageStore.get(removePid).isDirty() != null) {
            // 调用刷新磁盘方法
        }

        pageStore.remove(removePid);
        metaStore.remove(removePid);
    }
}
```

```java
import java.util.Arrays;

public class PageMetadata {
    private long[] accessHistory;
    private long lastAccessTime;
    private int k;

    public PageMetadata(int k, long currentTime) {
        this.k = k;
        this.accessHistory = new long[k];
        Arrays.fill(accessHistory, 0);
        this.lastAccessTime = currentTime;
        this.accessHistory[k - 1] = currentTime;
    }

    public long getLastAccessTime() {
        return lastAccessTime;
    }

    public long getBackwardKDistance() {
        return accessHistory[0] == 0 ? Long.MAX_VALUE : // 未满K次访问时视为无限大
                System.currentTimeMillis() - accessHistory[0];
    }

    public void updateAccess(long currentTime) {
        // 左移数组，移除最旧时间
        System.arraycopy(accessHistory, 1, accessHistory, 0, k - 1);
        // 更新时间
        this.lastAccessTime = currentTime;
        this.accessHistory[k - 1] = currentTime;
    }
}
```

