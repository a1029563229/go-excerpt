# Go 笔记

## 命令行的使用

```go
// 构建二进制文件
go build

// 安装全局命令
go install /usr/local/go/src/hello
```

## 基础

```go
// 定义变量
var variable_name variable_type;
variable_name = value;
var i, j, k int;
var c, ch byte;
var f, salary float32;
d = 42;

// 定义函数
func function_name( [parameter list] ) [return_types] {
  body of the function
}

// 访问内存地址
var a int = 10   
fmt.Printf("Address of a variable: %x\n", &a  )

// 定义复合结构类型
type struct_variable_type struct {
  member definition;
  member definition;
  ...
}

variable_name := struct_variable_type { value1, value2, ... };

// 使用 range 进行 for 循环
package main

import "fmt"

func main() {
   /* create a slice */
   numbers := []int{0,1,2,3,4,5,6,7,8} 
   
   /* print the numbers */
   for i:= range numbers {
      fmt.Println("Slice item",i,"is",numbers[i])
   }
   
   /* create a map*/
   countryCapitalMap := map[string] string {"France":"Paris","Italy":"Rome","Japan":"Tokyo"}
   
   /* print map using keys*/
   for country := range countryCapitalMap {
      fmt.Println("Capital of",country,"is",countryCapitalMap[country])
   }
   
   /* print map using key-value*/
   for country,capital := range countryCapitalMap {
      fmt.Println("Capital of",country,"is",capital)
   }
}

// 定义 map
var map_variable map[key_data_type]value_data_type
map_variable = make(map[key_data_type]value_data_type)

// 抛错
func Sqrt(value float64)(float64, error) {
   if(value < 0){
      return 0, errors.New("Math: negative number passed to Sqrt")
   }
   return math.Sqrt(value)
}
```