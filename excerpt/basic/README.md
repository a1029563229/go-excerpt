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
```