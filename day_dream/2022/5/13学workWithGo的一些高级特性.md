# 技术
## workWithGo
100分钟

关于go get

```bash
 ⚡ 05/13|21:25:11  code  go get 
go: no install location for directory /root/code/workWithGo/code outside GOPATH
        For more details see: 'go help gopath'
 ✘ ⚡ 05/13|21:28:02  code  go env -w GO111MODULE=on 
 ⚡ 05/13|21:28:19  code  go get                  
go: go.mod file not found in current directory or any parent directory.
        'go get' is no longer supported outside a module.
        To build and install a command, use 'go install' with a version,
        like 'go install example.com/cmd@latest'
        For more information, see https://golang.org/doc/go-get-install-deprecation
        or run 'go help get' or 'go help install'.
 ✘ ⚡ 05/13|21:28:21  code  go run memcache.go
memcache.go:7:2: no required module provides package github.com/bradfitz/gomemcache/memcache: go.mod file not found in current directory or any parent directory; see 'go help modules'
 ✘ ⚡ 05/13|21:28:37  code  go mod init code
go: creating new go.mod: module code
go: to add module requirements and sums:
        go mod tidy
 ⚡ 05/13|21:28:53  code  go run memcache.go
memcache.go:7:2: no required module provides package github.com/bradfitz/gomemcache/memcache; to add it:
        go get github.com/bradfitz/gomemcache/memcache
 ✘ ⚡ 05/13|21:29:09  code   go get github.com/bradfitz/gomemcache/memcache
go: downloading github.com/bradfitz/gomemcache v0.0.0-20220106215444-fb4bf637b56d
go: added github.com/bradfitz/gomemcache v0.0.0-20220106215444-fb4bf637b56d
 ⚡ 05/13|21:29:18  code  go run memcache.go                             
Error fetching from memcache dial tcp 127.0.0.1:11211: connect: connection refused
Error setting memcache item dial tcp 127.0.0.1:11211: connect: connection refused
```

# 锻炼
20分钟

# 总结
加2个自信币和2个自信点 