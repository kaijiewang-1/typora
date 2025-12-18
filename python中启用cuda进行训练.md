在 Python 中启用 GPU（CUDA）进行训练，主要是为了让深度学习框架（如 PyTorch、TensorFlow 等）利用 NVIDIA 显卡的并行计算能力加速训练。以下是详细的步骤和注意事项，适合新手一步步操作：

### **一、前置条件：硬件与软件准备**

要让 Python 启用 CUDA 加速，首先需要满足以下硬件和软件要求：

#### 1. **硬件要求**

- **NVIDIA 显卡**：必须拥有一块支持 CUDA 的 NVIDIA 显卡（如 GTX 10 系列及以上，RTX 20/30/40 系列等）。
  *如何检查？* 打开命令行输入 `nvidia-smi`（需先安装显卡驱动），若能显示显卡信息则表示支持。

#### 2. **软件要求**

- **显卡驱动**：安装 NVIDIA 官方最新驱动（下载地址）。
- **CUDA Toolkit**：NVIDIA 提供的并行计算平台（需与显卡驱动兼容）。
  *如何选择版本？* 参考深度学习框架的官方文档（如 PyTorch 2.0 推荐 CUDA 11.7+）。
  CUDA Toolkit 下载地址。
- **cuDNN**：NVIDIA 针对深度神经网络优化的加速库（需与 CUDA 版本匹配）。
  cuDNN 下载地址（需注册账号）。

### **二、安装支持 CUDA 的深度学习框架**

以最常用的 **PyTorch** 和 **TensorFlow** 为例，安装时需选择 GPU 版本。

#### **1. 安装 PyTorch（推荐）**

PyTorch 对 CUDA 的支持非常友好，安装时直接选择对应 CUDA 版本的命令即可。
*步骤：*

- 访问

   

  PyTorch 官网

  ，根据你的 CUDA 版本（通过

   

  ```
  nvidia-smi
  ```

   

  查看，如 CUDA 11.8）选择安装命令。

  示例（CUDA 11.8）：

  bash

  复制

  ```bash
  pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu118
  ```

- 安装完成后，验证是否启用 CUDA：

  python

  运行

  复制

  ```python
  import torch
  print(torch.cuda.is_available())  # 输出 True 表示成功启用 GPU
  print(torch.cuda.get_device_name(0))  # 显示你的显卡型号（如 NVIDIA RTX 3090）
  ```

#### **2. 安装 TensorFlow**

TensorFlow 的 GPU 版本需要与 CUDA、cuDNN 版本严格匹配（例如 TF 2.15 支持 CUDA 12.3）。
*步骤：*

- 查看 TensorFlow 官方 版本兼容表，选择与你的 CUDA 版本匹配的 TF 版本。

- 用 pip 安装 GPU 版本（示例安装 TF 2.15）：

  bash

  复制

  ```bash
  pip install tensorflow[and-cuda]==2.15.0
  ```

- 验证是否启用 CUDA：

  python

  运行

  复制

  ```python
  import tensorflow as tf
  print(tf.config.list_physical_devices('GPU'))  # 输出 GPU 信息表示成功
  ```

### **三、代码中启用 GPU 加速**

安装好框架后，需要在代码中显式将模型、数据（张量）移动到 GPU 上，框架才会使用 GPU 计算。

#### **PyTorch 示例**

python

运行

复制

```python
import torch

# 检查是否有可用的 GPU
device = torch.device("cuda" if torch.cuda.is_available() else "cpu")
print(f"使用设备：{device}")

# 创建模型并移动到 GPU
model = torch.nn.Linear(10, 2).to(device)  # .to(device) 关键！

# 创建数据并移动到 GPU
x = torch.randn(100, 10).to(device)
y = torch.randn(100, 2).to(device)

# 训练时，所有操作会自动在 GPU 上执行
output = model(x)
loss = torch.nn.functional.mse_loss(output, y)
loss.backward()
```

#### **TensorFlow 示例**

TensorFlow 2.x 默认会自动使用 GPU（如果可用），但也可以显式指定设备：

python

运行

复制

```python
import tensorflow as tf

# 检查 GPU 是否可用
print("可用 GPU 数量：", len(tf.config.list_physical_devices('GPU')))

# 显式指定使用 GPU（可选，通常自动分配）
with tf.device("/GPU:0"):  # 使用第一块 GPU
    # 定义模型
    model = tf.keras.Sequential([
        tf.keras.layers.Dense(64, activation='relu'),
        tf.keras.layers.Dense(1)
    ])
    model.compile(optimizer='adam', loss='mse')

    # 训练（数据会自动加载到 GPU）
    model.fit(x_train, y_train, epochs=10)
```

### **四、常见问题与解决**

1. **`torch.cuda.is_available()` 返回 False**
   - 检查 CUDA 版本与 PyTorch 版本是否匹配（例如 PyTorch 2.0 可能需要 CUDA 11.7+）。
   - 确认显卡驱动已正确安装（`nvidia-smi` 能正常显示显卡信息）。
   - 重新安装匹配的 PyTorch 版本（可能之前的安装包损坏）。
2. **TensorFlow 找不到 GPU**
   - 检查 TensorFlow 版本与 CUDA/cuDNN 是否兼容（参考 TF 版本兼容表）。
   - 确保 cuDNN 已正确安装（将 cuDNN 的 `bin`、`include`、`lib` 目录添加到系统环境变量 `PATH`）。
3. **GPU 显存不足**
   - 减少 batch size（批量大小）。
   - 使用 `torch.cuda.empty_cache()`（PyTorch）或 `tf.keras.backend.clear_session()`（TensorFlow）释放无用显存。

### **总结**

启用 GPU 加速的核心步骤：

1. 确认硬件（NVIDIA 显卡）和软件（驱动、CUDA、cuDNN）安装正确。
2. 安装支持 CUDA 的深度学习框架（如 PyTorch/TensorFlow GPU 版）。
3. 在代码中将模型和数据移动到 GPU 设备（PyTorch 用 `.to(device)`，TensorFlow 自动或显式指定）。

完成后，训练速度会显著提升（尤其是大模型或大数据集）！ 🚀