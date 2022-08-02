# 技术
90分钟
## 重学mysql
60分钟: 下次从 https://relph1119.github.io/mysql-learning-notes/#/mysql/02-MySQL%E7%9A%84%E8%B0%83%E6%8E%A7%E6%8C%89%E9%92%AE-%E5%90%AF%E5%8A%A8%E9%80%89%E9%A1%B9%E5%92%8C%E7%B3%BB%E7%BB%9F%E5%8F%98%E9%87%8F 开始

第一次见展示创建表语句
```sql
mysql> show create table engine_demo_table\G
*************************** 1. row ***************************
       Table: engine_demo_table
Create Table: CREATE TABLE `engine_demo_table` (
  `i` int(11) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8
1 row in set (0.00 sec)

mysql> 
```

## go并发
30分钟 下次: http://static.kancloud.cn/mutouzhang/go/596854

### 超时和取消 之 处理重复请求
#### 接受返回的第一个或最后一个结果
如果你的算法允许，或者你的并发进程是幂等的，那么可以简单地在下游进程中允许重复消息，并选择是否接受你收到的第一条或最后一条消息。

#### 检查goroutine许可
通过服务端询问请求端是否许可发往下一个阶段来防止重复请求

# 运动
30分钟

60仰卧起坐，30卧抬起

散步+小跑

# 总结
加2个自信币和2个自信点