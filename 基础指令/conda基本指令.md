这里为您整理了一份清晰的 Conda 常用命令速查表，方便您快速上手和管理您的Python环境。✨

### ⚙️ Conda 核心操作速查

| 功能类别       | 命令示例                                      | 作用说明                                                     |
| :------------- | :-------------------------------------------- | :----------------------------------------------------------- |
| **🔍 信息查看** | `conda --version`或 `conda -V`                | 查看Conda版本，验证是否安装成功。                            |
|                | `conda info --envs`或 `conda env list`        | **查看所有已创建的虚拟环境**。当前激活的环境前会带有星号 `*`。 |
| **🐍 环境管理** | `conda create --name myenv python=3.9`        | **创建环境**。创建一个名为 `myenv`且Python版本为3.9的新环境。 |
|                | `conda activate myenv`                        | **激活环境**。进入名为 `myenv`的环境，之后的操作（如安装包）都将在此环境中进行。 |
|                | `conda deactivate`                            | **退出环境**。退出当前激活的环境，返回基础环境（base）。     |
|                | `conda remove --name myenv --all`             | **删除环境**。彻底删除名为 `myenv`的环境及其中的所有包。     |
| **📦 包管理**   | `conda list`                                  | 列出当前激活环境中所有已安装的包。                           |
|                | `conda install numpy`                         | **安装包**。为当前环境安装 `numpy`包。                       |
|                | `conda install -n myenv numpy`                | 为指定的 `myenv`环境安装包，无需先激活该环境。               |
|                | `conda remove numpy`                          | 从当前环境中移除 `numpy`包。                                 |
|                | `conda update numpy`                          | 更新当前环境中的 `numpy`包。                                 |
| **🔄 高级技巧** | `conda env export > environment.yml`          | **导出环境**。将当前环境的精确配置导出到 `environment.yml`文件，便于共享和复现。 |
|                | `conda env create -f environment.yml`         | **从文件创建环境**。根据 `environment.yml`文件创建一个一模一样的新环境。 |
|                | `conda config --set auto_activate_base false` | **设置不默认进入base环境**。让终端启动时不自动激活基础环境。 |

### 💡 实用提示

1. **关于包管理工具的选择**：在Conda环境中，如果某个包无法通过 `conda install`安装，可以尝试使用 `pip install`。但通常建议**优先使用Conda命令**，因为它能更好地处理包之间的依赖关系，特别是涉及非Python库（如CUDA、MKL）时

   

2. **加速下载**：在国内下载包可能会比较慢，可以通过配置国内镜像源（如清华源）来大幅提升下载速度。配置命令可以在搜索结果中找到

   

3. **环境重命名**：Conda没有直接重命名环境的命令，但可以通过克隆旧环境再删除旧环境的方式实现：`conda create --name new_env --clone old_env`，然后 `conda remove --name old_env --all`