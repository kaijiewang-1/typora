# 《日常开发 Git 配置 + 常用命令速查手册》

适用对象：新手开发者、自用备忘、日常开发高频场景。
 原则：**命令可复制、按场景查找、少解释、多实用**。

> 参考 Git 官方配置文档、GitHub 行尾处理文档与 Git 凭据文档整理。`git config` 支持全局、系统、仓库级配置；`core.autocrlf` / `.gitattributes` 是跨平台换行符管理的核心配置；Git credential helper 用于避免反复输入 HTTPS 凭据。

------

# 🧩 第一部分：git config 配置

------

## 🧠 1. 用户名 / 邮箱配置

### 命令 1：配置全局用户名

- 📌 命令说明
   设置 Git 提交记录中的作者名称。
- 🧾 命令

```
git config --global user.name "Your Name"
```

- 💡 使用场景
   新电脑第一次安装 Git 后必须配置。
- ⚠️ 注意事项
   这不是 GitHub / GitLab 登录名，只是提交作者信息。
- 🧪 示例

```
git config --global user.name "Alice Zhang"
```

### 命令 2：配置全局邮箱

- 📌 命令说明
   设置 Git 提交记录中的作者邮箱。
- 🧾 命令

```
git config --global user.email "your_email@example.com"
```

- 💡 使用场景
   关联 GitHub / GitLab 提交记录。
- ⚠️ 注意事项
   邮箱建议与代码托管平台账号邮箱一致，否则提交可能无法关联头像或贡献记录。
- 🧪 示例

```
git config --global user.email "alice@example.com"
```

### 命令 3：查看当前配置

- 📌 命令说明
   查看当前 Git 配置。
- 🧾 命令

```
git config --list
```

- 💡 使用场景
   检查用户名、邮箱、代理、alias 是否配置成功。
- ⚠️ 注意事项
   同一个配置可能出现多次，仓库级配置会覆盖全局配置。
- 🧪 示例

```
git config --list | grep user
```

### 总结表格

| 命令                                     | 作用       | 常用场景     | 备注                      |
| ---------------------------------------- | ---------- | ------------ | ------------------------- |
| `git config --global user.name "Name"`   | 设置用户名 | 新电脑初始化 | 影响提交作者              |
| `git config --global user.email "email"` | 设置邮箱   | 关联平台账号 | 建议与 GitHub/GitLab 一致 |
| `git config --list`                      | 查看配置   | 排查配置问题 | 仓库配置优先级更高        |

------

## 🧠 2. 默认分支配置

### 命令：设置默认分支为 main

- 📌 命令说明
   设置 `git init` 新仓库时默认创建的分支名。
- 🧾 命令

```
git config --global init.defaultBranch main
```

- 💡 使用场景
   新项目统一使用 `main` 作为主分支。
- ⚠️ 注意事项
   只影响以后新建的仓库，不会修改已有仓库分支名。
- 🧪 示例

```
git init
git branch
```

### 总结表格

| 命令                                          | 作用         | 常用场景     | 备注           |
| --------------------------------------------- | ------------ | ------------ | -------------- |
| `git config --global init.defaultBranch main` | 设置默认分支 | 新项目初始化 | 不影响已有仓库 |

------

## 🧠 3. 编辑器设置

### 命令 1：设置 VS Code 为默认编辑器

- 📌 命令说明
   设置 Git 打开提交信息、rebase 信息时使用 VS Code。
- 🧾 命令

```
git config --global core.editor "code --wait"
```

- 💡 使用场景
   执行 `git commit`、`git rebase -i` 时使用 VS Code 编辑。
- ⚠️ 注意事项
   需要提前安装 `code` 命令行工具。
- 🧪 示例

```
git commit
```

### 命令 2：设置 Vim 为默认编辑器

- 📌 命令说明
   设置 Git 使用 Vim 编辑信息。
- 🧾 命令

```
git config --global core.editor "vim"
```

- 💡 使用场景
   服务器、Linux 环境常用。
- ⚠️ 注意事项
   新手可能不熟悉 Vim 退出方式。
- 🧪 示例

```
git rebase -i HEAD~3
```

### 总结表格

| 命令                                            | 作用         | 常用场景   | 备注         |
| ----------------------------------------------- | ------------ | ---------- | ------------ |
| `git config --global core.editor "code --wait"` | 设置 VS Code | 本地开发   | 推荐新手     |
| `git config --global core.editor "vim"`         | 设置 Vim     | 服务器环境 | 适合终端用户 |

------

## 🧠 4. 换行符处理：Windows / macOS / Linux

跨平台开发中，Windows 使用 `CRLF`，macOS / Linux 通常使用 `LF`。GitHub 文档也建议通过 `core.autocrlf` 或 `.gitattributes` 管理行尾，避免“明明没改文件却出现大量 diff”。

### Windows 推荐配置

- 📌 命令说明
   检出时转为 `CRLF`，提交时转为 `LF`。
- 🧾 命令

```
git config --global core.autocrlf true
```

- 💡 使用场景
   Windows 日常开发，与 macOS / Linux 同事协作。
- ⚠️ 注意事项
   仓库中建议统一保存 `LF`。
- 🧪 示例

```
git config --global core.autocrlf true
```

### macOS / Linux 推荐配置

- 📌 命令说明
   提交时将 `CRLF` 转为 `LF`，检出时不转换。
- 🧾 命令

```
git config --global core.autocrlf input
```

- 💡 使用场景
   macOS / Linux 开发环境。
- ⚠️ 注意事项
   不会在检出时自动改成 `CRLF`。
- 🧪 示例

```
git config --global core.autocrlf input
```

### 更推荐：项目内使用 `.gitattributes`

- 📌 命令说明
   用仓库级规则统一换行符。
- 🧾 命令

```
cat > .gitattributes << 'EOF'
* text=auto

*.sh text eol=lf
*.js text eol=lf
*.ts text eol=lf
*.json text eol=lf

*.bat text eol=crlf
*.cmd text eol=crlf

*.png binary
*.jpg binary
*.jpeg binary
*.gif binary
*.pdf binary
EOF
```

- 💡 使用场景
   团队项目、跨平台项目、前端 / 后端混合项目。
- ⚠️ 注意事项
   `.gitattributes` 应提交到仓库，团队共享。
- 🧪 示例

```
git add .gitattributes
git commit -m "chore: add gitattributes"
```

### 总结表格

| 命令                                      | 作用                    | 常用场景         | 备注               |
| ----------------------------------------- | ----------------------- | ---------------- | ------------------ |
| `git config --global core.autocrlf true`  | Windows 自动转换换行符  | Windows 开发     | 检出 CRLF，提交 LF |
| `git config --global core.autocrlf input` | macOS/Linux 提交时转 LF | macOS/Linux 开发 | 推荐               |
| `.gitattributes`                          | 仓库级换行规则          | 团队协作         | 最稳定             |

------

## 🧠 5. 代理配置：HTTP / HTTPS / SOCKS

### 命令 1：配置 HTTP 代理

- 📌 命令说明
   设置 Git 访问 HTTP 地址时使用代理。
- 🧾 命令

```
git config --global http.proxy http://127.0.0.1:7890
```

- 💡 使用场景
   公司网络、科学上网、访问 GitHub 慢或失败。
- ⚠️ 注意事项
   端口要与你本地代理工具一致。
- 🧪 示例

```
git clone https://github.com/user/repo.git
```

### 命令 2：配置 HTTPS 代理

- 📌 命令说明
   设置 Git 访问 HTTPS 地址时使用代理。
- 🧾 命令

```
git config --global https.proxy http://127.0.0.1:7890
```

- 💡 使用场景
   HTTPS clone / fetch / pull / push 失败。
- ⚠️ 注意事项
   大多数远程仓库地址是 HTTPS，需要配置 `https.proxy`。
- 🧪 示例

```
git fetch origin
```

### 命令 3：配置 SOCKS5 代理

- 📌 命令说明
   使用 SOCKS5 代理访问远程仓库。
- 🧾 命令

```
git config --global http.proxy socks5://127.0.0.1:7890
git config --global https.proxy socks5://127.0.0.1:7890
```

- 💡 使用场景
   本地代理工具提供 SOCKS5 端口。
- ⚠️ 注意事项
   确认代理协议是 `socks5://` 还是 `http://`。
- 🧪 示例

```
git ls-remote https://github.com/user/repo.git
```

### 命令 4：取消代理

- 📌 命令说明
   删除全局代理配置。
- 🧾 命令

```
git config --global --unset http.proxy
git config --global --unset https.proxy
```

- 💡 使用场景
   代理关闭后 Git 无法访问远程仓库。
- ⚠️ 注意事项
   如果配置了多个代理项，可能需要检查 `git config --list`。
- 🧪 示例

```
git config --list | grep proxy
```

### 操作系统差异

| 系统          | 常见代理地址                                       |
| ------------- | -------------------------------------------------- |
| Windows       | `http://127.0.0.1:7890`、`socks5://127.0.0.1:7890` |
| macOS / Linux | `http://127.0.0.1:7890`、`socks5://127.0.0.1:7890` |
| WSL           | 可能需要使用 Windows 宿主机 IP，而不是 `127.0.0.1` |

### 总结表格

| 命令                                      | 作用            | 常用场景        | 备注       |
| ----------------------------------------- | --------------- | --------------- | ---------- |
| `git config --global http.proxy ...`      | 配置 HTTP 代理  | clone 慢        | 看代理端口 |
| `git config --global https.proxy ...`     | 配置 HTTPS 代理 | GitHub 访问失败 | 高频       |
| `git config --global --unset http.proxy`  | 删除 HTTP 代理  | 代理失效        | 排查常用   |
| `git config --global --unset https.proxy` | 删除 HTTPS 代理 | push/fetch 异常 | 排查常用   |

------

## 🧠 6. 凭据管理 credential

Git 访问 HTTPS 仓库时可能要求用户名和密码；现代代码托管平台通常使用 Personal Access Token 或 OAuth token 作为密码。Git credential helper 可以缓存或保存凭据，减少反复输入。

### Windows：Git Credential Manager

- 📌 命令说明
   使用 Git Credential Manager 管理凭据。
- 🧾 命令

```
git config --global credential.helper manager
```

- 💡 使用场景
   Windows 下 HTTPS 拉取 / 推送 GitHub、GitLab 仓库。
- ⚠️ 注意事项
   如果凭据错误，可能需要到 Windows 凭据管理器中删除旧凭据。
- 🧪 示例

```
git pull
```

### macOS：osxkeychain

- 📌 命令说明
   使用 macOS 钥匙串保存 Git 凭据。
- 🧾 命令

```
git config --global credential.helper osxkeychain
```

- 💡 使用场景
   macOS 下避免反复输入用户名和 token。
- ⚠️ 注意事项
   如果 token 失效，需要从钥匙串中删除旧记录。
- 🧪 示例

```
git push origin main
```

### Linux：cache

- 📌 命令说明
   临时缓存 Git 凭据。
- 🧾 命令

```
git config --global credential.helper cache
```

- 💡 使用场景
   Linux 服务器上短时间避免重复输入。
- ⚠️ 注意事项
   默认缓存时间有限。
- 🧪 示例

```
git fetch
```

### Linux：store

- 📌 命令说明
   明文保存 Git 凭据。
- 🧾 命令

```
git config --global credential.helper store
```

- 💡 使用场景
   个人本地机器，且能接受明文保存风险。
- ⚠️ 注意事项
   不推荐在共享服务器上使用。
- 🧪 示例

```
git pull
```

### 总结表格

| 命令                            | 作用             | 常用场景 | 备注     |
| ------------------------------- | ---------------- | -------- | -------- |
| `credential.helper manager`     | Windows 凭据管理 | Windows  | 推荐     |
| `credential.helper osxkeychain` | macOS 钥匙串     | macOS    | 推荐     |
| `credential.helper cache`       | 临时缓存         | Linux    | 相对安全 |
| `credential.helper store`       | 明文保存         | 个人机器 | 谨慎使用 |

------

## 🧠 7. SSH vs HTTPS 使用说明

### HTTPS 远程地址

- 📌 命令说明
   使用 HTTPS 协议访问远程仓库。
- 🧾 命令

```
git clone https://github.com/user/repo.git
```

- 💡 使用场景
   新手、临时机器、公司网络只允许 HTTPS。
- ⚠️ 注意事项
   push 时可能需要 token 或 credential helper。
- 🧪 示例

```
git remote -v
```

### SSH 远程地址

- 📌 命令说明
   使用 SSH key 访问远程仓库。
- 🧾 命令

```
git clone git@github.com:user/repo.git
```

- 💡 使用场景
   长期开发机器、频繁 push / pull。
- ⚠️ 注意事项
   需要先配置 SSH key，并添加到 GitHub / GitLab。
- 🧪 示例

```
ssh -T git@github.com
```

### 切换远程地址

- 📌 命令说明
   修改已有仓库的远程地址。
- 🧾 命令

```
git remote set-url origin git@github.com:user/repo.git
```

- 💡 使用场景
   从 HTTPS 切换到 SSH。
- ⚠️ 注意事项
   切换后需要确认 SSH key 可用。
- 🧪 示例

```
git remote -v
```

### 总结表格

| 命令                            | 作用         | 常用场景         | 备注         |
| ------------------------------- | ------------ | ---------------- | ------------ |
| `git clone https://...`         | HTTPS 克隆   | 新手 / 临时环境  | 可能要 token |
| `git clone git@...`             | SSH 克隆     | 长期开发         | 推荐         |
| `git remote set-url origin ...` | 修改远程地址 | HTTPS / SSH 切换 | 常用         |

------

## 🧠 8. Git alias 别名

### 推荐 alias 组合

- 📌 命令说明
   为高频命令配置短别名。
- 🧾 命令

```
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.sw switch
git config --global alias.br branch
git config --global alias.cm commit
git config --global alias.cma "commit --amend"
git config --global alias.df diff
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "restore --staged"
git config --global alias.undo "reset --soft HEAD~1"
```

- 💡 使用场景
   高频开发操作提效。
- ⚠️ 注意事项
   团队脚本里不建议依赖个人 alias。
- 🧪 示例

```
git st
git lg
git undo
```

### 总结表格

| 命令          | 作用             | 常用场景   | 备注       |
| ------------- | ---------------- | ---------- | ---------- |
| `git st`      | 查看状态         | 高频       | `status`   |
| `git lg`      | 图形化日志       | 查分支     | 很实用     |
| `git undo`    | 撤销最近一次提交 | 本地误提交 | soft reset |
| `git unstage` | 取消暂存         | add 错文件 | 安全       |

------

## 🧠 9. 全局忽略文件 `.gitignore_global`

### 命令：配置全局 ignore

- 📌 命令说明
   忽略所有仓库中都不想提交的文件。
- 🧾 命令

```
git config --global core.excludesfile ~/.gitignore_global
```

- 💡 使用场景
   忽略系统文件、编辑器文件、临时文件。
- ⚠️ 注意事项
   项目相关忽略规则应写在项目 `.gitignore` 中。
- 🧪 示例

```
cat > ~/.gitignore_global << 'EOF'
.DS_Store
Thumbs.db
.idea/
.vscode/
*.log
*.tmp
EOF
```

### Windows 示例

```
git config --global core.excludesfile "%USERPROFILE%\.gitignore_global"
```

### macOS / Linux 示例

```
git config --global core.excludesfile ~/.gitignore_global
```

### 总结表格

| 命令                | 作用             | 常用场景        | 备注       |
| ------------------- | ---------------- | --------------- | ---------- |
| `core.excludesfile` | 设置全局忽略文件 | 系统 / IDE 文件 | 个人配置   |
| `.gitignore`        | 项目忽略规则     | 团队共享        | 应提交仓库 |

------

## 🧠 10. 大小写敏感：core.ignorecase

### 命令：配置大小写敏感

- 📌 命令说明
   控制 Git 是否忽略文件名大小写变化。
- 🧾 命令

```
git config core.ignorecase false
```

- 💡 使用场景
   需要识别 `User.js` 改名为 `user.js`。
- ⚠️ 注意事项
   Windows / macOS 默认文件系统常常大小写不敏感，Linux 通常大小写敏感。跨平台项目要格外注意。
- 🧪 示例

```
git mv User.js user.js
```

如果 Git 没识别大小写改名，可以分两步：

```
git mv User.js User_temp.js
git mv User_temp.js user.js
```

### 总结表格

| 命令                               | 作用           | 常用场景   | 备注         |
| ---------------------------------- | -------------- | ---------- | ------------ |
| `git config core.ignorecase false` | 不忽略大小写   | 文件重命名 | 仓库级更合适 |
| `git mv old new`                   | Git 识别重命名 | 大小写改名 | 推荐使用     |

------

## ⚡ 新电脑 Git 一键初始化配置

### Windows 推荐

```
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
git config --global init.defaultBranch main
git config --global core.autocrlf true
git config --global core.editor "code --wait"
git config --global credential.helper manager

git config --global alias.st status
git config --global alias.sw switch
git config --global alias.br branch
git config --global alias.cm commit
git config --global alias.df diff
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.unstage "restore --staged"
git config --global alias.undo "reset --soft HEAD~1"
```

### macOS 推荐

```
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
git config --global init.defaultBranch main
git config --global core.autocrlf input
git config --global core.editor "code --wait"
git config --global credential.helper osxkeychain

git config --global alias.st status
git config --global alias.sw switch
git config --global alias.br branch
git config --global alias.cm commit
git config --global alias.df diff
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.unstage "restore --staged"
git config --global alias.undo "reset --soft HEAD~1"
```

### Linux 推荐

```
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"
git config --global init.defaultBranch main
git config --global core.autocrlf input
git config --global core.editor "vim"
git config --global credential.helper cache

git config --global alias.st status
git config --global alias.sw switch
git config --global alias.br branch
git config --global alias.cm commit
git config --global alias.df diff
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.unstage "restore --staged"
git config --global alias.undo "reset --soft HEAD~1"
```

------

# 🚀 第二部分：Git 常用命令

------

## 🧠 📁 仓库初始化与克隆

### 命令 1：初始化仓库

- 📌 命令说明
   在当前目录创建 Git 仓库。
- 🧾 命令

```
git init
```

- 💡 使用场景
   本地新项目开始使用 Git 管理。
- ⚠️ 注意事项
   会创建隐藏目录 `.git`。
- 🧪 示例

```
mkdir my-project
cd my-project
git init
```

### 命令 2：克隆远程仓库

- 📌 命令说明
   从远程地址下载完整仓库。
- 🧾 命令

```
git clone <repo-url>
```

- 💡 使用场景
   加入已有项目。
- ⚠️ 注意事项
   HTTPS 可能需要 token，SSH 需要配置 SSH key。
- 🧪 示例

```
git clone git@github.com:user/repo.git
```

### 命令 3：克隆指定分支

- 📌 命令说明
   只克隆指定分支。
- 🧾 命令

```
git clone -b <branch-name> <repo-url>
```

- 💡 使用场景
   只需要某个开发分支。
- ⚠️ 注意事项
   分支名必须存在。
- 🧪 示例

```
git clone -b develop git@github.com:user/repo.git
```

### 总结表格

| 命令                          | 作用         | 常用场景   | 备注            |
| ----------------------------- | ------------ | ---------- | --------------- |
| `git init`                    | 初始化仓库   | 新项目     | 本地创建 `.git` |
| `git clone <url>`             | 克隆仓库     | 加入项目   | 高频            |
| `git clone -b <branch> <url>` | 克隆指定分支 | 只拉某分支 | 分支需存在      |

------

## 🧠 📌 文件操作：add / status / diff

### 命令 1：查看文件状态

- 📌 命令说明
   查看哪些文件被修改、新增、删除。
- 🧾 命令

```
git status
```

- 💡 使用场景
   提交前检查工作区。
- ⚠️ 注意事项
   这是最常用的安全检查命令。
- 🧪 示例

```
git status
```

### 命令 2：添加文件到暂存区

- 📌 命令说明
   将文件加入下一次提交。
- 🧾 命令

```
git add <file>
```

- 💡 使用场景
   准备提交某个文件。
- ⚠️ 注意事项
   `git add .` 会添加当前目录下所有改动，提交前要检查。
- 🧪 示例

```
git add src/main.js
```

### 命令 3：添加所有改动

- 📌 命令说明
   添加当前目录所有新增、修改、删除。
- 🧾 命令

```
git add .
```

- 💡 使用场景
   一次性提交当前改动。
- ⚠️ 注意事项
   小心误提交日志、临时文件、密钥文件。
- 🧪 示例

```
git add .
git status
```

### 命令 4：查看未暂存 diff

- 📌 命令说明
   查看工作区与暂存区之间的差异。
- 🧾 命令

```
git diff
```

- 💡 使用场景
   `git add` 前检查修改内容。
- ⚠️ 注意事项
   已经 `git add` 的内容不会显示在这里。
- 🧪 示例

```
git diff src/main.js
```

### 命令 5：查看已暂存 diff

- 📌 命令说明
   查看暂存区与上一次提交之间的差异。
- 🧾 命令

```
git diff --staged
```

- 💡 使用场景
   `git commit` 前最终检查。
- ⚠️ 注意事项
   也可使用 `git diff --cached`。
- 🧪 示例

```
git diff --staged
```

### 总结表格

| 命令                | 作用           | 常用场景   | 备注     |
| ------------------- | -------------- | ---------- | -------- |
| `git status`        | 查看状态       | 提交前检查 | 高频     |
| `git add <file>`    | 暂存指定文件   | 精确提交   | 推荐     |
| `git add .`         | 暂存全部       | 快速提交   | 小心误加 |
| `git diff`          | 查看未暂存差异 | add 前     | 高频     |
| `git diff --staged` | 查看已暂存差异 | commit 前  | 推荐     |

------

## 🧠 💾 提交相关：commit / amend

### 命令 1：提交代码

- 📌 命令说明
   将暂存区内容提交到本地仓库。
- 🧾 命令

```
git commit -m "commit message"
```

- 💡 使用场景
   完成一个功能、修复一个 bug 后提交。
- ⚠️ 注意事项
   提交信息应清晰描述改动。
- 🧪 示例

```
git commit -m "feat: add login page"
```

### 命令 2：修改最近一次提交信息

- 📌 命令说明
   修改最近一次 commit 的提交信息。
- 🧾 命令

```
git commit --amend -m "new commit message"
```

- 💡 使用场景
   提交信息写错。
- ⚠️ 注意事项
   如果该提交已经 push，谨慎 amend，可能需要 force push。
- 🧪 示例

```
git commit --amend -m "fix: correct login validation"
```

### 命令 3：把新改动合并进最近一次提交

- 📌 命令说明
   将当前暂存区内容合并到上一次 commit。
- 🧾 命令

```
git add .
git commit --amend --no-edit
```

- 💡 使用场景
   刚提交完发现漏了一个文件。
- ⚠️ 注意事项
   已 push 的提交不建议随意 amend。
- 🧪 示例

```
git add README.md
git commit --amend --no-edit
```

### 总结表格

| 命令                           | 作用             | 常用场景   | 备注         |
| ------------------------------ | ---------------- | ---------- | ------------ |
| `git commit -m "msg"`          | 提交             | 日常开发   | 高频         |
| `git commit --amend -m "msg"`  | 修改最近提交信息 | 信息写错   | 已 push 慎用 |
| `git commit --amend --no-edit` | 合并到最近提交   | 漏提交文件 | 已 push 慎用 |

------

## 🧠 🌿 分支管理：branch / checkout / switch

### 命令 1：查看分支

- 📌 命令说明
   查看本地分支。
- 🧾 命令

```
git branch
```

- 💡 使用场景
   查看当前在哪个分支。
- ⚠️ 注意事项
   当前分支前会显示 `*`。
- 🧪 示例

```
git branch
```

### 命令 2：查看所有分支

- 📌 命令说明
   查看本地和远程分支。
- 🧾 命令

```
git branch -a
```

- 💡 使用场景
   查找远程分支名称。
- ⚠️ 注意事项
   远程分支通常显示为 `remotes/origin/xxx`。
- 🧪 示例

```
git branch -a
```

### 命令 3：创建分支

- 📌 命令说明
   创建新分支，但不切换。
- 🧾 命令

```
git branch <branch-name>
```

- 💡 使用场景
   准备新功能分支。
- ⚠️ 注意事项
   新分支基于当前 HEAD 创建。
- 🧪 示例

```
git branch feature/login
```

### 命令 4：切换分支

- 📌 命令说明
   切换到已有分支。
- 🧾 命令

```
git switch <branch-name>
```

- 💡 使用场景
   在功能分支、主分支之间切换。
- ⚠️ 注意事项
   若当前有未提交改动，可能阻止切换。
- 🧪 示例

```
git switch develop
```

### 命令 5：创建并切换分支

- 📌 命令说明
   创建新分支并立即切换。
- 🧾 命令

```
git switch -c <branch-name>
```

- 💡 使用场景
   开始新功能开发。
- ⚠️ 注意事项
   推荐替代旧命令 `git checkout -b`。
- 🧪 示例

```
git switch -c feature/login
```

### 命令 6：删除本地分支

- 📌 命令说明
   删除已经合并的本地分支。
- 🧾 命令

```
git branch -d <branch-name>
```

- 💡 使用场景
   功能合并后清理本地分支。
- ⚠️ 注意事项
   未合并分支删除会失败；强删用 `-D`。
- 🧪 示例

```
git branch -d feature/login
```

### 总结表格

| 命令                   | 作用         | 常用场景   | 备注   |
| ---------------------- | ------------ | ---------- | ------ |
| `git branch`           | 查看本地分支 | 日常检查   | 高频   |
| `git branch -a`        | 查看所有分支 | 找远程分支 | 高频   |
| `git branch <name>`    | 创建分支     | 新功能     | 不切换 |
| `git switch <name>`    | 切换分支     | 日常开发   | 推荐   |
| `git switch -c <name>` | 创建并切换   | 新功能     | 高频   |
| `git branch -d <name>` | 删除分支     | 清理       | 已合并 |

------

## 🧠 🔀 合并与变基：merge / rebase

### 命令 1：合并分支

- 📌 命令说明
   将目标分支合并到当前分支。
- 🧾 命令

```
git merge <branch-name>
```

- 💡 使用场景
   将功能分支合并到主分支，或同步 develop。
- ⚠️ 注意事项
   可能产生 merge commit。
- 🧪 示例

```
git switch main
git merge feature/login
```

### 命令 2：变基

- 📌 命令说明
   将当前分支的提交移动到目标分支最新提交之后。
- 🧾 命令

```
git rebase <branch-name>
```

- 💡 使用场景
   让功能分支基于最新主分支，保持提交历史线性。
- ⚠️ 注意事项
   不要随意 rebase 已经共享给他人的分支。
- 🧪 示例

```
git switch feature/login
git fetch origin
git rebase origin/main
```

### 命令 3：解决冲突后继续 rebase

- 📌 命令说明
   解决冲突后继续变基流程。
- 🧾 命令

```
git add .
git rebase --continue
```

- 💡 使用场景
   rebase 中出现冲突。
- ⚠️ 注意事项
   冲突文件必须处理完并 add。
- 🧪 示例

```
git status
git add src/login.js
git rebase --continue
```

### 命令 4：放弃 rebase

- 📌 命令说明
   中止 rebase，回到 rebase 前状态。
- 🧾 命令

```
git rebase --abort
```

- 💡 使用场景
   rebase 冲突太多，不想继续。
- ⚠️ 注意事项
   这是救命命令。
- 🧪 示例

```
git rebase --abort
```

### 总结表格

| 命令                    | 作用        | 常用场景   | 备注                  |
| ----------------------- | ----------- | ---------- | --------------------- |
| `git merge <branch>`    | 合并分支    | 合并功能   | 可能产生 merge commit |
| `git rebase <branch>`   | 变基        | 整理历史   | 公共分支慎用          |
| `git rebase --continue` | 继续 rebase | 解决冲突后 | 必须先 add            |
| `git rebase --abort`    | 中止 rebase | 冲突太多   | 救命命令              |

------

## 🧠 🔙 回退操作：reset / revert / checkout 文件

### 命令 1：取消暂存

- 📌 命令说明
   将文件从暂存区移回工作区。
- 🧾 命令

```
git restore --staged <file>
```

- 💡 使用场景
   `git add` 错文件。
- ⚠️ 注意事项
   不会丢失文件修改。
- 🧪 示例

```
git restore --staged config.yml
```

### 命令 2：丢弃工作区修改

- 📌 命令说明
   放弃某个文件的本地修改。
- 🧾 命令

```
git restore <file>
```

- 💡 使用场景
   文件改乱了，想恢复到最近一次提交。
- ⚠️ 注意事项
   会丢失未提交修改，谨慎使用。
- 🧪 示例

```
git restore src/main.js
```

### 命令 3：软回退最近一次提交

- 📌 命令说明
   撤销最近一次 commit，但保留改动在暂存区。
- 🧾 命令

```
git reset --soft HEAD~1
```

- 💡 使用场景
   提交信息错了，或想重新整理提交。
- ⚠️ 注意事项
   不丢代码。
- 🧪 示例

```
git reset --soft HEAD~1
git commit -m "new message"
```

### 命令 4：混合回退最近一次提交

- 📌 命令说明
   撤销最近一次 commit，保留改动在工作区。
- 🧾 命令

```
git reset --mixed HEAD~1
```

- 💡 使用场景
   想重新选择要 add 的文件。
- ⚠️ 注意事项
   默认 `git reset HEAD~1` 等同于 mixed。
- 🧪 示例

```
git reset HEAD~1
git status
```

### 命令 5：硬回退

- 📌 命令说明
   回退到指定提交，并丢弃之后的所有改动。
- 🧾 命令

```
git reset --hard <commit-id>
```

- 💡 使用场景
   本地分支彻底回到某个版本。
- ⚠️ 注意事项
   危险命令，会丢失未保存改动。
- 🧪 示例

```
git reset --hard HEAD~1
```

### 命令 6：安全撤销某次提交

- 📌 命令说明
   创建一个新的提交，用来反向撤销指定 commit。
- 🧾 命令

```
git revert <commit-id>
```

- 💡 使用场景
   已经 push 的提交需要撤销。
- ⚠️ 注意事项
   比 reset 更适合公共分支。
- 🧪 示例

```
git revert a1b2c3d
```

### 总结表格

| 命令                          | 作用               | 常用场景     | 备注         |
| ----------------------------- | ------------------ | ------------ | ------------ |
| `git restore --staged <file>` | 取消暂存           | add 错文件   | 安全         |
| `git restore <file>`          | 丢弃修改           | 文件改乱     | 会丢改动     |
| `git reset --soft HEAD~1`     | 撤销提交保留暂存   | 改提交信息   | 安全         |
| `git reset --mixed HEAD~1`    | 撤销提交保留工作区 | 重选文件     | 常用         |
| `git reset --hard <id>`       | 硬回退             | 本地彻底回退 | 危险         |
| `git revert <id>`             | 反向撤销提交       | 已 push      | 推荐公共分支 |

------

## 🧠 🔄 同步远程：fetch / pull / push

### 命令 1：拉取远程信息但不合并

- 📌 命令说明
   获取远程最新分支和提交，不修改当前工作区。
- 🧾 命令

```
git fetch origin
```

- 💡 使用场景
   想先看看远程有什么变化。
- ⚠️ 注意事项
   比 `pull` 更安全。
- 🧪 示例

```
git fetch origin
git log --oneline --graph --decorate --all
```

### 命令 2：拉取并合并

- 📌 命令说明
   从远程拉取并合并到当前分支。
- 🧾 命令

```
git pull
```

- 💡 使用场景
   同步当前分支最新代码。
- ⚠️ 注意事项
   等价于 `fetch + merge`，可能产生冲突。
- 🧪 示例

```
git pull origin main
```

### 命令 3：拉取并 rebase

- 📌 命令说明
   从远程拉取后用 rebase 整理本地提交。
- 🧾 命令

```
git pull --rebase
```

- 💡 使用场景
   希望提交历史更线性。
- ⚠️ 注意事项
   冲突时需要解决后 `git rebase --continue`。
- 🧪 示例

```
git pull --rebase origin main
```

### 命令 4：推送当前分支

- 📌 命令说明
   将本地提交推送到远程仓库。
- 🧾 命令

```
git push
```

- 💡 使用场景
   本地提交完成后同步远程。
- ⚠️ 注意事项
   远程有新提交时可能被拒绝。
- 🧪 示例

```
git push origin main
```

### 命令 5：首次推送并关联远程分支

- 📌 命令说明
   推送本地分支，并设置 upstream。
- 🧾 命令

```
git push -u origin <branch-name>
```

- 💡 使用场景
   新建功能分支第一次 push。
- ⚠️ 注意事项
   后续可直接使用 `git push`。
- 🧪 示例

```
git push -u origin feature/login
```

### 命令 6：安全强推

- 📌 命令说明
   强制推送，但会检查远程是否被别人更新。
- 🧾 命令

```
git push --force-with-lease
```

- 💡 使用场景
   rebase 或 amend 后需要更新远程分支。
- ⚠️ 注意事项
   比 `--force` 安全，但仍需谨慎。
- 🧪 示例

```
git push --force-with-lease origin feature/login
```

### 总结表格

| 命令                          | 作用         | 常用场景        | 备注          |
| ----------------------------- | ------------ | --------------- | ------------- |
| `git fetch origin`            | 获取远程更新 | 安全同步        | 不改工作区    |
| `git pull`                    | 拉取并合并   | 日常同步        | 可能冲突      |
| `git pull --rebase`           | 拉取并变基   | 线性历史        | 推荐功能分支  |
| `git push`                    | 推送         | 提交后          | 高频          |
| `git push -u origin <branch>` | 首次推送     | 新分支          | 设置 upstream |
| `git push --force-with-lease` | 安全强推     | rebase/amend 后 | 慎用          |

------

## 🧠 🏷 标签管理：tag

### 命令 1：查看标签

- 📌 命令说明
   查看当前仓库所有 tag。
- 🧾 命令

```
git tag
```

- 💡 使用场景
   查看版本号。
- ⚠️ 注意事项
   默认按字典顺序显示。
- 🧪 示例

```
git tag
```

### 命令 2：创建轻量标签

- 📌 命令说明
   给当前提交打标签。
- 🧾 命令

```
git tag <tag-name>
```

- 💡 使用场景
   本地简单标记版本。
- ⚠️ 注意事项
   不包含额外说明信息。
- 🧪 示例

```
git tag v1.0.0
```

### 命令 3：创建附注标签

- 📌 命令说明
   创建带说明信息的 tag。
- 🧾 命令

```
git tag -a <tag-name> -m "message"
```

- 💡 使用场景
   正式版本发布。
- ⚠️ 注意事项
   推荐发布版本使用附注标签。
- 🧪 示例

```
git tag -a v1.0.0 -m "release v1.0.0"
```

### 命令 4：推送标签

- 📌 命令说明
   将 tag 推送到远程。
- 🧾 命令

```
git push origin <tag-name>
```

- 💡 使用场景
   发布版本。
- ⚠️ 注意事项
   创建 tag 后不会自动 push。
- 🧪 示例

```
git push origin v1.0.0
```

### 命令 5：删除标签

- 📌 命令说明
   删除本地标签。
- 🧾 命令

```
git tag -d <tag-name>
```

- 💡 使用场景
   本地 tag 打错。
- ⚠️ 注意事项
   如果远程也有，需要单独删除远程 tag。
- 🧪 示例

```
git tag -d v1.0.0
```

### 命令 6：删除远程标签

- 📌 命令说明
   删除远程仓库中的标签。
- 🧾 命令

```
git push origin --delete <tag-name>
```

- 💡 使用场景
   远程 tag 打错。
- ⚠️ 注意事项
   删除发布 tag 前应确认团队是否依赖该版本。
- 🧪 示例

```
git push origin --delete v1.0.0
```

### 总结表格

| 命令                             | 作用         | 常用场景 | 备注              |
| -------------------------------- | ------------ | -------- | ----------------- |
| `git tag`                        | 查看标签     | 查版本   | 高频              |
| `git tag <name>`                 | 创建轻量标签 | 本地标记 | 简单              |
| `git tag -a <name> -m "msg"`     | 创建附注标签 | 发布版本 | 推荐              |
| `git push origin <tag>`          | 推送标签     | 发布     | tag 不会自动 push |
| `git tag -d <tag>`               | 删除本地标签 | 打错 tag | 本地              |
| `git push origin --delete <tag>` | 删除远程标签 | 远程打错 | 慎用              |

------

## 🧠 🔍 日志查看：log / reflog

### 命令 1：查看提交日志

- 📌 命令说明
   查看当前分支提交历史。
- 🧾 命令

```
git log
```

- 💡 使用场景
   查看提交记录。
- ⚠️ 注意事项
   按 `q` 退出。
- 🧪 示例

```
git log
```

### 命令 2：单行日志

- 📌 命令说明
   简洁查看提交记录。
- 🧾 命令

```
git log --oneline
```

- 💡 使用场景
   快速找 commit id。
- ⚠️ 注意事项
   显示的是短 commit id。
- 🧪 示例

```
git log --oneline -5
```

### 命令 3：图形化日志

- 📌 命令说明
   以图形方式查看分支关系。
- 🧾 命令

```
git log --oneline --graph --decorate --all
```

- 💡 使用场景
   分支混乱、排查 merge / rebase。
- ⚠️ 注意事项
   推荐配置成 alias：`git lg`。
- 🧪 示例

```
git log --oneline --graph --decorate --all
```

### 命令 4：查看 reflog

- 📌 命令说明
   查看 HEAD 的移动记录。
- 🧾 命令

```
git reflog
```

- 💡 使用场景
   reset 错了、rebase 乱了、提交找不到了。
- ⚠️ 注意事项
   Git 救援神器。
- 🧪 示例

```
git reflog
git reset --hard HEAD@{1}
```

### 总结表格

| 命令                                         | 作用           | 常用场景     | 备注       |
| -------------------------------------------- | -------------- | ------------ | ---------- |
| `git log`                                    | 查看日志       | 查历史       | 按 q 退出  |
| `git log --oneline`                          | 简洁日志       | 找 commit id | 高频       |
| `git log --graph --oneline --decorate --all` | 图形日志       | 查分支       | 推荐 alias |
| `git reflog`                                 | 查看 HEAD 历史 | 救援         | 非常重要   |

------

## 🧠 🚑 紧急救援：stash / cherry-pick

### 命令 1：临时保存当前改动

- 📌 命令说明
   将当前未提交改动临时保存起来。
- 🧾 命令

```
git stash
```

- 💡 使用场景
   当前代码没写完，但需要切分支。
- ⚠️ 注意事项
   默认不保存未跟踪文件。
- 🧪 示例

```
git stash
git switch main
```

### 命令 2：保存改动并添加说明

- 📌 命令说明
   stash 时附带备注。
- 🧾 命令

```
git stash push -m "message"
```

- 💡 使用场景
   有多个 stash，需要区分。
- ⚠️ 注意事项
   推荐带说明。
- 🧪 示例

```
git stash push -m "login page half done"
```

### 命令 3：查看 stash 列表

- 📌 命令说明
   查看所有临时保存记录。
- 🧾 命令

```
git stash list
```

- 💡 使用场景
   找回之前保存的改动。
- ⚠️ 注意事项
   编号如 `stash@{0}`。
- 🧪 示例

```
git stash list
```

### 命令 4：恢复最近一次 stash

- 📌 命令说明
   恢复最近一次 stash，并从 stash 列表删除。
- 🧾 命令

```
git stash pop
```

- 💡 使用场景
   回到原分支后继续开发。
- ⚠️ 注意事项
   可能产生冲突。
- 🧪 示例

```
git switch feature/login
git stash pop
```

### 命令 5：应用 stash 但不删除

- 📌 命令说明
   恢复 stash 内容，但保留 stash 记录。
- 🧾 命令

```
git stash apply
```

- 💡 使用场景
   想在多个分支应用同一份改动。
- ⚠️ 注意事项
   不会自动删除 stash。
- 🧪 示例

```
git stash apply stash@{1}
```

### 命令 6：挑选某个提交

- 📌 命令说明
   将指定 commit 应用到当前分支。
- 🧾 命令

```
git cherry-pick <commit-id>
```

- 💡 使用场景
   只想把某个 bugfix 拿到当前分支。
- ⚠️ 注意事项
   可能产生冲突。
- 🧪 示例

```
git cherry-pick a1b2c3d
```

### 命令 7：中止 cherry-pick

- 📌 命令说明
   放弃当前 cherry-pick 操作。
- 🧾 命令

```
git cherry-pick --abort
```

- 💡 使用场景
   cherry-pick 冲突太多，不想继续。
- ⚠️ 注意事项
   会回到 cherry-pick 前状态。
- 🧪 示例

```
git cherry-pick --abort
```

### 总结表格

| 命令                      | 作用             | 常用场景    | 备注               |
| ------------------------- | ---------------- | ----------- | ------------------ |
| `git stash`               | 临时保存改动     | 切分支前    | 默认不含未跟踪文件 |
| `git stash push -m "msg"` | 带说明 stash     | 多个 stash  | 推荐               |
| `git stash list`          | 查看 stash       | 找临时改动  | 高频               |
| `git stash pop`           | 恢复并删除 stash | 继续开发    | 可能冲突           |
| `git stash apply`         | 恢复但保留 stash | 多分支应用  | 不删除             |
| `git cherry-pick <id>`    | 挑选提交         | 复制 bugfix | 可能冲突           |
| `git cherry-pick --abort` | 中止 cherry-pick | 冲突太多    | 救命               |

------

# 📌 高频命令组合：实际工作流

## 1. 新功能开发

```
git switch main
git pull --rebase
git switch -c feature/login

# 开发中
git status
git add .
git commit -m "feat: add login page"

git push -u origin feature/login
```

## 2. 提交前检查

```
git status
git diff
git add .
git diff --staged
git commit -m "fix: correct user validation"
```

## 3. 同步主分支最新代码

```
git fetch origin
git switch feature/login
git rebase origin/main
```

## 4. 刚提交完发现漏文件

```
git add missing-file.js
git commit --amend --no-edit
```

## 5. 撤销最近一次本地提交但保留代码

```
git reset --soft HEAD~1
```

## 6. 已 push 的提交需要撤销

```
git revert <commit-id>
git push
```

## 7. 分支乱了先看图

```
git log --oneline --graph --decorate --all
```

## 8. reset 错了找回来

```
git reflog
git reset --hard HEAD@{1}
```

------

# 🚨 常见问题与坑

------

## 1. 用户名 / 邮箱错误

- ❗问题描述
   提交记录显示成错误用户名或邮箱，GitHub / GitLab 不显示头像或贡献记录。
- 🔍原因分析
   `user.name` / `user.email` 配置错误，或者当前仓库设置覆盖了全局配置。
- ✅解决方案

```
git config --global user.name "Your Name"
git config --global user.email "your_email@example.com"

git config user.name
git config user.email
```

如果只想修改当前仓库：

```
git config user.name "Your Name"
git config user.email "your_email@example.com"
```

修改最近一次提交作者：

```
git commit --amend --reset-author
```

------

## 2. 换行符混乱

- ❗问题描述
   没改代码却出现大量 diff，或者 Windows / macOS / Linux 协作时文件反复变化。
- 🔍原因分析
   Windows 使用 `CRLF`，macOS / Linux 使用 `LF`，团队未统一换行符策略。
- ✅解决方案

Windows：

```
git config --global core.autocrlf true
```

macOS / Linux：

```
git config --global core.autocrlf input
```

项目内添加 `.gitattributes`：

```
cat > .gitattributes << 'EOF'
* text=auto
*.sh text eol=lf
*.js text eol=lf
*.ts text eol=lf
*.json text eol=lf
*.bat text eol=crlf
*.cmd text eol=crlf
*.png binary
*.jpg binary
EOF
```

------

## 3. 提交后想撤销

- ❗问题描述
   commit 后发现提交错了、漏文件了、提交信息写错了。
- 🔍原因分析
   提交已经进入本地历史，但还可以根据是否 push 选择处理方式。
- ✅解决方案

未 push，撤销提交但保留代码：

```
git reset --soft HEAD~1
```

未 push，修改最近一次提交信息：

```
git commit --amend -m "new message"
```

已 push，推荐安全撤销：

```
git revert <commit-id>
git push
```

------

## 4. push 被拒绝

- ❗问题描述
   执行 `git push` 时提示 rejected、non-fast-forward。
- 🔍原因分析
   远程分支有别人提交的新内容，本地不是最新。
- ✅解决方案

推荐：

```
git pull --rebase
git push
```

如果有冲突：

```
git status
# 手动解决冲突
git add .
git rebase --continue
git push
```

如果是 amend / rebase 后需要更新自己的远程功能分支：

```
git push --force-with-lease
```

------

## 5. 分支混乱

- ❗问题描述
   不知道自己在哪个分支，或者不知道某个提交在哪里。
- 🔍原因分析
   多分支开发、merge / rebase 后历史变复杂。
- ✅解决方案

查看当前分支：

```
git branch
```

查看所有分支：

```
git branch -a
```

查看图形日志：

```
git log --oneline --graph --decorate --all
```

查看最近 HEAD 移动记录：

```
git reflog
```

------

## 6. 代理导致无法 clone

- ❗问题描述
   `git clone` 卡住、超时、无法连接 GitHub / GitLab。
- 🔍原因分析
   代理未开、端口错误、Git 代理配置残留。
- ✅解决方案

查看代理：

```
git config --global --get http.proxy
git config --global --get https.proxy
```

配置代理：

```
git config --global http.proxy http://127.0.0.1:7890
git config --global https.proxy http://127.0.0.1:7890
```

删除代理：

```
git config --global --unset http.proxy
git config --global --unset https.proxy
```

测试远程：

```
git ls-remote https://github.com/user/repo.git
```

------

## 7. 凭据反复输入

- ❗问题描述
   每次 `pull` / `push` 都要求输入用户名和密码。
- 🔍原因分析
   没有配置 credential helper，或者旧 token / 密码失效。
- ✅解决方案

Windows：

```
git config --global credential.helper manager
```

macOS：

```
git config --global credential.helper osxkeychain
```

Linux 临时缓存：

```
git config --global credential.helper cache
```

Linux 明文保存：

```
git config --global credential.helper store
```

更推荐长期方案：切换 SSH。

```
git remote set-url origin git@github.com:user/repo.git
```

------

# ⭐ 推荐 alias 最终版

```
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.sw switch
git config --global alias.br branch
git config --global alias.cm commit
git config --global alias.cma "commit --amend"
git config --global alias.df diff
git config --global alias.dfs "diff --staged"
git config --global alias.lg "log --oneline --graph --decorate --all"
git config --global alias.last "log -1 HEAD --stat"
git config --global alias.unstage "restore --staged"
git config --global alias.undo "reset --soft HEAD~1"
git config --global alias.aliases "config --get-regexp alias"
```

常用效果：

```
git st
git df
git dfs
git lg
git undo
git unstage README.md
```

------

# ✅ 最小必背命令清单

| 场景                 | 命令                                         |
| -------------------- | -------------------------------------------- |
| 查看状态             | `git status`                                 |
| 查看改动             | `git diff`                                   |
| 添加文件             | `git add .`                                  |
| 提交                 | `git commit -m "msg"`                        |
| 查看日志             | `git log --oneline --graph --decorate --all` |
| 创建分支             | `git switch -c feature/name`                 |
| 切换分支             | `git switch branch-name`                     |
| 拉取远程             | `git pull --rebase`                          |
| 推送分支             | `git push -u origin branch-name`             |
| 撤销最近提交         | `git reset --soft HEAD~1`                    |
| 安全撤销已 push 提交 | `git revert <commit-id>`                     |
| 临时保存改动         | `git stash push -m "msg"`                    |
| 找回误操作           | `git reflog`                                 |