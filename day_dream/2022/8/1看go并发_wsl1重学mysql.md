# 技术

## concurrence in go
90分钟 下次从这里开始看: http://static.kancloud.cn/mutouzhang/go/596853

### Context存储和检索附加于请求的数据包
接下来我们讨论context包的另一个用处：存储和检索附加于请求的数据包。请记住，当一个函数创建一个goroutine和Context时，它通常会启动一个为请求提供服务的进程，并且子函数可能需要相关的请求信息。下面是一个示例：
```go
func main() {
	ProcessRequest("jane", "abc123")
}

func ProcessRequest(userID, authToken string) {
	ctx := context.WithValue(context.Background(), "userID", userID)
	ctx = context.WithValue(ctx, "authToken", authToken)
	HandleResponse(ctx)
}

func HandleResponse(ctx context.Context) {
	fmt.Printf("handling response for %v (%v)",
		ctx.Value("userID"),
		ctx.Value("authToken"),
	)
}
```
这会输出：
```
handling response for jane (abc123)
```
很简单的用法。不过也是有限制的：

- 你使用的key必须在Go中是可比较的，也就是说，== 和 != 必须能返回正确的结果。
- 返回值必须是并发安全的，这样才能从多个goroutine访问。

由于Context的键和值都被定义为interface{}，所以当试图检索值时，我们会失去其类型安全性。基于此，Go建议在context中存储和检索值时遵循一些规则。

首先，推荐你在包中自行定义key的类型，这样无论是否其他包执行相同的操作都可以防止context中的冲突。看下面这个例子：

```go
type foo int
type bar int

m := make(map[interface{}]int)
m[foo(1)] = 1
m[bar(1)] = 2

fmt.Printf("%v", m)
```
这会输出：

```
map[1:2 1:1]
```
**可以看到，虽然基础值是相同的，但不同类型的信息会在map中区分它们。由于你为包定义的key类型未导出，因此其他包不会与你在包中生成的key冲突。**

由于用于存储数据的key是非导出的，因此我们必须导出执行检索数据的函数。这很容易做到，因为它允许这些数据的使用者使用静态的，类型安全的函数。

当你把所有这些放在一起时，你会得到类似下面的例子：

```go
func main() {
	ProcessRequest("jane", "abc123")
}

type ctxKey int

const (
	ctxUserID    ctxKey = iota
	ctxAuthToken
)

func UserID(c context.Context) string {
	return c.Value(ctxUserID).(string)
}

func AuthToken(c context.Context) string {
	return c.Value(ctxAuthToken).(string)
}

func ProcessRequest(userID, authToken string) {
	ctx := context.WithValue(context.Background(), ctxUserID, userID)
	ctx = context.WithValue(ctx, ctxAuthToken, authToken)
	HandleResponse(ctx)
}

func HandleResponse(ctx context.Context) {
	fmt.Printf(
		"handling response for %v (auth: %v)",
		UserID(ctx),
		AuthToken(ctx),
	)
}
```
这会输出：

```
handling response for jane (auth: abc123)
```
在本例中，我们使用类型安全的方法来从Context获取值，如果消费者在不同的包中，他们不会知道或关心用于存储信息的key。 但是，这种技术会造成隐患。

在前面的例子中，我们假设HandleResponse存在于另一个名为response的包中，假设ProcessRequest包位于名为pross的包中。 pross包必须导入response包才能调用HandleResponse，但HandleResponse无法访问pross包中定义的访问函数，因为导入会形成循环依赖关系。由于用于Context中存储的key类型对于process包来说是私有的，所以response包无法检索这些数据！

这迫使我们创建以从多个位置导入的数据类型为中心的包。 虽然这不是一件坏事，但它是需要注意的。

context包非常简洁，但依然褒贬不一。在Go社区中一直存在争议。该包的取消操作功能相当受欢迎，但是在Context中存储任意数据的能力以及存储数据的类型不安全的造成了一些分歧。 虽然我们已经部分减缓了访问函数缺乏类型安全性的问题，但是仍然可以通过存储不正确的类型来引入错误。然而，更大的问题在于，开发人员到底应该在Context的实例中存储什么样的数据。

在context包的文档中这样写到：

使用context存储值仅适用于传输进程和API的请求附加数据，而不用于将可选参数传递给函数。

该说明十分含糊，“传输进程和API的请求”实在太过宽泛。我认为最好的解读方法是与开发组一起提出一些约定，并在代码评审中检查它们：

1. **数据应该是由进程传递的或与API相关。**如果你在进程的内存中生成数据，那么除非你也通过API传递数据，否则可能不是一个很好的候选应用程序。
2. **数据应该是不可变的。**如果可变，那么根据定义，你存储的内容肯定不是来自请求。
3. **数据应指向系统简单类型。**我们在上面已经讨论了关于使用安全类型的包导入问题，这个结论是很明显的。
4. **数据应该是纯粹的数据，而不是某种类型的函数。**消费者的逻辑应该是消耗这些数据。
5. **数据应该有助于操作，而不是驱动操作。**如果你的算法根据context中包含或不包含的内容而有所不同，那么就违背了“不用于将可选参数传递”的初衷。


这些不是硬性规定，但如果你发现自己的程序与以上约定由冲突，则可能需要考虑是否存在隐患或使用context是否必要。

另一个需要考虑的方面是该数据在使用之前可能需要经过多少层。如果在接受数据的位置和使用位置之间有几个框架和几十个函数，你可以考虑使用日志，并将数据添加为参数；或者你更愿意将它放在Context中，从而创建一个不可见的依赖关系。每种方法都有优点，最终这由你和你的团队做出决定。

我将**这5条约定做成了表格**，你可以将之作为参考：

| Data | 1 | 2 | 3 | 4 | 5 |
| --- | --- | --- | --- | --- | --- |
| Request ID | √ | √ | √ | √ | √ |
| User ID | √ | √ | √ | √ | |
| URL | √ | √ |  |  | |
| API Server Connection |  |  |  |  | |
| Authorization Token | √ | √ | √ | √ | |
| Request Token | √ | √ | √ |  | |

对于是否有必要使用context存储值，这里并没有简单的答案，具体取决于你的业务、算法和团队。

我留给你的最后建议是Context提供的取消功能非常有用，这样轻便的功能如果不用实在是太可惜了。

## 重学mysql-从根儿上理解mysql
30分钟 下次从这里开始: https://relph1119.github.io/mysql-learning-notes/#/mysql/01-%E8%A3%85%E4%BD%9C%E8%87%AA%E5%B7%B1%E6%98%AF%E4%B8%AA%E5%B0%8F%E7%99%BD-%E9%87%8D%E6%96%B0%E8%AE%A4%E8%AF%86MySQL 最后一行的mysql>是一个客户端的提示符，之后客户端发送给服务器的命令都需要写在这个提示符后边。
### 安装
wsl1 ubuntu18.04上面直接安装

```bash
sudo apt-get install mysql-server

sudo apt-get install mysql-client

sudo apt-get install libmysqlclient-dev
```

然后没有指定目录`/usr/local/mysql/bin`，而是把执行文件分散到了`/usr/bin`和`/usr/sbin`下面了，没有mysql.server

### 启动mysqld出问题
启动说没有管道`/var/run/mysqld`，那就创建一个这个目录

```bash
 ✘ ⚡ 08/1|20:09:59  learn  mysqld_safe
2022-08-01T12:10:05.800417Z mysqld_safe Logging to syslog.
2022-08-01T12:10:05.967989Z mysqld_safe Logging to '/var/log/mysql/error.log'.
2022-08-01T12:10:06.099380Z mysqld_safe Directory '/var/run/mysqld' for UNIX socket file don't exists.

 ✘ ⚡ 08/1|20:14:39  learn  mkdir /var/run/mysqld/mysqld.sock
mkdir: cannot create directory ‘/var/run/mysqld/mysqld.sock’: No such file or directory


 ✘ ⚡ 08/1|20:14:55  learn  mkdir -p /var/run/mysqld/mysqld.sock

 ✘ ⚡ 08/1|20:15:06  learn  mysqld_safe
2022-08-01T12:15:20.102736Z mysqld_safe Logging to syslog.
2022-08-01T12:15:20.280740Z mysqld_safe Logging to '/var/log/mysql/error.log'.
2022-08-01T12:15:21.081028Z mysqld_safe Starting mysqld daemon with databases from /var/lib/mysql
2022-08-01T12:15:23.102598Z mysqld_safe mysqld from pid file /var/run/mysqld/mysqld.pid ended
 ⚡ 08/1|20:15:23  learn 
```

# 运动
30分钟

60仰卧起坐，30卧抬起

散步+小跑

# 总结
加2个自信币和2个自信点