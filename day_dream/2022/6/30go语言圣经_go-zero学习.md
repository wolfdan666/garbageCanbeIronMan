# 技术
## go语言圣经
60分钟, 下次从这里开始: https://flydk.gitbooks.io/go/content/ch12/ch12-04.html

今天先不看资讯，因为暂时没看到更有趣的，先把基础知识学完

### reflect.TypeOf
其中 `TypeOf(3)` 调用将值 3 传给 `interface{}` 参数. 回到 7.5节接口值 的将一个具体的值转为接口类型会有一个隐式的接口转换操作, 它会创建一个包含两个信息的接口值: 操作数的动态类型(这里是int)和它的动态的值(这里是3).

因为 `reflect.TypeOf` 返回的是一个动态类型的接口值, 它总是返回具体的类型. 因此, 下面的代码将打印 `"*os.File"` 而不是 `"io.Writer"`. 稍后, 我们将看到能够表达接口类型的 `reflect.Type`.

```go
var w io.Writer = os.Stdout
fmt.Println(reflect.TypeOf(w)) // "*os.File"
```

要注意的是 `reflect.Type` 接口是满足 `fmt.Stringer` 接口的. 因为打印一个接口的动态类型对于调试和日志是有帮助的, `fmt.Printf` 提供了一个缩写 `%T` 参数, 内部使用 `reflect.TypeOf` 来输出:
```go
fmt.Printf("%T\n", 3) // "int"
```
reflect 包中另一个重要的类型是 Value. 一个 `reflect.Value` 可以装载任意类型的值. 函数 `reflect.ValueOf` 接受任意的 `interface{}` 类型, 并返回一个装载着其动态值的 `reflect.Value`. 和 `reflect.TypeOf` 类似, `reflect.ValueOf` 返回的结果也是具体的类型, 但是 `reflect.Value` 也可以持有一个接口值.

```go
v := reflect.ValueOf(3) // a reflect.Value
fmt.Println(v)          // "3"
fmt.Printf("%v\n", v)   // "3"
fmt.Println(v.String()) // NOTE: "<int Value>"
```
和 `reflect.Type` 类似, `reflect.Value` 也满足 `fmt.Stringer` 接口, 但是除非 Value 持有的是字符串, 否则 String 方法只返回其类型. 而使用 fmt 包的 `%v` 标志参数会对 `reflect.Values` 特殊处理.

## go-zero视频学习
30分钟， 下次学第三节

https://www.bilibili.com/video/BV1P3411p79J/ 

# 锻炼
走路70分钟_> 算锻炼20分钟

# 看书
20分钟

# 总结
加2个自信币和2个自信点