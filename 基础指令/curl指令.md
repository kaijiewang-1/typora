以下是curl命令的基本操作指南，特别适合用于测试和调试API：

## 1. 基本GET请求

bash

复制

```
# 最简单的GET请求
curl https://api.example.com/users

# 带详细信息的GET请求
curl -v https://api.example.com/users

# 只显示响应头
curl -I https://api.example.com/users

# 将响应保存到文件
curl -o response.json https://api.example.com/users
```

## 2. 带参数的GET请求

bash

复制

```
# 查询参数
curl "https://api.example.com/users?page=1&limit=10"

# 使用 -G 参数自动处理查询参数
curl -G https://api.example.com/search \
  -d "q=keyword" \
  -d "type=user"
```

## 3. POST请求

bash

复制

```
# 简单的POST请求
curl -X POST https://api.example.com/users

# 带JSON数据的POST请求
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John","email":"john@example.com"}'

# 从文件读取JSON数据
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d @data.json
```

## 4. 认证和头部信息

bash

复制

```
# 添加Bearer Token认证
curl -H "Authorization: Bearer your-token-here" \
  https://api.example.com/users

# 添加多个头部信息
curl -H "Content-Type: application/json" \
  -H "Authorization: Bearer token" \
  -H "X-API-Key: your-api-key" \
  https://api.example.com/users

# 基本认证
curl -u username:password https://api.example.com/users
```

## 5. PUT、PATCH、DELETE请求

bash

复制

```
# PUT请求
curl -X PUT https://api.example.com/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe"}'

# PATCH请求
curl -X PATCH https://api.example.com/users/1 \
  -H "Content-Type: application/json" \
  -d '{"email":"new@example.com"}'

# DELETE请求
curl -X DELETE https://api.example.com/users/1
```

## 6. 文件上传

bash

复制

```
# 上传文件
curl -X POST https://api.example.com/upload \
  -F "file=@/path/to/file.jpg" \
  -F "name=profile_picture"

# 多文件上传
curl -X POST https://api.example.com/upload \
  -F "files=@file1.jpg" \
  -F "files=@file2.jpg"
```

## 7. 调试和输出控制

bash

复制

```
# 显示详细请求信息（推荐调试使用）
curl -v https://api.example.com/users

# 只显示响应体（静默模式）
curl -s https://api.example.com/users

# 显示请求和响应头
curl -i https://api.example.com/users

# 跟踪重定向
curl -L https://api.example.com/users

# 显示请求时间
curl -w "@curl-format.txt" https://api.example.com/users
```

创建时间格式文件 `curl-format.txt`：

复制

```
time_namelookup:  %{time_namelookup}s\n
time_connect:  %{time_connect}s\n
time_appconnect:  %{time_appconnect}s\n
time_pretransfer:  %{time_pretransfer}s\n
time_redirect:  %{time_redirect}s\n
time_starttransfer:  %{time_starttransfer}s\n
----------\n
time_total:  %{time_total}s\n
```

## 8. 处理Cookie

bash

复制

```
# 保存Cookie到文件
curl -c cookies.txt https://api.example.com/login

# 使用保存的Cookie
curl -b cookies.txt https://api.example.com/dashboard

# 同时保存和使用Cookie
curl -c cookies.txt -b cookies.txt https://api.example.com/auth
```

## 9. 实用示例组合

bash

复制

```
# 完整的API测试示例
curl -X POST https://api.example.com/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"user","password":"pass"}' \
  -c cookies.txt \
  -v

# 然后使用获取的token
curl -H "Authorization: Bearer token-from-login" \
  https://api.example.com/users/me

# 测试分页API
curl "https://api.example.com/posts?page=1&per_page=20" \
  -H "Accept: application/json"
```

## 10. 常用选项总结

| 选项             | 说明                    |
| :--------------- | :---------------------- |
| `-X <METHOD>`    | 指定HTTP方法            |
| `-H <HEADER>`    | 添加请求头              |
| `-d <DATA>`      | 发送POST数据            |
| `-G`             | 将-d参数作为GET查询参数 |
| `-i`             | 显示响应头              |
| `-I`             | 只显示响应头            |
| `-v`             | 详细输出                |
| `-s`             | 静默模式                |
| `-o <FILE>`      | 输出到文件              |
| `-L`             | 跟随重定向              |
| `-u <USER:PASS>` | 基本认证                |
| `-b <COOKIE>`    | 使用Cookie              |
| `-c <FILE>`      | 保存Cookie到文件        |

## 11. 实际API测试示例

假设你的API文档如下：

- 基础URL: `https://api.example.com`
- 获取用户: `GET /users`
- 创建用户: `POST /users`

bash

复制

```
# 测试获取用户
curl -H "Content-Type: application/json" \
  https://api.example.com/users

# 测试创建用户
curl -X POST https://api.example.com/users \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Alice",
    "email": "alice@example.com",
    "age": 25
  }' \
  -v
```

这些curl命令可以帮助你在开发Android应用前先测试API接口是否正常工作。