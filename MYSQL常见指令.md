作为 MySQL 小白，掌握以下常见指令和操作可以快速上手数据库的基础使用。本文会从**连接数据库**、**数据库/表的基本操作**、**增删改查（CRUD）**、**索引与优化**、**事务**等核心场景展开，附带示例和注意事项。

### **一、连接 MySQL 数据库**

首先需要连接到 MySQL 服务，通常通过命令行工具（如 Linux 的终端、Windows 的 CMD/PowerShell 或 MySQL Workbench 图形化工具）。

#### 1. 命令行连接

bash

复制

```bash
# 格式：mysql -h 主机地址 -P 端口号 -u 用户名 -p
# 示例（本地连接，默认端口3306，root用户）：
mysql -h localhost -P 3306 -u root -p

# 输入后会提示输入密码（密码输入时不显示），登录成功后进入 MySQL 命令行。
```

- `-h`：指定服务器主机（本地可省略或写 `localhost`）。
- `-P`：指定端口（默认 3306，可省略）。
- `-u`：用户名（如 `root` 是管理员账号）。
- `-p`：提示输入密码（密码直接跟在 `-p` 后写 `mysql -u root -p123` 也行，但不安全，建议分开输入）。

#### 2. 退出连接

sql

复制

```sql
exit;   -- 或 quit; 或 \q;
```

### **二、数据库的基本操作**

数据库是存储数据的“容器”，先学会管理数据库本身。

#### 1. 查看所有数据库

sql

复制

```sql
SHOW DATABASES;   -- 显示当前 MySQL 服务中所有数据库
```

#### 2. 创建数据库

sql

复制

```sql
CREATE DATABASE 数据库名;   -- 示例：创建名为 mydb 的数据库
CREATE DATABASE IF NOT EXISTS mydb;   -- 如果不存在则创建（推荐）
```

- 可选参数：指定字符集（如

   

  ```
  CHARACTER SET utf8mb4
  ```

   

  支持中文和 emoji）：

  sql

  复制

  ```sql
  CREATE DATABASE mydb CHARACTER SET utf8mb4;
  ```

#### 3. 选择/切换数据库

sql

复制

```sql
USE 数据库名;   -- 示例：切换到 mydb 数据库（后续操作基于此库）
```

#### 4. 删除数据库（谨慎！）

sql

复制

```sql
DROP DATABASE 数据库名;   -- 示例：删除 mydb 数据库（数据无法恢复！）
DROP DATABASE IF EXISTS mydb;   -- 安全写法，避免报错
```

### **三、数据表的基本操作**

表是数据库的核心，用于存储结构化数据（类似 Excel 表格）。

#### 1. 查看当前数据库的所有表

sql

复制

```sql
SHOW TABLES;   -- 需先 USE 数据库，否则会报错
```

#### 2. 创建表

需要定义表的**字段名**、**数据类型**和**约束**（如主键）。

**语法格式**：

sql

复制

```sql
CREATE TABLE 表名 (
  字段1 数据类型 [约束],
  字段2 数据类型 [约束],
  ...
  [主键约束]
);
```

**示例**：创建一个用户表 `user`，包含 id（主键）、姓名、年龄、邮箱、注册时间。

sql

复制

```sql
CREATE TABLE user (
  id INT PRIMARY KEY AUTO_INCREMENT,   -- 主键，自动递增
  name VARCHAR(50) NOT NULL,           -- 姓名，非空，最多50字符
  age TINYINT UNSIGNED,                -- 年龄，无符号（0-255）
  email VARCHAR(100) UNIQUE,           -- 邮箱，唯一约束（不重复）
  create_time DATETIME DEFAULT NOW()   -- 注册时间，默认当前时间
);
```

**常用数据类型**：

- 整数：`INT`（4字节）、`TINYINT`（1字节，如年龄）、`BIGINT`（8字节，如ID）。
- 字符串：`VARCHAR(n)`（可变长度，n≤255）、`CHAR(n)`（定长，如手机号 `CHAR(11)`）、`TEXT`（长文本）。
- 时间：`DATETIME`（日期时间，如 '2024-05-20 12:00:00'）、`DATE`（仅日期）、`TIMESTAMP`（时间戳，范围小但省空间）。
- 数值：`FLOAT`（浮点数）、`DECIMAL(M,D)`（精确小数，如金额 `DECIMAL(10,2)` 表示共10位，2位小数）。

**约束**：

- `PRIMARY KEY`：主键（唯一标识一行，不可重复且非空）。
- `AUTO_INCREMENT`：自增（仅整数类型，常配合主键）。
- `NOT NULL`：非空（字段必须有值）。
- `UNIQUE`：唯一（字段值不可重复）。
- `DEFAULT`：默认值（插入数据时未指定则用默认值）。

#### 3. 查看表结构

sql

复制

```sql
DESCRIBE 表名;   -- 简写 DESC 表名；示例：DESC user;
```

输出会显示字段名、类型、是否允许空、键类型等信息。

#### 4. 修改表结构（ALTER TABLE）

常见操作：添加字段、修改字段、删除字段。

**示例**：

- 添加一个

   

  ```
  phone
  ```

   

  字段（手机号，唯一）：

  sql

  复制

  ```sql
  ALTER TABLE user ADD COLUMN phone CHAR(11) UNIQUE;
  ```

- 修改

   

  ```
  age
  ```

   

  字段的类型为

   

  ```
  SMALLINT
  ```

  （更省空间）：

  sql

  复制

  ```sql
  ALTER TABLE user MODIFY COLUMN age SMALLINT UNSIGNED;
  ```

- 重命名

   

  ```
  phone
  ```

   

  字段为

   

  ```
  mobile
  ```

  ：

  sql

  复制

  ```sql
  ALTER TABLE user CHANGE COLUMN phone mobile CHAR(11) UNIQUE;
  ```

- 删除

   

  ```
  mobile
  ```

   

  字段：

  sql

  复制

  ```sql
  ALTER TABLE user DROP COLUMN mobile;
  ```

#### 5. 删除表（谨慎！）

sql

复制

```sql
DROP TABLE 表名;   -- 示例：DROP TABLE user; （数据无法恢复！）
DROP TABLE IF EXISTS user;   -- 安全写法
```

### **四、增删改查（CRUD）：核心操作**

CRUD 是数据库最常见的操作，对应**插入（Create）、查询（Read）、更新（Update）、删除（Delete）**。

#### 1. 插入数据（INSERT）

**语法**：

sql

复制

```sql
-- 方式1：指定字段名（推荐，避免字段顺序错误）
INSERT INTO 表名 (字段1, 字段2, ...) VALUES (值1, 值2, ...);

-- 方式2：插入所有字段（需按表结构顺序）
INSERT INTO 表名 VALUES (值1, 值2, ...);
```

**示例**：

sql

复制

```sql
-- 插入一条用户数据（指定字段）
INSERT INTO user (name, age, email) 
VALUES ('张三', 25, 'zhangsan@example.com');

-- 插入多条数据（批量插入）
INSERT INTO user (name, age, email) 
VALUES 
  ('李四', 28, 'lisi@example.com'),
  ('王五', 22, 'wangwu@example.com');
```

- 字符串和日期需用单引号 `' '` 包裹（MySQL 支持双引号，但建议单引号）。
- 自增字段（如 `id`）和有默认值的字段（如 `create_time`）可省略。

#### 2. 查询数据（SELECT）

查询是最复杂的操作，基础用法如下：

**基础查询**：

sql

复制

```sql
-- 查询所有字段（* 表示所有列）
SELECT * FROM user;

-- 查询指定字段（推荐，明确需求）
SELECT name, age, email FROM user;

-- 给字段起别名（AS 可省略）
SELECT name AS 姓名, age AS 年龄 FROM user;
```

**条件查询（WHERE）**：

sql

复制

```sql
-- 查询年龄大于25岁的用户
SELECT * FROM user WHERE age > 25;

-- 查询姓名是“张三”或邮箱包含“example”的用户（OR/AND）
SELECT * FROM user WHERE name = '张三' OR email LIKE '%example%';

-- 模糊查询（LIKE）：% 表示任意字符，_ 表示单个字符
SELECT * FROM user WHERE email LIKE 'zhang%';   -- 邮箱以 zhang 开头

-- 范围查询（BETWEEN...AND...）
SELECT * FROM user WHERE age BETWEEN 20 AND 30;   -- 年龄20-30岁

-- 排除某个值（IN / NOT IN）
SELECT * FROM user WHERE id NOT IN (1, 2);   -- 排除id为1和2的用户
```

**排序（ORDER BY）**：

sql

复制

```sql
-- 按年龄降序排列（DESC：降序，ASC：升序，默认升序）
SELECT * FROM user ORDER BY age DESC;
```

**分页查询（LIMIT）**：

sql

复制

```sql
-- 查询第1-10条数据（偏移量0，取10条）
SELECT * FROM user LIMIT 0, 10;

-- 更简洁写法（查询第11-20条）
SELECT * FROM user LIMIT 10 OFFSET 10;   -- 或 LIMIT 10, 10（旧写法）
```

**聚合与分组（GROUP BY）**：

sql

复制

```sql
-- 统计总用户数（COUNT）
SELECT COUNT(*) AS 总用户数 FROM user;

-- 统计不同年龄的用户数量（GROUP BY 分组，COUNT(*) 统计每组数量）
SELECT age, COUNT(*) AS 人数 FROM user GROUP BY age;

-- 过滤分组（HAVING，类似 WHERE 但作用于分组后）
SELECT age, COUNT(*) AS 人数 FROM user GROUP BY age HAVING 人数 > 1;
```

#### 3. 更新数据（UPDATE）

**语法**：

sql

复制

```sql
UPDATE 表名 SET 字段1=值1, 字段2=值2 ... WHERE 条件;
```

- **必须加 WHERE 条件**！否则会更新全表数据（危险！）。

**示例**：

sql

复制

```sql
-- 将 id=1 的用户邮箱改为 'new_zhangsan@example.com'
UPDATE user SET email = 'new_zhangsan@example.com' WHERE id = 1;

-- 批量更新：所有年龄小于20岁的用户，年龄+1
UPDATE user SET age = age + 1 WHERE age < 20;
```

#### 4. 删除数据（DELETE）

**语法**：

sql

复制

```sql
DELETE FROM 表名 WHERE 条件;
```

- **必须加 WHERE 条件**！否则会删除全表数据（危险！）。

**示例**：

sql

复制

```sql
-- 删除 id=3 的用户
DELETE FROM user WHERE id = 3;

-- 清空表（删除所有数据，比 DELETE 快，但无法回滚）
TRUNCATE TABLE user;   -- 注意：TRUNCATE 是 DDL 操作，会重置自增ID
```

### **五、索引：提升查询效率**

索引是数据库的“目录”，能加速查询，但会增加插入/更新/删除的开销。

#### 1. 创建索引

sql

复制

```sql
-- 给 email 字段创建普通索引（加快按邮箱查询的速度）
CREATE INDEX idx_email ON user(email);

-- 给 name 字段创建唯一索引（同时保证唯一性）
CREATE UNIQUE INDEX idx_name ON user(name);

-- 建表时直接创建索引（推荐）
CREATE TABLE user (
  id INT PRIMARY KEY AUTO_INCREMENT,
  name VARCHAR(50) NOT NULL,
  email VARCHAR(100),
  INDEX idx_email (email)   -- 建表时创建索引
);
```

#### 2. 查看索引

sql

复制

```sql
SHOW INDEX FROM user;   -- 查看 user 表的所有索引
```

#### 3. 删除索引

sql

复制

```sql
DROP INDEX 索引名 ON 表名;   -- 示例：DROP INDEX idx_email ON user;
```

### **六、事务：保证数据一致性**

事务是一组操作的集合，要么全部成功，要么全部失败（如转账：A转100给B，A扣钱和B加钱必须同时完成）。

#### 1. 事务操作

sql

复制

```sql
-- 开始事务
START TRANSACTION;   -- 或 BEGIN;

-- 执行操作（例如：A扣100，B加100）
UPDATE account SET balance = balance - 100 WHERE id = 1;
UPDATE account SET balance = balance + 100 WHERE id = 2;

-- 提交事务（所有操作生效）
COMMIT;

-- 回滚事务（所有操作撤销）
ROLLBACK;
```

#### 2. 自动提交

MySQL 默认开启自动提交（每条 SQL 单独作为一个事务）。关闭自动提交后可手动控制事务：

sql

复制

```sql
SET autocommit = 0;   -- 关闭自动提交（0：关闭，1：开启）
-- 执行多个操作...
COMMIT;   -- 手动提交
-- 或 ROLLBACK; 回滚
SET autocommit = 1;   -- 恢复自动提交
```

### **七、其他常用指令**

#### 1. 查看 MySQL 版本

sql

复制

```sql
SELECT VERSION();   -- 输出 MySQL 版本号（如 8.0.33）
```

#### 2. 查看当前用户

sql

复制

```sql
SELECT USER();   -- 输出当前连接的用户名和主机（如 'root@localhost'）
```

#### 3. 备份与恢复数据库（命令行工具）

- 

  备份

  （用

   

  ```
  mysqldump
  ```

   

  工具，需在终端执行，非 MySQL 命令行）：

  bash

  复制

  ```bash
  mysqldump -u 用户名 -p 数据库名 > 备份文件路径.sql
  # 示例：备份 mydb 数据库到 mydb_backup.sql
  mysqldump -u root -p mydb > /home/user/mydb_backup.sql
  ```

- 

  恢复

  （登录 MySQL 命令行后执行）：

  sql

  复制

  ```sql
  SOURCE 备份文件路径.sql;   -- 示例：SOURCE /home/user/mydb_backup.sql;
  ```

### **注意事项（避坑指南）**

1. **字段名和表名的大小写**：Linux 系统下 MySQL 默认区分大小写（受 `lower_case_table_names` 配置影响），Windows 不区分，建议统一用小写。
2. **字符串用单引号**：虽然 MySQL 支持双引号，但 SQL 标准是单引号，避免混淆。
3. **备份数据**：执行 `DROP`、`TRUNCATE`、`DELETE` 前务必备份，尤其是生产环境！
4. **索引不是越多越好**：索引会占用空间，降低写入速度，只在高频查询字段添加。
5. **自增 ID 重置**：`TRUNCATE TABLE` 会重置自增 ID（从1开始），`DELETE` 不会。

### **学习资源推荐**

- 图形化工具：MySQL Workbench（官方）、Navicat（付费）、DBeaver（免费）。
- 练习平台：LeetCode 数据库题、《SQL 必知必会》书籍。