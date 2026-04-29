# YOLOv8 图像分类任务训练操作手册

**适用环境：Windows + RTX 4060 Laptop + Ultralytics CLI + 预训练分类模型**

本文档面向实际项目落地，目标是让工程师能够完成：

**数据准备 → 训练 → 验证 → 推理 → 导出 → 排查 → 调参优化**

任务类型为 **image classification**，不是 detection / segmentation。Ultralytics 官方分类数据集采用 `train / val / test` 目录，每个 split 下按类别建立子文件夹；分类训练可直接指定数据集根目录，不需要检测任务常见的 `data.yaml`。

------

## 0. 项目约定

### 0.1 推荐目录

```
D:\projects\yolov8_cls_project\
├─ dataset\
│  ├─ train\
│  │  ├─ normal\
│  │  └─ abnormal\
│  ├─ val\
│  │  ├─ normal\
│  │  └─ abnormal\
│  └─ test\
│     ├─ normal\
│     └─ abnormal\
├─ runs\
└─ scripts\
```

### 0.2 类别命名建议

建议使用英文目录名：

```
normal
abnormal
```

不建议直接使用中文目录名：

```
正常
异常
```

原因：Windows、Python 包、ONNX 推理端、部署服务之间可能存在编码差异。工程中建议用英文类别名，最终业务展示时再映射为中文。

------

# 1. 环境准备与依赖检查

## 1.1 安装 Ultralytics

进入你的 Python 环境后执行：

```
pip install -U ultralytics
```

检查安装是否成功：

```
yolo version
```

也可以查看 Python 中是否能导入：

```
python -c "import ultralytics; print(ultralytics.__version__)"
```

Ultralytics 官方支持通过 CLI 使用 `yolo` 命令进行 train、val、predict、export 等模式操作。

------

## 1.2 检查 PyTorch 与 CUDA 是否可用

执行：

```
python -c "import torch; print('torch:', torch.__version__); print('cuda available:', torch.cuda.is_available()); print('cuda version:', torch.version.cuda); print('gpu:', torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')"
```

期望输出类似：

```
torch: 2.x.x+cuXXX
cuda available: True
cuda version: 12.x
gpu: NVIDIA GeForce RTX 4060 Laptop GPU
```

若 `cuda available: False`，训练会走 CPU，速度会非常慢。

------

## 1.3 检查 YOLO 是否识别 GPU

执行一个最小训练前检查：

```
yolo checks
```

如果环境正常，应能看到 Python、PyTorch、CUDA、GPU 等信息。

------

## 1.4 Windows 环境常见坑

### 1.4.1 路径建议使用英文路径

推荐：

```
D:\projects\yolov8_cls_project\dataset
```

不推荐：

```
D:\项目\图像分类\数据集
```

虽然很多情况下中文路径可以工作，但工程部署、日志、ONNX 导出、第三方推理框架中更容易出问题。

------

### 1.4.2 路径包含空格时必须加引号

错误示例：

```
yolo classify train data=D:\my project\dataset model=yolov8n-cls.pt
```

正确示例：

```
yolo classify train data="D:\my project\dataset" model=yolov8n-cls.pt
```

------

### 1.4.3 权限问题

不要把项目放在：

```
C:\Program Files\
C:\Windows\
```

建议放在：

```
D:\projects\
C:\Users\<你的用户名>\projects\
```

否则可能出现日志无法写入、权重无法保存、缓存文件创建失败等问题。

------

### 1.4.4 CUDA 版本冲突

典型现象：

```
torch.cuda.is_available() = False
```

或训练时报：

```
CUDA error
DLL load failed
```

排查顺序：

```
nvidia-smi
python -c "import torch; print(torch.__version__, torch.version.cuda, torch.cuda.is_available())"
```

原则：

1. `nvidia-smi` 能看到 RTX 4060。
2. PyTorch 是 CUDA 版本，不是 CPU-only 版本。
3. 不要求本机 CUDA Toolkit 版本与 PyTorch 完全一致，但 NVIDIA 驱动必须支持当前 PyTorch CUDA 运行时。

------

# 2. 数据集准备规范

## 2.1 正确的 YOLOv8 分类目录结构

Ultralytics 分类任务使用按类别文件夹组织的数据结构。官方分类数据集规范为 `train`、`test`，可选 `val`，每个目录下按类别名建立子目录。

你的项目应组织为：

```
dataset/
├─ train/
│  ├─ normal/
│  │  ├─ 000001.jpg
│  │  ├─ 000002.jpg
│  │  └─ ...
│  └─ abnormal/
│     ├─ 000001.jpg
│     ├─ 000002.jpg
│     └─ ...
├─ val/
│  ├─ normal/
│  └─ abnormal/
└─ test/
   ├─ normal/
   └─ abnormal/
```

------

## 2.2 分类任务不需要 data.yaml

**结论：YOLOv8 分类任务不需要 `data.yaml`。**

训练时直接把 `data` 指向数据集根目录即可：

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt
```

不要写成检测任务形式：

```
yolo detect train data=data.yaml model=yolov8n.pt
```

分类任务应使用：

```
yolo classify train ...
```

并使用分类模型：

```
yolov8n-cls.pt
yolov8s-cls.pt
yolov8m-cls.pt
...
```

------

## 2.3 当前数据划分建议

总数据量：1500 张。

比例：

```
train : val : test = 7 : 2 : 1
```

建议数量：

| 类别     | 总数 | train 70% | val 20% | test 10% |
| -------- | ---- | --------- | ------- | -------- |
| normal   | 1000 | 700       | 200     | 100      |
| abnormal | 500  | 350       | 100     | 50       |
| 合计     | 1500 | 1050      | 300     | 150      |

------

## 2.4 数据检查要点

### 2.4.1 检查类别数量

PowerShell 执行：

```
Get-ChildItem "D:\projects\yolov8_cls_project\dataset\train\normal" -File | Measure-Object
Get-ChildItem "D:\projects\yolov8_cls_project\dataset\train\abnormal" -File | Measure-Object

Get-ChildItem "D:\projects\yolov8_cls_project\dataset\val\normal" -File | Measure-Object
Get-ChildItem "D:\projects\yolov8_cls_project\dataset\val\abnormal" -File | Measure-Object

Get-ChildItem "D:\projects\yolov8_cls_project\dataset\test\normal" -File | Measure-Object
Get-ChildItem "D:\projects\yolov8_cls_project\dataset\test\abnormal" -File | Measure-Object
```

------

### 2.4.2 检查错误标签

分类任务的标签来自文件夹名，因此错误标签通常表现为：

```
normal 文件夹中混入 abnormal 图片
abnormal 文件夹中混入 normal 图片
```

工程建议：

1. 每个类别随机抽样 50～100 张人工复核。
2. 重点检查模型预测错误的样本。
3. 训练后查看混淆矩阵，把 `abnormal → normal` 的错分图片重点复核。

可先用 Windows 文件资源管理器人工检查，也可以用训练后预测结果筛选低置信度样本。

------

### 2.4.3 检查损坏图片

执行：

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=1 imgsz=224 batch=16 device=0
```

如果存在损坏图片，训练初始化阶段通常会报错或跳过异常样本。先跑 1 个 epoch 是工程上常用的“数据冒烟测试”。

------

## 2.5 是否建议数据增强

**结论：建议使用适度数据增强，但不要过强。**

原因：

当前数据只有 1500 张，其中异常类 500 张，属于中小规模二分类任务。适度增强有助于提升泛化能力，尤其是异常样本较少时。

但不建议盲目使用过强增强。分类任务中，如果异常特征依赖细节纹理、微小缺陷、局部颜色变化，过强的旋转、裁剪、颜色扰动可能破坏标签语义。

建议起步：

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=80 imgsz=224 batch=32 device=0 degrees=5 translate=0.05 scale=0.1 fliplr=0.5
```

如果异常方向有业务含义，例如上下颠倒后不再合理，则不要开启大角度旋转或翻转。

------

# 3. 模型选择与初始化

## 3.1 必须使用预训练分类模型

本项目要求使用预训练模型。推荐起步模型：

```
yolov8n-cls.pt
```

训练命令中的模型参数应写为：

```
model=yolov8n-cls.pt
```

不要使用：

```
model=yolov8n.pt
```

`yolov8n.pt` 是检测模型，不是分类模型。

Ultralytics 分类模型会对整张图片输出一个类别标签和置信度，适合图像级正常 / 异常判断。

------

## 3.2 模型规模说明

YOLOv8 分类模型通常有以下规模：

| 模型             | 特点                   | 适用场景                     |
| ---------------- | ---------------------- | ---------------------------- |
| `yolov8n-cls.pt` | 最小、最快、显存占用低 | 小数据集、快速验证、边缘部署 |
| `yolov8s-cls.pt` | 精度和速度平衡         | 数据较干净、希望提升精度     |
| `yolov8m-cls.pt` | 更强表达能力           | 类别差异细微、数据量更多     |
| `yolov8l-cls.pt` | 精度更强、显存更高     | 较大数据集、复杂业务         |
| `yolov8x-cls.pt` | 最大、最慢             | 高精度离线场景、大数据集     |

Ultralytics YOLOv8 系列包含不同规模模型，并支持训练、验证、推理和导出等工程流程。

------

## 3.3 当前项目推荐模型

当前数据：

```
总量：1500 张
类别：2 类
类别比例：normal 1000 / abnormal 500
GPU：RTX 4060 Laptop
```

推荐策略：

### 第一阶段：基线模型

使用：

```
yolov8n-cls.pt
```

原因：

1. 数据量不大。
2. 二分类任务不需要一开始就上大模型。
3. 训练快，便于快速发现数据问题。
4. 对 RTX 4060 Laptop 友好。

------

### 第二阶段：精度提升

如果 `yolov8n-cls.pt` 出现以下情况，可以尝试：

```
yolov8s-cls.pt
```

适用条件：

1. 验证集准确率已经接近收敛，但异常类仍有明显漏判。
2. 训练集和验证集差距不大，说明不是明显过拟合。
3. 错误样本确实存在细粒度差异，需要更强特征表达。

------

### 不建议一开始使用大模型

不建议直接使用：

```
yolov8m-cls.pt
yolov8l-cls.pt
yolov8x-cls.pt
```

原因：

1. 1500 张数据容易过拟合。
2. 训练更慢。
3. 调参周期变长。
4. 工程收益未必明显。

------

# 4. 训练流程

## 4.1 训练前冒烟测试

先跑 1 个 epoch，确认数据路径、GPU、类别读取均正常：

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=1 imgsz=224 batch=16 device=0 workers=4 project="D:\projects\yolov8_cls_project\runs" name=smoke_test
```

检查点：

1. 是否使用 CUDA。
2. 是否识别到 2 个类别。
3. 是否能正常读取 train / val。
4. 是否能保存结果到 `runs`。

------

## 4.2 基础训练命令，可直接运行

推荐首轮正式训练：

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=80 imgsz=224 batch=32 device=0 workers=4 project="D:\projects\yolov8_cls_project\runs" name=yolov8n_cls_baseline pretrained=True
```

训练完成后，权重通常保存在：

```
D:\projects\yolov8_cls_project\runs\yolov8n_cls_baseline\weights\best.pt
D:\projects\yolov8_cls_project\runs\yolov8n_cls_baseline\weights\last.pt
```

工程上优先使用：

```
best.pt
```

------

## 4.3 带轻量增强的推荐训练命令

如果确认数据读取无误，建议使用以下命令作为正式版本：

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=100 imgsz=224 batch=32 device=0 workers=4 project="D:\projects\yolov8_cls_project\runs" name=yolov8n_cls_aug pretrained=True patience=20 degrees=5 translate=0.05 scale=0.1 fliplr=0.5
```

说明：

1. `patience=20`：验证指标长时间不提升时提前停止。
2. `degrees=5`：小角度旋转。
3. `translate=0.05`：轻微平移。
4. `scale=0.1`：轻微缩放。
5. `fliplr=0.5`：50% 概率水平翻转。

如果业务图像不允许水平翻转，例如左右方向有明确含义，则设置：

```
fliplr=0.0
```

------

## 4.4 参数说明

### epochs

推荐：

```
80～120
```

当前数据量不大，起步使用：

```
epochs=100
```

判断依据：

1. 如果 30～50 epoch 后验证准确率不再提升，可减少 epochs 或依赖 `patience`。
2. 如果训练仍在提升且验证集同步提升，可适当增加到 150。
3. 如果训练准确率升高但验证准确率下降，说明过拟合，不应继续增加 epochs。

------

### batch

RTX 4060 Laptop 常见显存为 8GB，也可能存在不同功耗和显存配置。分类任务 `imgsz=224` 下，建议：

```
batch=32 起步
batch=64 可尝试
batch=16 保守稳定
```

推荐首轮：

```
batch=32
```

如果显存溢出，改为：

```
batch=16
```

如果显存余量很大，尝试：

```
batch=64
```

------

### imgsz

分类任务常用：

```
224
```

推荐：

```
imgsz=224
```

如果异常特征很小、依赖细节，可尝试：

```
imgsz=320
```

但 `imgsz=320` 会增加显存占用和训练时间。

工程建议：

1. 第一轮：`imgsz=224`
2. 若异常类漏判严重且缺陷很小：尝试 `imgsz=320`
3. 不建议一开始直接 `imgsz=640`，分类任务通常没有必要

------

### device

使用第一张 NVIDIA GPU：

```
device=0
```

强制 CPU：

```
device=cpu
```

本项目应使用：

```
device=0
```

------

### workers

Windows 下数据加载多进程有时比 Linux 更容易出问题。建议：

```
workers=4
```

如果卡住、报错或训练启动异常，改成：

```
workers=0
```

稳定后再尝试：

```
workers=2
workers=4
```

------

## 4.5 Windows 路径写法

推荐写法：

```
data="D:\projects\yolov8_cls_project\dataset"
project="D:\projects\yolov8_cls_project\runs"
```

也可以使用 `/`：

```
data="D:/projects/yolov8_cls_project/dataset"
project="D:/projects/yolov8_cls_project/runs"
```

不要混用未转义路径和空格。

------

# 5. 训练过程解读

## 5.1 训练日志中的核心信息

训练时通常会看到类似信息：

```
Epoch    GPU_mem    train/loss    metrics/accuracy_top1    metrics/accuracy_top5
1/100    1.2G       0.623         0.780                    1.000
...
```

不同版本日志字段可能略有差异，但分类任务重点关注：

```
train/loss
val/loss
metrics/accuracy_top1
metrics/accuracy_top5
```

------

## 5.2 loss 含义

`loss` 表示模型预测结果与真实类别之间的差异。

工程判断：

| 现象                           | 说明                                     |
| ------------------------------ | ---------------------------------------- |
| train loss 持续下降            | 模型正在学习                             |
| train loss 不下降              | 学习率、数据标签、模型配置可能有问题     |
| train loss 下降，val loss 上升 | 可能过拟合                               |
| train loss 和 val loss 都很高  | 数据难、标签错、模型不足或训练参数不合适 |

------

## 5.3 accuracy 含义

分类任务中 accuracy 表示预测类别是否与真实类别一致。

例如验证集 300 张，预测正确 270 张：

```
accuracy = 270 / 300 = 90%
```

------

## 5.4 Top-1 accuracy

Top-1 accuracy 是分类任务的核心指标。

含义：

```
模型置信度最高的类别是否等于真实类别
```

对于你的二分类任务：

```
normal / abnormal
```

Top-1 accuracy 就是最直观的分类准确率。

工程上需要注意：

```
总体 Top-1 accuracy 高，不代表 abnormal 类识别一定好。
```

因为你的数据存在类别不平衡：

```
normal: 1000
abnormal: 500
```

如果模型偏向 normal，总体准确率可能仍然看起来不低，但 abnormal 漏检严重。

------

## 5.5 Top-5 accuracy

Top-5 accuracy 表示真实类别是否出现在模型置信度前 5 的类别中。

对于二分类任务，Top-5 accuracy 没有实际区分意义，通常会接近或等于 1.0。

本项目重点看：

```
Top-1 accuracy
confusion matrix
abnormal 类召回情况
```

------

## 5.6 如何判断是否收敛

认为模型基本收敛的条件：

1. `train/loss` 下降趋缓。
2. `val/loss` 不再明显下降。
3. `accuracy_top1` 多个 epoch 内提升很小。
4. `best.pt` 长时间不再更新。

工程建议：

如果连续 20 个 epoch 验证指标无明显提升，使用：

```
patience=20
```

让训练自动提前停止。

------

## 5.7 如何判断是否过拟合

典型过拟合表现：

| 指标             | 表现             |
| ---------------- | ---------------- |
| train loss       | 持续下降         |
| train accuracy   | 持续升高         |
| val loss         | 不降反升         |
| val accuracy     | 停滞或下降       |
| confusion matrix | 某类错误持续严重 |

处理方法：

1. 减少 epochs。
2. 使用 `patience`。
3. 增加轻量数据增强。
4. 清理错误标签。
5. 换小模型，例如从 `yolov8s-cls.pt` 回退到 `yolov8n-cls.pt`。
6. 增加异常类真实样本。

------

# 6. 模型评估与验证

## 6.1 使用 best.pt 在验证集上评估

训练后执行：

```
yolo classify val model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" data="D:\projects\yolov8_cls_project\dataset" imgsz=224 batch=32 device=0
```

Ultralytics 支持通过 CLI 使用 `val` 模式验证模型表现。

------

## 6.2 使用 test 集评估

如果你的 `dataset` 中包含 `test`，可以执行：

```
yolo classify val model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" data="D:\projects\yolov8_cls_project\dataset" split=test imgsz=224 batch=32 device=0
```

工程建议：

1. `val` 用于训练过程调参。
2. `test` 只用于最终验收。
3. 不要反复用 test 集调参，否则 test 指标会失真。

------

## 6.3 关键指标解释

### Top-1 accuracy

核心指标：

```
预测第一名类别是否正确
```

二分类项目中主要看它。

------

### Top-5 accuracy

二分类项目中参考意义弱。

因为只有 2 类，真实类别几乎必然在 Top-5 中。

------

## 6.4 Confusion Matrix 解读

训练或验证完成后，结果目录中通常会生成混淆矩阵图，例如：

```
runs\yolov8n_cls_aug\confusion_matrix.png
```

二分类混淆矩阵重点关注：

| 真实类别 | 预测 normal    | 预测 abnormal  |
| -------- | -------------- | -------------- |
| normal   | 正常识别正确   | 正常误判为异常 |
| abnormal | 异常漏判为正常 | 异常识别正确   |

本项目最关键的是：

```
abnormal 被预测成 normal 的数量
```

这代表异常漏检。

如果业务目标是异常检测，异常漏检通常比正常误报更严重。

------

## 6.5 工程验收建议

不要只看整体 accuracy。建议记录：

```
overall accuracy
normal accuracy
abnormal accuracy
abnormal recall
abnormal false negative count
```

如果异常类测试集 50 张，其中 10 张被预测为 normal：

```
abnormal recall = 40 / 50 = 80%
```

这说明异常漏检率：

```
20%
```

对很多工业质检场景来说可能不可接受。

------

# 7. 推理与预测

## 7.1 单张图片预测

```
yolo classify predict model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" source="D:\projects\yolov8_cls_project\dataset\test\abnormal\000001.jpg" imgsz=224 device=0
```

输出会包含预测类别和置信度。

示例：

```
abnormal 0.93
normal 0.07
```

表示模型认为该图片是 abnormal，置信度为 0.93。

------

## 7.2 文件夹批量预测

```
yolo classify predict model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" source="D:\projects\yolov8_cls_project\dataset\test" imgsz=224 device=0 save=True
```

预测结果通常保存在：

```
runs\classify\predict\
```

如果想指定输出目录：

```
yolo classify predict model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" source="D:\projects\yolov8_cls_project\dataset\test" imgsz=224 device=0 save=True project="D:\projects\yolov8_cls_project\runs" name=predict_test
```

------

## 7.3 对业务图片目录预测

例如生产图片目录：

```
yolo classify predict model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" source="D:\production_images" imgsz=224 device=0 save=True project="D:\projects\yolov8_cls_project\runs" name=predict_production
```

------

# 8. 模型导出

Ultralytics 支持将训练好的模型导出为多种部署格式，CLI 中使用 `yolo export`，并通过 `format` 指定目标格式。

------

## 8.1 导出 ONNX

```
yolo export model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" format=onnx imgsz=224 opset=12 simplify=True
```

导出后通常生成：

```
best.onnx
```

### ONNX 适用场景

适合：

1. C++ 推理服务。
2. ONNX Runtime。
3. OpenVINO。
4. TensorRT 转换前中间格式。
5. 跨平台部署。

工程建议：

```
Windows / Linux 服务端部署优先考虑 ONNX。
```

------

## 8.2 导出 TorchScript

```
yolo export model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" format=torchscript imgsz=224
```

导出后通常生成：

```
best.torchscript
```

### TorchScript 适用场景

适合：

1. PyTorch 生态内部署。
2. Python 服务推理。
3. C++ LibTorch 推理。
4. 与 PyTorch 训练链路保持一致的项目。

工程建议：

```
如果部署端已经使用 PyTorch，优先考虑 TorchScript。
如果部署端追求跨框架兼容，优先考虑 ONNX。
```

TorchScript 是 PyTorch 生态中的模型序列化与部署方式，可用于没有完整 Python 训练代码的运行环境。

------

# 9. 常见问题与排查

## 9.1 类别不平衡导致模型偏向 normal

### 现象

1. 总体 accuracy 不低。
2. abnormal 经常被预测为 normal。
3. confusion matrix 中 `abnormal → normal` 数量较多。

### 原因

当前数据：

```
normal: 1000
abnormal: 500
```

normal 是 abnormal 的 2 倍，模型可能倾向于预测 normal。

### 解决方法

优先级从高到低：

#### 方法 1：补充 abnormal 样本

最有效。

目标比例建议：

```
normal : abnormal = 1 : 1
```

或至少：

```
normal : abnormal <= 1.5 : 1
```

------

#### 方法 2：对 abnormal 做数据增强

建议对异常类补充增强样本，但要保证增强后仍符合业务语义。

可做：

```
轻微旋转
轻微平移
轻微缩放
亮度轻微扰动
轻微模糊
```

慎用：

```
大角度旋转
过度裁剪
强颜色变化
上下翻转
```

------

#### 方法 3：重点分析异常漏检样本

对预测错误样本做复核：

```
yolo classify predict model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" source="D:\projects\yolov8_cls_project\dataset\test\abnormal" imgsz=224 device=0 save=True project="D:\projects\yolov8_cls_project\runs" name=check_abnormal_errors
```

重点查看被预测成 normal 的图片，判断：

1. 是否标错。
2. 是否异常特征太弱。
3. 是否图片质量差。
4. 是否需要更高分辨率。
5. 是否需要采集更多同类异常。

------

## 9.2 训练不收敛

### 现象

```
loss 不下降
accuracy 长期低
预测结果接近随机
```

### 排查

#### 检查数据结构

```
dataset/train/normal
dataset/train/abnormal
dataset/val/normal
dataset/val/abnormal
```

#### 检查是否误用了检测模型

错误：

```
model=yolov8n.pt
```

正确：

```
model=yolov8n-cls.pt
```

#### 检查学习率

可以尝试降低学习率：

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=100 imgsz=224 batch=32 device=0 lr0=0.001
```

#### 检查标签质量

如果 normal / abnormal 边界本身不清晰，模型会很难收敛。

------

## 9.3 GPU 未被使用

### 现象

训练很慢，日志中显示 CPU，或任务管理器 GPU 占用低。

### 排查命令

```
nvidia-smi
python -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')"
```

### 训练时显式指定 GPU

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=100 imgsz=224 batch=32 device=0
```

如果仍然不用 GPU，通常是 PyTorch CUDA 版本安装问题。

------

## 9.4 CUDA 版本冲突

### 现象

```
CUDA unavailable
CUDA error
DLL load failed
```

### 处理步骤

1. 确认驱动：

```
nvidia-smi
```

1. 确认 PyTorch CUDA：

```
python -c "import torch; print(torch.__version__); print(torch.version.cuda); print(torch.cuda.is_available())"
```

1. 如果 PyTorch 是 CPU 版，需要重新安装支持 CUDA 的 PyTorch。

------

## 9.5 Windows 路径问题

### 现象

```
FileNotFoundError
Dataset not found
No images found
```

### 解决

使用绝对路径：

```
data="D:\projects\yolov8_cls_project\dataset"
```

路径有空格必须加引号：

```
data="D:\my project\dataset"
```

尽量避免中文路径和特殊字符。

------

## 9.6 batch 过大导致显存溢出

### 现象

```
CUDA out of memory
```

### 解决

从：

```
batch=64
```

改为：

```
batch=32
```

仍报错则改为：

```
batch=16
```

如果 `imgsz=320` 报错，先回退：

```
imgsz=224
```

------

# 10. 性能优化建议

## 10.1 类别不平衡处理

当前比例：

```
normal: 1000
abnormal: 500
```

这是 2:1，不算极端，但对异常检测业务已经需要关注。

推荐优先级：

### 优先级 1：增加 abnormal 样本

目标：

```
abnormal 从 500 增加到 800～1000
```

这是最有效的优化方式。

------

### 优先级 2：构建更合理的验证集和测试集

确保 val / test 中 abnormal 覆盖不同类型：

```
轻微异常
明显异常
边界异常
不同光照
不同背景
不同设备来源
```

不要让 test 集只包含容易样本。

------

### 优先级 3：异常类定向增强

对 abnormal 做轻量增强，尤其是：

```
亮度变化
轻微旋转
轻微缩放
轻微平移
少量模糊
```

不要增强出不符合业务真实分布的图片。

------

## 10.2 batch size 推荐

RTX 4060 Laptop 建议：

| imgsz | 推荐 batch | 说明           |
| ----- | ---------- | -------------- |
| 224   | 32         | 推荐起步       |
| 224   | 64         | 显存足够时尝试 |
| 320   | 16         | 更稳           |
| 320   | 32         | 显存足够时尝试 |

推荐命令：

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=100 imgsz=224 batch=32 device=0 workers=4
```

------

## 10.3 学习率建议

默认学习率通常可以作为第一轮基线。

如果训练不稳定，尝试：

```
lr0=0.001
```

命令：

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=100 imgsz=224 batch=32 device=0 lr0=0.001
```

如果训练很慢、loss 下降很弱，可以尝试：

```
lr0=0.005
```

但不要频繁同时改多个参数。工程上一次只改一个关键参数，方便复盘。

------

## 10.4 是否需要数据增强

**需要，但应适度。**

推荐配置：

```
degrees=5 translate=0.05 scale=0.1 fliplr=0.5
```

完整命令：

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=100 imgsz=224 batch=32 device=0 workers=4 project="D:\projects\yolov8_cls_project\runs" name=yolov8n_cls_aug pretrained=True patience=20 degrees=5 translate=0.05 scale=0.1 fliplr=0.5
```

如果异常方向敏感，改为：

```
fliplr=0.0
```

------

## 10.5 是否需要更换模型规模

### 当前推荐

```
首选 yolov8n-cls.pt
```

### 何时换 yolov8s-cls.pt

满足以下条件时尝试：

1. `yolov8n-cls.pt` 已经稳定收敛。
2. 训练集和验证集差距不大。
3. abnormal 漏检仍然偏多。
4. 错误样本不是明显标注错误。
5. GPU 显存允许。

命令：

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8s-cls.pt epochs=100 imgsz=224 batch=32 device=0 workers=4 project="D:\projects\yolov8_cls_project\runs" name=yolov8s_cls_aug pretrained=True patience=20 degrees=5 translate=0.05 scale=0.1 fliplr=0.5
```

### 何时不要换大模型

如果出现：

```
train accuracy 高
val accuracy 低
val loss 上升
```

这通常是过拟合，不应换更大模型。

应优先：

1. 清洗数据。
2. 增加异常样本。
3. 增强数据。
4. 降低 epochs。
5. 使用更小模型。

------

# 11. 推荐执行流程

## Step 1：检查环境

```
pip install -U ultralytics
yolo version
python -c "import torch; print(torch.__version__); print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')"
```

------

## Step 2：检查数据结构

```
dataset/
├─ train/
│  ├─ normal/
│  └─ abnormal/
├─ val/
│  ├─ normal/
│  └─ abnormal/
└─ test/
   ├─ normal/
   └─ abnormal/
```

------

## Step 3：冒烟测试

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=1 imgsz=224 batch=16 device=0 workers=4 project="D:\projects\yolov8_cls_project\runs" name=smoke_test
```

------

## Step 4：正式训练

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=100 imgsz=224 batch=32 device=0 workers=4 project="D:\projects\yolov8_cls_project\runs" name=yolov8n_cls_aug pretrained=True patience=20 degrees=5 translate=0.05 scale=0.1 fliplr=0.5
```

------

## Step 5：验证模型

```
yolo classify val model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" data="D:\projects\yolov8_cls_project\dataset" imgsz=224 batch=32 device=0
```

------

## Step 6：测试集评估

```
yolo classify val model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" data="D:\projects\yolov8_cls_project\dataset" split=test imgsz=224 batch=32 device=0
```

------

## Step 7：单张预测

```
yolo classify predict model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" source="D:\projects\yolov8_cls_project\dataset\test\abnormal\000001.jpg" imgsz=224 device=0
```

------

## Step 8：批量预测

```
yolo classify predict model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" source="D:\projects\yolov8_cls_project\dataset\test" imgsz=224 device=0 save=True project="D:\projects\yolov8_cls_project\runs" name=predict_test
```

------

## Step 9：导出 ONNX

```
yolo export model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" format=onnx imgsz=224 opset=12 simplify=True
```

------

## Step 10：导出 TorchScript

```
yolo export model="D:\projects\yolov8_cls_project\runs\yolov8n_cls_aug\weights\best.pt" format=torchscript imgsz=224
```

------

# 12. 最终工程建议

对于当前项目，推荐首个可落地版本采用：

```
模型：yolov8n-cls.pt
imgsz：224
batch：32
epochs：100
patience：20
device：0
workers：4
增强：轻量增强
```

首轮命令：

```
yolo classify train data="D:\projects\yolov8_cls_project\dataset" model=yolov8n-cls.pt epochs=100 imgsz=224 batch=32 device=0 workers=4 project="D:\projects\yolov8_cls_project\runs" name=yolov8n_cls_v1 pretrained=True patience=20 degrees=5 translate=0.05 scale=0.1 fliplr=0.5
```

验收时不要只看总体 accuracy，必须重点看：

```
abnormal 类是否被误判为 normal
abnormal 类召回率
confusion matrix
低置信度样本
错误样本是否存在标注问题
```

若异常漏检严重，优化优先级是：

```
补充 abnormal 样本 > 清洗错误标签 > 异常类定向增强 > 提高 imgsz > 尝试 yolov8s-cls.pt
```