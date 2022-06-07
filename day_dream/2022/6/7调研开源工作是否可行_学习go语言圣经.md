# 调研行业
40分钟
昨天看到了PPIO可以远程办公做开源，和Databend很像，然后发现貌似现在也只有这两家在这样干，而且PPIO的薪资还是偏低的(~~可能是已经把工程师租房少花的成本已经提前计算了...但工程师不一定回了农村啊~~)

所以就调研了一下开源工作的可行性: https://www.zhihu.com/question/355691918

## 关于吃饭
然后发现就是中国的开源商业性还在成长阶段，还是不足以支持持续性"吃饭"，所以还是的找个厂上班，提升技术，持续观望

## 关于热爱
个人对开源当做理想来热爱还是可以的，但是手写英文文档还是有点蹩脚的，既然没饭吃，还让自己这么蹩脚干嘛

# 技术
## github个人页再思考
20分钟
参考自: https://github.com/kautukkundan/Awesome-Profile-README-templates/blob/master/elaborate/JoeyBling.md

- 发现语言统计的也可以换主题: https://github.com/anuraghazra/github-readme-stats
- 发现标签图标其实不用自己保存...可以在这个网站生成: https://shields.io/

## go语言圣经
40分钟

下次第7章从这里开始: https://flydk.gitbooks.io/go/content/ch7/ch7-08.html
第8章下次从这里开始: https://flydk.gitbooks.io/go/content/ch8/ch8-10.html

### 优美的适配器类型--就怕自己在实际项目中可能想不到使用它...到时候有这种需求再说，多理解这个使用场景
在下面的程序中，我们创建一个ServeMux并且使用它将URL和相应处理/list和/price操作的handler联系起来，这些操作逻辑都已经被分到不同的方法中。然后我门在调用ListenAndServe函数中使用ServeMux为主要的handler。

gopl.io/ch7/http3
```go
func main() {
    db := database{"shoes": 50, "socks": 5}
    mux := http.NewServeMux()
    mux.Handle("/list", http.HandlerFunc(db.list))
    mux.Handle("/price", http.HandlerFunc(db.price))
    log.Fatal(http.ListenAndServe("localhost:8000", mux))
}

type database map[string]dollars

func (db database) list(w http.ResponseWriter, req *http.Request) {
    for item, price := range db {
        fmt.Fprintf(w, "%s: %s\n", item, price)
    }
}

func (db database) price(w http.ResponseWriter, req *http.Request) {
    item := req.URL.Query().Get("item")
    price, ok := db[item]
    if !ok {
        w.WriteHeader(http.StatusNotFound) // 404
        fmt.Fprintf(w, "no such item: %q\n", item)
        return
    }
    fmt.Fprintf(w, "%s\n", price)
}
```
让我们关注这两个注册到handlers上的调用。第一个db.list是一个**方法值** (§6.4)，它是下面这个类型的值。

```go
func(w http.ResponseWriter, req *http.Request)
```
也就是说db.list的调用会援引一个接收者是db的database.list方法。所以db.list是一个实现了handler类似行为的函数，但是因为它没有方法（理解：该方法没有它自己的方法），所以它不满足http.Handler接口并且不能直接传给mux.Handle。

语句http.HandlerFunc(db.list)是一个转换而非一个函数调用，因为http.HandlerFunc是一个类型。它有如下的定义：

net/http

```go
package http

type HandlerFunc func(w ResponseWriter, r *Request)

func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
    f(w, r)
}
```
HandlerFunc显示了在Go语言接口机制中一些不同寻常的特点。**这是一个实现了接口http.Handler的方法的函数类型。ServeHTTP方法的行为是调用了它的函数本身。因此HandlerFunc是一个让函数值满足一个接口的适配器**，这里函数和这个接口仅有的方法有相同的函数签名。实际上，这个技巧让一个单一的类型例如database以多种方式满足http.Handler接口：一种通过它的list方法，一种通过它的price方法等等。

# 锻炼
20分钟

# 总结
加2个自信币和2个自信点