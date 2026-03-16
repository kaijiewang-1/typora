有几种常用方法可以将 Python 环境导出到 requirements.txt 文件：

## 方法一：使用 pip freeze（最常用）

```
bash# 导出所有已安装的包
pip freeze > requirements.txt
```

**优点：**

- 简单快速
- 包含所有依赖包（包括子依赖）
- 指定具体版本号

**缺点：**

- 会导出所有包，包括项目不需要的

------

## 方法二：使用 pipreqs（推荐）

只导出项目中实际使用的包：

```
bash# 1. 安装 pipreqs
pip install pipreqs

# 2. 导出项目依赖（在项目根目录执行）
pipreqs . --encoding=utf8 --force
```

**参数说明：**

- `.` - 当前目录
- `--encoding=utf8` - 指定编码避免中文路径错误
- `--force` - 覆盖已存在的 requirements.txt

**优点：**

- 只扫描项目中实际 import 的包
- 不包含未使用的依赖
- 文件更简洁

------

## 方法三：使用 pip-tools（更专业）

```
bash# 1. 安装 pip-tools
pip install pip-tools

# 2. 创建 requirements.in 文件，手动列出直接依赖
echo "fastapi" > requirements.in
echo "uvicorn" >> requirements.in

# 3. 生成 requirements.txt（包含所有子依赖）
pip-compile requirements.in
```

**优点：**

- 区分直接依赖和间接依赖
- 更好的版本控制

------

## 方法四：手动导出主要包

```
bash# 只导出顶层包（不包含子依赖）
pip list --format=freeze | grep -v "^\-e" > requirements.txt
```

------

## 针对您的项目

根据您的项目，我建议使用 **pipreqs**，它会自动扫描 main.py 中的导入：

```
bash# 进入项目目录
cd e:\code\smartaqua\AquaGarden\software\backend

# 使用 pipreqs 生成
pipreqs . --encoding=utf8 --force
```

生成的 requirements.txt 会包含：

```
fastapi
uvicorn
python-jose
passlib
sqlalchemy
opencv-python
numpy
python-multipart
websockets
```

------

## 验证 requirements.txt

生成后，可以在新环境中测试：

```
bash# 创建虚拟环境
python -m venv venv

# 激活虚拟环境
# Windows:
venv\Scripts\activate
# Linux/Mac:
source venv/bin/activate

# 安装依赖
pip install -r requirements.txt
```

------

## 最佳实践建议

1. **开发环境**：使用 `pipreqs` 生成简洁的依赖列表
2. **生产环境**：使用 `pip freeze` 确保版本完全一致
3. **定期更新**：使用 `pip list --outdated` 检查可更新的包
4. **版本锁定**：重要项目建议使用 `poetry` 或 `pipenv` 管理依赖