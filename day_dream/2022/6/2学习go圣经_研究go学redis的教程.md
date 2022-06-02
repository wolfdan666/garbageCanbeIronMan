# 技术
## 学习go圣经
60分钟

下次第7章从这里开始: https://flydk.gitbooks.io/go/content/ch7/ch7-07.html
第8章下次从这里开始: https://flydk.gitbooks.io/go/content/ch8/ch8-10.html

### 接口值是否可比较
接口值可以使用==和!＝来进行比较。两个接口值相等仅当它们都是nil值，或者它们的动态类型相同并且动态值也根据这个动态类型的==操作相等。因为接口值是可比较的，所以它们可以用在map的键或者作为switch语句的操作数。

然而，如果两个接口值的动态类型相同，但是这个动态类型是不可比较的（比如切片），将它们进行比较就会失败并且panic:
```go
var x interface{} = []int{1, 2, 3}
fmt.Println(x == x) // panic: comparing uncomparable type []int
```
考虑到这点，接口类型是非常与众不同的。其它类型要么是安全的可比较类型（如基本类型和指针）要么是完全不可比较的类型（如切片，映射类型，和函数），但是在比较接口值或者包含了接口值的聚合类型时，我们必须要意识到潜在的panic。同样的风险也存在于使用接口作为map的键或者switch的操作数。只能比较你非常确定它们的动态值是可比较类型的接口值。

### 警告：一个包含nil指针的接口不是nil接口
```go
if out != nil {
    out.Write([]byte("done!\n")) // panic: nil pointer dereference
}
```
当main函数调用函数f时，它给f函数的out参数赋了一个*bytes.Buffer的空指针，所以out的动态值是nil。然而，它的动态类型是*bytes.Buffer，意思就是out变量是一个包含空指针值的非空接口（如图7.5），所以防御性检查out!=nil的结果依然是true。


### sort接口
```go
package sort

type Interface interface {
    Len() int
    Less(i, j int) bool // i, j are indices of sequence elements
    Swap(i, j int)
}
```
为了对序列进行排序，我们需要定义一个实现了这三个方法的类型，然后对这个类型的一个实例应用sort.Sort函数。思考对一个字符串切片进行排序，这可能是最简单的例子了。下面是这个新的类型StringSlice和它的Len，Less和Swap方法。

```go
type StringSlice []string
func (p StringSlice) Len() int           { return len(p) }
func (p StringSlice) Less(i, j int) bool { return p[i] < p[j] }
func (p StringSlice) Swap(i, j int)      { p[i], p[j] = p[j], p[i] }
```
**实现之后，StringSlice就是一个(is a)sort接口了，就可以调用接口方法了。**现在我们可以通过像下面这样将一个切片转换为一个StringSlice类型来进行排序：

```go
sort.Sort(StringSlice(names))
```

### Reverse的sort
```go
sort.Sort(sort.Reverse(byArtist(tracks)))
```
sort.Reverse函数值得进行更近一步的学习，因为它使用了(§6.3)章中的组合，这是一个重要的思路。sort包定义了一个不公开的struct类型reverse，它嵌入了一个sort.Interface。reverse的Less方法调用了内嵌的sort.Interface值的Less方法，但是通过交换索引的方式使排序结果变成逆序。

**以前一直不理解这个Reverse，以为是先排序后再倒序，现在才发现是接口封装了一下，然后重写了Less方法**
```go
package sort

type reverse struct{ Interface } // that is, sort.Interface

func (r reverse) Less(i, j int) bool { return r.Interface.Less(j, i) }

func Reverse(data Interface) Interface { return reverse{data} }
```

reverse的另外两个方法Len和Swap隐式地由原有内嵌的sort.Interface提供。因为reverse是一个不公开的类型，所以导出函数Reverse返回一个包含原有sort.Interface值的reverse类型实例。

## 找看redis go资料
40分钟
结论是没有太好的教程，还是自己看菜鸟教程加看一些文章吧

# 锻炼
20分钟

# 总结
加2个自信币和2个自信点