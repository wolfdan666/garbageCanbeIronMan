# 技术
90分钟
## concurrence in go
60分钟 , 看完了这本书，后面的一些部分由于翻译版没有更新，所以看的是英文原本，用的**知云文献翻译**，感觉还不错，挺好，终于看完了go并发，舒服了

### work-stealing
All things considered, stealing continuations are considered to be theoretically supe‐ rior to stealing tasks, and therefore it is best to queue the continuation and not the goroutine

综合考虑所有因素，偷接被认为在理论上优于偷接任务，因此最好是排队继续而不是goroutine

So why don’t all work-stealing algorithms implement continuation stealing? Well, continuation stealing usually requires support from the compiler. Luckily, Go has its own compiler, and continuation stealing is how Go’s work-stealing algorithm is implemented. Languages that don’t have this luxury usually implement task, or socalled “child, ” stealing as a library.

那么，为什么不是所有的工作窃取算法都实现了延续窃取呢?嗯，延续窃取通常需要编译器的支持。幸运的是，Go有自己的编译器，延续窃取是Go工作窃取算法的实现方式。没有这种奢侈的语言通常实现任务，或所谓的“孩子”，窃取作为一个库。

#### GMP模型
Go’s scheduler has three main concepts:
G
 A goroutine.
M
 An OS thread (also referenced as a machine in the source code).
P
 A context (also referenced as a processor in the source code).

In our discussion about work stealing, M is equivalent to T, and P is equivalent to the
work deque (changing `GOMAXPROCS` changes how many of these are allocated). The G
is a goroutine, but keep in mind it represents the current `state` of a goroutine, most
notably its program counter (PC). This allows a G to represent a continuation so Go
can do continuation stealing.

## mysql
30分钟 下次从这里开始: https://relph1119.github.io/mysql-learning-notes/#/mysql/02-MySQL%E7%9A%84%E8%B0%83%E6%8E%A7%E6%8C%89%E9%92%AE-%E5%90%AF%E5%8A%A8%E9%80%89%E9%A1%B9%E5%92%8C%E7%B3%BB%E7%BB%9F%E5%8F%98%E9%87%8F?id=%e6%9c%8d%e5%8a%a1%e5%99%a8%e7%a8%8b%e5%ba%8f%e8%bf%90%e8%a1%8c%e8%bf%87%e7%a8%8b%e4%b8%ad%e8%ae%be%e7%bd%ae
### 系统配置项
```bash
mysqld --default-storage-engine=MyISAM --max-connections=10
```

可以看到`default_storage_engine`和`max_connections`这两个系统变量的值已经被修改了。有一点需要注意的是，**对于启动选项来说，如果启动选项名由多个单词组成，各个单词之间用短划线-或者下划线_连接起来都可以，但是对应的系统变量之间必须使用下划线_连接起来。**

# 运动
30分钟

60仰卧起坐，30卧抬起

散步+小跑

# 总结
加2个自信币和2个自信点