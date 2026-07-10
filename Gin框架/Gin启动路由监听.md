```go
func main() {
    //创建一个默认的路由
    router := gin.Default()
    router.GET("/index", func(context *gin.Context) {
       context.String(200, "Hello World")
    })

    //启动监听
    router.Run(":8080")
    //这里的Run底层封装了一个server.ListenAndServe()
}
```