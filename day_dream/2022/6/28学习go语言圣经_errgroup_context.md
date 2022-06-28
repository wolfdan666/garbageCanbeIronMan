# 技术
100分钟， 下次从这里开始: https://flydk.gitbooks.io/go/content/ch12/ch12-01.html

## go语言圣经
60分钟

```go
letters := make([]rune, 0, len(s))
for _, r := range s {
    if unicode.IsLetter(r) {
        letters = append(letters, unicode.ToLower(r))
    }
}
```
这个改进提升性能约35%，报告结果是基于2,000,000次迭代的平均运行时间统计。
```bash
$ go test -bench=.
PASS
BenchmarkIsPalindrome-8 2000000                      697 ns/op
ok      gopl.io/ch11/word2      1.468s
```
**如这个例子所示，快的程序往往是伴随着较少的内存分配。**

## 示例函数
下面是IsPalindrome函数对应的示例函数：

```go
func ExampleIsPalindrome() {
    fmt.Println(IsPalindrome("A man, a plan, a canal: Panama"))
    fmt.Println(IsPalindrome("palindrome"))
    // Output:
    // true
    // false
}
```

根据示例函数的后缀名部分，godoc这个web文档服务器会将示例函数关联到某个具体函数或包本身，因此ExampleIsPalindrome示例函数将是IsPalindrome函数文档的一部分，Example示例函数将是包文档的一部分。

示例函数的第二个用处是，在go test执行测试的时候也会运行示例函数测试。如果示例函数内含有类似上面例子中的// Output:格式的注释，那么测试工具会执行这个示例函数，然后检查示例函数的标准输出与注释是否匹配。

## 反射
探讨Go语言的反射特性，看看它可以给语言增加哪些表达力，以及在两个至关重要的API是如何使用反射机制的：一个是fmt包提供的字符串格式化功能，另一个是类似encoding/json和encoding/xml提供的针对特定协议的编解码功能。对于我们在4.6节中看到过的text/template和html/template包，它们的实现也是依赖反射技术的。**然后，反射是一个复杂的内省技术，不应该随意使用，因此，尽管上面这些包内部都是用反射技术实现的，但是它们自己的API都没有公开反射相关的接口。**

强大的但是比较脆弱工具，因为没用好，容易造成大问题。

## context，errgroup学习
40分钟

示例暂未看懂: https://www.liwenzhou.com/posts/Go/error-in-goroutine/
还未看完: https://wizardforcel.gitbooks.io/go42/content/content/42_37_context.html

# 运动
20分钟

# 读书
30分钟

# 总结
加2个自信币和2个自信点