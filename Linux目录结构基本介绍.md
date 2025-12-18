# Linux采用标准化的目录结构（遵循FHS，Filesystem Hierarchy Standard），核心目标是统一目录组织方式，方便系统管理、软件安装及跨发行版兼容。以下是主要顶级目录及其功能的详细说明：

### **1. 根目录 `/`**

- **功能**：所有文件和目录的起点，是Linux文件系统的“树根”。
- **特点**：必须包含引导系统、修复系统所需的关键文件（如内核、initramfs、启动脚本等）。

### **2. 基础命令与工具目录**

#### **`/bin`（Binary）**

- 存放**所有用户可用的基本命令**（如`ls`、`cp`、`mv`、`cat`、`echo`），系统启动或单用户模式（救援模式）时必需。

#### **`/sbin`（System Binary）**

- 存放**系统管理员（root）专用的管理命令**（如`fdisk`、`shutdown`、`iptables`、`ifconfig`），普通用户无权限执行（或无需执行）。

### **3. 配置文件目录 `/etc`**

- **功能**：存放系统和应用的**静态配置文件**（文本格式为主）。

- 

  常见内容

  ：

  - 用户账户：`passwd`、`shadow`、`group`；
  - 网络配置：`network/interfaces`、`resolv.conf`；
  - 服务配置：`nginx/nginx.conf`、`mysql/my.cnf`；
  - 启动脚本：`rc.local`、`profile`（环境变量）。

### **4. 设备文件目录 `/dev`**

- **功能**：存储**硬件设备的抽象文件**（“设备节点”），系统通过操作这些文件与硬件交互。

- 

  常见设备

  ：

  - 磁盘：`sda`（SCSI/SATA磁盘）、`nvme0n1`（NVMe固态）；
  - 终端：`tty`（控制台）、`pts/0`（伪终端）；
  - 虚拟设备：`null`（黑洞设备）、`zero`（零设备）、`random`（随机数）。

### **5. 进程与系统信息目录 `/proc`**

- **功能**：**虚拟文件系统**，动态反映内核、进程及硬件状态（不占用物理磁盘空间）。

- 

  常见内容

  ：

  - 进程信息：`/proc/[PID]/`（如`/proc/1/`是init进程的信息）；
  - 系统状态：`cpuinfo`（CPU信息）、`meminfo`（内存信息）、`diskstats`（磁盘统计）。

### **6. 内核与硬件信息目录 `/sys`**

- **功能**：**虚拟文件系统**，提供内核设备模型、硬件拓扑及驱动信息的接口（比`/proc`更结构化）。
- **常见用途**：查看设备驱动（`/sys/class/net/`显示网络设备驱动）、调整硬件参数（如`/sys/devices/system/cpu/cpu0/cpufreq/scaling_governor`调整CPU调速器）。

### **7. 可变数据目录 `/var`**

- **功能**：存放**运行时动态变化的数据**（日志、缓存、数据库等）。

- 

  核心子目录

  ：

  - `/var/log`：系统和服务日志（如`auth.log`认证日志、`syslog`系统日志）；
  - `/var/lib`：应用状态数据（如`mysql/data`存储数据库文件、`docker/`存储容器数据）；
  - `/var/cache`：应用缓存（如`apt/cache`存储下载的软件包）；
  - `/var/tmp`：临时文件（重启后保留，比`/tmp`更持久）。

### **8. 临时文件目录 `/tmp`**

- **功能**：存放**短期临时文件**（程序运行时生成，重启后可能被清空，取决于系统配置）。

### **9. 用户程序目录 `/usr`**

- **功能**：存放**非系统启动必需的用户级程序和数据**（“Unix System Resources”的缩写，历史遗留命名）。

- 

  核心子目录

  ：

  - `/usr/bin`：普通用户的非基础命令（如`python3`、`git`）；
  - `/usr/sbin`：管理员的非基础命令（如`apache2ctl`、`useradd`）；
  - `/usr/lib`：应用程序的库文件（对应`/lib`，但不含内核模块）；
  - `/usr/local`：**本地编译安装的软件**（避免污染系统默认目录，如手动编译的Python、Nginx）。

### **10. 用户家目录 `/home`**

- **功能**：存放**普通用户的个人文件**（如文档、配置、下载内容），每个用户对应一个子目录（如`/home/zhangsan`）。

### **11. root用户家目录 `/root`**

- **功能**：root用户的专属家目录（区别于普通用户的`/home`）。

### **12. 启动相关目录 `/boot`**

- **功能**：存放**系统启动必需的文件**（内核、initramfs、引导加载程序如GRUB的配置）。
- **常见内容**：`vmlinuz-5.4.0-xx-generic`（内核镜像）、`initrd.img-xx`（初始RAM磁盘）、`grub/grub.cfg`（GRUB配置）。

### **13. 库文件目录 `/lib`**

- **功能**：存放**系统和核心命令依赖的共享库文件**（如`.so`结尾的动态链接库），对应`/bin`和`/sbin`的依赖。

### **14. 挂载点目录 `/mnt`、`/media`**

- **功能**：临时挂载外部存储设备（如U盘、移动硬盘、ISO镜像）。
- **区别**：`/mnt`用于手动临时挂载；`/media`用于自动挂载（如插入U盘后系统自动挂载到`/media/zhangsan/USB-NAME`）。

### **15. 第三方软件目录 `/opt`**

- **功能**：存放**第三方大型软件或独立组件**（如Oracle数据库、某些商业软件，避免与`/usr/local`冲突）。

### **16. 服务数据目录 `/srv`**

- **功能**：存放**服务程序提供的文件数据**（如Web服务器的网站文件、FTP服务器的共享数据，现代Linux中较少使用，常被`/var/www`等替代）。

### **总结**

Linux目录结构通过FHS实现标准化，核心逻辑是“按用途分类”：

- **静态配置**：`/etc`；
- **动态数据**：`/var`；
- **用户程序**：`/usr`；
- **硬件交互**：`/dev`、`/proc`、`/sys`；
- **启动与基础命令**：`/boot`、`/bin`、`/sbin`。

理解这些目录的功能，有助于高效管理系统、定位文件及排查问题。



# linux常见命令

以下是 Linux 系统中**最常用的命令分类整理**，覆盖文件操作、系统管理、网络、权限等核心场景，附**用途、常用选项、示例**，适合新手快速上手和日常查阅。

## **一、文件与目录操作**

### 1. 查看目录内容：`ls`

**用途**：列出目录下的文件/子目录。

**常用选项**：

- `-l`：详细列表（权限、大小、修改时间）。
- `-a`：显示隐藏文件（以`.`开头的文件）。
- `-h`：人性化显示文件大小（如 `1K`、`2M`）。
- `-d`：仅显示目录本身（不展开内容）。

**示例**：

bash

复制

```
ls -lh   # 详细列表+人性化大小
ls -a    # 显示所有文件（含隐藏）
ls -ld /home/user/documents  # 查看documents目录本身的信息
```

### 2. 切换目录：`cd`

**用途**：切换当前工作目录。

**常用路径**：

- `~`：用户家目录（如 `/home/user`）。
- `.`：当前目录。
- `..`：上级目录。
- `/`：根目录。

**示例**：

bash

复制

```
cd ~          # 回到用户家目录
cd documents  # 进入当前目录下的documents子目录
cd ../Desktop # 切换到上级目录的Desktop
cd /usr/local # 绝对路径切换到/usr/local
```

### 3. 查看当前路径：`pwd`

**用途**：显示当前工作目录的绝对路径。

**示例**：

bash

复制

```
pwd   # 输出类似 /home/user/documents
```

### 4. 创建目录：`mkdir`

**用途**：创建新目录。

**常用选项**：

- `-p`：递归创建多级目录（若父目录不存在则自动创建）。

**示例**：

bash

复制

```
mkdir mydir          # 创建mydir目录
mkdir -p a/b/c       # 递归创建a→b→c三级目录
```

### 5. 删除文件/目录：`rm`

**用途**：删除文件或目录（**谨慎使用！**）。

**常用选项**：

- `-f`：强制删除（忽略不存在的文件，不提示确认）。
- `-r`：递归删除目录及其内容（删除目录必须加此选项）。

**示例**：

bash

复制

```
rm test.txt          # 删除test.txt文件（需确认）
rm -f test.txt       # 强制删除（无确认提示）
rm -r mydir          # 递归删除mydir目录及其所有内容（危险！）
```

### 6. 复制文件/目录：`cp`

**用途**：复制文件或目录。

**常用选项**：

- `-r`：递归复制目录（复制目录必须加此选项）。
- `-i`：覆盖前提示确认（默认已开启）。

**示例**：

bash

复制

```
cp file1.txt file2.txt   # 复制file1.txt为file2.txt
cp -r dir1 dir2          # 递归复制dir1目录到dir2（若dir2不存在则创建）
cp /home/user/a.txt ~/   # 复制a.txt到用户家目录
```

### 7. 移动/重命名文件/目录：`mv`

**用途**：移动文件/目录，或重命名。

**示例**：

bash

复制

```
mv file1.txt file2.txt   # 重命名file1.txt为file2.txt
mv file.txt ~/Documents  # 移动file.txt到Documents目录
mv dir1 dir2             # 重命名dir1为dir2（若dir2存在则移动）
```

## **二、文件查看与编辑**

### 1. 查看文件内容：`cat`

**用途**：快速查看小文件的全部内容（大文件不建议，会刷屏）。

**示例**：

bash

复制

```
cat /etc/os-release   # 查看系统版本信息
cat log.txt           # 查看log.txt的全部内容
```

### 2. 分页查看大文件：`less`/`more`

**用途**：分页查看大文件（`less`功能更强大，支持上下翻页）。

**常用操作（`less`）**：

- `空格`：下一页；`b`：上一页。
- `/关键词`：搜索关键词（`n`下一个，`N`上一个）。
- `q`：退出。

**示例**：

bash

复制

```
less /var/log/syslog   # 分页查看系统日志
more large_file.log    # 分页查看大文件（仅上下翻页）
```

### 3. 查看文件头部/尾部：`head`/`tail`

**用途**：查看文件的前几行或后几行。

**常用选项**：

- `-n 数字`：指定显示的行数（如 `-n 10`显示前10行）。
- `-f`：实时追踪文件新增内容（常用于查看日志）。

**示例**：

bash

复制

```
head -n 20 /etc/passwd   # 查看/etc/passwd的前20行
tail -n 50 log.txt       # 查看log.txt的最后50行
tail -f /var/log/nginx/access.log  # 实时追踪Nginx访问日志（按Ctrl+C退出）
```

### 4. 文本编辑器：`nano`/`vim`

**`nano`（新手友好）**：

bash

复制

```
nano filename.txt   # 打开文件（底部显示快捷键，如Ctrl+O保存，Ctrl+X退出）
```

**`vim`（高级编辑器）**：

- 模式切换：命令模式（默认）→ 插入模式（按 `i`）→ 底部命令模式（按 `:`）。
- 常用操作：
  - 插入模式：输入文本。
  - 保存退出：`:wq`（保存并退出）；`:q`（退出，未修改）；`:q!"`（强制退出不保存）。
  - 搜索：`/关键词`（向下搜索），`?关键词`（向上搜索）。

## **三、系统信息与资源管理**

### 1. 查看系统版本：`uname`

**用途**：显示内核和系统信息。

**常用选项**：

- `-a`：显示所有信息（内核版本、主机名、架构等）。

**示例**：

bash

复制

```
uname -a   # 输出类似 Linux ubuntu 5.4.0-xx-generic #xx-Ubuntu SMP ...
```

### 2. 查看进程：`ps`/`top`/`htop`

**`ps`（静态进程快照）**：

**常用选项**：

- `aux`：显示所有用户的所有进程（详细信息）。

**示例**：

bash

复制

```
ps aux | grep "nginx"   # 查找所有包含"nginx"的进程
```

**`top`（动态进程监控）**：

- 实时显示CPU、内存占用，进程列表（按 `q`退出）。
- 按 `M`按内存排序，按 `P`按CPU排序。

**`htop`（更友好的交互式监控）**：

需先安装：`sudo apt install htop`，功能类似 `top`，支持鼠标操作。

### 3. 查看磁盘空间：`df`/`du`

**`df`（磁盘总空间）**：

**常用选项**：

- `-h`：人性化显示大小（如 `1G`、`500M`）。

**示例**：

bash

复制

```
df -h   # 查看各分区的磁盘占用（根目录/、/home等）
```

**`du`（目录/文件大小）**：

**常用选项**：

- `-h`：人性化显示。
- `-s`：仅显示总大小（不展开子目录）。

**示例**：

bash

复制

```
du -sh /home/user/documents   # 查看documents目录的总大小
du -h --max-depth=1 /var/log  # 查看/var/log下一级目录的大小
```

### 4. 关机/重启：`shutdown`/`reboot`

**示例**：

bash

复制

```
shutdown -h now    # 立即关机
shutdown -r 10     # 10分钟后重启
reboot             # 立即重启（等价于shutdown -r now）
```

## **四、网络相关命令**

### 1. 测试网络连通性：`ping`

**用途**：测试与目标IP/域名的连通性。

**示例**：

bash

复制

```
ping www.baidu.com   # 持续发送ICMP包（按Ctrl+C停止）
ping -c 5 192.168.1.1  # 发送5个包后停止
```

### 2. 查看IP地址：`ip`/`ifconfig`

**`ip`（现代工具，推荐）**：

bash

复制

```
ip addr show   # 显示所有网络接口的IP地址（如eth0、wlan0）
ip route show  # 查看路由表
```

**`ifconfig`（传统工具，需安装net-tools）**：

bash

复制

```
sudo apt install net-tools  # 若未安装
ifconfig       # 显示网络接口信息（IP、MAC、流量统计）
```

### 3. 查看网络连接：`netstat`/`ss`

**`netstat`（传统工具）**：

bash

复制

```
netstat -tunlp   # 查看所有TCP/UDP连接的进程（-t:TCP, -u:UDP, -n:数字IP, -l:监听状态, -p:进程）
```

**`ss`（更高效，推荐）**：

bash

复制

```
ss -tunlp        # 功能同上，性能更好
```

### 4. 下载文件：`curl`/`wget`

**`curl`（支持多种协议，可交互）**：

bash

复制

```
curl https://www.example.com   # 下载网页内容并输出到终端
curl -O https://example.com/file.zip  # 下载文件并保存为file.zip（-O：输出到文件）
```

**`wget`（专注下载）**：

bash

复制

```
wget https://example.com/large_file.iso  # 下载大文件（支持断点续传：-c选项）
```

### 5. 远程连接：`ssh`/`scp`

**`ssh`（远程登录）**：

bash

复制

```
ssh user@192.168.1.100   # 登录IP为192.168.1.100的服务器（user是用户名）
ssh -p 2222 user@example.com  # 指定端口（默认22）
```

**`scp`（远程文件传输）**：

- 上传文件：`scp local_file user@remote_ip:remote_path`
- 下载文件：`scp user@remote_ip:remote_file local_path`

**示例**：

bash

复制

```
scp ~/documents/report.pdf user@192.168.1.100:/home/user/  # 上传report.pdf到服务器
scp user@192.168.1.100:/var/log/nginx/access.log ./  # 下载access.log到本地当前目录
```

## **五、权限管理**

### 1. 修改文件/目录权限：`chmod`

**用途**：调整文件/目录的读（r）、写（w）、执行（x）权限。

**两种模式**：

- **符号模式**：`chmod [用户][+/-/=][权限] 文件`
  - 用户：`u`（所有者）、`g`（所属组）、`o`（其他用户）、`a`（所有用户）。
  - 权限：`r`（读）、`w`（写）、`x`（执行）。
- **数字模式**：权限用3位数字表示（每位0-7，`r=4, w=2, x=1`）。

**示例**：

bash

复制

```
chmod u+x script.sh   # 给所有者添加执行权限
chmod g-w file.txt    # 移除所属组的写权限
chmod 755 mydir       # 数字模式：所有者7（rwx），所属组5（r-x），其他用户5（r-x）
```

### 2. 修改文件/目录所有者：`chown`

**用途**：修改文件/目录的所有者（需root权限）。

**示例**：

bash

复制

```
sudo chown user:user file.txt   # 修改所有者为user，所属组为user（格式：用户:组）
sudo chown -R user:admin dir/   # 递归修改dir目录及其内容的所有者为user，组为admin
```

### 3. 以管理员权限执行命令：`sudo`

**用途**：临时以root权限执行命令（需当前用户在sudoers列表中）。

**示例**：

bash

复制

```
sudo apt install nginx   # 以root权限安装nginx
sudo systemctl restart nginx  # 重启nginx服务
```

## **六、压缩与解压**

### 1. 打包/压缩：`tar`

**用途**：将多个文件/目录打包为一个`.tar`文件（可结合压缩工具）。

**常用选项**：

- `-c`：打包（create）。
- `-x`：解包（extract）。
- `-z`：用gzip压缩/解压（生成`.tar.gz`）。
- `-j`：用bzip2压缩/解压（生成`.tar.bz2`）。
- `-v`：显示详细过程。

**示例**：

bash

复制

```
tar -czvf mydir.tar.gz mydir   # 打包并gzip压缩mydir目录为mydir.tar.gz
tar -xzvf mydir.tar.gz         # 解压mydir.tar.gz到当前目录
```

### 2. 解压`.zip`文件：`unzip`

**用途**：解压`.zip`格式的压缩包（需安装：`sudo apt install unzip`）。

**示例**：

bash

复制

```
unzip file.zip   # 解压file.zip到当前目录
unzip -d target_dir file.zip  # 解压到指定目录target_dir
```

## **七、查找与搜索**

### 1. 查找文件/目录：`find`

**用途**：在指定路径下查找文件/目录（支持名称、类型、大小等条件）。

**示例**：

bash

复制

```
find /home/user -name "*.txt"   # 在/home/user下查找所有.txt文件
find /var/log -type f -size +10M  # 查找/var/log下大于10MB的普通文件（-type f：文件，-size +10M：大于10M）
```

### 2. 搜索文件内容：`grep`

**用途**：在文件或命令输出中搜索匹配的关键词。

**常用选项**：

- `-i`：忽略大小写。
- `-r`：递归搜索目录下的所有文件。
- `-n`：显示匹配行的行号。

**示例**：

bash

复制

```
grep "error" /var/log/syslog   # 在syslog中搜索包含"error"的行
grep -r "TODO" ~/projects/     # 递归搜索~/projects目录下所有文件中的"TODO"
ps aux | grep "nginx"          # 搜索包含"nginx"的进程（结合管道）
```

### 3. 快速查找文件路径：`locate`

**用途**：通过数据库快速查找文件路径（需先更新数据库：`sudo updatedb`）。

**示例**：

bash

复制

```
locate nginx.conf   # 查找nginx.conf文件的路径（可能在/etc/nginx/nginx.conf）
```

## **八、实用技巧**

### 1. 通配符：`*`/`?`/`[]`

- `*`：匹配任意字符（如 `*.txt`匹配所有.txt文件）。
- `?`：匹配单个字符（如 `file?.txt`匹配file1.txt、fileA.txt）。
- `[]`：匹配括号内的任意一个字符（如 `[ab].txt`匹配a.txt、b.txt）。

### 2. 管道：`|`

**用途**：将前一个命令的输出作为后一个命令的输入。

**示例**：

bash

复制

```
ls -l | grep "Jun 10"   # 查看今天（6月10日）修改的文件
cat log.txt | tail -n 10  # 查看log.txt的最后10行
```

### 3. 重定向：`>`/`>>`/`<`

- `>`：覆盖写入文件（如 `echo "hello" > file.txt`）。
- `>>`：追加写入文件（如 `echo "world" >> file.txt`）。
- `<`：从文件读取输入（如 `cat < file.txt`等价于 `cat file.txt`）。

### 4. 查看命令手册：`man`

**用途**：查看命令的详细帮助文档（按 `q`退出）。

**示例**：

bash

复制

```
man ls   # 查看ls命令的手册
man chmod  # 查看chmod命令的用法
```

### 5. 显示命令路径：`which`

**用途**：查找命令的可执行文件路径。

**示例**：

bash

复制

```
which python3   # 输出类似 /usr/bin/python3
which git       # 输出git的安装路径
```

------

以上是 Linux 最常用的命令，覆盖日常开发、运维的核心场景。建议通过实践逐步熟悉，遇到复杂操作时可结合 `man`命令查看详细文档~