# 基础知识

```go
i := 1
fmt.Println("Write ", i, " as ")
switch i {
case 1:
	fmt.Println("one")
case 2:
	fmt.Println("two")
case 3:
	fmt.Println("three")
}

// 在 Go 中输出 one，如果是 Javascript 则输出 one two three
// 在 Go 中匹配到匹配项后会自动跳出 switch

// time.Weekday，返回星期几
func (t Time) Weekday() Weekday {
	return absWeekday(t.abs())
}

// 星期几的枚举类
// 定义枚举
type Weekday int

const (
	Sunday Weekday = iota
	Monday
	Tuesday
	Wednesday
	Thursday
	Friday
	Saturday
)

// switch 进行条件匹配
t := time.Now()
switch {
case t.Hour() < 12:
	fmt.Println("It's before noon")
default:
	fmt.Println("It's after noon")
}

// 匹配类型
whatAmI := func(i interface{}) {
	// 获取类型 interface{}.(type)
	switch t := i.(type) {
	case bool:
		fmt.Println("I'm a bool")
	case int:
		fmt.Println("I'm a int")
	default:
		// 输出类型 %T
		fmt.Printf("Don't know type %T\n", t)
	}
}

// slice 添加多个元素
s = append(s, "e", "f")

// 克隆数组
c := make([]string, len(s))
copy(c, s)

// 二维 Slices
twoD := make([][]int, 3)
for i := 0; i < 3; i++ {
	innerLen := i + 1
	twoD[i] = make([]int, innerLen)
	for j := 0; j < innerLen; j++ {
		twoD[i][j] = i + j
	}
}

// 删除 map 中的某个 key
delete(m, "k2")
fmt.Println("map: ", m)

// 检测 map 中的某个 Key 是否存在
_, prs := m["k2"]
fmt.Println("prs: ", prs) // false

// 限制请求频率
func main() {
	requests := make(chan int, 5)
	for i := 0; i < 5; i++ {
		requests <- i
	}
	close(requests)

	limiter := time.Tick(200 * time.Millisecond)

	for req := range requests {
		<-limiter
		fmt.Println("request: ", req, time.Now())
	}
}

// 同类型省略部分类型声明
func (s byLength) Swap(i, j int)

// 比较两个时间的早晚
p(then.Before(now))
p(then.After(now))
p(then.Equal(now))

// 计算两个时间的差距
diff := now.Sub(then)
p(diff)

p(diff.Hours())
p(diff.Minutes())
p(diff.Seconds())
p(diff.Nanoseconds())

// 对时间做加减法计算
p(then.Add(diff))
p(then.Add(-diff))

now := time.Now()
// 时间戳：秒
secs := now.Unix()
// 时间戳：纳秒
nanos := now.UnixNano()
// 时间戳：毫秒
millis := nanos / 1000000


// 随机数
rand.Intn(100)
rand.Float64()

// 字符串转换为数字
strconv.ParseFloat("1.234", 64)
strconv.ParseInt("0x1c8", 0, 64)

// 加密
s := "sha1 this string"
h := sha1.New()
h.Write([]byte(s))
bs := h.Sum(nil)

// 操作环境变量
// BAR=2 go run main.go
os.Setenv("FOO", "1")
fmt.Println("FOO: ", os.Getenv("FOO"))
fmt.Println("BAR: ", os.Getenv("BAR"))

for _, e := range os.Environ() {
	fmt.Println(e)
}

// map 的使用技巧
type client chan<- string

var (
	entering = make(chan client)
	leaving  = make(chan client)
	messages = make(chan string)
)

func broadcaster() {
	clients := make(map[client]bool)
	for {
		select {
		case msg := <-messages:
			for cli := range clients {
				cli <- msg
			}
		case cli := <-entering:
			clients[cli] = true
		case cli := <-leaving:
			delete(clients, cli)
			close(cli)
		}
	}
}

// 简单控制并发/线程数的方法
var sema = make(chan struct{}, 20)

func dirents(dir string) []os.FileInfo {
	sema <- struct{}{}
	defer func() { <-sema }()
	entries, err := ioutil.ReadDir(dir)
	if err != nil {
		fmt.Fprintf(os.Stderr, "du1: %v\n", err)
		return nil
	}
	return entries
}

// ReadAll是很常用的一个方法，用来一次性的读取io.Reader当中的数据。
// ioutil.ReadAll(resp.Body) 会先将所有的响应读出来放到内存中。如果文件太大，那么就会消耗很多内存。这样是不明智的。
// io 包提供了 io.Copy() 方法，该方法实现了两个文件句柄之间的拷贝。
ioutil.ReadAll()

// 格式化 bytes
fmt.Printf("%s", bytes)
// 返回更详细的错误信息
return nil, fmt.Errorf("parsing %s as HTML: %v", url, err)

// 反射的基本用法（基本数据类型）
func display(path string, v reflect.Value) {
	switch v.Kind() {
	case reflect.Invalid:
		fmt.Printf("%s = invalid\n", path)
	case reflect.Slice, reflect.Array:
		for i := 0; i < v.Len(); i++ {
			display(fmt.Sprintf("%s[%d]", path, i), v.Index(i))
		}
	case reflect.Struct:
		for i := 0; i < v.NumField(); i++ {
			fieldPath := fmt.Sprintf("%s.%s", path, v.Type().Field(i).Name)
			display(fieldPath, v.Field(i))
		}
	case reflect.Map:
		for _, key := range v.MapKeys() {
			display(fmt.Sprintf("%s[%s]", path, formatAtom(key)), v.MapIndex(key))
		}
	case reflect.Ptr:
		if v.IsNil() {
			fmt.Printf("%s = nil\n", path)
		} else {
			display(fmt.Sprintf("(*%s)", path), v.Elem())
		}
	case reflect.Interface:
		if v.IsNil() {
			fmt.Printf("%s = nil\n", path)
		} else {
			fmt.Printf("%s.type = %s\n", path, v.Elem().Type())
			display(path+".value", v.Elem())
		}
	default: // 基本类型、通道、函数
		fmt.Printf("%s = %s\n", path, formatAtom(v))
	}
}

// formatAtom 格式化一个值，且不分析它的内部结构
func formatAtom(v reflect.Value) string {
	switch v.Kind() {
	case reflect.Invalid:
		return "invalid"
	case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
		return strconv.FormatInt(v.Int(), 10)
	case reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64, reflect.Uintptr:
		return strconv.FormatUint(v.Uint(), 10)
		// ... 为简化起见，省略了浮点数和复数的分支...
	case reflect.Bool:
		return strconv.FormatBool(v.Bool())
	case reflect.String:
		return strconv.Quote(v.String())
	case reflect.Chan, reflect.Func, reflect.Ptr, reflect.Slice, reflect.Map:
		return v.Type().String() + " 0x" + strconv.FormatUint(uint64(v.Pointer()), 16)
	default: // reflect.Array, reflect.Struct, reflect.Interface
		return v.Type().String() + " value"
	}
}

// 正则的基本使用
tagRE := regexp.MustCompile("[0-9]{3}?")
tag := tagRE.FindString(target)
```