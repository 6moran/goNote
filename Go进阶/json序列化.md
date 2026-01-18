### `json`数据格式说明：

​	任何数据类型都可以通过`json`来表示

​	`json`键值对是用来保存数据的一种方式

```json
[{"name":"aaa","age":20,"address":["北京","上海"]}
,{"name":"mary","age":15,"address":["深圳","成都"]}]
```

**`json`序列化操作：**

- **结构体序列化：**

```go
// 结构体序列化
func testStruct() {
    mon := Monster{
       Name:  "牛魔王",
       Age:   500,
       Birth: "2011-02-10",
       Sal:   2332,
       Skill: "牛魔拳",
    }
    //序列化函数，参数是一个空接口
    data, err := json.Marshal(&mon)
    if err != nil {
       fmt.Println("序列化失败", err)
    }
    fmt.Println(string(data))
}
```

- **Map序列化：**

```go
// Map序列化
func testMap() {
    a := make(map[string]interface{})
    a["name"] = "狐狸"
    a["age"] = 23
    a["address"] = "洪崖洞"

    data, err := json.Marshal(a)
    if err != nil {
       fmt.Println("序列化失败", err)
    }
    fmt.Println(string(data))
}
```

- **切片序列化：**

```go
// 切片序列化
func testSlice() {
    slice := make([]map[string]interface{}, 0)
    m1 := make(map[string]interface{})
    m1["name"] = "jack"
    m1["age"] = 10
    m1["address"] = []string{"成都", "北京"}
    slice = append(slice, m1)

    m2 := make(map[string]interface{})
    m2["name"] = "tom"
    m2["age"] = 12
    m2["address"] = []string{"上海", "北京"}
    slice = append(slice, m2)

    data, err := json.Marshal(slice)
    if err != nil {
       fmt.Println("序列化失败", err)
    }
    fmt.Println(string(data))
}
```

- **基本数据类型序列化：**

```go
// 对基本数据类型序列化（意义不大）
func testInt() {
    num := 3244
    data, err := json.Marshal(num)
    if err != nil {
       fmt.Println("序列化失败", err)
    }
    fmt.Println(string(data))
}
```

**结果：**![image-20250921105830859](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20250921105830859.png)

注意事项：

对于结构体的序列化，如果我们希望序列化后的key的名字，由我们自己指定，那么可以给struct指定一个tag标签即可（如果直接修改结构体的字段名字，如果是小写那么json包不能访问，无法序列化成功）

```go
type Monster struct {
    Name  string `json:"monster_name"` //反射机制
    Age   int    `json:"monster_age"`
    Birth string `json:"moster_birth"`
    Sal   float64
    Skill string
}
```

序列化后结果：

![image-20250921111957097](C:\Users\16053\AppData\Roaming\Typora\typora-user-images\image-20250921111957097.png)

**`json`反序列化操作：**

反序列化要保证json和go中类型一致

- **结构体反序列化：**

  ```go
  type Monster struct {
      Name  string `json:"monster_name"` //反射机制
      Age   int    `json:"monster_age"`
      Birth string `json:"moster_birth"`
      Sal   float64
      Skill string
  }
  
  // 将json字符串反序列化为struct
  func unmarshalStruct() {
      str := "{\"monster_name\":\"牛魔王\",\"monster_age\":500,\"moster_birth\":\"2011-02-10\",\"Sal\":2332,\"Skill\":\"牛魔拳\"}"
  
      mon := Monster{}
      //第一个参数是需要反序列化的json字符串的byte切片，第二个参数是结构体mon的地址
      err := json.Unmarshal([]byte(str), &mon)
      if err != nil {
         fmt.Println("反序列化失败", err)
      }
      fmt.Println(mon)
  }
  ```

- **Map反序列化：**

  ```go
  // 将json字符串反序列化为map
  func unmarshalMap() {
      str := "{\"address\":\"洪崖洞\",\"age\":23,\"name\":\"狐狸\"}"
      a := make(map[string]interface{})
      //反序列化时map也可以不make，Unmarshal会自动make
      err := json.Unmarshal([]byte(str), &a)
      if err != nil {
         fmt.Println("反序列化失败", err)
      }
      fmt.Println(a)
  }
  ```

- **切片反序列化：**

  ```go
  // 将json字符串反序列化为切片
  func unmarshalslice() {
      str := "[{\"address\":[\"成都\",\"北京\"],\"age\":10,\"name\":\"jack\"},{\"address\":[\"上海\"," +
         "\"北京\"],\"age\":12,\"name\":\"tom\"}]"
      a := make([]map[string]interface{}, 0)
      err := json.Unmarshal([]byte(str), &a)
      if err != nil {
         fmt.Println("反序列化失败", err)
      }
      fmt.Println(a)
  
  }
  ```

