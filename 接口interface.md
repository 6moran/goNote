### 接口：

```go
// 接口，接口里面的方法是没有方法体的
type Usb interface {
    Start()
    Stop()
}

type Phone struct {
}

func (p Phone) Start() {
    fmt.Println("手机开始工作---")
}

func (p Phone) Stop() {
    fmt.Println("手机停止工作---")
}

type Camera struct {
}

func (c Camera) Start() {
    fmt.Println("相机开始工作---")
}

func (c Camera) Stop() {
    fmt.Println("相机停止工作---")
}

type Computer struct {
}

// 编写一个方法几首一个Usb接口类型变量
// 实现一个接口，就是指实现了该接口所声明的所有方法
func (c Computer) Working(usb Usb) {
    usb.Start()
    usb.Stop()
}

func main() {
    computer := Computer{}
    phone := Phone{}
    camera := Camera{}

    computer.Working(phone)
    computer.Working(camera)
}
```

golang的接口并不是显式实现，只要定义一个结构体实现了该接口的所有方法，那么它就实现了该接口。（所以一个结构体可能实现多个接口）

### 注意事项：

1.接口本身不能创建实例，但是可以指向一个实现了该接口的自定义类型的变量

```go
type AIterface interface {
	Say()
}

type Student struct {
	Name string
}

func (stu Student) Say() {
	fmt.Println("执行Say")
}

stu := Student{}
var inter AIterface = stu
inter.Say()
```

2.只要是自定义数据类型，就可以实现接口，不仅仅是结构体类型

```go
type integer int

func (i integer) Say() {
	fmt.Println("integar 执行")
}

var i integer = 2
per.Do(i)
```

3.一个自定义类型可以实现多个接口

4.接口中不能含有变量

5.一个接口可以嵌入多个别的接口，例如（A嵌入B、C接口）这时如果要实现A接口就需要把B、C接口也全部实现。

![image-20250914164259968](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20250914164259968.png)

6.interface类型默认为一个指针（引用类型），如果没有初始化就使用那么会输出nil

7.空接口interface{}没有任何方法，所以所有的类型都实现了空接口，即我们可以把任何变量赋给空接口

```go
//方法一
type D interface {
}
var d D = stu
fmt.Println(d)

//方法二
var t interface{} = 23
fmt.Println(t)

//空接口运用type switch
switch v := v.(type) {
    case int:
        fmt.Println("Integer:", v)
    case string:
        fmt.Println("String:", v)
    default:
        fmt.Println("Unknown type")
}
```

8.接口不能有两个相同的方法，嵌入得来的也不能一样

```go
type B interface {
    test1()
    test2()
}

// 这里如果test2的参数不一样，就会编译错误：方法重复了
type C interface {
    test2()
    test3()
}
type A interface {
    B
    C
}
```

9.接口的类型转换

**类型断言（用于将接口类型转换为指定类型）**

**(用于从接口类型中提取其底层值)**

`value.(type)`

```go
func main() {
    var i interface{} = "Hello, World"
    str, ok := i.(string)
    if ok {
       fmt.Printf("'%s' is a string\n", str)
    } else {
       fmt.Println("conversion failed")
    }
}

```

**类型转换（用于将一个接口类型转换为另一个接口类型）**

`T(value)`

在类型转换中，我们必须保证要转换的值和目标接口类型之间是兼容的，否则编译器会报错。

