# Gorm

## 一、使用步骤

### 1、安装gorm

```go
go get -u gorm.io/gorm
```

### 2、安装MySQL驱动

```go
go get -u gorm.io/driver/mysql
```

### 3、连接MySQL

```go
import (
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

func main() {
  dsn := "user:pass@tcp(127.0.0.1:3306)/dbname?charset=utf8mb4&parseTime=True&loc=Local"
  db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
}
```

**注意：**想要正确的处理 `time.Time` ，您需要带上 `parseTime` 参数；想要正确显示本地时间，需要带上 `loc` 参数，否则默认是UTC（世界时间）；要支持完整的 UTF-8 编码，您需要将 `charset=utf8` 更改为 `charset=utf8mb4`

### 4、Gorm连接配置

```go
type Config struct {
  // 禁用默认事务，gorm在单个创建和更新时都会开启事务以保持数据一致性
  SkipDefaultTransaction bool
  // 自定义的命名策略
  NamingStrategy schema.Namer
  // 保存完整的关联
  FullSaveAssociations bool
  // 自定义logger
  Logger logger.Interface
  // 自定义nowfunc，用于注入CreatedAt和UpdatedAt字段
  NowFunc func() time.Time
  // 只生成sql不执行
  DryRun bool
  // 使用预编译语句
  PrepareStmt bool
  // 建立连接后，ping一下数据库
  DisableAutomaticPing bool
  // 在迁移数据库时忽略外键
  DisableForeignKeyConstraintWhenMigrating bool
  // 在迁移数据库时忽略关联引用
  IgnoreRelationshipsWhenMigrating bool
  // 禁用嵌套事务
  DisableNestedTransaction bool
  // 运行全局更新，就是不加where的update
  AllowGlobalUpdate bool
  // 对表的所有字段进行查询
  QueryFields bool
  // 批量创建的size
  CreateBatchSize int
  // 启用错误转换
  TranslateError bool

  // ClauseBuilders clause builder
  ClauseBuilders map[string]clause.ClauseBuilder
  // ConnPool db conn pool
  ConnPool ConnPool
  // Dialector database dialector
  Dialector
  // Plugins registered plugins
  Plugins map[string]Plugin

  callbacks  *callbacks
  cacheStore *sync.Map
}
```

这些配置都可以在`&gorm.Config{}`中设置







## 二、扩展操作

### 1、自定义驱动

GORM 允许通过 `DriverName` 选项自定义 MySQL 驱动，例如：

```go
import (
  _ "example.com/my_mysql_driver"
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

db, err := gorm.Open(mysql.New(mysql.Config{
  DriverName: "my_mysql_driver",
  DSN: "gorm:gorm@tcp(localhost:9910)/gorm?charset=utf8&parseTime=True&loc=Local",
}), &gorm.Config{})
```

一般情况下用不到...



### 2、现有的数据库连接

GORM 允许通过一个现有的数据库连接来初始化 `*gorm.DB`

```go
import (
  "database/sql"
  "gorm.io/driver/mysql"
  "gorm.io/gorm"
)

sqlDB, err := sql.Open("mysql", "mydb_dsn")
gormDB, err := gorm.Open(mysql.New(mysql.Config{
  Conn: sqlDB,
}), &gorm.Config{})
```

同上...