# Golang

## 基础
### 数据类型

```go
bool                                          // 不允许将布尔型强制转换
string                                        // go 中通过反引号`表示语法块
int  int8  int16  int32(rune)  int64          // go 中不允许将整型强制转换为布尔型；rune 代表了一个 Unicode 字符
uint uint8(byte) uint16 uint32 uint64 uintptr // byte 代表一个 ASCII 字符；由于uint8的取值范围是 0~255，因此也可以很好的表示RGB十进制的取值范围；uintptr 被设定为足够存放一个指针
                                              // int 与 uint 自适应32与64位平台
byte                                          // uint8 的别名，表示一个 ASCII 字符
rune                                          // int32 的别名，表示一个 Unicode 码点（UTF-8字符）
float32 float64                               // 应该尽可能使用 float64，因为 math 包中所有有关数学运算的函数都会要求接收这个类型
                                              // float32占用 4 字节（单精度），float64则占用 8 字节（双精度），指数和小数默认为float64
complex64 complex128                          // 复数类型，分别表示 32/64 位实数和虚数，complex128 为复数的默认类型
```

### make & new

|              | `make()`                              | `new()`                             |
|--------------|---------------------------------------|-------------------------------------|
| **适用类型** | 引用类型：`slice`、`map`、`channel`   | 值类型：`int`、`string`、`struct` 等 |
| **返回值**   | 已初始化的类型本身                    | 指向零值的指针 `*T`                 |
| **使用频率** | 常用                                  | 极少（`var` 声明已自动初始化为零值）|

**注意：**
- 引用类型零值为 `nil`，未经 `make` 初始化直接赋值会引发 `panic: assignment to entry in nil map`
- `cap()` 查看容量，`len()` 查看长度（均适用于 `make` 类型）
- `for...range` 可遍历所有 `make` 类型：
  - `slice` / `array`：返回 `(index, value)`
  - `map`：返回 `(key, value)`
  - `channel`：返回 `(value, ok)`，`ok` 表示通道是否已关闭

## 标准库

### 文本与字符串
- `fmt`：格式化输出，常用函数`fmt.Println()`, `fmt.Sprintf()`, `fmt.Printf()`, `fmt.Errorf()`
- `strings`：字符串处理，常用函数 `strings.Contains()`, `strings.Split()`, `strings.Join()`, `strings.ReplaceAll()`, `strings.HasPrefix()`
- `strconv`：字符串与基本类型间的类型转换
    - `strconv.Atoi()`：字符串转整数
    - `strconv.Itoa()`：整数转字符串
    - `strconv.ParseInt()`：字符串转整数
    - `strconv.FormatFloat()`：浮点数转字符串
- `bytes`：处理 `[]byte` 切片
- `regexp`：正则表达式

### 错误处理
- `errors`：
    - `errors.New()`：创建一个新的错误
    - `errors.Is()`：判断错误链中是否存在某个错误
    - `errors.As()`：提取错误链中的特定错误类型
    - `errors.Join()`：合并多个错误

### 并发控制
- `sync`：常用并发原语
    - `sync.Mutex`：互斥锁
    - `sync.RWMutex`：读写锁，适用于读多写少的场景，读并发性能更好
    - `sync.Once`：保证某段代码只执行一次，常用于单例初始化
    - `sync.WaitGroup`：等待一组 goroutine 全部完成
    - `sync.Map`：并发安全的 map
    - `sync.Pool`：并发安全的对象池，可复用临时对象以减少 GC 压力，如在高并发 HTTP 服务中复用 `bytes.Buffer`
- `sync/atomic`：原子操作，常用函数 `atomic.AddInt64()`, `atomic.LoadInt64()`, `atomic.CompareAndSwapInt64()`
- `context`：在 goroutine 间传递截止时间、取消信号和请求级数据
    - `context.Background()`：返回空的根 context，通常作为顶层起点
    - `context.WithTimeout()`：创建带超时时间的 context，到期自动取消
    - `context.WithDeadline()`：创建带截止时间的 context，到期自动取消
    - `context.WithCancel()`：创建可手动取消的 context
    - `context.WithValue()`：创建携带键值对的 context，用于传递请求级数据

### 文件系统
- `io`：最核心的 I/O 抽象接口，如`io.Reader`, `io.Writer`, `io.Closer`，常用函数`io.ReadAll()`, `io.Copy()`
- `os`：跨平台的文件操作、环境变量读取、进程交互等，常用函数 `os.Open()`, `os.ReadFile()`, `os.WriteFile()`, `os.Getenv()`, `os.MkdirAll()`
- `bufio`：包装 `io.Reader`/`io.Writer` 提供带缓冲区的读写，常用函数 `bufio.NewScanner()`, `bufio.NewReader()`
- `path/filepath`：处理文件路径，如`filepath.Join()`, `filepath.Split()`, `filepath.Abs()`, `filepath.Ext()`

### 网络
- `net/http`：HTTP 客户端和服务端，常用函数 `http.Get()`, `http.Post()`, `http.ListenAndServe()`, `http.HandleFunc()`
- `net`：TCP、UDP、IP、Unix Socket 等网络编程，常用函数 `net.Listen()`, `net.Dial()`, `net.ResolveTCPAddr()`
- `net/url`：URL 解析与编码，常用函数 `url.Parse()`, `url.QueryEscape()`, `url.QueryUnescape()`

### 序列化与编码
- `encoding/json`：常用函数 `json.Marshal()`, `json.Unmarshal()`
- `encoding/base64`
- `encoding/xml`
- `encoding/csv`
- `encoding/hex`

### slices/maps
- `slices`：
    - `slices.Contains()`：判断切片是否包含某个元素
    - `slices.Sort()`：对切片进行排序
    - `slices.Compact()`：对切片进行去重
    - `slices.Compare()`：逐元素比较两个切片，返回 0（相等）、-1（s1 < s2）或 +1（s1 > s2）
- `maps`：
    - `maps.Keys()`：返回 map 所有键组成的切片（顺序不固定）
    - `maps.Values()`：返回 map 所有值组成的切片
    - `maps.Clone()`：浅拷贝一个 map
    - `maps.Copy(dst, src)`：将 src 中所有键值对复制到 dst
    - `maps.Equal(m1, m2)`：判断两个 map 是否包含相同的键值对

### 时间与数值计算
- `time`：
    - `time.Now()`：获取当前时间
    - `time.NewTicker()`：定时触发，返回 `<-chan Time`，使用后需 `Ticker.Stop()` 释放资源
    - `time.After()`：一段时间后触发，返回 `<-chan Time`，不可复用
    - `time.Sleep()`：休眠一段时间
    - `time.Since()`：计算时间差，返回 `Duration`
    - `time.Duration`：时间间隔
- `math`：
    - `math.MaxInt64`, `math.MaxFloat64`, `math.Pi`, `math.E` 等常量
    - `math.Abs()`, `math.Pow()`, `math.Sqrt()` 等数学函数
- `math/rand/v2`：普通业务逻辑的随机数生成，如生成随机数验证码、抽奖等
- `crypto/rand`：密码学安全的随机数生成，如生成 UUID、密钥、加密随机数等

### 日志
- `log/slog`：结构化日志库（支持 JSON 格式输出、日志级别控制、Key-Value 键值对日志）

### 运行时
- `flag`：Go 标准库的 `flag` 包可满足基本的命令行参数解析需求，但缺乏必填参数校验、短选项/长选项别名等功能。实际项目中通常选用功能更完整的第三方库 `spf13/cobra`。
- `reflect`：在运行时动态检查变量类型、获取结构体 Tag、动态调用方法（如 ORM 框架和序列化框架大量使用）
- `runtime`：运行时信息与控制
    - `runtime.NumGoroutine()`：当前活跃的 goroutine 数量
    - `runtime.NumCPU()`：当前机器的逻辑 CPU 数量
    - `runtime.GOMAXPROCS(n)`：设置可并行执行的最大 CPU 数，传 0 仅查询不修改
    - `runtime.Gosched()`：主动让出当前 goroutine 的 CPU 时间片
    - `runtime.GC()`：手动触发一次 GC
    - `runtime.Stack(buf, all)`：将当前 goroutine（或所有 goroutine）的调用栈写入 buf

### 测试
- `testing`：Go 内置测试框架，核心类型为 `testing.T`（单元测试）、`testing.B`（基准测试）、`testing.F`（模糊测试），但缺少断言函数，实际项目中通常搭配第三方库 `github.com/stretchr/testify` 使用：
    - `testify/assert`：断言失败后继续执行后续测试
    - `testify/require`：断言失败后立即终止当前测试
    - `testify/mock`：生成 mock 对象，用于隔离外部依赖

## 自带工具

- **`run`**：编译并运行当前模块
- **`test`**：支持单元测试（TestXxx）、基准性能测试（BenchmarkXxx）、模糊测试（FuzzXxx）、代码覆盖率（`-cover`）、竞态检测（`-race`）
- `build`
    - **`-ldflags`**：传递参数给链接器
        - **`-X`**：动态注入变量值，如版本号、构建时间
        - `-w`：去掉 DWARF 调试信息
        - `-s`：移除符号表和调试信息
    - **`-race`**：开启竞态检测
    - `-trimpath`：移除源码路径（推荐生产环境使用）
    - `-x`：打印编译时调用的底层指令
- **`mod`**
    - **`init`**：初始化模块（生成 go.mod）
    - **`tidy`**：整理依赖（自动增删无用包）
    - `vendor`：将依赖拷贝到 vendor 目录
    - `download`：手动下载依赖包
    - `verify`：校验模块校验和
- **`get`**：添加、修改或升级项目的第三方依赖版本，并更新 go.mod
- `fmt`：格式化代码
- `vet`：静态代码分析工具，检查潜在的代码问题（如错误的 Printf 格式化参数、`sync.Mutex` 被值拷贝等锁误用）
- `env`：查看或修改 Go 的环境变量
- **`install`**：将二进制文件安装到 `$GOPATH/bin`（或 `$GOBIN`）目录下，常用于安装 CLI 工具
- `tool`
    - **`pprof`**：CPU、内存（Heap）、Goroutine、阻塞（Block）等性能分析工具，可以生成交互式命令行或 Web 架构的分析图表
    - **`trace`**：可视化执行追踪工具，可以通过浏览器查看具体的 Goroutine 调度、垃圾回收（GC）暂停时间以及网络 I/O 阻塞情况
    - **`cover`**：将 `go test -coverprofile` 生成的覆盖率文件渲染为 HTML 网页，直观查看哪行代码没被测试覆盖到
    - `compile`：Go 编译器，可用于查看汇编代码（如 `go tool compile -S main.go`）
    - `link`：Go 链接器
    - `cgo`：处理 Go 与 C 语言互操作的工具
    - `nm`：查看二进制文件中的符号表
    - `objdump`：对 Go 二进制文件进行反汇编
    - `addr2line`：将二进制文件的指令地址转换为源码对应的文件名和行号
- `generate`：配合代码中的 `//go:generate` 注释，自动触发代码生成脚本（如生成枚举映射、Protobuf 代码等）
- `work`：方便在本地同时修改和调试多个相互依赖的 Go 模块
- `clean`：清理编译产生的临时文件、对象文件和缓存
- `fix`：自动修复（不推荐直接使用，容易破坏代码逻辑）

## 底层原理

- [Go语言原本](https://golang.design/under-the-hood/)
- [Go语言101](https://gfw.go101.org/article/101.html)

### GMP
### GC
### 调度器