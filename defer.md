**defer的作用：**

```
func add(num1 int, num2 int) int {
    //defer先把它后面的语句压入栈中，也会将相关的值同时拷贝入栈中，不会随着函数后面的变化而变化
    defer fmt.Println("num1=", num1) //30
    defer fmt.Println("num2=", num2) //60
    //栈的特点是先进后出，所以先输出num2，在输出num1

    num1 += 90 //num1=120
    num2 += 50 //num2=110

    var sum int = num1 + num2
    fmt.Println("sum=", sum) //230
    return sum

}
fmt.Println("----------------------------------")
add(30, 60)
```

**执行结果：**![image-20250907094126406](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20250907094126406.png)



------



