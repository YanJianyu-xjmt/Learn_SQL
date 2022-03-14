# SQL 和

## 联结

select 。。 left join   on

## 排序

order by 反序 desc

## 去重~~~~

Distinct 用法 

```
Select DISTINCT 
```

## Having 用法

having 子句是作用筛选满足条件的组 在分组之后 过滤数据， 条件包含聚组函数，但是WHERE 在条件分组之前应用，having在分组条件之前用

## 子查询

- 就是()括起来，然后相当于于子表比如
  
  ```'
  select email form (select count(1) as t,email)r where r.t > 1
  ```
  
  然后在填结果子查询的复杂度可以认识o2 的，就是找到一行执行一次子查询

## 多行匹配

180 leetcode

```
SELECT DISTINCT a.Num AS ConsecutiveNums
FROM Logs a, Logs b,Logs c
WHERE a.Num = b.Num and b.Num=c.Num and a.id=b.id-1 and b.id = c.id -1;
```

多行可以直接起别名

## 判断是不是空

is NULL

## 子查询的理解

```
SELECT DISTINCT (SELECT name FROM Department WHERE p.departmentId = id) AS `Department`,
p.name AS `Employee`,p.salary AS `Salary` 
FROM Employee p,
(SELECT DISTINCT e.departmentId did,(SELECT DISTINCT salary FROM Employee WHERE departmentId = e.departmentId ORDER BY salary DESC LIMIT 2,1) s FROM Employee e) l
WHERE (l.s is NULL AND l.did = p.departmentId) OR (p.salary >= l.s AND l.did = p.departmentId)
ORDER BY p.salary DESC;
```

LEETCODE  185题目

可以理解为一个临时表，比如理解这里的子查询就找到第三小的值，然后判断

## 删除DELETE

```
DELETE p
FROM Person p,
(select MIN(id)mids,email FROM Person GROUP BY email)l
WHERE l.email = p.email AND l.mids < p.id;
```

LEETCODE 196 这里问题就是 delete 加上目标p ，

## 多表查询

随便搞，只要起名子九ok

## 时间查询

```
SELECT DISTINCT p0.id

FROM Weather p0,Weather p1

WHERE datediff(p0.recordDate,p1.recordDate)=1

AND p1.Temperature < p0.Temperature;
```

LEETCODE 197

## 联合主键

就是多个联合起来的主键，加起来不一样就行，比如 诺基亚 N90 苹果N90 苹果 A10这样

## Gorm

### Create Record

```
db.Select("Name", "Age", "CreatedAt").Create(&user)
// 这里可以绑定 结构体，非常牛
```

```
db.Omit("Name", "Age", "CreatedAt").Create(&user)
```

这里Omit 的意思是 忽略没有指定的

### batch Insert

- 更加有效率的插入

```
var users = []User{{name: "jinzhu_1"}, ...., {Name: "jinzhu_10000"}}

// 数量为 100
db.CreateInBatches(users, 100)
```

也可以在定义的时候使用

```
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  CreateBatchSize: 1000,
})

db := db.Session(&gorm.Session{CreateBatchSize: 1000})
```

## 创建关联，就是两个结构体

```
type CreditCard struct {
  gorm.Model
  Number   string
  UserID   uint
}

type User struct {
  gorm.Model
  Name       string
  CreditCard CreditCard
}

db.Create(&User{
  Name: "jinzhu",
  CreditCard: CreditCard{Number: "411111111111"}
})
```

组合感觉 要先grom.Model 这样

也能跳过一些组合的

```
db.Omit("CreditCard").Create(&user)

// skip all associations
db.Omit(clause.Associations).Create(&user)
```

## Default Values

在定义结构体的时候，可以设置default 参数

```
type User struct {
  ID   int64
  Name string `gorm:"default:galeone"`
  Age  int64  `gorm:"default:18"`
}
```

### Upsert 和冲突

然后还有高级查询的 FirstOrInit FirstOrCreate

## 查询

### 检索单个对象

- GORM 有First Take Last 等方法，方便检索，相当于添加了 LIMIT 1条件，如果没有找到，返回ErrRecordNotFound 错误
  
  ```
  // 获取第一条记录（主键升序）
  db.First(&user)
  // SELECT * FROM users ORDER BY id LIMIT 1;
  
  // 获取一条记录，没有指定排序字段
  db.Take(&user)
  // SELECT * FROM users LIMIT 1;
  
  // 获取最后一条记录（主键降序）
  db.Last(&user)
  // SELECT * FROM users ORDER BY id DESC LIMIT 1;
  
  result := db.First(&user)
  result.RowsAffected // 返回找到的记录数
  result.Error        // returns error or nil
  
  // 检查 ErrRecordNotFound 错误
  errors.Is(result.Error, gorm.ErrRecordNotFound)
  ```

## 用主键检索

如果主键是数字类型，可以使用内联条件 来检索对象，

## 检索全部对象

全局索引

```
// 获取全部记录
result := db.Find(&users)
// SELECT * FROM users;

result.RowsAffected // 返回找到的记录数，相当于 `len(users)`
result.Error        // returns error
```

### WHERE 字段

```
db.Where(&User{Name: "jinzhu"}, "name", "Age").Find(&users)
// SELECT * FROM users WHERE name = "jinzhu" AND age = 0;

db.Where(&User{Name: "jinzhu"}, "Age").Find(&users)
```

### 内联条件

```
// 根据主键获取记录，如果是非整型主键
db.First(&user, "id = ?", "string_primary_key")
// SELECT * FROM users WHERE id = 'string_primary_key';

// Plain SQL
db.Find(&user, "name = ?", "jinzhu")
// SELECT * FROM users WHERE name = "jin
zhu";
// 内联条件
db.Find(&users, "name <> ? AND age > ?", "jinzhu", 20)
// SELECT * FROM users WHERE name <> "jinzhu" AND age > 20;

// Struct
db.Find(&users, User{Age: 20})
// SELECT * FROM users WHERE age = 20;

// Map
db.Find(&users, map[string]interface{}{"age": 20})
```

### 选择特定字段

这就检索字段，默认情况下，GORM会检索

```
db.Select("name", "age").Find(&users)
// SELECT name, age FROM users;

db.Select([]string{"name", "age"}).Find(&users)
// SELECT name, age FROM users;

db.Table("users").Select("COALESCE(age,?)", 42).Rows()
// SELECT COALESCE(age,'42') FROM users;
```

### 排序ORDER

```
db.Order("age desc, name").Find(&users)
// SELECT * FROM users ORDER BY age desc, name;

// 多个 order
db.Order("age desc").Order("name").Find(&users)
```

### Group BY & Having

Model 函数，

```
type result struct {
  Date  time.Time
  Total int
}

db.Model(&User{}).Select("name, sum(age) as total").Where("name LIKE ?", "group%").Group("name").First(&result)
// SELECT name, sum(age) as total FROM `users` WHERE name LIKE "group%" GROUP BY `name` LIMIT 1


db.Model(&User{}).Select("name, sum(age) as total").Group("name").Having("name = ?", "group").Find(&result)
// SELECT name, sum(age) as total FROM `users` GROUP BY `name` HAVING name = "group"

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Rows()
for rows.Next() {
  ...
}

rows, err := db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Rows()
for rows.Next() {
  ...
}

type Result struct {
  Date  time.Time
  Total int64
}
db.Table("orders").Select("date(created_at) as date, sum(amount) as total").Group("date(created_at)").Having("sum(amount) > ?", 100).Scan(&results)
```

### 去重Distinct

```
db.Distinct("name", "age").Order("name, age desc").Find(&results)
```

### Joins 预加载

```
type User struct {
    Id  int
    Age int
}

type Order struct {
    UserId     int
    FinishedAt *time.Time
}

query := db.Table("order").Select("MAX(order.finished_at) as latest").Joins("left join user user on order.user_id = user.id").Where("user.age > ?", 18).Group("order.user_id")
db.Model(&Order{}).Joins("join (?) q on order.finished_at = q.latest", query).Scan(&results)
```

### Scan

```
type Result struct {
  Name string
  Age  int
}

var result Result
db.Table("users").Select("name", "age").Where("name = ?", "Antonio").Scan(&result)

// Raw SQL
db.Raw("SELECT name, age FROM users WHERE name = ?", "Antonio").Scan(&result)
```

扫描

## SELECT  查询

Locking(For UPDATE )

```
db.Clauses(clause.Locking{Strength: "UPDATE"}).Find(&users)
// SELECT * FROM `users` FOR UPDATE

db.Clauses(clause.Locking{
  Strength: "SHARE",
  Table: clause.Table{Name: clause.CurrentTable},
}).Find(&users)
// SELECT * FROM `users` FOR SHARE OF `users`

db.Clauses(clause.Locking{
  Strength: "UPDATE",
  Options: "NOWAIT",
}).Find(&users)
// SELECT * FROM `users` FOR UPDATE NOWAIT
```

这里看出来

### 子查询可以嵌套

```
db.Where("amount > (?)", db.Table("orders").Select("AVG(amount)")).Find(&orders)
// SELECT * FROM "orders" WHERE amount > (SELECT AVG(amount) FROM "orders");

subQuery := db.Select("AVG(age)").Where("name LIKE ?", "name%").Table("users")
db.Select("AVG(age) as avgage").Group("name").Having("AVG(age) > (?)", subQuery).Find(&results)
// SELECT AVG(age) as avgage FROM `users` GROUP BY `name` HAVING AVG(age) > (SELECT AVG(age) FROM `users` WHERE name LIKE "name%")
```

这里很合适

### FirstOrCreate

```
// 未找到 user，则根据给定条件创建一条新纪录
db.FirstOrCreate(&user, User{Name: "non_existing"})
// INSERT INTO "users" (name) VALUES ("non_existing");
// user -> User{ID: 112, Name: "non_existing"}

// 找到了 `name` = `jinzhu` 的 user

// Attr 是如果没有用根据这个创建
db.Where(User{Name: "jinzhu"}).FirstOrCreate(&user)
// 未找到 user，根据条件和 Assign 属性创建记录
db.Where(User{Name: "non_existing"}).Attrs(User{Age: 20}).FirstOrCreate(&user)
// SELECT * FROM users WHERE name = 'non_existing' ORDER BY id LIMIT 1;
// INSERT INTO "users" (name, age) VALUES ("non_existing", 20);
// user -> User{ID: 112, Name: "non_existing", Age: 20}

// 找到了 `name` = `jinzhu` 的 user，则忽略 Attrs
db.Where(User{Name: "jinzhu"}).Attrs(User{Age: 20}).FirstOrCreate(&user)
```

获取匹配，如果没有创建一条新的记录

## 优化器，索引提示

```
import "gorm.io/hints"

db.Clauses(hints.New("MAX_EXECUTION_TIME(10000)")).Find(&User{})
// SELECT * /*+ MAX_EXECUTION_TIME(10000) */ FROM `users

db.Clauses(hints.UseIndex("idx_user_name")).Find(&User{})
// SELECT * FROM `users` USE INDEX (`idx_user_name`)

db.Clauses(hints.ForceIndex("idx_user_name", "idx_user_id").ForJoin()).Find(&User{})
// SELECT * FROM `users` FORCE INDEX FOR JOIN (`idx_user_name`,`idx_user_id`)"
`
```

可以查执行计划

### 迭代

```
rows, err := db.Model(&User{}).Where("name = ?", "jinzhu").Rows()
defer rows.Close()

for rows.Next() {
  var user User
  // ScanRows 方法用于将一行记录扫描至结构体
  db.ScanRows(rows, &user)

  // 业务逻辑...
}
```

### Pluck 这里是用于查询单个列的

```
var ages []int64
db.Model(&users).Pluck("age", &ages)

var names []string
db.Model(&User{}).Pluck("name", &names)

db.Table("deleted_users").Pluck("name", &names)

// Distinct Pluck
db.Model(&User{}).Distinct().Pluck("Name", &names)
// SELECT DISTINCT `name` FROM `users`

// 超过一列的查询，应该使用 `Scan` 或者 `Find`，例如：
db.Select("name", "age").Scan(&users)
db.Select("name", "age").Find(&users)
```

如果要查多列要 用 select 和scan

## 更新

### 保存所有字段

```
db.First(&user)

user.Name = "jinzhu 2"
user.Age = 100
db.Save(&user)
```

## 更新单个列

```
// 条件更新
db.Model(&User{}).Where("active = ?", true).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE active=true;

// User 的 ID 是 `111`
db.Model(&user).Update("name", "hello")
// UPDATE users SET name='hello', updated_at='2013-11-17 21:34:10' WHERE id=111;

// 根据条件和 model 的值进行更新
db.Model(&user).Where("active = ?", true).Update("name", "hello")
```

### 更新多列

```
db.Model(&user).Updates(User{Name: "hello", Age: 18, Active: false})
// UPDATE users SET name='hello', age=18, updated_at = '2013-11-17 21:34:10' WHERE id = 111;

// 根据 `map` 更新属性
db.Model(&user).Updates(map[string]interface{}{"name": "hello", "age": 18, "active": false})
```

### 检查字段是否有更新

gorm 提供了changed方法， 可以被用在before update hook 里面，返回是否有changed方法是  只能用在update updates 方法一起使用，（用事务 begin就没屌用）

```
func (u *User) BeforeUpdate(tx *gorm.DB) (err error) {
  // 如果 Role 字段有变更
    if tx.Statement.Changed("Role") {
    return errors.New("role not allowed to change")
    }

  if tx.Statement.Changed("Name", "Admin") { // 如果 Name 或 Role 字段有变更
    tx.Statement.SetColumn("Age", 18)
  }

  // 如果任意字段有变更
    if tx.Statement.Changed() {
        tx.Statement.SetColumn("RefreshedAt", time.Now())
    }
    return nil
}

db.Model(&User{ID: 1, Name: "jinzhu"}).Updates(map[string]interface{"name": "jinzhu2"})
// Changed("Name") => true
db.Model(&User{ID: 1, Name: "jinzhu"}).Updates(map[string]interface{"name": "jinzhu"})
// Changed("Name") => false, 因为 `Name` 没有变更
db.Model(&User{ID: 1, Name: "jinzhu"}).Select("Admin").Updates(map[string]interface{
  "name": "jinzhu2", "admin": false,
})
// Changed("Name") => false, 因为 `Name` 没有被 Select 选中并更新

db.Model(&User{ID: 1, Name: "jinzhu"}).Updates(User{Name: "jinzhu2"})
// Changed("Name") => true
db.Model(&User{ID: 1, Name: "jinzhu"}).Updates(User{Name: "jinzhu"})
// Changed("Name") => false, 因为 `Name` 没有变更
db.Model(&User{ID: 1, Name: "jinzhu"}).Select("Admin").Updates(User{Name: "jinzhu2"})
// Changed("Name") => false, 因为 `Name` 没有被 Select 选中并更新
```

### update 时候修改

在before要修改，如果是完整的用save 如果没有用setcolumn

```
func (user *User) BeforeSave(tx *gorm.DB) (err error) {
  if pw, err := bcrypt.GenerateFromPassword(user.Password, 0); err == nil {
    tx.Statement.SetColumn("EncryptedPassword", pw)
  }

  if tx.Statement.Changed("Code") {
    s.Age += 20
    tx.Statement.SetColumn("Age", s.Age+20)
  }
}

db.Model(&user).Update("Name", "jinzhu")
```

## 事务

### 关闭默认的事务

```
// Globally disable
db, err := gorm.Open(sqlite.Open("gorm.db"), &gorm.Config{
  SkipDefaultTransaction: true,
})

// Continuous session mode
tx := db.Session(&Session{SkipDefaultTransaction: true})
tx.First(&user, 1)
tx.Find(&users)
tx.Model(&user).Update("Age", 18)
```

如果 create/update/delete 在一个事务种，可以保证一致性，如果不需要关闭，大约提升30%性能

### 事务

```
db.Transaction(func(tx *gorm.DB) error {
  // do some database operations in the transaction (use 'tx' from this point, not 'db')
  if err := tx.Create(&Animal{Name: "Giraffe"}).Error; err != nil {
    // return any error will rollback
    return err
  }

  if err := tx.Create(&Animal{Name: "Lion"}).Error; err != nil {
    return err
  }

  // return nil will commit the whole transaction
  return nil
})
```

为了 感觉就是说一种定义方法

### 人为控制

```
// begin a transaction
tx := db.Begin()

// do some database operations in the transaction (use 'tx' from this point, not 'db')
tx.Create(...)

// ...

// rollback the transaction in case of error
tx.Rollback()

// Or commit the transaction
tx.Commit()
```

简单粗暴



### SavePoint。RollbackTo

```
tx := db.Begin()
tx.Create(&user1)

tx.SavePoint("sp1")
tx.Create(&user2)
tx.RollbackTo("sp1") // Rollback user2

tx.Commit() // Commit user1

```

保存结果
