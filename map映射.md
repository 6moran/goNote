### map的使用：

map[key:value key:value....]

*map是无序的*

- **map的定义：**

  ```go
  //定义map（映射），只声明map是没有分配内存空间的，必须通过make函数进行初始化，才会分配空间
  var a map[int]string
  //map，将键值对（一对匹配的信息）相关联，通过key可以得到value
  a = make(map[int]string, 10)
  a[20231514] = "张三"
  a[20231516] = "李四"
  a[20231517] = "王五"
  //例这里面key是20231514，value是张三
  fmt.Println(a)
  
  //方法二：
  b := make(map[int]string)
  b[20201414] = "任期"
  fmt.Println(b)
  
  //方法三：
  c := map[int]string{
      20141413: "张四",
      20144549: "赵八",
  }
  fmt.Println(c)
  
  //*****注意*****
  //key的部分不能重复，否则会替换，value可以重复，一样会保存
  a[20231517] = "赵六"
  a[20231519] = "李四"
  fmt.Println(a) //王五会被替换为赵六，value李四会存在两个
  ```

- **map操作：**

  1. 增加：`b[20201414] = "任期"`

  2. 删除：`delete(c, 20141413)`

  3. 查找：

     ```go
     value, flag := c[20144549]
     fmt.Println("value =", value)
     fmt.Println("flag =", flag)
     ```

     找到时：![image-20250908203522913](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20250908203522913.png)

     不存在时：![image-20250908203551449](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20250908203551449.png)

  4. 修改：`b[20201414] = "王五"`

  5. 获取长度：len(a)

  6. 遍历for range：

     1. 普通map的遍历：

        ```go
        //for range遍历
        for key, val := range a {
            fmt.Printf("key = %d   ", key)
            fmt.Println("value =", val)
        }
        ```
  
     2. 二维映射的定义和遍历：
  
        ```go
        //二维映射
        d := make(map[string]map[int]string)
        //赋值
        d["班级1"] = make(map[int]string, 3)
        d["班级1"][2323] = "任刚"
        d["班级2"] = make(map[int]string, 2)
        d["班级2"][2353] = "负刚"
        fmt.Println(d)
        
        for k, v := range d {
            //fmt.Printf("key = %s  value = %v\n", k, v)
            fmt.Printf("%s:\n", k)
            for k2, v2 := range v {
               fmt.Printf("key = %d  value = %v\n", k2, v2)
            }
        }
        ```
  
     