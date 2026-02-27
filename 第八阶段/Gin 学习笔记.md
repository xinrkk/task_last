# Gin 学习笔记

> —康思晟

## 一、Gin 初识

### 1. Gin 的基础语法

**安装 Gin**

bash

```
go get -u github.com/gin-gonic/gin
```



**最简单的 Gin 程序**

go

```
package main

import (
    "github.com/gin-gonic/gin"
    "net/http"
)

func main() {
    // 创建路由
    r := gin.Default()
    
    // 定义路由
    r.GET("/ping", func(c *gin.Context) {
        c.JSON(http.StatusOK, gin.H{
            "message": "pong",
        })
    })
    
    // 启动服务
    r.Run(":8080")  // 监听 8080 端口
}
```



### 2. 使用 Gin 启动一个 HTTP 服务

go

```
package main

import "github.com/gin-gonic/gin"

func main() {
    r := gin.Default()
    
    // GET请求
    r.GET("/hello", func(c *gin.Context) {
        c.String(200, "Hello World")
    })
    
    // POST请求
    r.POST("/user", func(c *gin.Context) {
        name := c.PostForm("name")
        c.JSON(200, gin.H{
            "name": name,
        })
    })
    
    // 启动服务
    r.Run(":8080")  // 访问 http://localhost:8080
}
```



## 二、Web 后端的基本结构

### 1. MVC 架构分层

text

```
myproject/
├── main.go              // 入口文件
├── controllers/         // 控制器（处理请求）
│   └── user.go
└── models/             // 模型（定义数据）
    └── user.go
```

MVC架构的三个核心组件分别是：**模型（Model）**、**视图（View）**和**控制器（Controller）**。

#### 1. 模型（Model）

- **职责**：负责管理应用程序的数据和业务逻辑。
- **具体功能**：
  - 直接与数据库进行交互（如数据的增删改查）。
  - 定义处理数据的规则（如验证、计算）。
  - 当数据发生变化时，它通常会通知视图（View）进行更新。

#### 2. 视图（View）

- **职责**：负责数据的展示和用户界面。
- **具体功能**：
  - 呈现模型（Model）提供的数据给用户。
  - 接收用户的输入（如鼠标点击、键盘敲击），并将其传递给控制器（Controller）。
  - 通常不包含复杂的业务逻辑，只负责显示（HTML、CSS、GUI等）。

#### 3. 控制器（Controller）

- **职责**：负责接收用户的输入并调用模型和视图来完成具体的操作。它是连接模型和视图的桥梁。
- **具体功能**：
  - 解析来自用户的请求（例如：用户点击了“保存”按钮）。
  - 决定调用哪个模型（Model）来处理数据。
  - 决定调用哪个视图（View）来渲染结果。
  - 包含应用程序的主要调度逻辑。

**代码示例：**

go

```
// main.go
package main

import (
    "myproject/controllers"
    "github.com/gin-gonic/gin"
)

func main() {
    r := gin.Default()
    
    r.GET("/users", controllers.GetUsers)
    r.GET("/users/:id", controllers.GetUser)
    
    r.Run(":8080")
}

// controllers/user.go
package controllers

import (
    "github.com/gin-gonic/gin"
    "myproject/models"
)

func GetUsers(c *gin.Context) {
    users := models.GetAllUsers()
    c.JSON(200, users)
}

func GetUser(c *gin.Context) {
    id := c.Param("id")
    user := models.GetUserById(id)
    c.JSON(200, user)
}

// models/user.go
package models

type User struct {
    ID   string `json:"id"`
    Name string `json:"name"`
}

func GetAllUsers() []User {
    return []User{
        {ID: "1", Name: "张三"},
        {ID: "2", Name: "李四"},
    }
}

func GetUserById(id string) User {
    return User{ID: id, Name: "张三"}
}
```



## 三、API 初识

### 1. API 基本概念

API 就是接口，后端提供，前端调用。

### 2. RESTful API 最简单理解

**用不同的请求方式表示操作：**

- `GET` - 获取数据
- `POST` - 创建数据
- `PUT` - 修改数据
- `DELETE` - 删除数据

**示例：**

go

```
r.GET("/users")      // 获取所有用户
r.GET("/users/1")    // 获取ID为1的用户
r.POST("/users")     // 创建新用户
r.PUT("/users/1")    // 修改ID为1的用户
r.DELETE("/users/1") // 删除ID为1的用户
```



## 四、身份校验

### 1. Cookie

go

```
// 设置cookie
r.GET("/set-cookie", func(c *gin.Context) {
    c.SetCookie("username", "张三", 3600, "/", "localhost", false, true)
    c.String(200, "cookie已设置")
})

// 获取cookie
r.GET("/get-cookie", func(c *gin.Context) {
    username, _ := c.Cookie("username")
    c.String(200, "用户名：" + username)
})
```



### 2. Session

bash

```
go get github.com/gin-contrib/sessions
```



go

```
import (
    "github.com/gin-contrib/sessions"
    "github.com/gin-contrib/sessions/cookie"
)

func main() {
    r := gin.Default()
    store := cookie.NewStore([]byte("secret"))
    r.Use(sessions.Sessions("session", store))
    
    r.GET("/login", func(c *gin.Context) {
        session := sessions.Default(c)
        session.Set("user", "张三")
        session.Save()
        c.String(200, "登录成功")
    })
    
    r.GET("/info", func(c *gin.Context) {
        session := sessions.Default(c)
        user := session.Get("user")
        c.String(200, "当前用户：" + user.(string))
    })
}
```



### 3. JWT

go

```
package main

import (
    "github.com/gin-gonic/gin"
    "github.com/golang-jwt/jwt/v5"
    "time"
)

var jwtKey = []byte("my_secret_key")

// 登录 - 生成token
r.POST("/login", func(c *gin.Context) {
    // 验证用户名密码（简化版）
    username := c.PostForm("username")
    password := c.PostForm("password")
    
    if username == "admin" && password == "123456" {
        // 生成token
        token := jwt.NewWithClaims(jwt.SigningMethodHS256, jwt.MapClaims{
            "username": username,
            "exp":      time.Now().Add(time.Hour * 24).Unix(),
        })
        
        tokenString, _ := token.SignedString(jwtKey)
        c.JSON(200, gin.H{"token": tokenString})
    } else {
        c.JSON(401, gin.H{"error": "用户名或密码错误"})
    }
})

// 需要登录的接口
r.GET("/profile", func(c *gin.Context) {
    // 获取token
    tokenString := c.GetHeader("Authorization")
    if tokenString == "" {
        c.JSON(401, gin.H{"error": "未登录"})
        return
    }
    
    // 去掉Bearer前缀
    if len(tokenString) > 7 {
        tokenString = tokenString[7:]
    }
    
    // 验证token
    token, _ := jwt.Parse(tokenString, func(token *jwt.Token) (interface{}, error) {
        return jwtKey, nil
    })
    
    if claims, ok := token.Claims.(jwt.MapClaims); ok && token.Valid {
        username := claims["username"]
        c.JSON(200, gin.H{"username": username})
    } else {
        c.JSON(401, gin.H{"error": "token无效"})
    }
})
```
