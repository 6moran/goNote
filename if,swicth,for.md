**if:**

```go
var count int = 40
if count < 30 {
	fmt.Println("口罩不足")
}
//建议省略括号
```



**switch:**

```go
var num int = 6
switch num {
    case 1:
       fmt.Println("1")
       //每个case后面默认省略一个break
    case 2:
       fmt.Println("2")
       fallthrough
       //fallthrough表示switch穿透,直接输出后面一个的case
    case 3:
       fmt.Println("3")
    case 4:
       fmt.Println("4")
    default:
       fmt.Println("其他")
}
```



**for:**

```go
//形式1
var sum int = 0
for i := 0; i <= 5; i++ {
    sum += i
}

//形式2，类似while
i2 := 0
for i2 <= 5 {
    fmt.Println(i2)
    i2++
}

//死循环
for {
	fmt.Println("Goland")
}

var str string = "hello golang"
//普通for循环：按照字节进行遍历输出的（暂时先不使用中文）
for i := 0; i < len(str); i++ {
    fmt.Printf("%c", str[i])
}
fmt.Println()

//for range形式
var str1 string = "hello golang 你好"
//索引给到i，具体值给到value(一个汉字三个字节)，for range对字符进行遍历
for i, value := range str1 {
    fmt.Printf("索引为%d，具体的值为%c\n", i, value)
}
```

