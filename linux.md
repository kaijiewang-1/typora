# Linux 和 Bash 入门

本教程将帮助 Linux 新手开始在默认通过 WSL 安装的 Linux 中的 Ubuntu 发行版上安装和更新软件包，并使用 Bash 命令行中的一些基本命令。



## 安装和更新软件

可以使用运行分发版的首选包管理器直接从命令行安装和更新软件程序。

例如，在 Ubuntu 中，首先通过运行 `sudo apt update`来更新可用的软件列表。 然后，您可以使用 `sudo apt-get install` 命令直接获取软件，然后输入您要安装的程序名称:

Bash

```bash
sudo apt-get install <app_name>
```

若要更新已安装的程序，可以运行：

Bash

```bash
sudo apt update && sudo apt upgrade
```

![升级和更新](https://user-images.githubusercontent.com/98557455/183468063-35b00e76-d11a-4260-aa3c-9f8e0dab2e47.gif)

 提示

Linux 的不同发行版通常具有不同的包管理器，需要使用特定于关联包管理器的安装命令。 例如，Arch Linux 的主包管理器被称为 [`pacman`](https://wiki.archlinux.org/title/Pacman)，安装命令将是 `sudo pacman -S <app_name>`。 openSUSE 的主包管理器叫做 [`Zypper`](https://doc.opensuse.org/documentation/tumbleweed/zypper/)，安装命令是 `sudo zypper install <app_name>`。 Alpine 的主包管理器称为 [Alpine Package Keeper （`apk`），](https://wiki.alpinelinux.org/wiki/Alpine_Package_Keeper) 安装命令将是 `sudo apk add <app_name>`。 Red Hat 分发版（如 CentOS）的两个主要包管理器是 [YUM 和 RPM](https://www.redhat.com/blog/how-manage-packages) ，安装命令可以是 `sudo yum install <app_name>` 或 `sudo rpo -i <app_name>`。 请参阅你正在使用的分发文档，了解可用于安装和更新软件的工具。



## 处理文件和目录

若要查看当前正在使用的目录的路径，请使用 `pwd` 以下命令：

Bash

```bash
pwd
```

若要创建新目录，请使用命令 `mkdir`，后接您要创建的目录名称。

Bash

```bash
mkdir hello_world
```

若要更改目录，请使用 `cd` 命令后跟要导航到的目录的名称：

Bash

```bash
cd hello_world
```

若要查看当前正在使用的目录中的内容，请键入 `ls` 命令行：

Bash

```bash
ls
```

![directory-and-file-commands1](https://user-images.githubusercontent.com/98557455/183470971-7b188fdd-bb01-44e0-ac17-56f246ffd78e.gif)

默认情况下，该 `ls` 命令将仅打印所有文件和目录的名称。 若要获取其他信息，例如上次修改文件时或文件权限，请使用标志 `-l`：

Bash

```bash
ls -l
```

可以通过 `touch` 命令创建一个新文件，后面接要创建的文件名称：

Bash

```bash
touch hello_world.txt
```

可以使用任何下载的图形文本编辑器或 VS Code 远程 – WSL 扩展编辑文件。 可以 [在此处](https://learn.microsoft.com/zh-cn/windows/wsl/tutorials/wsl-vscode)了解有关 VS Code 入门的详细信息。

如果希望直接从命令行编辑文件，则需要使用命令行编辑器，例如 `vim`， `emacs`或 `nano`。 许多分发版都安装了其中一个或多个程序，但你始终可以按照 [上述](https://github.com/MicrosoftDocs/WSL/edit/linux-tutorial/WSL/tutorials/linux.md#installing-and-updating-software)指南中概述的安装说明来安装这些程序。

若要使用首选的编辑方法编辑文件，只需运行程序名称，然后运行要编辑的文件的名称：

Bash

```bash
code hello_world.txt
```

Bash

```bash
notepad.exe hello_world.txt
```

要在命令行中查看文件的内容，请使用 `cat` 命令，然后指定您要读取的文件：

Bash

```bash
cat hello_world.txt
```

![目录和文件命令2](https://user-images.githubusercontent.com/98557455/183481394-25bc0b2f-3d6d-465f-8f0b-aa5393f88727.gif)



## 使用管道和重定向运算符

管道 `|` 将一个命令的输出作为输入重定向到另一个命令。 例如， `lhscmd | rhscmd` 将输出定向 `lhscmd` 到 `rhscmd`。 可以通过多种方式使用管道，通过命令行快速完成任务。 下面是有关如何使用管道的几个简单示例。

假设你想要快速对文件的内容进行排序。 请看下面的 fruits.txt 示例：

Bash

```bash
$ cat fruits.txt
Orange
Banana
Apple
Pear
Plum
Kiwi
Strawberry
Peach
```

可以使用管道快速对此列表进行排序：

Bash

```bash
$ cat fruits.txt | sort
Apple
Banana
Kiwi
Orange
Peach
Pear
Plum
Strawberry
```

默认情况下，命令的 `cat` 输出将发送到标准输出;但是， `|` 允许我们改为将输出作为输入重定向到另一个命令 `sort`。

另一个用例是搜索。 可以使用 `grep` 一个有用的命令来搜索特定搜索字符串的输入。

Bash

```bash
cat fruits.txt | grep P
Pear
Plum
Peach
```

还可以使用重定向运算符，例如 `>` 将输出传递给文件或流。 例如，如果要使用 fruit.txt的已排序内容创建新的 .txt 文件：

Bash

```bash
$ cat fruits.txt | sort > sorted_fruit.txt
$ cat sorted_fruit.txt
Apple
Banana
Kiwi
Orange
Peach
Pear
Plum
Strawberry
```

默认情况下，命令的 `sort` 输出将发送到标准输出;但是， `>` 运算符允许我们改为将输出重定向到名为 sorted_fruits.txt的新文件中。

可以通过多种有趣的方式使用管道和重定向运算符，以便直接从命令行完成任务。