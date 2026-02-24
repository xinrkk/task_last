# MySQL 学习笔记

> —康思晟

## 一、MySQL 前世今生

### 1.1 发展历程

- **1979年**: 瑞典公司TcX开发出UNIREG数据库工具
- **1995年**: MySQL 1.0 发布，名字来源于创始人女儿名字 "My"
- **2000年**: 采用GPL开源协议，开始流行
- **2008年**: Sun公司以10亿美元收购MySQL AB
- **2010年**: Oracle收购Sun公司，MySQL归入Oracle旗下
- **2018年**: MySQL 8.0 正式版发布，成为主流版本

### 1.2 为什么选择MySQL

- **开源免费**: 社区版完全免费，降低开发成本
- **性能优秀**: 查询速度快，支持高并发
- **跨平台**: 支持Windows/Linux/macOS
- **生态完善**: 丰富的周边工具和文档
- **易于上手**: 学习曲线平缓，适合初学者

------

## 二、MySQL 环境搭建

### 2.1. **准备虚拟机**
在 VMware 或 VirtualBox 中安装 Linux。可以从 CentOS 7/8 等版本入手，网络适配器可选择 NAT 模式以方便共享主机网络 。
### 2.2. **安装 MySQL**：
   - **下载**：从官网获取适合 Linux 的 MySQL 社区版 RPM 包或 tar 包，上传至虚拟机 。
   - **安装**：以 CentOS 7 为例，通常通过 `yum` 或 `rpm` 命令安装下载好的包。安装后启动 MySQL 服务：`systemctl start mysqld` 。
   - **安全配置**：运行 `mysql_secure_installation` 来设置 root 密码、移除匿名用户、禁止 root 远程登录等 。

------

## 三、MySQL 基础概念

### 3.1 数据表结构


**常用数据类型:**

| 类型         | 描述       | 示例                  |
| :----------- | :--------- | :-------------------- |
| INT          | 整数       | 18, 100               |
| VARCHAR(n)   | 变长字符串 | '张三'                |
| CHAR(n)      | 定长字符串 | '001'                 |
| TEXT         | 长文本     | 文章内容              |
| DECIMAL(m,d) | 精确小数   | 95.5                  |
| DATE         | 日期       | '2023-01-01'          |
| DATETIME     | 日期时间   | '2023-01-01 10:30:00' |
| TIMESTAMP    | 时间戳     | 自动记录时间          |

**约束条件:**

- **NOT NULL**: 不能为空
- **UNIQUE**: 值唯一
- **PRIMARY KEY**: 主键，唯一标识一条记录
- **FOREIGN KEY**: 外键，关联其他表
- **DEFAULT**: 默认值
- **AUTO_INCREMENT**: 自动递增

### 3.2 表之间关联方式

**1. 一对一关系**

sql

```
-- 用户表
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(50)
);

-- 用户详情表（一对一）
CREATE TABLE user_profiles (
    user_id INT PRIMARY KEY,
    address VARCHAR(200),
    phone VARCHAR(20),
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```



**2. 一对多关系**

sql

```
-- 班级表（一）
CREATE TABLE classes (
    id INT PRIMARY KEY,
    class_name VARCHAR(50)
);

-- 学生表（多）
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    class_id INT,
    FOREIGN KEY (class_id) REFERENCES classes(id)
);
```



**3. 多对多关系**

sql

```
-- 学生表
CREATE TABLE students (
    id INT PRIMARY KEY,
    name VARCHAR(50)
);

-- 课程表
CREATE TABLE courses (
    id INT PRIMARY KEY,
    course_name VARCHAR(50)
);

-- 选课表（中间表）
CREATE TABLE student_courses (
    student_id INT,
    course_id INT,
    score DECIMAL(5,2),
    PRIMARY KEY (student_id, course_id),
    FOREIGN KEY (student_id) REFERENCES students(id),
    FOREIGN KEY (course_id) REFERENCES courses(id)
);
```



### 3.3 SQL 基本操作（CRUD）

**增**

sql

```
INSERT INTO student VALUES (1, '张三', 18);
```



**删**

sql

```
DELETE FROM student WHERE id = 1;
```



**改**

sql

```
UPDATE student SET age = 20 WHERE name = '张三';
```



**查**

sql

```
SELECT * FROM student;
SELECT name FROM student WHERE age > 18;
```
