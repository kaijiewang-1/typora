Next time: Before starting a second instance, stop the first one with Ctrl+C in the terminal where it is running. If you are not sure what is using 8090:

```powershell
netstat -ano | findstr :8090

tasklist /FI "PID eq <PID_from_LISTENING_line>"
```

Alternative: Keep the old server running and start another copy on a different port, for example:

```powershell
mvn spring-boot:run -Dspring-boot.run.arguments="--server.port=8091"
```

# 开发时遇到“端口被占用”简直是家常便饭，不用重启电脑，几行命令就能轻松搞定！

根据你的操作系统，选择以下对应的“查杀两步法”即可：

### 🍎 Linux / macOS 系统

**第一步：查端口，找 PID**

在终端输入以下命令（把 `8080`换成你的实际端口号）：

```bash
lsof -i :8080
```

*如果提示没有 `lsof`，可以用这个备胎命令：`netstat -tunlp | grep 8080`*

命令会列出占用该端口的程序，记下第一列的程序名或第二列的 **PID**（进程号）。

**第二步：杀进程，放端口**

直接用 `kill -9`加上刚才查到的 PID 强制秒杀（比如 PID 是 1234）：

```bash
kill -9 1234
```

*如果是系统级进程，记得加上 `sudo`提权：*

```bash
sudo kill -9 1234
```

------

### 🪟 Windows 系统

**第一步：查端口，找 PID**

按下 `Win + R`，输入 `cmd`回车打开黑框，输入（同样把 `8080`换成你的端口）：

```cmd
netstat -ano | findstr 8080
```

最后一列显示的数字就是占用该端口的 **PID** 。

**第二步：杀进程，放端口**

拿到 PID 后（比如是 1234），接着敲入这行命令强制终结它：

```cmd
taskkill /pid 1234 /f
```

*提示：如果遇到“拒绝访问”，关掉 CMD 然后用管理员身份重新打开再试一次即可。*