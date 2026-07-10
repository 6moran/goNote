## 一、查询参数(URL)

```go
func query(c *gin.Context) {
    //普通查询
    fmt.Println(c.Query("user"))
    //带bool的查询，会返回是否有该参数
    fmt.Println(c.GetQuery("user"))
    //如果该参数有多个值，能得到
    fmt.Println(c.GetQueryArray("user"))
    //如果参数是map
    fmt.Println(c.GetQueryMap("user"))
    //可以设置默认值，当该参数不存在时
    fmt.Println(c.DefaultQuery("user","李四"))
}

func main() {
    router := gin.Default()
    router.GET("/query", query)
    router.Run()
}
```



## 二、多路路由参数

```go
func param(c *gin.Context) {
    fmt.Println(c.Param("id"))
    fmt.Println(c.Param("book"))
}

func main() {
    router := gin.Default()
    //gin框架中只能使用:而不是{}
    router.GET("/param/:id", param)
    router.GET("/param/:id/:book", param)
    router.Run()
}
```



## 三、表单参数

```go
func formData(c *gin.Context) {
    //普通获取form表单参数
    fmt.Println(c.PostForm("name"))
    //获取多个form表单参数
    fmt.Println(c.PostFormArray("name"))
    //带默认值的获取form表单参数
    fmt.Println(c.DefaultPostForm("name", "付涛"))
}

func main() {
    router := gin.Default()
    router.POST("/form", formData)
    router.Run()
}
```



## 四、原始参数

```go
//得到原始数据
func raw(c *gin.Context) {
	body, _ := c.GetRawData()
	fmt.Println(string(body))
}

func main() {
	router := gin.Default()
	router.POST("/raw", raw)
	router.Run()
}

//form-data
----------------------------881378550787652012031532
Content-Disposition: form-data; name="name"

futao
----------------------------881378550787652012031532
Content-Disposition: form-data; name="age"

12
----------------------------881378550787652012031532--


//x-www-form-urlencoded
name=futao&age=12


//json
{
    "name":"futao",
    "age":12
}
```



## 五、请求头参数

```go
//从请求头中拿到参数
func header(c *gin.Context) {
	//这里的c.Request.Header是一个map[string][]sting
	//如果使用Header["key"]来获取请求头参数，那么key需要注意大小写
	//如果使用GetHeader或Header.Get函数来获取参数，那么不需要区分大小写
	contentType := c.Request.Header
	//contentType := c.GetHeader("Content-Type")
	//content := c.Request.Header.Get("key")
	fmt.Println(contentType)
}

func main() {
	router := gin.Default()
	router.Get("/header", header)
	router.Run()
}
```



## 六、四大请求方式

通常使用Restful风格，它是一种网络应用中资源定位和资源操作的风格，不是标准也不是协议。

GET：从服务器中取出资源。

POST：在服务器中新建一个资源。

PUT：在服务器中更新资源（客户端提供完整资源数据）。

PATCH：在服务器中更新资源（客户端提供需要修改的资源数据）。

DELETE：从服务器删除资源。



```go
// 以文字资源为例，这里的资源通常使用复数形式。
GET /articles 文章列表
GET /articles/:id 文章详情
POST /articles 添加文章
PUT /articles/:id 修改某一篇文章
DELETE /articles/:id 删除某一篇文章
```

