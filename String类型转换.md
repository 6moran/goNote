### 基本数据类型的默认值

| 数据类型   | 默认值 |
| ---------- | ------ |
| 整数类型   | 0      |
| 浮点类型   | 0      |
| 布尔类型   | false  |
| 字符串类型 | ""     |



------



### 数据类型的转换

**1.基本数据类型转String**

```go
var n1 int = 180
var n2 float32 = float32(n1)
```

**方法1:**

```go
var n1 int = 18
var n2 float32 = 3.3
var s1 string = fmt.Sprintf("%d", n1)
fmt.Printf("s1的数据类型为:%T, s1=%v\n", s1, s1)
var s2 string = fmt.Sprintf("%f", n2)
fmt.Printf("s2的数据类型为:%T, s2=%v\n", s2, s2)
```

**方法2:**

```go
import "strconv"
var s3 string = strconv.FormatInt(int64(n1), 10) 
//第一个参数必须是int64类型,第二个参数为10进制
fmt.Printf("s3的数据类型为:%T, s3=%v\n", s3, s3)
var s4 string = strconv.FormatFloat(n2, 'f', 9, 64)
//第二个参数: 'f' (-ddd.dddd) 第三个参数: 9 保留小数点后面9位  第四个参数:表示其为float64类型的
fmt.Printf("s4的数据类型为:%T, s4=%v\n", s4, s4)
var s5 string = strconv.FormatBool(true)
fmt.Printf("s5的数据类型为:%T, s5=%v\n", s5, s5)
```

```go
func main() {
    num := 123
    str := strconv.Itoa(num)
    fmt.Printf("整数 %d  转换为字符串为：'%s'\n", num, str)
}
```

**2.String转基本数据类型**

```go
import "strconv"

var str1 string = "true"
var b bool
b, _ = strconv.ParseBool(str1)
//ParseBool的返回值有两个,(value bool , err error)
//value就是我们的得到布尔类型的数值,err为出现的错误
fmt.Printf("b的类型是: %T, b=%v\n", b, b)

var str2 string = "19"
var num1 int64
num1, _ = strconv.ParseInt(str2, 10, 64)
fmt.Printf("num1的类型是: %T, num1=%v\n", num1, num1)

var str3 string = "3.14"
var num3 float64
num3, _ = strconv.ParseFloat(str3, 64)
fmt.Printf("num3的类型是: %T, num3=%v", num3, num3)
```

```go
func main() {
    str := "123"
    num, err := strconv.Atoi(str)
    if err != nil {
       fmt.Println("转换错误:", err)
    } else {
       fmt.Printf("字符串 '%s' 转换为整数为：%d\n", str, num)
    }
}
```

