下面是一份面向初学者的 SSH + VS Code Remote SSH 科普指导。

# 1. 什么是 SSH？

**SSH** 全称是 **Secure Shell**，中文通常叫“安全外壳协议”。

它是一种用于**安全远程登录和管理服务器**的网络协议。你可以把它理解成：

> 在自己电脑上打开一个终端，通过加密通道连接到另一台电脑或服务器，并像坐在那台机器前一样操作它。

例如，你在本地电脑上输入：

```
ssh user@192.168.1.10
```

就可以登录到 IP 为 `192.168.1.10` 的远程服务器，用户名是 `user`。

SSH 常用于：

- 远程登录 Linux 服务器
- 远程执行命令
- 上传、下载文件
- 远程开发
- GitHub / GitLab 免密码拉取代码
- 端口转发
- 运维管理
- 云服务器管理

------

# 2. SSH 为什么安全？

SSH 的核心特点是：**加密通信**。

传统的远程登录方式，比如 Telnet，是明文传输的。用户名、密码、命令内容都有可能被窃听。

SSH 会对通信内容进行加密，主要保护：

- 登录密码
- 文件传输内容
- 终端命令
- 返回结果
- Git 操作中的认证信息

SSH 常见认证方式有两种：

## 方式一：密码登录

例如：

```
ssh root@1.2.3.4
```

然后输入服务器密码。

优点是简单。

缺点是：

- 容易被暴力破解
- 每次登录都要输入密码
- 安全性不如密钥登录

## 方式二：密钥登录

SSH 密钥登录使用一对密钥：

| 类型             | 作用                           |
| ---------------- | ------------------------------ |
| 私钥 private key | 放在你自己的电脑上，绝不能泄露 |
| 公钥 public key  | 放到远程服务器上，可以公开     |

简单理解：

> 公钥像锁，私钥像钥匙。服务器保存你的锁，只有你本地的钥匙能打开。

常见密钥文件：

```
~/.ssh/id_rsa
~/.ssh/id_rsa.pub
```

或者更推荐的新格式：

```
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

其中：

- `id_ed25519` 是私钥，**不要发给别人**
- `id_ed25519.pub` 是公钥，可以放到服务器上

------

# 3. SSH 的基本工作流程

以登录服务器为例：

```
ssh user@server_ip
```

流程大致是：

1. 你的电脑向服务器发起连接
2. 服务器证明“我是这台服务器”
3. 双方建立加密通道
4. 你用密码或密钥证明“我是这个用户”
5. 登录成功后，你就可以远程操作服务器

第一次连接时，通常会看到：

```
The authenticity of host '1.2.3.4' can't be established.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

输入：

```
yes
```

之后服务器的身份信息会记录到：

```
~/.ssh/known_hosts
```

------

# 4. 如何安装 SSH 客户端？

## Windows

Windows 10 / 11 通常已经内置 OpenSSH。

可以在 PowerShell 或 CMD 中检查：

```
ssh -V
```

如果能看到类似：

```
OpenSSH_for_Windows_...
```

说明已经安装。

如果没有，可以在：

```
设置 → 应用 → 可选功能 → 添加功能 → OpenSSH Client
```

中安装。

## macOS

macOS 默认自带 SSH。

检查命令：

```
ssh -V
```

## Linux

大多数 Linux 也自带 SSH 客户端。

如果没有，可以安装：

Ubuntu / Debian：

```
sudo apt update
sudo apt install openssh-client
```

CentOS / RHEL：

```
sudo yum install openssh-clients
```

Fedora：

```
sudo dnf install openssh-clients
```

------

# 5. 如何连接一台服务器？

最基本格式：

```
ssh 用户名@服务器地址
```

例如：

```
ssh root@123.45.67.89
```

或者：

```
ssh ubuntu@my-server.com
```

如果服务器 SSH 端口不是默认的 `22`，比如是 `2222`：

```
ssh -p 2222 user@123.45.67.89
```

注意：

```
-p
```

是小写，用于指定端口。

------

# 6. 如何生成 SSH 密钥？

推荐使用 `ed25519`：

```
ssh-keygen -t ed25519 -C "your_email@example.com"
```

然后一路回车即可。

默认会生成：

```
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

查看公钥：

```
cat ~/.ssh/id_ed25519.pub
```

Windows PowerShell 中可以用：

```
type $env:USERPROFILE\.ssh\id_ed25519.pub
```

------

# 7. 如何把公钥放到服务器？

服务器上每个用户都有一个文件：

```
~/.ssh/authorized_keys
```

你需要把本地的 `.pub` 公钥内容追加到服务器的这个文件中。

## 方法一：使用 ssh-copy-id

macOS / Linux 常用：

```
ssh-copy-id user@server_ip
```

如果端口不是 22：

```
ssh-copy-id -p 2222 user@server_ip
```

## 方法二：手动复制

先登录服务器：

```
ssh user@server_ip
```

然后执行：

```
mkdir -p ~/.ssh
chmod 700 ~/.ssh
nano ~/.ssh/authorized_keys
```

把本地的公钥内容粘贴进去，保存退出。

然后设置权限：

```
chmod 600 ~/.ssh/authorized_keys
```

------

# 8. SSH 配置文件是什么？

SSH 配置文件可以让你不用每次输入很长的命令。

配置文件位置：

## macOS / Linux

```
~/.ssh/config
```

## Windows

```
C:\Users\你的用户名\.ssh\config
```

你可以写：

```
Host myserver
    HostName 123.45.67.89
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

之后就可以直接连接：

```
ssh myserver
```

而不用每次写：

```
ssh -i ~/.ssh/id_ed25519 -p 22 ubuntu@123.45.67.89
```

常见配置项：

```
Host myserver
    HostName 123.45.67.89
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
```

含义：

| 配置项              | 说明                           |
| ------------------- | ------------------------------ |
| Host                | 连接别名                       |
| HostName            | 真实服务器地址                 |
| User                | 登录用户名                     |
| Port                | SSH 端口                       |
| IdentityFile        | 私钥路径                       |
| ServerAliveInterval | 每隔多少秒发送保活包，防止断连 |

------

# 9. 如何在 VS Code 中使用 SSH？

VS Code 可以通过 **Remote - SSH** 插件连接远程服务器，然后直接在远程服务器上编辑代码、运行命令、调试程序。

这对开发非常有用，尤其是：

- 服务器上有 GPU
- 项目只能在 Linux 环境跑
- 远程机器配置更强
- 本地电脑只是用来写代码
- 云服务器部署项目
- 远程调试 Python / Node.js / Go / C++ 项目

------

# 10. VS Code Remote SSH 使用步骤

## 第一步：安装 VS Code

下载并安装 VS Code。

## 第二步：安装插件

打开 VS Code，进入扩展市场，搜索：

```
Remote - SSH
```

安装 Microsoft 发布的插件。

通常插件名是：

```
Remote - SSH
```

也可以安装整个扩展包：

```
Remote Development
```

------

## 第三步：确保本地可以用终端 SSH 登录

在 VS Code 之外，先打开系统终端测试：

```
ssh user@server_ip
```

如果这个命令都连不上，VS Code 也大概率连不上。

建议先保证下面这个命令成功：

```
ssh myserver
```

其中 `myserver` 是你在 `~/.ssh/config` 中配置的别名。

------

## 第四步：配置 SSH config

例如你的服务器信息是：

```
服务器 IP：123.45.67.89
用户名：ubuntu
端口：22
私钥：~/.ssh/id_ed25519
```

那么配置：

```
Host myserver
    HostName 123.45.67.89
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

Windows 中路径可以写成：

```
Host myserver
    HostName 123.45.67.89
    User ubuntu
    Port 22
    IdentityFile C:/Users/YourName/.ssh/id_ed25519
```

注意 Windows 路径建议使用 `/`，不要写成反斜杠：

```
C:/Users/YourName/.ssh/id_ed25519
```

------

## 第五步：在 VS Code 中连接服务器

在 VS Code 中：

1. 按 `F1` 或 `Ctrl + Shift + P`
2. 输入：

```
Remote-SSH: Connect to Host...
```

1. 选择你的主机，比如：

```
myserver
```

1. 第一次连接时，选择远程系统类型，比如：

```
Linux
```

1. 等待 VS Code 在远程服务器上安装 VS Code Server
2. 连接成功后，左下角会显示类似：

```
SSH: myserver
```

------

## 第六步：打开远程目录

连接成功后，点击：

```
File → Open Folder
```

然后输入远程服务器上的目录，比如：

```
/home/ubuntu/myproject
```

或者：

```
/var/www/html
```

之后你就可以像本地一样编辑远程服务器文件。

------

# 11. VS Code Remote SSH 的实际体验

连接成功后，VS Code 的这些功能其实运行在远程服务器上：

- 文件浏览
- 终端
- Git
- Python 环境
- Node.js 环境
- 调试器
- 插件的远程部分

你打开 VS Code 终端，会发现命令运行在服务器上：

```
pwd
hostname
python --version
nvidia-smi
```

例如：

```
pwd
```

可能返回：

```
/home/ubuntu/project
```

说明你操作的是远程服务器，而不是本地电脑。

------

# 12. 常见 SSH 指令

## 12.1 基本登录

```
ssh user@host
```

示例：

```
ssh ubuntu@123.45.67.89
```

------

## 12.2 指定端口登录

```
ssh -p 2222 user@host
```

示例：

```
ssh -p 2222 ubuntu@123.45.67.89
```

------

## 12.3 指定私钥登录

```
ssh -i ~/.ssh/id_ed25519 user@host
```

示例：

```
ssh -i ~/.ssh/mykey ubuntu@123.45.67.89
```

------

## 12.4 使用配置别名登录

配置文件：

```
Host prod
    HostName 123.45.67.89
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

登录：

```
ssh prod
```

------

## 12.5 执行远程命令

不用进入交互式 shell，也可以远程执行命令：

```
ssh user@host "ls -la"
```

示例：

```
ssh ubuntu@123.45.67.89 "df -h"
```

查看服务器磁盘空间。

------

## 12.6 上传文件：scp

从本地上传到服务器：

```
scp local_file user@host:/remote/path/
```

示例：

```
scp app.py ubuntu@123.45.67.89:/home/ubuntu/
```

指定端口时注意是大写 `-P`：

```
scp -P 2222 app.py ubuntu@123.45.67.89:/home/ubuntu/
```

注意：

- `ssh` 用小写 `-p`
- `scp` 用大写 `-P`

------

## 12.7 下载文件：scp

从服务器下载到本地：

```
scp user@host:/remote/file ./local/path/
```

示例：

```
scp ubuntu@123.45.67.89:/home/ubuntu/result.txt .
```

下载目录需要加 `-r`：

```
scp -r ubuntu@123.45.67.89:/home/ubuntu/project .
```

------

## 12.8 使用 rsync 同步文件

`rsync` 比 `scp` 更适合同步项目，因为它只传输变化的部分。

本地同步到远程：

```
rsync -avz ./project/ user@host:/remote/project/
```

示例：

```
rsync -avz ./myapp/ ubuntu@123.45.67.89:/home/ubuntu/myapp/
```

远程同步到本地：

```
rsync -avz user@host:/remote/project/ ./project/
```

指定端口：

```
rsync -avz -e "ssh -p 2222" ./project/ user@host:/remote/project/
```

------

## 12.9 查看 SSH 连接调试信息

如果连不上，可以加 `-v`：

```
ssh -v user@host
```

更多调试信息：

```
ssh -vv user@host
```

极详细：

```
ssh -vvv user@host
```

------

## 12.10 端口转发：本地端口转发

常见场景：服务器上有一个服务运行在远程的 `localhost:8888`，你想在本地浏览器访问。

命令：

```
ssh -L 本地端口:远程地址:远程端口 user@host
```

示例：

```
ssh -L 8888:localhost:8888 ubuntu@123.45.67.89
```

然后你本地浏览器访问：

```
http://localhost:8888
```

就相当于访问服务器上的：

```
http://localhost:8888
```

这常用于：

- Jupyter Notebook
- TensorBoard
- Web 服务调试
- 数据库管理端口转发

------

## 12.11 后台建立端口转发

```
ssh -N -L 8888:localhost:8888 user@host
```

其中：

| 参数 | 说明                     |
| ---- | ------------------------ |
| `-N` | 不执行远程命令，只做转发 |
| `-L` | 本地端口转发             |

------

## 12.12 反向端口转发

让远程服务器访问你本地的服务：

```
ssh -R 远程端口:localhost:本地端口 user@host
```

示例：

```
ssh -R 9000:localhost:3000 ubuntu@123.45.67.89
```

意思是：远程服务器访问自己的 `localhost:9000` 时，会转发到你本地电脑的 `localhost:3000`。

------

## 12.13 动态代理

```
ssh -D 1080 user@host
```

这会在本地创建一个 SOCKS5 代理：

```
localhost:1080
```

------

## 12.14 保持连接不断开

```
ssh -o ServerAliveInterval=60 user@host
```

或者写入配置：

```
Host myserver
    HostName 123.45.67.89
    User ubuntu
    ServerAliveInterval 60
    ServerAliveCountMax 3
```

------

# 13. 常见 Linux 远程操作命令

SSH 登录进去后，常用这些命令。

## 查看当前路径

```
pwd
```

## 查看文件

```
ls
ls -la
```

## 进入目录

```
cd /path/to/dir
```

## 创建目录

```
mkdir mydir
mkdir -p parent/child
```

## 删除文件

```
rm file.txt
```

## 删除目录

```
rm -r mydir
```

慎用：

```
rm -rf mydir
```

千万不要乱执行：

```
rm -rf /
```

## 复制文件

```
cp source.txt target.txt
```

复制目录：

```
cp -r source_dir target_dir
```

## 移动或重命名

```
mv old.txt new.txt
```

## 查看文件内容

```
cat file.txt
less file.txt
head file.txt
tail file.txt
tail -f log.txt
```

其中：

```
tail -f log.txt
```

常用于实时查看日志。

## 查看磁盘空间

```
df -h
```

## 查看目录大小

```
du -sh *
```

## 查看内存

```
free -h
```

## 查看进程

```
ps aux
top
htop
```

## 查找进程

```
ps aux | grep python
```

## 杀掉进程

```
kill PID
kill -9 PID
```

## 查看端口占用

```
ss -tulnp
```

或：

```
lsof -i :端口号
```

示例：

```
lsof -i :8000
```

------

# 14. VS Code Remote SSH 常见操作

## 打开远程终端

在 VS Code 中：

```
Terminal → New Terminal
```

这个终端是在远程服务器上运行的。

## 安装远程插件

连接 SSH 后，VS Code 插件会分成本地和远程两类。

例如 Python 插件需要安装在远程环境中。

在扩展页面中可能看到：

```
Install in SSH: myserver
```

点击即可安装到远程服务器。

## 选择 Python 解释器

连接远程服务器后：

1. `Ctrl + Shift + P`
2. 输入：

```
Python: Select Interpreter
```

1. 选择远程服务器上的 Python，例如：

```
/home/ubuntu/miniconda3/envs/myenv/bin/python
```

## 打开远程项目

可以打开：

```
/home/ubuntu/project
```

不建议直接打开：

```
/
```

因为根目录文件太多，VS Code 扫描会很慢。

------

# 15. VS Code Remote SSH 配置示例

## 普通服务器

```
Host dev
    HostName 123.45.67.89
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519
```

连接：

```
ssh dev
```

VS Code 中选择：

```
Remote-SSH: Connect to Host...
dev
```

------

## 多台服务器

```
Host dev
    HostName 123.45.67.89
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host test
    HostName 123.45.67.90
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519

Host prod
    HostName 123.45.67.91
    User root
    Port 2222
    IdentityFile ~/.ssh/prod_key
```

------

## 通过跳板机连接服务器

有些内网服务器不能直接访问，需要先连跳板机。

```
Host jump
    HostName jump.example.com
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519

Host inner
    HostName 10.0.0.5
    User ubuntu
    IdentityFile ~/.ssh/id_ed25519
    ProxyJump jump
```

连接内网机器：

```
ssh inner
```

VS Code 也可以直接连接 `inner`。

------

# 16. GitHub / GitLab 中的 SSH

SSH 还常用于 Git 代码仓库认证。

例如 GitHub 仓库地址：

```
git@github.com:username/repo.git
```

克隆：

```
git clone git@github.com:username/repo.git
```

测试 GitHub SSH 是否配置成功：

```
ssh -T git@github.com
```

如果看到类似：

```
Hi username! You've successfully authenticated...
```

说明成功。

注意：GitHub 不会给你一个普通 shell，所以即使认证成功，也不会让你登录到 GitHub 的服务器终端。

------

# 17. 常见问题排查

## 问题一：Permission denied

常见报错：

```
Permission denied (publickey).
```

可能原因：

- 用户名写错了
- 私钥不对
- 公钥没有放到服务器
- 服务器禁用了密码登录
- `authorized_keys` 权限不对
- VS Code 没找到正确私钥

排查：

```
ssh -v user@host
```

检查 SSH 实际使用了哪个私钥。

------

## 问题二：Connection timed out

```
ssh: connect to host 123.45.67.89 port 22: Connection timed out
```

可能原因：

- 服务器没开机
- IP 写错
- 安全组没开放 22 端口
- 防火墙阻止
- 网络不通
- 服务器 SSH 服务没启动

云服务器尤其要检查：

- 云厂商安全组
- 服务器防火墙
- SSH 端口

------

## 问题三：Connection refused

```
ssh: connect to host 123.45.67.89 port 22: Connection refused
```

说明目标主机能访问，但对应端口没有服务。

可能原因：

- SSH 服务没启动
- 端口写错
- SSH 改成了别的端口

服务器上检查：

```
sudo systemctl status ssh
```

或：

```
sudo systemctl status sshd
```

启动：

```
sudo systemctl start ssh
```

或：

```
sudo systemctl start sshd
```

------

## 问题四：Host key verification failed

```
Host key verification failed.
```

可能原因：

- 服务器重装过
- IP 对应的机器变了
- `known_hosts` 中记录的指纹不一致

可以删除旧记录：

```
ssh-keygen -R 服务器IP
```

例如：

```
ssh-keygen -R 123.45.67.89
```

然后重新连接。

注意：如果你不确定服务器是否真的重装过，要小心中间人攻击风险。

------

## 问题五：UNPROTECTED PRIVATE KEY FILE

```
WARNING: UNPROTECTED PRIVATE KEY FILE!
```

说明私钥权限太开放。

Linux / macOS 修复：

```
chmod 600 ~/.ssh/id_ed25519
```

`.ssh` 目录权限：

```
chmod 700 ~/.ssh
```

------

## 问题六：VS Code 一直卡在 Installing VS Code Server

可以尝试：

1. 确认终端 `ssh myserver` 可以正常登录
2. 删除远程服务器上的 VS Code Server 缓存：

```
rm -rf ~/.vscode-server
```

1. 重新连接 VS Code
2. 检查远程服务器磁盘空间：

```
df -h
```

1. 检查是否缺少必要工具：

```
which bash
which tar
which curl
which wget
```

------

# 18. SSH 安全建议

## 不建议长期使用 root 直接登录

可以创建普通用户：

```
sudo adduser devuser
sudo usermod -aG sudo devuser
```

然后用普通用户登录。

## 推荐使用密钥登录

比密码登录更安全，也更适合自动化。

## 禁用密码登录

确认密钥登录可用后，可以修改：

```
sudo nano /etc/ssh/sshd_config
```

设置：

```
PasswordAuthentication no
```

然后重启 SSH：

```
sudo systemctl restart ssh
```

或：

```
sudo systemctl restart sshd
```

## 禁用 root 登录

```
PermitRootLogin no
```

然后重启 SSH 服务。

## 修改默认端口

例如改成 2222：

```
Port 2222
```

然后注意放开防火墙和云安全组。

## 使用防火墙

Ubuntu 常见：

```
sudo ufw allow 2222/tcp
sudo ufw enable
```

------

# 19. SSH 与 VS Code 的推荐实践

建议你采用这种工作方式：

1. 本地生成 SSH 密钥
2. 把公钥放到服务器
3. 配置 `~/.ssh/config`
4. 本地终端确认可以 `ssh myserver`
5. VS Code 安装 Remote - SSH
6. VS Code 连接 `myserver`
7. 打开远程项目目录
8. 在远程终端运行项目
9. 用 Git 管理代码

典型配置：

```
Host myserver
    HostName 123.45.67.89
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
```

典型连接：

```
ssh myserver
```

典型 VS Code 使用：

```
Remote-SSH: Connect to Host...
myserver
```

------

# 20. 常用命令速查表

| 场景                    | 命令                                      |
| ----------------------- | ----------------------------------------- |
| 登录服务器              | `ssh user@host`                           |
| 指定端口                | `ssh -p 2222 user@host`                   |
| 指定私钥                | `ssh -i ~/.ssh/key user@host`             |
| 生成密钥                | `ssh-keygen -t ed25519 -C "email"`        |
| 复制公钥                | `ssh-copy-id user@host`                   |
| 上传文件                | `scp file user@host:/path/`               |
| 下载文件                | `scp user@host:/path/file .`              |
| 上传目录                | `scp -r dir user@host:/path/`             |
| 同步目录                | `rsync -avz ./dir/ user@host:/path/`      |
| 调试连接                | `ssh -v user@host`                        |
| 本地端口转发            | `ssh -L 8888:localhost:8888 user@host`    |
| 后台端口转发            | `ssh -N -L 8888:localhost:8888 user@host` |
| 删除 known_hosts 旧记录 | `ssh-keygen -R host`                      |
| 测试 GitHub SSH         | `ssh -T git@github.com`                   |

------

# 21. 一个完整示例

假设你有一台云服务器：

```
IP：123.45.67.89
用户名：ubuntu
端口：22
```

## 本地生成密钥

```
ssh-keygen -t ed25519 -C "me@example.com"
```

## 复制公钥到服务器

```
ssh-copy-id ubuntu@123.45.67.89
```

如果没有 `ssh-copy-id`，手动把：

```
~/.ssh/id_ed25519.pub
```

内容复制到服务器：

```
~/.ssh/authorized_keys
```

## 配置 SSH

编辑：

```
~/.ssh/config
```

加入：

```
Host myserver
    HostName 123.45.67.89
    User ubuntu
    Port 22
    IdentityFile ~/.ssh/id_ed25519
    ServerAliveInterval 60
```

## 测试连接

```
ssh myserver
```

## VS Code 连接

1. 安装 `Remote - SSH`
2. `Ctrl + Shift + P`
3. 输入：

```
Remote-SSH: Connect to Host...
```

1. 选择：

```
myserver
```

1. 打开远程目录：

```
/home/ubuntu/project
```

之后就可以在 VS Code 里远程写代码了。

------

# 22. 初学者最容易混淆的点

## SSH 是协议，不是软件本身

`ssh` 命令是 OpenSSH 客户端，SSH 是协议。

## 公钥可以公开，私钥不能公开

`.pub` 可以给服务器、GitHub、GitLab。

没有 `.pub` 后缀的私钥千万不要发给别人。

## SSH 的端口默认是 22

但很多服务器为了安全会改成别的端口。

## VS Code Remote SSH 不是把文件下载到本地

它是在远程服务器上打开和编辑文件，只是界面显示在本地。

## VS Code 终端是在远程机器上运行

连接成功后，VS Code 里的终端命令会在服务器执行。

## SSH config 可以极大简化操作

强烈建议配置别名，不要每次都手敲长命令。

------

# 23. 推荐学习顺序

你可以按这个顺序练习：

1. 学会 `ssh user@host`
2. 学会生成密钥 `ssh-keygen`
3. 学会配置 `~/.ssh/config`
4. 学会 `scp` 上传下载文件
5. 学会 VS Code Remote SSH
6. 学会端口转发 `ssh -L`
7. 学会查看日志、进程、磁盘、端口
8. 学会排查 `Permission denied`、`Connection timed out` 等错误

掌握这些后，日常远程开发和服务器管理基本就够用了。