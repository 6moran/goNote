init：初始化函数，可以进行一些初始化操作

**（在main函数之前执行）**

```go
package main

import "fmt"

func init() {
    fmt.Println("init执行了")
}

func main() {
    fmt.Println("main执行了")
}
```

执行结果:

![image-20250907081356818](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20250907081356818.png)



------

执行顺序：import>init>main

