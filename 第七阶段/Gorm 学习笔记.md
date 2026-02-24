# Gorm 学习笔记

> —康思晟

## 一、ORM 和 Gorm

### 1. ORM 概念和作用

**ORM**（Object Relational Mapping，对象关系映射）是一种程序设计技术，用于在关系型数据库和面向对象编程语言之间建立映射关系。

**核心思想**：

- 将数据库中的表映射为编程语言中的类（结构体）
- 将表中的记录映射为类的实例（对象）
- 将表中的字段映射为对象的属性

**ORM 的主要作用**：

| 作用           | 说明                                                         |
| :------------- | :----------------------------------------------------------- |
| 简化数据库操作 | 开发者可以使用面向对象的方式操作数据库，无需编写复杂的 SQL 语句 |
| 提高开发效率   | 减少重复的 SQL 编写工作，加快开发速度                        |
| 代码可维护性   | 数据库操作集中在模型层，便于维护和修改                       |
| 数据库抽象     | 同一套代码可以适配不同的数据库系统                           |
| 防止 SQL 注入  | ORM 框架通常会做参数转义，提高安全性                         |

### 2. Gorm 概念和优势

**Gorm** 是 Go 语言中最流行的 ORM 库，它是一个功能强大、开发者友好的 ORM 框架。

**Gorm 的主要优势**：

| 优势       | 说明                                            |
| :--------- | :---------------------------------------------- |
| 全功能 ORM | 支持关联、事务、迁移、批量操作等完整功能        |
| 易于使用   | API 设计简洁直观，学习成本低                    |
| 性能优秀   | 通过预处理语句、连接池等机制保证性能            |
| 自动迁移   | 支持自动创建、更新数据库表结构                  |
| 钩子方法   | 支持 Before/After Create、Update、Delete 等钩子 |
| 链式调用   | 支持链式操作，代码更加流畅                      |
| 日志支持   | 内置日志模块，方便调试                          |
| 软删除     | 内置软删除支持，防止数据永久丢失                |

## 二、Gorm 基本使用

### 1. 安装


```
# 打开终端，在你的项目目录下执行
go get -u gorm.io/gorm
go get -u gorm.io/driver/mysql
```



### 2. 连接数据库


```
package main

import (
    "gorm.io/driver/mysql"
    "gorm.io/gorm"
    "log"
)

func main() {
    // 连接数据库
    // 格式: "用户名:密码@tcp(IP:端口)/数据库名?参数"
    dsn := "root:123456@tcp(127.0.0.1:3306)/testdb?charset=utf8mb4&parseTime=True&loc=Local"
    
    db, err := gorm.Open(mysql.Open(dsn), &gorm.Config{})
    if err != nil {
        log.Fatal("连接失败:", err)
    }
    
    // 连接成功
    fmt.Println("数据库连接成功!")
}
```



------

## 三、Gorm 基础操作

### 1. 定义模型


```
// 定义一个用户表
type User struct {
    ID     uint   // 主键
    Name   string // 姓名
    Age    int    // 年龄
    Email  string // 邮箱
}

// 自动创建表（如果表不存在）
db.AutoMigrate(&User{})
```



### 2. 增删改查

#### 增加数据（Create）


```
// 创建一条数据
user := User{
    Name:  "张三",
    Age:   20,
    Email: "zhangsan@qq.com",
}

result := db.Create(&user) // 插入数据

fmt.Println(user.ID)       // 打印自动生成的ID
fmt.Println(result.Error)  // 打印错误（没有就是nil）
```



#### 查询数据（Retrieve）


```
var user User
var users []User

// 查询单条
db.First(&user, 1)                 // 查询ID=1的数据
db.First(&user, "name = ?", "张三") // 查询名字叫张三的

// 查询多条
db.Find(&users)                     // 查询所有
db.Find(&users, "age > ?", 18)      // 查询年龄大于18的
```



#### 更新数据（Update）


```
// 先查询再更新
db.First(&user, 1)
user.Age = 25
db.Save(&user) // 保存修改

// 直接更新
db.Model(&User{}).Where("id = ?", 1).Update("age", 30)
```



#### 删除数据（Delete）


```
// 删除ID=1的数据
db.Delete(&User{}, 1)

// 条件删除
db.Where("age < ?", 18).Delete(&User{})
```



### 3. 常用查询


```
var users []User

// 条件查询
db.Where("age > ?", 18).Find(&users)           // 年龄大于18
db.Where("name LIKE ?", "%张%").Find(&users)    // 名字包含"张"

// 排序
db.Order("age desc").Find(&users)               // 年龄降序

// 分页
db.Limit(10).Offset(0).Find(&users)             // 每页10条，第1页

// 计数
var count int64
db.Model(&User{}).Where("age > ?", 18).Count(&count) // 统计年龄大于18的人数

```
