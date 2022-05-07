# 技术
## ladder
10分钟

gcp taiwan 节点的ip被ban了

gcp换ip，只要把一个ip先设置为静态ip，然后把静态ip绑定到无，然后节点就会生成新的ip，然后删除掉静态ip即可

参考: https://blog.twofei.com/800/

然后ssh到新的ip，root密码和之前的一样，可以去看v2ray的管理面板，发现这个东西会自动把配置里面的ip更新为当前ip，所以这个v2ray的管理脚本还是挺智能的，爽，剩下工作就是告知自己的使用者更换一下配置就行了

## wsl1和wsl2的代理
10分钟
```bash
# wsl1的方式
# alias setproxy="export ALL_PROXY=socks5://127.0.0.1:10808" (只用访问当地的10808) , wsl1是和windows共用网络
# wsl2指向 resolv.conf里面的windows网关，以及访问局域网端口10810 （高版本的v2rayN），wsl2是类似虚拟机，有较完整内核
# 因为重启可能会导致网关以及ip变化，以及resolv.conf一般不会配置其他的nameserver,所以可以用如下bash获取网关
host_ip=$(cat /etc/resolv.conf |grep "nameserver" |cut -f 2 -d " ")
alias setproxy="export ALL_PROXY=socks5://$host_ip:10810"
alias unsetproxy="unset ALL_PROXY"
setproxy
```

## go语言圣经
150分钟

大部分Go语言核心团队的成员都参与了该书校对工作，因此该书的质量是可以完全放心的。

**同时，单凭阅读和学习其语法结构并不能真正地掌握一门编程语言，必须进行足够多的编程实践——亲自编写一些程序并研究学习别人写的程序。**要从利用Go语言良好的特性使得程序模块化，充分利用Go的标准函数库以Go语言自己的风格来编写程序。书中包含了上百个精心挑选的习题，希望大家能先用自己的方式尝试完成习题，然后再参考官方给出的解决方案。

```bash
# go get 需要go.mod了
# go mod init 和 设置了 go proxy,以及设置了 go GO111MODULE还是失败，所以直接git 下载代码
 ⚡ 05/7|17:51:03  code  go get gopl.io/ch1/helloworld
go: go.mod file not found in current directory or any parent directory.
        'go get' is no longer supported outside a module.
        To build and install a command, use 'go install' with a version,
        like 'go install example.com/cmd@latest'
        For more information, see https://golang.org/doc/go-get-install-deprecation
        or run 'go help get' or 'go help install'.
 ✘ ⚡ 05/7|17:51:23  code  go install gopl.io/ch1/helloworld
go: 'go install' requires a version when current directory is not in a module
        Try 'go install gopl.io/ch1/helloworld@latest' to install the latest version
 ✘ ⚡ 05/7|17:52:31  code  go install gopl.io/ch1/helloworld@latest
go: gopl.io/ch1/helloworld@latest: module gopl.io/ch1/helloworld: Get "https://goproxy.io/gopl.io/ch1/helloworld/@v/list": dial tcp: lookup goproxy.io on 172.22.128.1:53: read udp 172.22.132.78:40480->172.22.128.1:53: i/o timeout
```

go mod init 和 设置了 go proxy,以及设置了 go GO111MODULE还是失败，所以直接git 下载代码 ：  git@github.com:adonovan/gopl.io.git

```bash
go run helloworld.go
Hello, 世界

go build helloworld.go
$ ./helloworld
```

### 包的概念
一个包由位于单个目录下的一个或多个.go源代码文件组成，目录定义包的作用。每个源文件都以一条package声明语句开始，这个例子里就是package main，表示该文件属于哪个包，紧跟着一系列导入（import）的包，之后是存储在这个文件里的程序语句。

main包比较特殊。它定义了一个独立可执行的程序，而不是一个库。在main里的main 函数 也很特殊，它是整个程序执行时的入口（译注：C系语言差不多都这样）。main函数所做的事情就是程序做的。当然了，main函数一般调用其它包里的函数完成很多工作（如：fmt.Println）。


### 简单摘要
s[m:n]这个切片，0 ≤ m ≤ n ≤ len(s)，包含n-m个元素。

## go语言运行引发的wsl选择问题
**因为github只添加了我windows下的ssh的密钥，所以代码clone到windows，然后wsl访问`/mnt/c`去编译运行**

并且可以创建软链接，让wsl用户目录，快速到`/mnt/c`下代码目录

```bash
# 错误链接方式: ln -s go_learn ~/code/go_learn . https://blog.csdn.net/neve_give_up_dan/article/details/124637360
 ⚡ 05/7|18:24:01  Code  ln -s /mnt/c/Users/shanl/Documents/Code/go_learn ~/code/go_learn
 ⚡ 05/7|18:24:06  Code  ll ~/code/go_learn
lrwxrwxrwx 1 root root 8 May  7 18:24 /root/code/go_learn -> /mnt/c/Users/shanl/Documents/Code/go_learn
 ⚡ 05/7|18:24:11  Code  pwd
/mnt/c/Users/shanl/Documents/Code
```

**发现wsl2进入windows的目录cd一下都超级卡...然后对于网络代理，还需要一个网关来转发，实在是卡到了极点，加上自己其实没有内核方面的相关需求，就一个简单的看系统状态的需求，其实可以直接看windows的（后面发现wsl1也能显示了），所以决定换回wsl1**

主要点是
1. 访问windows目录太卡
2. 网络转发也卡
3. 如果有时候想开VMware，wsl是不支持的，而wsl1可以关掉HV虚拟服务，然后开VMware

测试了一下，确实wsl1快一点

# 锻炼
10分钟

# 总结
加2个自信币，3个自信点，一个未来自信币