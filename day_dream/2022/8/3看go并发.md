# 技术

## concurrence in go
120分钟 下次从这里开始看: http://static.kancloud.cn/mutouzhang/go/596859

### 心跳
建议阅读原文

### 请求并发复制处理
需要平衡复制请求的代价和处理时间的收益 造成的性价比是否是想要的，可接受的

```go
doWork := func(done <-chan interface{}, id int, wg *sync.WaitGroup, result chan<- int) {

	started := time.Now()
	defer wg.Done()

	// 模拟随机加载
	simulatedLoadTime := time.Duration(1+rand.Intn(5)) * time.Second
	select {
	case <-done:
	case <-time.After(simulatedLoadTime):
	}

	select {
	case <-done:
	case result <- id:
	}

	took := time.Since(started)
	// 显示处理程序将花费多长时间
	if took < simulatedLoadTime {
		fmt.Printf("%v took time %v lower than simulatedLoadTime %v\n", id, took, simulatedLoadTime)
		took = simulatedLoadTime
	}
	fmt.Printf("%v took %v\n", id, took)
}

done := make(chan interface{})
result := make(chan int)

var wg sync.WaitGroup
wg.Add(10)

for i := 0; i < 10; i++ { //1
	go doWork(done, i, &wg, result)
}

firstReturned := <-result //2
close(done)               //3
wg.Wait()

fmt.Printf("Received an answer from #%v\n", firstReturned)


/* 
 ⚡ 08/3|17:05:41  test  go run tmp.go
4 took time 1.0013932s lower than simulatedLoadTime 3s
4 took 3s
7 took time 1.0014065s lower than simulatedLoadTime 5s
7 took 5s
0 took 1.0012962s
2 took 1.0014013s
3 took time 1.0011971s lower than simulatedLoadTime 2s
3 took 2s
1 took time 1.001413s lower than simulatedLoadTime 4s
1 took 4s
9 took time 1.0013894s lower than simulatedLoadTime 2s
9 took 2s
5 took 1.0009741s
6 took time 1.0014642s lower than simulatedLoadTime 3s
6 took 3s
8 took time 1.0015575s lower than simulatedLoadTime 2s
8 took 2s
Received an answer from #5
*/
```

### 速率限制
如果你曾经使用过API来获取服务，那么你可能经受过与速率限制相抗衡。速率限制使得某种资源每次访问的次数受限。资源可以是任何东西：API连接，磁盘I/O，网络包，错误。

你有没有想过为什么会需要制定速率限制？为什么不允许无限制地访问系统？最明显的答案是，通过对系统进行速率限制，可以防止整个系统遭受攻击。如果恶意用户可以在他们的资源允许的情况下极快地访问你的系统，那么他们可以做各种事情。

例如，他们可以用日志消息或有效的请求填满服务器的磁盘。**如果你的日志配置错误，他们甚至可能会执行恶意的操作，然后提交足够的请求，将任何活动记录从日志中移出并放入/dev/null中导致日志系统完全崩溃。他们可能试图暴力访问资源，或者执行分布式拒绝服务攻击。如果你没有对系统进行请求限制，你的系统就成了一个走在街上不穿衣服的大美女。（这个例子过于形象了，离谱）**

后面的文章要结合代码和运行结果仔细看，所以推荐看原文，实现之后，还是很优美的
```go
func Open() *APIConnection {
	return &APIConnection{
		apiLimit: MultiLimiter( //1
			rate.NewLimiter(Per(2, time.Second), 2),
			rate.NewLimiter(Per(10, time.Minute), 10)),
		diskLimit:    MultiLimiter(rate.NewLimiter(rate.Limit(1), 1)),      //2
		networkLimit: MultiLimiter(rate.NewLimiter(Per(3, time.Second), 3)) //3
	}
}

type APIConnection struct {
	networkLimit,
	diskLimit,
	apiLimit RateLimiter
}

func (a *APIConnection) ReadFile(ctx context.Context) error {
	err := MultiLimiter(a.apiLimit, a.diskLimit).Wait(ctx) //4
	if err != nil {
		return err
	}
	// Pretend we do work here
	return nil
}

func (a *APIConnection) ResolveAddress(ctx context.Context) error {
	err := MultiLimiter(a.apiLimit, a.networkLimit).Wait(ctx) //5
	if err != nil {
		return err
	}
	// Pretend we do work here
	return nil
}
/* 
01:40:15 ResolveAddress
01:40:15 ReadFile
01:40:16 ReadFile
01:40:17 ResolveAddress
01:40:17 ResolveAddress
01:40:17 ReadFile
01:40:18 ResolveAddress
01:40:18 ResolveAddress
01:40:19 ResolveAddress
01:40:19 ResolveAddress
01:40:21 ResolveAddress
01:40:27 ResolveAddress
01:40:33 ResolveAddress
01:40:39 ReadFile
01:40:45 ReadFile
01:40:51 ReadFile
01:40:57 ReadFile
01:41:03 ReadFile
01:41:09 ReadFile
01:41:15 ReadFile
01:41:15 Done.
*/
```

### Goroutines异常行为修复
监控程序 (还有使用的地方需要去看下原文)
```go
type startGoroutineFn func(done <-chan interface{}, pulseInterval time.Duration) (heartbeat <-chan interface{}) //1

newSteward := func(timeout time.Duration, startGoroutine startGoroutineFn) startGoroutineFn { //2

	return func(done <-chan interface{}, pulseInterval time.Duration) <-chan interface{} {

		heartbeat := make(chan interface{})

		go func() {
			defer close(heartbeat)

			var wardDone chan interface{}
			var wardHeartbeat <-chan interface{}
			startWard := func() { //3
				wardDone = make(chan interface{})                             //4
				wardHeartbeat = startGoroutine(or(wardDone, done), timeout/2) //5
			}
			startWard()
			pulse := time.Tick(pulseInterval)

		monitorLoop:
			for { //6
				timeoutSignal := time.After(timeout)
				for {
					select {
					case <-pulse:
						select {
						case heartbeat <- struct{}{}:
						default:
						}
					case <-wardHeartbeat: //7
						continue monitorLoop
					case <-timeoutSignal: //8
						log.Println("steward: ward unhealthy; restarting")
						close(wardDone)
						startWard()
						continue monitorLoop
					case <-done:
						return
					}
				}
			}
		}()

		return heartbeat
	}
}

doWorkFn := func(done <-chan interface{}, intList ...int) (startGoroutineFn, <-chan interface{}) { //1

	intChanStream := make(chan (<-chan interface{})) //2
	intStream := bridge(done, intChanStream)

	doWork := func(done <-chan interface{}, pulseInterval time.Duration) <-chan interface{} { //3
		intStream := make(chan interface{}) //4
		heartbeat := make(chan interface{})

		go func() {
			defer close(intStream)
			select {
			case intChanStream <- intStream: //5
			case <-done:
				return
			}

			pulse := time.Tick(pulseInterval)

			for {
			valueLoop:
				for _, intVal := range intList {
					if intVal < 0 {
						log.Printf("negative value: %v\n", intVal) //6
						return
					}

					for {
						select {
						case <-pulse:
							select {
							case heartbeat <- struct{}{}:
							default:
							}
						case intStream <- intVal:
							continue valueLoop
						case <-done:
							return
						}
					}
				}
			}
		}()
		return heartbeat
	}
	return doWork, intStream
}
```

**使用这样的方式可以确保你的系统保持健康，此外，相信系统崩溃的减少也能大幅度降低开发过程中猝死的几率。愿诸君健康工作，准点下班。**

最后一句话，泪目了

# 运动
30分钟

60仰卧起坐，30卧抬起

散步+小跑

# 总结
加2.5个自信币和2.5个自信点