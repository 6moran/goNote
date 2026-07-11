# Gorm（传统版本）


## 一、创建

```go
type User struct {
    Id       int
    Name     string
    Sex      int
    Birthday *time.Time			//指针能存null
}

func main() {
    dsn := "root:1458963@tcp(127.0.0.1:3306)/gorm_test?parseTime=True"
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
       fmt.Println(err)
       return
    }

    t := time.Date(2005, 1, 2, 0, 0, 0, 0, time.Local)
	user := User{
		Name:     "付涛",
		Sex:      1,
		Birthday: &t,
	}
	result := db.Create(&user)
    //打印错误
	fmt.Println(result.Error)

}
```



### 用指定字段创建记录

```go
//指定要添加的属性
db.Select("Name", "Age", "CreatedAt").Create(&user)
// INSERT INTO `users` (`name`,`age`,`created_at`) VALUES ("jinzhu", 18, "2020-07-04 11:05:21.775")
```

忽略指定字段创建记录

```go
db.Omit("Name", "Age", "CreatedAt").Create(&user)
// INSERT INTO `users` (`birthday`,`updated_at`) VALUES ("2020-01-01 00:00:00.000", "2020-07-04 11:05:21.775")
//这里就是忽略了Name、Age、CreateAt，只有birthday和updated_at，这俩字段在user里面
```



### 批量创建记录

```go
//普通批量创建
t := time.Now()
users := []*User{
    {Name: "Jinzhu", Birthday: &t},
    {Name: "Jackson", Birthday: &t},
}

result := db.Create(users)		//这里传切片地址
if result.Error != nil {
    fmt.Println(result.Error)
    return
}

//分批批量创建，第二个数字参数代表每批创建几条记录
result := db.CreateInBatches(users, 2)
if result.Error != nil {
    fmt.Println(result.Error)
    return
}
```





### 通过map创建记录

```go
//单条记录
db.Model(&User{}).Create(map[string]interface{}{
    "Name": "jinzhu",
})

//批量创建
db.Model(&User{}).Create([]map[string]interface{}{
    {"Name": "jinzhu_1"},
    {"Name": "jinzhu_2"},
})
```

**注意**：当使用map来创建时，钩子方法不会执行，关联不会被保存且不会回写主键。





## 二、查询

### First、Last、Take、Find

```go
//如果相关 model 没有定义主键，那么将按 model 的第一个字段进行排序。
//通过主键顺序排序，然后取第一个
user := User{}
//这里表示查users表，把结果赋给user
db.First(&user)
fmt.Println(user)
// SELECT * FROM users ORDER BY id LIMIT 1;

//通过主键逆序排序，然后取第一个
user2 := User{}
db.Last(&user2)
fmt.Println(user2)
// SELECT * FROM users ORDER BY id DESC LIMIT 1;

//直接取一个，一般是第一个
user3 := User{}
db.Debug().Take(&user3)
fmt.Println(user3)
// SELECT * FROM users LIMIT 1;

//查询全部对象
users := []User{}
result := db.Find(&users)
//result里面有受影响的行数和err
fmt.Println(result.RowsAffected)
//值赋给users了
fmt.Println(users)
// SELECT * FROM users;
```



### Model、Table指定表

```go
type User struct {
	Id       int
	Name     string
	Sex      int
	Birthday *time.Time
}

type User2 struct {
	Id   int
	Name string
}

//Model指定，通过结构体推倒表名
user3 := User{}
user1 := User2{}
//这里是User{}解析为users，查users表，将结果放到user1，use1也可以是result（map[string]interface{}）
//如果是Model，那么后面跟的First和Last里面可以传map，会正常工作
//只会查询user1拥有的字段
db.Model(&user3).First(&user1)
fmt.Println(user1)

//Table指定，直接写死表名
result := map[string]interface{}{}
//如果是Table，那么后面跟的First和Last里面不可以传map，不会正常工作
db.Table("users").Take(&result)
fmt.Println(result)
```



### 根据主键检索

```go
//当主键是数字类型时
db.First(&user, 21)
db.First(&user, "21")
//相当于SELECT * FROM users WHERE id = 21;

//注意，当传的结构体对应的主键中有值时，会自动根据该值查询，只有结构体才行
db.First(&User{Id: 22})
//相当于SELECT * FROM users WHERE id = 22;

//注意两者不能一起用
db.First(&User{Id: 22},21)
//相当于SELECT * FROM users WHERE id = 22 AND id = 21;
```

```go
//当主键是字符串类型时
//需要多一步？，避免sql注入
db.First(&user, "id = ?", "1b74413f-f3b8-409f-ac47-e8c062e3472a")
//相当于SELECT * FROM users WHERE id = "1b74413f-f3b8-409f-ac47-e8c062e3472a";
```



### 特殊字段特殊查询

```go
type User struct {
  ID           string `gorm:"primarykey;size:16"`
  Name         string `gorm:"size:24"`
  DeletedAt    gorm.DeletedAt `gorm:"index"`
}

var user = User{ID: 15}
db.First(&user)
// 相当于 SELECT * FROM `users` WHERE `users`.`id` = '15' AND `users`.`deleted_at` IS NULL ORDER BY `users`.`id` LIMIT 1
//这里有一个gorm.DeletedAt字段，它代表着软删除，gorm会自动添加为空条件，避免你查询到已经标记为删除了的数据
```



### 条件查询(Where、Or)

Where和Not，这里Not是对条件的否定查询

```go
users := []User{}
db.Where("name = ?", "付涛").Find(&users)
//SELECT * FROM `users` WHERE name = '付涛'
db.Not("name = ?", "付涛").Find(&users)
//SELECT * FROM `users` WHERE NOT name = '付涛'

db.Where("name in ?", []string{"Bob", "fcy"}).Last(&users)
//SELECT * FROM `users` WHERE name in ('Bob','fcy') ORDER BY `users`.`id` DESC LIMIT 1
db.Not("name in ?", []string{"Bob", "fcy"}).Last(&users)
//SELECT * FROM `users` WHERE NOT name in ('Bob','fcy') ORDER BY `users`.`id` DESC LIMIT 1

lastTime := time.Date(2004, 1, 1, 1, 1, 1, 1, time.Local)
db.Where("name = ? and birthday > ?", "付涛", lastTime).Find(&users)
//SELECT * FROM `users` WHERE name = '付涛' and birthday > '2004-01-01 01:01:01'


//注意，结构体中主键有值时，会把这个也解析为一个条件，切片则不会
//注意，只有主键才会!
user := User{Id: 22}
db.Where("name = ?", "付涛").First(&user)
//SELECT * FROM `users` WHERE name = '付涛' AND `users`.`id` = 22 ORDER BY `users`.`id` LIMIT 1
```

gorm中的条件是可以嵌套的，在嵌套时，And条件直接用`Where()`，Or的话需要用`Or()`

```go
users := []User{}
db.Debug().Where("name = ?", "付涛").Where("id = ?", "22").Or("name = ?", "Bob").Where("id = ?", "24").Find(&users)
//SELECT * FROM `users` WHERE name = '付涛' AND id = '22' OR name = 'Bob' AND id = '24'


//这样更容易理解
db.Where(
  db.Where("pizza = ?", "pepperoni").Where(db.Where("size = ?", "small").Or("size = ?", "medium")),
).Or(
  db.Where("pizza = ?", "hawaiian").Where("size = ?", "xlarge"),
).Find(&Pizza{})
// SQL: SELECT * FROM `pizzas` WHERE (pizza = "pepperoni" AND (size = "small" OR size = "medium")) OR (pizza = "hawaiian" AND size = "xlarge")

```

这样写的好处是有利于条件的拼接。





### 通过结构体、Map或切片条件查询

```go
//结构体
//不要这样干，会有重复的条件，因为Find也会拿主键当条件
//注意：当使用struct进行查询时，GORM只会使用非零字段进行查询，这意味着如果字段的值为0、''、false或其他零值，则不会用于构建查询条件，例如这里的Sex如果为0就不会自动构建，Map和切片是可以的
user := User{Id: 22, Sex: 1}
//要么再新建一个载体
user2 := User{}
db.Where(&user).Find(&user2)
//SELECT * FROM `users` WHERE `users`.`id` = 22 AND `users`.`sex` = 1


// Map
db.Where(map[string]interface{}{"name": "jinzhu", "age": 20}).Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 20;


// Slice of primary keys
db.Where([]int64{20, 21, 22}).Find(&users)
// SELECT * FROM users WHERE id IN (20, 21, 22);
```

**当使用结构体时，可以指定只用指定的字段**

```go
//结构体默认是全部字段构建where，这时我们可以指定几个字段用
user := User{Id: 21, Name: "Bob"}
user2 := []User{}
db.Where(&user, "sex").Find(&user2)
//SELECT * FROM `users` WHERE `users`.`sex` = 0
```



### 内联条件查询

内联条件查询将不使用where，而是将条件通过where的方式内联在First或Find方法中。

```go
//普通内联
user := User{}
db.Debug().First(&user, "name = ?", "Alice")
//SELECT * FROM `users` WHERE name = 'Alice' ORDER BY `users`.`id` LIMIT 1

//结构体内联
user2 := User{}
db.Debug().Find(&user2, User{Name: "付涛"})
//SELECT * FROM `users` WHERE `users`.`name` = '付涛'

//map内联
user3 := User{}
db.Debug().Find(&user3, map[string]interface{}{"Name": "Bob"})
//SELECT * FROM `users` WHERE `users`.`Name` = 'Bob'
```



### 选择查询的字段(Select、Distinct)

`Select()`和`Distinct()`，区别是后者会去重

```go
//只查询name和age字段
db.Select("name", "age").Find(&users)
//SELECT name, age FROM users;


db.Distinct("name", "age").Find(&users)
//SELECT distinct name, age FROM users;
```



### 排序

```go
//排序
db.Order("age desc, name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

//嵌套排序
db.Order("age desc").Order("name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;
```



### 分页查询

```go
db.Limit(3).Find(&users)
// SELECT * FROM users LIMIT 3;

users := []User{}
db.Limit(2).Offset(3).Find(&users)
//SELECT * FROM `users` LIMIT 2 OFFSET 3

//注意，offest必须和limit一起出现，单独一个offest报错


//这里后面的Limit(-1)和offest(-1)的意思为清除前面的偏移量，后面的Limit(-1)和offest(-1)可以分开使用
users1 := []User{}
users2 := []User{}
db.Limit(2).Offset(2).Find(&users1).Limit(-1).Offset(-1).Find(&users2)
//SELECT * FROM `users` LIMIT 2 OFFSET 2
//SELECT * FROM `users`
```



### Group和Having

```go
type result struct {
    Name  string
    Count int
}

//用法一
res := []result{}
db.Model(&User{}).Select("name, count(*) as count").Group("name").Having("count > ?", 1).Find(&res)
//SELECT name, count(*) as count FROM `users` GROUP BY `name` HAVING count > 1

//用法二
rows, err := db.Table("users").Select("name,count(*)").Group("name").Rows()
defer rows.Close()
for rows.Next() {
    var name string
    var sex int
    err = rows.Scan(&name, &sex)
    if err != nil {
        fmt.Println(err)
    }
    fmt.Println(name, sex)
}

//用法三
res := []result{}
db.Model(&User{}).Select("name, count(*) as count").Group("name").Having("count > ?", 1).Scan(&res)
```



### Joins连接

```go
type User struct {
    Id       int
    Name     string
    Sex      int
    Birthday *time.Time
    RoleID   int		//这两个必须写，否则检查不到关联
    Role     Role
}

type Role struct {
    Id   int
    Name string
}

db.Joins("Role").Find(&users)
//SELECT `users`.`id`,`users`.`name`,`users`.`sex`,`users`.`birthday`,`users`.`role_id`,`Role`.`id` AS `Role__id`,`Role`.`name` AS `Role__name` FROM `users` LEFT JOIN `roles` `Role` ON `users`.`role_id` = `Role`.`id`

//自然连接且多一个条件
db.Debug().InnerJoins("Role", db.Where(&User{Id: 1})).Find(&users)
//SELECT `users`.`id`,`users`.`name`,`users`.`sex`,`users`.`birthday`,`users`.`role_id`,`Role`.`id` AS `Role__id`,`Role`.`name` AS `Role__name` FROM `users` INNER JOIN `roles` `Role` ON `users`.`role_id` = `Role`.`id` AND `Role`.`id` = 1

```







## 三、更新

### 创建或更新(Save)

```go
db.Save(&user)
//对于传进来的user，如果user的主键对应的字段没有值或是零值，那么即视为创建操作
//如果有主键对应的字段，但是表中没有该主键值，那么会先修改再创建，但是修改后被影响的行为0
//如果有主键且有效，那么即为修改操作，会将表中的值修改为user对应字段的值

//因为Save可能会产生歧义，所以不用，使用Updates方法
```



### 更新(Update)

```go
//普通条件修改
db.Debug().Model(&User{}).Where("id = 3").Update("name", "付涛")
//UPDATE `users` SET `name`='付涛' WHERE id = 3

//如果要使用表达式，用gorm.Expr()
db.Debug().Model(&User{}).Where("id = 3").Update("salary", gorm.Expr("salary * ?", 2))

//User含主键自动构建条件
db.Debug().Model(&User{Id: 21}).Update("sex", 0)
//UPDATE `users` SET `sex`=0 WHERE `id` = 21
```



### 根据结构体和Map更新

```go
// 根据 `struct` 更新属性，只会更新非零值的字段
//这里Active是false，也就是零值，所以不出现在条件
db.Model(&user).Updates(User{Name: "hello", Age: 18, Active: false})
// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;

// 根据 `map` 更新属性
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET name='hello', age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;
```



### 选中和忽略字段进行更新

```go 
// 选择 Map 的字段
// User 的 ID 是 `111`:
db.Model(&user).Select("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET name='hello' WHERE id=111;

db.Model(&user).Omit("name").Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
// UPDATE users SET age=18, active=false, updated_at='2013-11-17 21:34:10' WHERE id=111;

// 选择 Struct 的字段（会选中零值的字段）
db.Model(&user).Select("Name", "Age").Updates(User{Name: "new_name", Age: 0})
// UPDATE users SET name='new_name', age=0 WHERE id=111;

// 选择所有字段（选择包括零值字段的所有字段）
db.Model(&user).Select("*").Updates(User{Name: "jinzhu", Role: "admin", Age: 0})

// 选择除 Role 外的所有字段（包括零值字段的所有字段）
db.Model(&user).Select("*").Omit("Role").Updates(User{Name: "jinzhu", Role: "admin", Age: 0})
```







## 四、删除

```go
//带主键的结构体删除
db.Debug().Delete(&User{Id: 21})
//DELETE FROM `users` WHERE `users`.`id` = 21

//带条件的删除
db.Debug().Where("name = ?", "付涛").Delete(&User{Id: 22})
//DELETE FROM `users` WHERE name = '付涛' AND `users`.`id` = 22

//根据主键内联删除
db.Debug().Delete(&User{}, 23)
//DELETE FROM `users` WHERE `users`.`id` = 23
db.Debug().Delete(&User{}, []int{1, 2, 3})
//DELETE FROM `users` WHERE `users`.`id` IN (1,2,3)



//批量删除
var users = []User{{Id: 1}, {Id: 2}, {Id: 3}}
db.Debug().Delete(&users)
//DELETE FROM `users` WHERE `users`.`id` IN (1,2,3)
//内联模糊查询删除
db.Debug().Delete(&users, "name LIKE ?", "%jinzhu%")
//DELETE FROM `users` WHERE name LIKE '%jinzhu%' AND `users`.`id` IN (1,2,3)
```



### 全局删除

```go
//gorm中直接操作全局是不被允许的
db.Delete(&User{}).Error // gorm.ErrMissingWhereClause
//gorm不支持结构体的普通字段定义条件来删除
db.Delete(&[]User{{Name: "jinzhu1"}, {Name: "jinzhu2"}}).Error // gorm.ErrMissingWhereClause

//如果非要操作全局，那么以下几个方案
//方法一
db.Where("1 = 1").Delete(&User{})
// DELETE FROM `users` WHERE 1=1

//方法二
db.Exec("DELETE FROM users")
// DELETE FROM users

//方法三
db.Session(&gorm.Session{AllowGlobalUpdate: true}).Delete(&User{})
// DELETE FROM users
```



### 返回删除行的数据

```go
// 回写所有的列
var users []User
DB.Clauses(clause.Returning{}).Where("role = ?", "admin").Delete(&users)
// DELETE FROM `users` WHERE role = "admin" RETURNING *
// users => []User{{ID: 1, Name: "jinzhu", Role: "admin", Salary: 100}, {ID: 2, Name: "jinzhu.2", Role: "admin", Salary: 1000}}

// 回写指定的列
// 这里column是一个结构体，里面的Name字段用于定义被返回的字段名，即指定要返回的字段
DB.Clauses(clause.Returning{Columns: []clause.Column{{Name: "name"}, {Name: "salary"}}}).Where("role = ?", "admin").Delete(&users)
// DELETE FROM `users` WHERE role = "admin" RETURNING `name`, `salary`
// users => []User{{ID: 0, Name: "jinzhu", Role: "", Salary: 100}, {ID: 0, Name: "jinzhu.2", Role: "", Salary: 1000}}

```



### 软删除

如果你的结构体包含gorm.DeletedAt字段，那么该结构体自动获得软删除能力。

当调用`Delete`时，GORM并不会从数据库中删除该记录，而是将该记录的`DeleteAt`设置为当前时间，而后的一般查询方法将无法查找到此条记录。

```go
db.Where("age = ?", 20).Delete(&User{})
// UPDATE users SET deleted_at="2013-10-29 10:23" WHERE age = 20;

// 查询时会忽略该行数据
db.Where("age = 20").Find(&user)
// SELECT * FROM users WHERE age = 20 AND deleted_at IS NULL;
```



**查找被软删除的记录（Unscoped）**

```go
db.Unscoped().Where("age = 20").Find(&users)
// SELECT * FROM users WHERE age = 20;
```

**永久删除**

```go
//查出来删掉
db.Unscoped().Delete(&order)
// DELETE FROM orders WHERE id=10;
```

![[Pasted image 20260521121249.png]]

除此之外，GORM也支持原生SQL的执行，你可以选择使用GORM来拼接SQL也可以直接执行原生SQL
