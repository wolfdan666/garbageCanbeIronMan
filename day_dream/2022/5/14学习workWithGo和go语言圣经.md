# 技术
110分钟

## workWithGo
30分钟

学完了,之前产生的2.5个支线自信币添加到自信币

看完了Working with MySQL Database，但是在家没有环境测试，所以pass

**When defining a struct you do not explicitly specify the interface it implements. Go determines automatically if a given type satisfies an interface if it defines all the methods.**

```go
package main

import "fmt"

type Vehicle interface {
    Alert() string
}

type Car struct { }

func (c Car) Alert() string {
    return "Honk! Honk!"
}

type Bicycle struct { }

func (b Bicycle) Alert() string {
    return "Ring! Ring!"
}

func main() {
    c := Car{}
    b := Bicycle{}

    vehicles := []Vehicle{c, b}
    for _, v := range vehicles {
        fmt.Println(v.Alert())
    }
}
```

## go语言圣经
80分钟

golang中的map是hash实现的

map的迭代顺序并不确定，从实践来看，该顺序随机，每次运行都会变化。这种设计是有意为之的，因为能防止程序依赖特定遍历顺序，而这是无法保证的。

```
%d          十进制整数
%x, %o, %b  十六进制，八进制，二进制整数。
%f, %g, %e  浮点数： 3.141593 3.141592653589793 3.141593e+00
%t          布尔：true或false
%c          字符（rune） (Unicode码点)
%s          字符串
%q          带双引号的字符串"abc"或带单引号的字符'c'
%v          变量的自然形式（natural format）
%T          变量的类型
%%          字面上的百分号标志（无操作数）
```

# 锻炼
10分钟

# 总结
加2个自信币2个自信点