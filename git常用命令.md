# Git 常用命令手册（小白友好版）

Git 是目前最流行的**分布式版本控制系统**，用于管理代码的修改历史、协作开发和版本回溯。本手册整理了 Git 最常用的命令，涵盖**初始化、提交、分支、远程、撤销**等场景，附**用途、语法、示例**，帮你快速上手！

## 一、前置概念（必懂）

在使用命令前，先理解 Git 的核心模型：

- **工作区（Working Directory）**：你编辑文件的本地目录（比如 `my-project` 文件夹）。
- **暂存区（Staging Area/Index）**：临时存放待提交的修改（相当于“草稿箱”）。
- **本地仓库（Local Repository）**：`.git` 隐藏文件夹，保存所有历史版本。
- **远程仓库（Remote Repository）**：GitHub/Gitee 等线上仓库，用于协作。

## 二、基础命令（初始化与文件操作）

### 1. 初始化仓库

**用途**：将本地目录变成 Git 仓库（生成 `.git` 隐藏文件夹）。
​**语法**​：`git init [目录路径]`
​**示例**​：

bash

复制

```bash
# 在当前目录初始化仓库
git init

# 在指定目录初始化（比如 ./my-project）
git init ./my-project
```

### 2. 查看文件状态

**用途**：检查工作区/暂存区的修改状态（哪些文件改了、哪些在暂存区、哪些未跟踪）。
​**语法**​：`git status`
​**示例**​：

bash

复制

```bash
git status
# 输出示例：
# On branch main
# Changes not staged for commit:
#   (use "git add <file>..." to update what will be committed)
#   (use "git restore <file>..." to discard changes in working directory)
#         modified:   README.md
#
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#         src/index.js
```

### 3. 添加文件到暂存区

**用途**：将工作区的修改放入“草稿箱”，准备提交。
​**语法**​：

- `git add <文件名>`：添加单个文件（比如 `git add README.md`）。
- `git add .`：添加当前目录**所有修改/新增**的文件（最常用）。
- `git add <目录名>`：添加某个目录下的所有修改（比如 `git add src/`）。
  ​**示例**​：

bash

复制

```bash
# 添加所有修改到暂存区
git add .
```

### 4. 提交到本地仓库

**用途**：将暂存区的修改保存为**本地版本**（生成一个提交记录）。
​**语法**​：`git commit -m "提交说明"`
​**关键**​：`-m` 后面的说明要**清晰简洁**​（比如“修复登录按钮样式”“新增用户模块”），方便后续回溯。
​**示例**​：

bash

复制

```bash
git commit -m "初始化项目：添加 README 和基础结构"
```

### 5. 查看提交历史

**用途**：查看本地仓库的所有提交记录（包括提交人、时间、说明、哈希值）。
​**语法**​：

- `git log`：查看完整历史（按 `q` 退出）。
- `git log --oneline`：查看简洁历史（单行显示，推荐）。
- `git log --graph`：查看分支合并的图形化历史。
  ​**示例**​：

bash

复制

```bash
git log --oneline
# 输出示例：
# a1b2c3d (HEAD -> main) 初始化项目：添加 README 和基础结构
# d4e5f6a 第一次提交：创建 README.md
```

## 三、分支管理（核心协作功能）

分支是 Git 的灵魂，用于**隔离不同功能的修改**（比如“开发新功能”和“修复 bug”互不影响）。

### 1. 查看本地分支

**用途**：列出所有本地分支（`*` 标记当前所在分支）。
​**语法**​：`git branch`
​**示例**​：

bash

复制

```bash
git branch
# 输出示例：
# * main
#   dev
#   feature/login
```

### 2. 创建分支

**用途**：基于当前分支创建新分支（不会切换到新分支）。
​**语法**​：`git branch <分支名>`
​**示例**​：

bash

复制

```bash
# 创建名为 feature/search 的分支
git branch feature/search
```

### 3. 切换分支

**用途**：切换到指定分支（开始在该分支上工作）。
​**语法**​：

- `git checkout <分支名>`：传统方式（Git 1.x 常用）。
- `git switch <分支名>`：Git 2.23+ 推荐（更直观，“切换”的语义更明确）。
  ​**示例**​：

bash

复制

```bash
# 切换到 feature/search 分支（两种方式任选其一）
git checkout feature/search
# 或
git switch feature/search
```

### 4. 创建并切换分支（一步到位）

**用途**：同时创建新分支并切换过去（最常用）。
​**语法**​：`git checkout -b <分支名>` 或 `git switch -c <分支名>`
​**示例**​：

bash

复制

```bash
# 创建并切换到 dev 分支
git switch -c dev
```

### 5. 合并分支

**用途**：将指定分支的修改合并到当前分支（比如把 `feature/login` 的修改合并到 `main`）。
​**步骤**​：

1. 切换到目标分支（比如 `main`）：`git switch main`
2. 合并指定分支：`git merge <要合并的分支名>`
   ​**示例**​：

bash

复制

```bash
# 把 feature/login 分支的修改合并到 main
git switch main
git merge feature/login
```

### 6. 删除分支

**用途**：删除本地已合并的分支（未合并的分支需加 `-D` 强制删除）。
​**语法**​：

- `git branch -d <分支名>`：删除已合并的分支（安全）。
- `git branch -D <分支名>`：强制删除未合并的分支（危险，慎用）。
  ​**示例**​：

bash

复制

```bash
# 删除已合并的 feature/search 分支
git branch -d feature/search
```

## 四、远程仓库操作（协作关键）

远程仓库（比如 GitHub）用于**共享代码**和**团队协作**。

### 1. 添加远程仓库

**用途**：将本地仓库与线上远程仓库关联（`origin` 是远程仓库的默认名称）。
​**语法**​：`git remote add origin <远程仓库地址>`
​**示例**​：

bash

复制

```bash
# 关联 GitHub 仓库（地址从 GitHub 仓库页复制）
git remote add origin https://github.com/your-username/your-repo.git
```

### 2. 推送本地分支到远程

**用途**：将本地分支的修改上传到远程仓库（首次推送需加 `-u` 设置默认上游）。
​**语法**​：`git push -u origin <本地分支名>`
​**后续推送**​：若已设置上游，直接 `git push` 即可。
​**示例**​：

bash

复制

```bash
# 首次推送 main 分支到远程（设置 origin/main 为上游）
git push -u origin main

# 后续推送（直接 push）
git push
```

### 3. 拉取远程更新

**用途**：从远程仓库获取最新修改，并**合并到本地分支**（相当于 `git fetch` + `git merge`）。
​**语法**​：`git pull origin <远程分支名>`
​**示例**​：

bash

复制

```bash
# 拉取远程 main 分支的更新并合并到本地 main
git pull origin main
```

### 4. 获取远程更新（不合并）

**用途**：仅获取远程仓库的修改，但不合并到本地（用于查看远程变化）。
​**语法**​：`git fetch origin`
​**示例**​：

bash

复制

```bash
# 获取远程所有分支的更新
git fetch origin
```

### 5. 查看远程仓库地址

**用途**：确认当前仓库关联的远程地址。
​**语法**​：`git remote -v`
​**示例**​：

bash

复制

```bash
git remote -v
# 输出示例：
# origin  https://github.com/your-username/your-repo.git (fetch)
# origin  https://github.com/your-username/your-repo.git (push)
```

## 五、撤销与回溯（救急必备）

### 1. 撤销工作区的修改

**用途**：丢弃工作区中某个文件的修改（回到最后一次 `git add` 或 `git commit` 的状态）。
​**语法**​：`git checkout -- <文件名>`
​**示例**​：

bash

复制

```bash
# 丢弃 README.md 的修改
git checkout -- README.md
```

### 2. 撤销暂存区的修改

**用途**：将暂存区的文件移回工作区（比如不小心 `git add` 了不该加的文件）。
​**语法**​：`git reset HEAD <文件名>`
​**示例**​：

bash

复制

```bash
# 将 src/index.js 从暂存区移回工作区
git reset HEAD src/index.js
```

### 3. 撤销最后一次提交

**用途**：回退到上一次提交（有两种模式：保留修改/彻底删除）。
​**语法**​：

- `git reset --soft HEAD^`：撤销提交，保留修改到暂存区（适合“修改提交说明”）。
- `git reset --hard HEAD^`：彻底撤销提交，修改回到工作区（**慎用**，会丢失最后一次提交的修改）。
  ​**说明**​：`HEAD^` 表示“上一个提交”，`HEAD~n` 表示“前 n 个提交”。
  ​**示例**​：

bash

复制

```bash
# 彻底撤销最后一次提交（回到上一次的状态）
git reset --hard HEAD^
```

### 4. 安全撤销提交（推荐团队协作）

**用途**：生成一个新的提交来“抵消”指定提交的修改（不会改变历史，适合已推送到远程的提交）。
​**语法**​：`git revert <提交哈希>`
​**示例**​：

bash

复制

```bash
# 撤销哈希为 a1b2c3d 的提交
git revert a1b2c3d
```

## 六、标签管理（版本发布）

标签用于**标记重要版本**（比如“v1.0.0 发布”），比提交哈希更易读。

### 1. 查看标签

**用途**：列出所有本地标签。
​**语法**​：`git tag`
​**示例**​：

bash

复制

```bash
git tag
# 输出示例：
# v1.0.0
# v1.1.0
```

### 2. 创建标签

**语法**：

- `git tag <标签名>`：创建轻量标签（无说明，推荐少用）。
- `git tag -a <标签名> -m "标签说明"`：创建附注标签（有说明，推荐）。
  ​**示例**​：

bash

复制

```bash
# 创建附注标签（标记 v1.0.0 发布）
git tag -a v1.0.0 -m "正式发布 1.0 版本"
```

### 3. 推送标签到远程

**用途**：将本地标签上传到远程仓库（默认不会自动推送）。
​**语法**​：`git push origin <标签名>` 或 `git push origin --tags`（推送所有标签）。
​**示例**​：

bash

复制

```bash
# 推送 v1.0.0 标签到远程
git push origin v1.0.0
```

### 4. 删除标签

**用途**：删除本地/远程标签。
​**语法**​：

- `git tag -d <标签名>`：删除本地标签。
- `git push origin --delete <标签名>`：删除远程标签。
  ​**示例**​：

bash

复制

```bash
# 删除本地 v1.0.0 标签
git tag -d v1.0.0

# 删除远程 v1.0.0 标签
git push origin --delete v1.0.0
```

## 七、常用技巧与注意事项

1. **提交说明规范**：用“动作+内容”格式（比如“fix: 修复登录页密码框样式”“feat: 新增购物车功能”），方便后续排查问题。
2. **解决冲突**：`git pull` 或 `git merge` 时可能遇到冲突，需手动编辑冲突文件（标记为 `<<<<<<<`、`=======`、`>>>>>>>` 的部分），然后 `git add` 和 `git commit` 解决。
3. **别名简化命令**：可以用 `git config` 设置别名（比如 `git config --global alias.st status`，之后用 `git st` 代替 `git status`）。

## 总结：Git 命令速查表

|      场景      |            命令             |
| :------------: | :-------------------------: |
|   初始化仓库   |         `git init`          |
|    查看状态    |        `git status`         |
|  添加到暂存区  |         `git add .`         |
|   提交到本地   |   `git commit -m "说明"`    |
|  查看提交历史  |     `git log --oneline`     |
| 创建并切换分支 |   `git switch -c 分支名`    |
|  推送本地分支  | `git push -u origin 分支名` |
|  拉取远程更新  |  `git pull origin 分支名`   |
| 撤销工作区修改 |  `git checkout -- 文件名`   |
|  安全撤销提交  |    `git revert 提交哈希`    |

跟着本手册练习几次，你就能熟练掌握 Git 的核心操作啦！遇到问题可随时查 Git 官方文档（https://git-scm.com/doc）～ 😊



## LF 和CRLF的报错问题



这个警告是 Git 在处理 **换行符（Line Ending）**时发出的提示，属于正常现象，但可以根据需求选择是否消除。以下是详细解释和解决方案：

------

### **问题原因**

Git 在不同操作系统中对换行符的处理方式不同：

- **Windows**：默认使用 `CRLF`（回车+换行，`\r\n`）
- **Linux/macOS**：默认使用 `LF`（换行，`\n`）

当你在一个操作系统中修改文件后，Git 会自动将换行符转换为该系统的格式。此警告表示：

> **当前文件中有以 `LF`结尾的行，下次 Git 修改它时会自动转换为 `CRLF`（仅限 Windows）。**

------

### **解决方案**

#### **1. 忽略警告（推荐）**

如果：

- 你仅在 Windows 上开发
- 不需要跨平台协作
- 文件内容不受换行符影响（如文本文件、Python/JavaScript 代码等）

则无需处理，直接忽略警告即可。Git 会在提交时自动统一为 `LF`，检出时转为 `CRLF`。

------

#### **2. 消除警告（配置 Git）**

如果你希望统一换行符规则，可以配置 Git 的行为：

##### **(1) 针对 Windows 用户**

在 Git Bash 中运行以下命令：

bash

复制

```
git config --global core.autocrlf true
```

效果：

- **提交（Commit）时**：Git 自动将 `CRLF`转为 `LF`（统一仓库中的换行符为 `LF`）。
- **检出（Checkout）时**：Git 自动将 `LF`转为 `CRLF`（确保本地文件用 Windows 风格换行符）。

------

##### **(2) 针对 Linux/macOS 用户**

在 Git Bash 中运行：

bash

复制

```
git config --global core.autocrlf input
```

效果：

- **提交时**：自动将 `CRLF`转为 `LF`（保持仓库统一为 `LF`）。
- **检出时**：保留 `LF`（不转换）。

------

##### **(3) 强制所有文件使用 LF（跨平台协作推荐）**

在仓库根目录创建 `.gitattributes`文件，添加以下内容：

gitattributes

复制

```
* text=auto eol=lf
```

然后提交此文件：

bash

复制

```
git add .gitattributes
git commit -m "Fix line endings with .gitattributes"
```

效果：

- 所有文本文件统一使用 `LF`换行符。
- 跨平台协作时不会出现换行符混乱。

------

### **验证配置**

运行以下命令检查文件换行符状态：

bash

复制

```
git ls-files --eol
```

输出示例：

复制

```
i/lf    w/crlf  attr/text=auto eol=crlf README.md
```

- `i/lf`：仓库中存储为 `LF`
- `w/crlf`：工作区显示为 `CRLF`（Windows 用户正常）

------

### **注意事项**

1. **已存在的文件**：如果已有文件混合了 `CRLF`和 `LF`，需手动修复：

   bash

   复制

   ```
   git add --renormalize .
   git commit -m "Normalize line endings"
   ```

2. **二进制文件**：确保 `.gitattributes`中排除二进制文件（如图片、PDF）：

   gitattributes

   复制

   ```
   *.png binary
   *.jpg binary
   ```

3. **团队协作**：建议所有成员统一配置 `.gitattributes`，避免换行符冲突。

------

### **总结**

- **警告无害**：可直接忽略，Git 会自动处理换行符。
- **统一规范**：通过 `.gitattributes`或 `core.autocrlf`配置，确保团队换行符一致。
- **跨平台项目**：强制使用 `LF`是最佳实践。





## GIT  LFS 大文件问题解决

根据GitHub的提示，您上传的文件超过了推荐大小限制。以下是解决方案：

**步骤说明：**

1. **安装Git LFS**

bash

复制

```
# 根据操作系统选择安装方式
# Windows/macOS通过官网下载安装包
# Linux使用包管理器：
curl -s https://packagecloud.io/install/repositories/github/git-lfs/script.deb.sh | sudo bash
sudo apt-get install git-lfs
```

1. **初始化Git LFS**

bash

复制

```
git lfs install
```

1. **追踪大文件类型**

bash

复制

```
git lfs track "*.zip"  # 将所有zip文件加入追踪
git add .gitattributes  # 提交追踪规则
```

1. **迁移历史大文件（如有旧提交）**

bash

复制

```
git lfs migrate import --include="*.zip" --everything
```

1. **正常提交流程**

bash

复制

```
git add 大一/大物实验.zip
git commit -m "Add large file with LFS"
git push origin main
```

**注意事项：**

- 确保团队所有成员都安装Git LFS
- 每月免费额度：1GB存储 + 1GB流量
- 查看用量：https://github.com/settings/billing
- 如果持续需要大文件存储，可考虑GitHub Packages或其他云存储方案

**验证结果：**

推送后会看到类似日志：

复制

```
Uploading LFS objects: 100% (1/1), 72 MB | 0 B/s, done.
```

如果仍有问题，可使用以下命令检查：

bash

复制

```
git lfs ls-files  # 查看待上传的LFS文件
git lfs env       # 查看当前LFS环境配置
```



# 在Git中处理文件冲突是协作开发时常见的操作，以下是针对小白的详细解决步骤和实用技巧：

------

### **一、理解冲突的产生**

当多人同时修改同一文件的同一部分时，Git无法自动合并，会标记冲突（如`<<<<<<< HEAD`、`=======`、`>>>>>>>`）。冲突通常发生在以下场景：

- 

  **多人修改同一行代码**（如样式或逻辑）。

- 

  **删除与修改冲突**（一人删除文件，另一人修改）。

- 

  **重命名冲突**（一人重命名文件，另一人修改原文件）

  

------

### **二、解决冲突的步骤**

#### **1. 查看冲突文件**

- 

  运行 `git status`，会列出所有冲突文件（显示为 **both modified**）

- 

  示例输出：

  复制

  ```
  both modified: index.html
  ```

#### **2. 查看冲突内容**

- 

  用文本编辑器或IDE（如VS Code）打开冲突文件，会看到类似以下标记：

  html

  下载

  复制

  预览

  ```
  <<<<<<< HEAD
  <h1>当前分支的标题</h1>
  =======
  <h1>其他分支的标题</h1>
  >>>>>>> feature-branch
  ```

  **`<<<<<<< HEAD`**：当前分支的代码。

  **`=======`**：分隔符。

  **`>>>>>>> feature-branch`**：其他分支的代码

  

  

#### **3. 手动解决冲突**

- 

  **保留一方修改**：删除另一方代码及冲突标记。

  html

  下载

  复制

  预览

  ```
  <h1>当前分支的标题</h1>  <!-- 仅保留当前分支的代码 -->
  ```

- 

  **合并双方修改**：根据需求整合内容。

  html

  下载

  复制

  预览

  ```
  <h1>合并后的标题</h1>
  ```

  **删除冲突标记**：确保文件中无`<<<<<<<`、`=======`、`>>>>>>>`残留

  

#### **4. 标记冲突已解决**

运行 `git add <文件名>`将解决后的文件加入暂存区。

bash

复制

```
git add index.html
```

#### **5. 提交合并结果**

运行 `git commit`完成合并：

bash

复制

```
git commit -m "解决index.html冲突"
```

------

### **三、使用工具简化操作**

#### **1. 命令行工具**

**查看差异**：用 `git diff`查看冲突细节。

**快速放弃修改**：若冲突无法解决，可运行 `git checkout -- <文件名>`丢弃本地修改



#### **2. 图形化工具**

**VS Code**：内置合并界面，支持一键选择保留哪一方代码

**GitKraken/Sourcetree**：可视化解决冲突，适合新手



------

### **四、预防冲突的技巧**

1. 

   **频繁同步代码**：

   合并前先执行 `git pull`拉取远程最新代码。

   完成小功能后及时提交，减少单次修改量

   

2. 

   **分支管理**：

   每人独立开发时使用分支（如 `feature/login`），完成后合并到主分支。

   避免直接在 `main`分支上开发

   

3. 

   **代码规范**：

   

   统一代码风格（如缩进、命名），减少格式冲突。

   使用工具（如Prettier）自动格式化代码

   

------

### **五、常见问题处理**

**冲突未解决无法提交？**

运行 `git status`检查是否遗漏文件。

确保所有冲突文件已执行 `git add`

**误删文件或代码？**

使用 `git restore <文件名>`恢复未暂存的修改



**复杂冲突处理**：

若涉及多个文件或历史提交，可用 `git log --merge`查看冲突提交记录



------

### **总结**

解决Git冲突的核心是：**识别冲突文件 → 手动/工具合并 → 提交结果**。通过分支管理、频繁同步代码和规范开发流程，可以有效减少冲突发生。遇到问题时，善用Git命令和可视化工具能事半功倍！