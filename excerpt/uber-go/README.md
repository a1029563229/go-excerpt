# Uber Go 语言代码规范

- 零值 Mutex 是有效的

```go
var mu sync.Mutex
mu.Lock()
```

- 如果你使用结构体指针，mutex 可以非指针形式作为结构体的组成字段，或者更好的方式是直接嵌入到结构体中。如果是私有结构体类型或是实现 Mutex 接口的类型，我们可以使用嵌入 mutex 的方法：

```go
type SMap struct {
  mu sync.Mutex // 对于导出类型，请使用私有锁

  data map[string]string
}

func NewSMap() *SMap {
  return &SMap{
    data: make(map[string]string)
  }
}

func (m *SMap) Get(k string) string {
  m.mu.Lock()
  defer m.mu.Unlock()

  return m.data[k]
}
```


- 在边界处拷贝 Slices 和 Maps
- slices 和 maps 包含了指向底层数据的指针，因此在需要复制它们时要特别注意。
- 请注意，当 map 和 slice 作为函数参数传入时，如果您存储了对它们的引用，则用户可以对其进行修改

```go
func (d *Driver) SetTrips(trips []Trip) {
  d.trips = make([]Trip, len(trips))
  copy(d.trips, trips)
}

trips := ...
d1.SetTrips(trips)

// 这里我们修改 trips[0]，但不会影响到 d1.trips
trips[0] = ...
```

- 返回 slices 或 maps 时，同样请注意用户对暴露内部状态的 map 和 slice 的修改

```go
type Stats struct {
  mu sync.Mutex

  counters map[string]int
}

func (s *Stats) Snapshot() map[string]int {
  s.mu.Lock()
  defer s.mu.Unlock()

  result := make(map[string]int, len(s.counters))
  for k, v := range s.counters {
    result[k] = v
  }
  return result
}

// snapshot 现在是一个拷贝
snapshot := stats.Snapshot()
```

- 使用 defer 释放资源，诸如文件和锁。
- 当有多个 return 分支时，很容易遗忘 unlock；
- 使用 defer 释放资源会使代码更加具有可读性
- Defer 的开销非常小，只有在您可以证明函数执行时间处于纳秒级的程度时，才应避免这样做。使用 defer 提高可读性是值得的，因为使用它们的成本微不足道。尤其适用于那些不仅仅是简单内存访问的较大的方法，在这些方法中其他计算的资源消耗远超过 defer。

```go
p.Lock()
defer p.Unlock()

if p.count < 10 {
  return p.count
}

p.count++
return p.count
```

- Channel 的 size 要么是 1，要么是无缓冲的
- channel 通常 size 应为 1 或是无缓冲的。默认情况下，channel 是无缓冲的，其 size 为 0。任何其他尺寸都必须经过严格的审查。考虑如何确定大小，是什么阻止了 channel 在负载下被填满并阻止写入，以及发生这种情况时发生了什么。

```go
// 大小：1
c := make(chan int, 1)

// 无缓冲 channel，大小为 0
c := make(chan int)
```

- 枚举从 1 开始

```go
type Operation int

const (
  Add Operation = iota + 1
  Subtract
  Multiply
)

// Add=1, Subtract=2, Multiply=3
```

### 错误处理

- 如果客户端需要检测错误，并且您使用创建了一个简单的错误 errors.New，请使用一个错误变量：

```go
// package foo

var ErrCouldNotOpen = errors.New("could not open")

func Open() error {
  return ErrCouldNotOpen
}

// package bar
if err := foo.Open(); err != nil {
  if err == foo.ErrCouldNotOpen {
    // ...handle
  } else {
    panic("unknown error")
  }
}
```

- 如果您有可能需要客户端检测的错误，并且想向其中添加更多信息（例如，它不是静态字符串），则应使用自定义类型：

```go
type errNotFound struct {
  file string
}

// error 类型需要实现 Error() 方法
func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func open(file string) error {
  return errorFound{file: file}
}

func use() {
  if err := open(filename); err != nil {
    if _, ok := err.(errNotFound); ok {
      /// ...handle
    } else {
      panic("unknown error")
    }
  }
}
```

- 直接导出自定义错误类型时要小心，因为它们已经成为程序包公共 API 的一部分。最好公开匹配器以检查错误：

```go
// package foo

type errNotFound struct {
  file string
}

func (e errNotFound) Error() string {
  return fmt.Sprintf("file %q not found", e.file)
}

func IsNotFoundError(err error) bool {
  _, ok := err.(errNotFound)
  return ok
}

func Open(file string) error {
  return errNotFound{file: file}
}

// package bar

if err := foo.Open("foo"); err != nil {
  if foo.IsNotFoundError(err) {
    // handle
  } else {
    panic("unknown error")
  }
}
```

- 处理类型断言失败
- type assertion 的单个返回值形式针对不正确的类型将产生 panic。因此，请始终使用“comma ok”的惯用法

```go
t, ok := i.(string)
if !ok {
  // 优雅的处理错误
}
```

- 不要 panic
- 在生产环境中运行的代码必须避免出现 panic。panic 是 cascading failures 级联失败的主要根源。如果发生错误，该函数必须返回错误，并允许调用方决定如何处理它。
- panic/recover 不是错误处理策略。仅当发生不可恢复的事情（例如：nil 引用）时，程序才必须 panic。程序初始化是一个例外：程序启动时应使程序中止的不良情况可能会引起 panic。

```go
func foo(bar string) error {
  if (len(bar) == 0) {
    return errors.New("bar must not be empty")
  }
  //...
  return nil
}

func main() {
  if len(os.Args) != 2 {
    fmt.Println("USAGE: foo <bar>")
    os.Exit(1)
  }
  if err := foo(os.Args[1]); err != nil {
    panic(err)
  }
}
```

- 避免可变全局变量
- 使用选择依赖注入方式避免改变全局变量。既适用于函数指针又适用于其他值类型：

```go
type singer struct {
  now func() time.Time
}

func newSigner() *signer {
  return &signer{
    now: time.Now,
  }
}

func (s *singer) Sign(msg string) string {
  now := s.now()
  return signWithTime(msg, now)
}
```

### 规范

- 一致性
- 相似的声明放在一组

```go
// 同类型的常量、变量和类型声明放在一起
const (
  a = 1
  b = 2
)

var (
  a = 1
  b = 2
)

type (
  Area float64
  Volume float64
)

// 仅将相关的声明放在一组，不要将不相关的声明放在一组

// 分组使用的位置没有限制，例如：你可以在函数内部使用它们：
func f() string {
  var (
    red   = color.New(0xff0000)
    green = color.New(0x00ff00)
    blue  = color.New(0x0000ff)
  )
}
```