# 一、Gin日志

## 日志输入至文件

```go
func main() {
    ////创建日志文件
    //f, _ := os.Create("gin.log")
    //这样写每次追加写入，并且文件不存在时会创建文件
    //这样要控制打印的日志的量
    f, _ := os.OpenFile("gin.log", os.O_CREATE|os.O_APPEND|os.O_WRONLY, 0666)
    ////将日志输入至文件
    //gin.DefaultWriter = io.Writer(f)
    //这里可以在多个地方打印日志，批量输入
    gin.DefaultWriter = io.MultiWriter(f, os.Stdout)
    router := gin.Default()
    router.GET("/", func(c *gin.Context) {
       c.JSON(200, gin.H{"msg": "啊哈哈"})
    })
    router.Run()
}
```







## 日志信息和路由格式

```go
//这里会显示你所有的路由，这里是3个，如果是匿名函数的话，是func
//这里表示main包下的main文件下的func1的路由
[GIN-debug] GET    /                         --> main.main.func1 (3 handlers)
```

可以自定义格式

```go
//自定义打印格式
gin.DebugPrintRouteFunc = func(httpMethod, absolutePath, handlerName string, nuHandlers int) {
    log.Printf("[ feng ] %s %s %s %d \n",
               httpMethod,
               absolutePath,
               handlerName,
               nuHandlers,
              )
}

//这里绑定了两个路由后的结果
2026/03/26 16:02:56 [ feng ] GET / main.main.func2 3 
2026/03/26 16:02:56 [ feng ] GET /index main.main.func3 3

//同时请求的log的显示也可以自定义
//gin中默认的
---------------------------------------------------------------------------------
// defaultLogFormatter is the default log format function Logger middleware uses.
var defaultLogFormatter = func(param LogFormatterParams) string {
	var statusColor, methodColor, resetColor, latencyColor string
	if param.IsOutputColor() {
		statusColor = param.StatusCodeColor()
		methodColor = param.MethodColor()
		resetColor = param.ResetColor()
		latencyColor = param.LatencyColor()
	}

	switch {
	case param.Latency > time.Minute:
		param.Latency = param.Latency.Truncate(time.Second * 10)
	case param.Latency > time.Second:
		param.Latency = param.Latency.Truncate(time.Millisecond * 10)
	case param.Latency > time.Millisecond:
		param.Latency = param.Latency.Truncate(time.Microsecond * 10)
	}

	return fmt.Sprintf("[GIN] %v |%s %3d %s|%s %8v %s| %15s |%s %-7s %s %#v\n%s",
		param.TimeStamp.Format("2006/01/02 - 15:04:05"),
		statusColor, param.StatusCode, resetColor,
		latencyColor, param.Latency, resetColor,
		param.ClientIP,
		methodColor, param.Method, resetColor,
		param.Path,
		param.ErrorMessage,
	)
}
---------------------------------------------------------------------------------

//我们只需要仿照这个写一个即可
```

![image-20260326165630201](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20260326165630201.png)





## gin设置

```go
//查看全部路由
fmt.Println(router.Routes())

//结果
[{GET / main.main.func2 0x7ff783d5e720} {GET /index main.main.func3 0x7ff783d5e800}]
//这里我注册了两个路由，所以会显示
```





如果不想看到这些debug日志，我们可以切换环境

![image-20260326164903507](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20260326164903507.png)

```go
//可以设置模式，默认状态下是debug模式，会显示很多日志，设置了这个就只在请求时显示日志
gin.SetMode(gin.ReleaseMode)
```



# 二、Logrus第三方日志库

 ```go
 //下载logrus
 go get github.com/sirupsen/logrus
 ```



Logrus的日志是有输出等级的，默认是info，Logrus只会输出 >= 当前等级的日志

```go
//日志等级
Panic > Fatal > Error > Warn > Info > Debug > Trace

//这里默认是Info，那么就看不到Debug和Trace的日志

//修改日志等级
logrus.SetLevel(logrus.DebugLevel)

//得到当前日志等级
logrus.GetLevel()
```



### 设置特定字段

```go
//可以批量设置，也可以单个设置，
log := logrus.WithField("app", "goland").WithField("sex", "男")

//注意这里是log.WithFields而不是logrus.WithFields，前者是追加，后者是覆盖
log = log.WithFields(logrus.Fields{
    "name": "付涛",
    "age":  23,
})
log.Errorf("你好")


//结果
time="2026-03-27T15:07:33+08:00" level=error msg="你好" age=23 app=goland name="付涛" sex="男"
```



### 设置日志输出样式

```go
//设置输出样式为json
logrus.SetFormatter(&logrus.JSONFormatter{})
//设置输出样式为text，默认是text
logrus.SetFormatter(&logrus.TextFormatter{})

//注意，设置输出样式时，有属性设置
logrus.SetFormatter(&logrus.TextFormatter{
    ForceColors:     true,		//是否启用颜色控制
    FullTimestamp:   true,		//是否显示完整时间戳
    TimestampFormat: "2006-01-02 15:04:05",		//时间格式
})

//json样式
{"age":23,"app":"goland","level":"error","msg":"你好","name":"付涛","sex":"男","time":"2026-03-27T15:12:41+08:00"}

//text样式
time="2026-03-27T15:11:56+08:00" level=error msg="你好" age=23 app=goland name="付涛" sex="男"
```



