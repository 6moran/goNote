## 一、响应JSON

```go
type User struct {
    UserName string `json:"user_name"`
    Age      int    `json:"age"`
}
```

```go
 func response(context *gin.Context) {
    //json响应结构体
    user := User{
       UserName: "符成岩",
       Age:      10,
    }
    context.JSON(200, user)

    //json响应map
    userMap := map[string]string{
      "username": "任富航",
      "age":      "20",
    }
    context.JSON(200, userMap)

    //json响应gin框架中原始的值gin.H(map[string]any)
    context.JSON(200, gin.H{
     "user_name": "付涛",
     "age":       23,
    })
}
```



## 二、响应XML和YAML

```go
func response(context *gin.Context) {
    //响应XML
    context.XML(200, gin.H{"username": "付涛", "message": "cs", "data": gin.H{"user": "futao"}})
    //响应YAML
    context.YAML(200, gin.H{"username": "付涛", "message": "cs", "data": gin.H{"user": "futao"}})
}
```

![image-20260315113155483](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20260315113155483.png)

![image-20260315113442977](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20260315113442977.png)



## 三、响应HTML

```go
func main() {
    //创建一个默认的路由
    router := gin.Default()
    //加载模版目录下的所有文件
    router.LoadHTMLGlob("template/*")
    router.GET("/response", response)
    //启动监听
    router.Run(":8080")
}

func response(context *gin.Context) {
    //这里这个nil起始就是往html文件中传的值
    context.HTML(200, "index.html", nil)
} 
```

![image-20260315114254386](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20260315114254386.png)



## 四、文件响应

```go
func main() {
    //创建一个默认的路由
    router := gin.Default()
    //绑定一个文件，第一个参数是网页的请求路径，第二个参数是本地文件路径
    router.StaticFile("/home", "static/home")
    //绑定一个文件目录，第一个参数是网页请求路径前缀，第二个路径是该前缀对应的目录
    router.StaticFS("/static", http.Dir("static/img"))
    //启动监听
    router.Run(":8080")
}
```

![image-20260315162600633](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20260315162600633.png)

## 五、重定向响应

```go
func response(context *gin.Context) {
    context.Redirect(302, "http://www.baidu.com")
} 
```



## 六、设置响应头

```go
func response(context *gin.Context) {
    context.Header("key","value")
} 
```



