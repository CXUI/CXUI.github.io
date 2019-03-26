---
layout: post
title:  "Go 的编译与工具学习笔记"
date:   2019-03-25 13:00:00

categories: go
tags: tip
author: "Victor"
---

## 编译

### go build
将源码编译为可执行文件。

* `go build` 无参数编译：如果源码中没有依赖 GOPATH 的包引用，就可以使用无参数编译，会在当前目录下生成与目录同名的可执行文件。
* `go build 文件列表`：默认选择文件列表中第一个源码文件作为可执行文件名输出，列表中的文件必须是同一个包的源码。
* `go build -o myexec main.go lib.go`：-o 参数可以指定输出可执行文件名。
* `go build 包`：包名是相对于 src 目录开始算，`go build -o main directory/go-project`

### go run
编译源码并直接运行 main() 函数，不会在当前目录留下可执行文件。

```
go run -gcflags "-m -l" main.go
```

`-gcflags` 参数是编译参数，后面 `-m` 表示进行内存分配分析 `-l` 表示避免程序内联，也就是避免进行程序优化。

### go install
将编译文件的中间文件放在 GOPATH 的 pkg 目录下，并将编译结果放在 GOPATH 的 bin 目录下。

### go get
`go get 远程包` 比如 `go get github.com/davyxu/cellnet` 自动获取并编译，根据实际情况可能在 GOPATH 的 bin 或者 pkg 目录下。

## 测试

单元测试，是指对软件中的最小可测试单元进行检查和验证。单元是认为规定的最小的被测功能模块。单元测试实在软件开发过程中要进行的最低级别的测试活动，应在个例的情况下进行测试。

* 命名时以 _test 结尾
* 测试方法函数以 Test 为前缀
* 在附加参数中添加 `-v` 可以让测试时显示详细的流程

```go
import "testing"
func TestHelloWorld(t *testing.T) {
   t.Log("hello world")
}
```

* go test 指定文件时默认执行文件内的所有测试用例。可以用 `-run` 参数选择需要的测试用例单独执行
* `-run` 跟随的测试用例的名称支持正则表达式，使用 `-run TestA$` 只执行 TestA 测试用例

```go
func TestFailNow(t *testing.T) {
   t.FailNow() // 终止当前测试用例
}
func TestFail(t *testing.T) {
   t.Fail() // 标记失败但继续执行
}
```

## 基准检验

基准测试可以测试一段程序的运行性能及耗费 CPU 的程度。对一个测试用例的默认测试时间是 1 秒，若函数返回时间不足 1 秒，那么 `b.N` 会指数递增。

```go
func Benchmark_Add(b *testing.B) {
   var n int
   for i := 0; i < b.N; i++ {
      n++
   }
}

func Benchmark_Alloc(b *testing.B) {
   for i := 0; i < b.N; i++ {
      fmt.Sprintf("%d", i)
   }
}
```

* 测试代码需要保证函数可重入性及无状态
* 测试代码不能使用全局变量等带有记忆性质的数据结构
* 避免多次运行同一段代码时的环境不一致，不能假设 N 值范围，而使用基准测试框架提供的 `b.N`

`go test -v -bench=. -benchtime=5s -benchmem benchmark_test.go`

* `-bench=.` 表示运行 benchmark_test.go 文件里的所有基准测试
* `-benchtime=5s` 可以自定 benchmem
* `-benchmem` 参数以显示内存分配情况
* `-bench=Alloc` 表示只测试 Benchmark_Alloc 函数

```
[I] ➜ go test -v -bench=Alloc -benchmem benchmark_test.go
goos: darwin
goarch: amd64
Benchmark_Alloc-8   	20000000	       103 ns/op	      16 B/op	       2 allocs/op
PASS
ok  	command-line-arguments	2.195s
```

20000000 表示测试的次数，103 ns/op 表示每一个操作耗费多少时间，16 B/op 表示每次调用分配 16 个字节，2 allocs/op 表示每次调用两次分配。

有些测试需要一定启动和初始化时间，会影响测试结果的精准性。我们可以手动控制计时器，从而让计时器只在需要的区间进行测试。

```go
func Benchmark_Add_TimerControl(b *testing.B) {
   // 重置计时器
   b.ResetTimer()
   // 停止计时器
   b.StopTimer()
   // 开始计时器
   b.StartTimer()

   var n int
   for i := 0; i < b.N; i++ {
      n++
   }
}
```

## 性能分析

Go 工具链中的 go pprof 可以快速分析及定位各种性能问题，比如 CPU 消耗、内存分配及阻塞分析。

首先需要使用 runtime.pprof 包嵌入带分析程序的入口和结束处，下面这个包更好用。

```bash
go get github.com/pkg/profile
```

为了保证性能分析数据的合理性，分析的最短时间是 1 秒，性能分析需要可执行配合才能生成分析结果，因此使用命令行对程序进行编译。

```go
import (
	"time"
	"github.com/pkg/profile"
)

func joinSlice() []string {
	var arr []string
	for i := 0; i < 10000; i++ {
		arr = append(arr, "arr")
	}
	return arr
}

func main() {
	stopper := profile.Start(profile.CPUProfile, profile.ProfilePath("."))

	defer stopper.Stop()

	joinSlice()

	time.Sleep(time.Second)
}
```

```
go build -o cpu cpu.go
./cpu
go tool pprof --pdf cpu cpu.pprof > cpu.pdf
```

上面的意思在当前目录输出 cpu.pprof 文件，在使用 go tool 工具链输出 pdf 格式文件，这个过程会用到 Graphviz 工具，需要提前安装。

