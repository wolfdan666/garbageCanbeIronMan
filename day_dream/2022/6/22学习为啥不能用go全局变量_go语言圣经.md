# 技术
## review
60分钟(已经减去了发呆时间)

[A theory of modern Go](https://peter.bourgon.org/blog/2017/06/09/theory-of-modern-go.html)
2017 06 09
tl;dr: magic is bad; global state is magic → no package level vars; no func init

The single best property of Go is that it is basically non-magical. With very few exceptions, a straight-line reading of Go code leaves no ambiguity about definitions, dependency relationships, or runtime behavior. This makes Go relatively easy to read, which in turn makes it relatively easy to maintain, which is the single highest virtue of industrial programming.

But there are a few ways that magic can creep in. One unfortunately very common way is through the use of global state. Package-global objects can encode state and/or behavior that is hidden from external callers. Code that calls on those globals can have surprising side effects, which subverts the reader’s ability to understand and mentally model the program.

Functions (including methods) are basically the only mechanism that Go has to build abstraction. Consider the following function definition.

```go
func NewObject(n int) (*Object, error)
```
By convention, we expect that functions of the form NewXxx are type constructors. That expectation is validated when we see that the function returns a pointer to an Object, and an error. From this we can deduce that the constructor function may or may not succeed, and if it fails, that we will receive an error telling us why. We observe that the function takes a single int parameter, which we assume controls some aspect or capability of the returned Object. Presumably, there is some constraint on n, which, if not met, will result in an error. But because the function takes no other parameter, we expect it should have no other effect, beyond (hopefully) allocating some memory.

By reading the function signature alone, we are able to make all of these deductions, and build a mental model of this function. This process, applied repeatedly and recursively from the first line of func main, is how we read and understand programs.

Now, consider if this were the body of the function.

```go
func NewObject(n int) (*Object, error) {
	row := dbconn.QueryRow("SELECT ... FROM ... WHERE ...")
	var id string
	if err := row.Scan(&id); err != nil {
		logger.Log("during row scan: %v", err)
		id = "default"
	}
	resource, err := pool.Request(n)
	if err != nil {
		return nil, err
	}
	return &Object{
		id:  id,
		res: resource,
	}, nil
}
```
The function invokes a package global database/sql.Conn, to make a query against some unspecified database; a package global logger, to output a string of arbitrary format to some unknown location; and a package global pool object of some kind, to request a resource of some kind. All of these operations have side effects that are completely invisible from an inspection of the function signature. There is no way for a caller to predict any of these things will happen, except by reading the function and diving to the definition of all of the globals.

Consider this alternative signature.

```go
func NewObject(db *sql.DB, pool *resource.Pool, n int, logger log.Logger) (*Object, error)
```
By lifting each of the dependencies into the signature as parameters, we allow readers to accurately model the scope and potential behaviors of the function. The caller knows exactly what the function needs to do its work, and can provide them accordingly.

If we’re designing the public API for this package, we can even take it one helpful step further.

```go
// RowQueryer models part of a database/sql.DB.
type RowQueryer interface {
	QueryRow(string, ...interface{}) *sql.Row
}

// Requestor models the requesting side of a resource.Pool.
type Requestor interface {
	Request(n int) (*resource.Value, error)
}

func NewObject(q RowQueryer, r Requestor, n int, logger log.Logger) (*Object, error) {
	// ...
}
```
By modeling each concrete object as an interface, capturing only the methods we use, we allow callers to swap in alternative implementations. This reduces source-level coupling(耦合) between packages, and enables us to mock out the concrete dependencies in tests. Testing the original version of the code, with concrete package-level globals, involves tedious and error-prone swapping-out of components.

If all of our constructors and functions take their dependencies explicitly, then we no longer have any use for globals. Instead, we can construct all of our database connections, our loggers, our resource pools, in our func main, so that future readers can very clearly map out a component graph. And we can very explicitly pass those dependencies to the components that use them, so that we eliminate the comprehension-subverting magic of globals. Also, observe that if we have no global variables, we have no more use for func init, whose only purpose is to instantiate or mutate package-global state. We can then look at all uses of func init with appropriate suspicion: what is this code doing? Why is it not in func main, where it belongs?

It’s not only possible, but quite easy, and actually extremely refreshing, to write Go programs that are practically free of global state. In my experience, programming in this way is not noticeably slower or more tedious than using global variables to shrink function definitions. On the contrary: when a function signature reliably and completely describes the behavior-scope of the function body, we can reason about, refactor, and maintain code in the large much more efficiently. Go kit has been written in this style since the very beginning, to its great benefit.

— – -

From this, we can develop a theory of modern Go. Based on the words of Dave “Humbug” Cheney, I propose the following guidelines:

- No package level variables
- No func init

There are exceptions, of course. But from these rules, the other practices follow naturally.

### 感想
确实，全局变量虽然可以减少定义时的代码量，但坏处是对库调用者隐藏了太多的东西，而且不利于库调用者理解都是干了啥，并且会在报错的时候，让调用者完全没有办法

而通过接口式的方式暴露出需要的参数，方便调用者可以更加方便的进行参数修改，减少代码耦合性

### 问题
还是不能使用单例...有人和我一样的疑问，而且是在2年前...现在还没解决，我真的挺无语的

https://forum.golangbridge.org/t/how-to-resolve-those-golang-ci-lint-issue/17168

所以我被迫提问: https://stackoverflow.com/questions/72709336/how-to-use-sync-once-in-golangci-lint-i-got-err-once-is-a-global-variable-goc

### 尝试以及新问题
```go
var _backTask backTask //go:embed test
```
确实没有全局变量报错，但是出了新问题:(我之前还以为是双斜线必须和文本之间有空格的comment的那个warning，结果是另外的warning)
The "embed" package must be imported when using go:embed directives.embed

### stackoverflow最终结果
我好不容易回答了一个C/C++的问题，然后获得了1个赞，stackoverflow声誉达到11点，想着再问一个好问题，去达到15点，拥有投票权，结果被两个大佬批了（stackoverflow的veto(否决):This question does not show any research effort; it is unclear or not useful），说我完全可以不用这个全局检测，真的尴尬，然后声誉掉到了7点

vote: 投票
veto: 否决

不删除问题吧，我觉得有些人应该也会犯我的错，用这个留作教训

> There are many cases where package-level variables are appropriate and this is one of them. Check to see if there's a source code comment that you can use to silence the warning from the problem tool. – 
RedBlue
 4 hours ago

> You have a lot of options: 1. Do not run golangcli-lint at all. 2. Disable the noglobals rule. 3. Do not use a global. – 
Volker
 4 hours ago


不过还好让我知道了不能全信golangci-lint中集成的一些lint，因为有些lint比较极端，看过了[A theory of modern Go](https://peter.bourgon.org/blog/2017/06/09/theory-of-modern-go.html)，其实也应该知道这不是要禁止所有的global variables...

## go语言圣经
60分钟, 下次: https://flydk.gitbooks.io/go/content/ch10/ch10-07.html 10.7.4. 包文档
### 包匿名导入
如果只是导入一个包而并不使用导入的包将会导致一个编译错误。但是有时候我们只是想利用导入包而产生的副作用：**它会计算包级变量的初始化表达式和执行导入包的init初始化函数**（§2.6.2）。这时候我们需要抑制“unused import”编译错误，我们可以用下划线_来重命名导入的包。像往常一样，下划线_为空白标识符，并不能被访问。

### 工作区结构
GOPATH对应的工作区目录有三个子目录。**其中src子目录用于存储源代码。**每个包被保存在与$GOPATH/src的相对路径为包导入路径的子目录中，例如gopl.io/ch1/helloworld相对应的路径目录。我们看到，一个GOPATH工作区的src目录中可能有多个独立的版本控制系统，例如gopl.io和golang.org分别对应不同的Git仓库。**其中pkg子目录用于保存编译后的包的目标文件，bin子目录用于保存编译后的可执行程序，例如helloworld可执行程序。**

第二个环境变量GOROOT用来指定Go的安装目录，还有它自带的标准库包的位置。GOROOT的目录结构和GOPATH类似，因此存放fmt包的源代码对应目录应该为$GOROOT/src/fmt。用户一般不需要设置GOROOT，默认情况下Go语言安装工具会将其设置为安装的目录路径。

### go get一些事情
go get命令获取的代码是真实的本地存储仓库，而不仅仅只是复制源文件，因此你依然可以使用版本管理工具比较本地代码的变更或者切换到其它的版本。例如golang.org/x/net包目录对应一个Git仓库：
```bash
$ cd $GOPATH/src/golang.org/x/net
$ git remote -v
origin  https://go.googlesource.com/net (fetch)
origin  https://go.googlesource.com/net (push)
```
需要注意的是导入路径含有的网站域名和本地Git仓库对应远程服务地址并不相同，真实的Git地址是go.googlesource.com。这其实是Go语言工具的一个特性，可以让包用一个自定义的导入路径，但是真实的代码却是由更通用的服务提供，例如googlesource.com或github.com。因为页面 https://golang.org/x/net/html 包含了如下的元数据，它告诉Go语言的工具当前包真实的Git仓库托管地址：
```bash
$ go build gopl.io/ch1/fetch
$ ./fetch https://golang.org/x/net/html | grep go-import
<meta name="go-import"
      content="golang.org/x/net git https://go.googlesource.com/net">
```

`go get -u`命令只是简单地保证每个包是最新版本，如果是第一次下载包则是比较方便的；但是对于发布程序则可能是不合适的，因为本地程序可能需要对依赖的包做精确的版本依赖管理。通常的解决方案是使用vendor的目录用于存储依赖包的固定版本的源代码，对本地依赖的包的版本更新也是谨慎和持续可控的。在Go1.5之前，一般需要修改包的导入路径，所以复制后golang.org/x/net/html导入路径可能会变为gopl.io/vendor/golang.org/x/net/html。最新的Go语言命令已经支持vendor特性，但限于篇幅这里并不讨论vendor的具体细节。不过可以通过`go help gopath`命令查看Vendor的帮助文档。

(译注：墙内用户在上面这些命令的基础上，还需要学习用翻墙来go get。) -- 笑了，害

### go build,go install
`go install`命令和`go build`命令很相似，但是它会保存每个包的编译成果，而不是将它们都丢弃。被编译的包会被保存到`$GOPATH/pkg`目录下，目录路径和 src目录路径对应，可执行程序被保存到`$GOPATH/bin`目录。（很多用户会将`$GOPATH/bin`添加到可执行程序的搜索列表中。）还有，`go install`命令和`go build`命令都不会重新编译没有发生变化的包，这可以使后续构建更快捷。为了方便编译依赖的包，`go build -i`命令将安装每个目标所依赖的包。

```go
func main() {
    fmt.Println(runtime.GOOS, runtime.GOARCH)
}
```
下面以64位和32位环境分别编译和执行：

```bash
$ go build gopl.io/ch10/cross
$ ./cross
darwin amd64
$ GOARCH=386 go build gopl.io/ch10/cross
$ ./cross
darwin 386
```
有些包可能需要针对不同平台和处理器类型使用不同版本的代码文件，以便于处理底层的可移植性问题或为一些特定代码提供优化。如果一个文件名包含了一个操作系统或处理器类型名字，例如net_linux.go或asm_amd64.s，Go语言的构建工具将只在对应的平台编译这些文件。还有一个特别的构建注释参数可以提供更多的构建过程控制。例如，文件中可能包含下面的注释：

```go
// +build linux darwin
```
在包声明和包注释的前面，该构建注释参数告诉go build只在编译程序对应的目标操作系统是Linux或Mac OS X时才编译这个文件。下面的构建注释则表示不编译这个文件：

```go
// +build ignore
```
更多细节，可以参考go/build包的构建约束部分的文档。

```bash
$ go doc go/build
```

# 锻炼
20分钟步行+8分钟 61仰卧起坐+46俯卧撑

# 读书
32分钟

# 总结
加3个自信币3个自信点