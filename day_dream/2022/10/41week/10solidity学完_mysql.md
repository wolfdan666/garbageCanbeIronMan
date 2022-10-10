# 技术
## 根儿mysql
90分钟
### 18章 InnoDB的buffer pool
#### chunk_size注意点
innodb_buffer_pool_chunk_size的值并不包含缓存页对应的控制块的内存空间大小，所以
实际上InnoDB向操作系统申请连续内存空间时，每个chunk的大小要比innodb_buffer_pool_chunk_size
的值大一些，约5%。

#### non-youngs和made not young, young-making rate等的理解
non-youngs/s ：**代表每秒由于不满足时间限制而不能从 old 区域移动到 young 区域头部的节点数量。**

Page made not young ：在将 innodb_old_blocks_time 设置的值大于0时，首次访问或者后续访问某个处在 old 区域的节点时由于不符合时间间隔的限制而不能将其移动到 young 区域头部时， Page made not young 的值会加1。

这里需要注意，对于处在 young 区域的节点，如果由于它在 young 区域的1/4处而导致它没有被移动到young 区域头部，这样的访问并不会将 Page made not young 的值加1。（**这里是不能将其搞成young，而不是从young搞成old**）

young-making rate ：表示在过去某段时间，平均访问1000次页面，有多少次访问使页面移动到 young 区域的头部了。

需要大家注意的一点是，这里统计的将页面移动到 young 区域的头部次数不仅仅包含从 old 区域移动到young 区域头部的次数，还包括从 young 区域移动到 young 区域头部的次数（访问某个 young 区域的节点，只要该节点在 young 区域的1/4处往后，就会把它移动到 young 区域的头部）。

not (young-making rate) ：表示在过去某段时间，平均访问1000次页面，有多少次访问没有使页面移动到 young 区域的头部。

需要大家注意的一点是，这里统计的没有将页面移动到 young 区域的头部次数不仅仅包含因为设置了innodb_old_blocks_time 系统变量而导致访问了 old 区域中的节点但没把它们移动到 young 区域的次数，还包含因为该节点在 young 区域的前1/4处而没有被移动到 young 区域头部的次数。

### 19章 从猫爷被杀说起-事务简介
下次从这里开始: 19.1.1 原子性（Atomicity）

### 小感想
发现很多mysql实现原理里面的知识点，涉及到的术语，都是我以前在SDS里面听过的但没完全理解的术语，所以属于是储存通用了，挺好的，可以帮助自己多了解一些术语的含义

## solidity入门学完
2小时

# 运动
10分钟

60仰卧起坐，30卧抬起

# 总结
+3.5自信币+3.5自信点
