# 技术
共计100分钟

## go test
共计45分钟, 直接ctrl点击import里面的库就能跳转看到go的官方包说明，真的爽

15分钟紧急学习go的单元测试

30min 研究[gopkg.in/check.v1](https://pkg.go.dev/gopkg.in/check.v1@v1.0.0-20201130134442-10cb98267c6c#pkg-constants) 

因为 gopkg.in/check.v1 , 中的 setUpSuite函数，所以又看了一下 [Package suite](https://pkg.go.dev/github.com/stretchr/testify/suite)


## workWithGo
55分钟 workWithGo

提前学Working with Go - Testing

### 变量和包命名不要命名成一样的
```go
// 这里json_data 千万不要命名成为 json  ， 否则会导致下面的 json.Unmarshal 无法识别...
json_data, err := json.Marshal(p)
if err != nil {
    fmt.Println("JSON Encoding Error", err)
}

fmt.Println(string(json_data))

json_str := `{ "Name": "Marcus", "City": "San Jose"}`
var person Person

if err := json.Unmarshal([]byte(json_str), &person); err != nil {
    fmt.Println("Error parsing JSON: ", err)
}
```

### 解码json数组文件，文件不会填写 
```go
var people []Person

file, err := ioutil.ReadFile("names.json")
if err != nil {
    fmt.Println("Error reading file")
}

// the names.json file has an array of person objects,
// so read into people
if err := json.Unmarshal(file, &people); err != nil {
    fmt.Println("Error parsing JSON", err)
}

// output result
fmt.Println(people)
```

### go test小问题
```bash
go test
go: go.mod file not found in current directory or any parent directory; see 'go help modules'


go env -w GO111MODULE=auto
go test
# _/root/code/workWithGo/code [_/root/code/workWithGo/code.test]
./commandline.go:14:6: main redeclared in this block
        ./arrays.go:5:6: other declaration of main
./ctrl_stu.go:5:6: main redeclared in this block
        ./arrays.go:5:6: other declaration of main
./dateAndTime.go:27:6: main redeclared in this block
        ./arrays.go:5:6: other declaration of main
./directories.go:10:6: main redeclared in this block

# 重开一个子文件去做go test
```

# 锻炼
暴雨中行走50分钟--->算锻炼20分钟

# 总结
加2个自信币和2个自信点