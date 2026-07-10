## 一、常用验证器

```go
//必填字段
required	//不能为空，并且不能没有这个字段，例：binding:"required"，注意这里如果全是空格的话是能通过的

//针对字符串的长度
min			//最小长度，例：binding:"min=5"
max 		//最大长度，例：binding:"max=10"
len			//长度，例：binding:"len=6"

//针对数字大小
eq			//等于，例：binding:"eq=3"
ne			//不等于
gt			//大于
gte			//大于等于
lt			//小于
lte			//小于等于

//针对同级字段
eqfield		//等于其他字段的值，例：Password string `binding:"eqfield=ConfirmPassword"`
nefield		//不等于其他字段的值

//忽略字段
-			//忽略字段，例：binding:"-"
```

 



## 二、gin内置验证器

```go
//枚举
oneof 		//例：binding:"oneof=red green"，表示该字段只能是red或green

//字符串
contains		//包含，例：binding:"contains=fengfeng"，即该字段必须包含fengfeng
excludes		//不包含
startswith		//字符串前缀，即必须以xxx开头
endswith		//字符串后缀，即必须以xxx结尾

//数组
dive			//dive后面的tag标签就是就是针对数组中的每一个元素的验证

//网络验证
ip
ipv4
ipv6
uri			//统一资源标示符
url			//统一资源定位符

//日期验证
datetime	//例：binding:"datetime=2006-01-02 15:04:05"，注意，这里面的时间是固定的，2006年1月2日下午的3点4分5秒
```





## 三、自定义错误的验证信息

```go
type SignUserInfo struct {
	Name       string `json:"name" binding:"required,min=4"`
	Password   string `json:"password"`
	RePassword string `json:"re_password"`
}

func GetValidMsg(err error) string {
	//将err接口断言为具体类型
	if errs, ok := err.(validator.ValidationErrors); ok {
		//循环每一个错误信息
		//根据报错字段名，获取结构体的具体字段
		for _, e := range errs {
            //此处可以提出为函数
            //如果需要用到e.Field字段名，通常用json字段名
			switch e.Tag() {
			case "required":
				return "该字段不能为空"
			case "min":
				return fmt.Sprintf("长度不能小于%s", e.Param())
			}
		}
	}
	return ""
}

func main() {
	router := gin.Default()
	router.POST("/register", func(c *gin.Context) {
		var user SignUserInfo
		err := c.ShouldBindJSON(&user)
		if err != nil {
			fmt.Println(err)
			c.JSON(400, gin.H{"msg": GetValidMsg(err)})
			return
		}
		c.JSON(200, user)

	})

	router.Run()
}
```



## 四、自定义验证器

```go
// 校验逻辑函数
func signValid(fl validator.FieldLevel) bool {
	//校验逻辑
	var nameList []string = []string{"峰峰", "张三", "付涛"}
	for _, nameStr := range nameList {
		name := fl.Field().Interface().(string)
		if name == nameStr {
			return false
		}
	}
	return true
}


//注册为校验器
if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
    v.RegisterValidation("sign", signValid) //注册为校验tag
}
```

**测试用例：**

```go
type SignUserInfo struct {
    Name       string `json:"name" binding:"required,min=1,sign"`	//sign在这
    Password   string `json:"password"`
    RePassword string `json:"re_password"`
}

func GetValidMsg(err error) string {
    //将err接口断言为具体类型
    if errs, ok := err.(validator.ValidationErrors); ok {
       //循环每一个错误信息
       //根据报错字段名，获取结构体的具体字段
       for _, e := range errs {
          switch e.Tag() {
          case "required":
             return "该字段不能为空"
          case "min":
             return fmt.Sprintf("长度不能小于%s", e.Param())
          case "sign":
             return "包含敏感词"		//自定义错误
          }
       }

    }
    return ""
}

// 校验逻辑函数
func signValid(fl validator.FieldLevel) bool {
    //校验逻辑
    var nameList []string = []string{"峰峰", "张三", "付涛"}
    for _, nameStr := range nameList {
       name := fl.Field().Interface().(string)
       if name == nameStr {
          return false
       }
    }
    return true
}

func main() {
    router := gin.Default()

    if v, ok := binding.Validator.Engine().(*validator.Validate); ok {
       v.RegisterValidation("sign", signValid) //注册为校验tag
    }

    router.POST("/register", func(c *gin.Context) {
       var user SignUserInfo
       err := c.ShouldBindJSON(&user)
       if err != nil {
          fmt.Println(err)
          c.JSON(400, gin.H{"msg": GetValidMsg(err)})		//调用
          return
       }
       c.JSON(200, user)

    })

    router.Run()
}
```

**结果：**

<img src="C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20260324204605319.png" alt="image-20260324204605319" style="zoom: 80%;" />

