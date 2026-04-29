# YOLOv8 目标检测任务训练操作指导手册

**适用环境：Windows + RTX 4060 Laptop + Ultralytics CLI + LabelImg YOLO 标注**

------

## 0. 文档定位

本文档面向实际项目落地，目标是让初级工程师可以直接完成：

```
数据整理 → 标注检查 → data.yaml 配置 → 模型训练 → 验证测试 → 推理预测 → 模型导出 → 问题排查
```

本手册适用于 **YOLOv8 目标检测 detection**，不是 classification 分类任务。

检测任务需要：

```
原始图片
YOLO 格式 .txt 标注文件
data.yaml
```

Ultralytics CLI 的基本格式是 `yolo TASK MODE ARGS`，例如检测训练使用 `yolo detect train ...`。

------

# 1. 环境准备与依赖检查

## 1.1 安装 Ultralytics

在已经配置好的 Python 环境中执行：

```
pip install -U ultralytics
```

检查安装结果：

```
python -c "import ultralytics; print(ultralytics.__version__)"
```

检查 `yolo` 命令是否可用：

```
yolo version
```

也可以执行：

```
yolo checks
```

`yolo checks` 可用于查看 Ultralytics、Python、PyTorch、CUDA、GPU 等环境信息。Ultralytics 官方 CLI 支持通过 `yolo` 命令执行 train、val、predict、export 等模式。

------

## 1.2 验证 PyTorch 是否识别 CUDA

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

如果输出：

```
cuda available: False
```

说明当前 PyTorch 没有正确使用 CUDA，训练会走 CPU。

------

## 1.3 确认训练使用 RTX 4060 Laptop

训练命令中显式指定：

```
device=0
```

示例：

```
yolo detect train model=yolov8n.pt data=D:/your_project/dataset/data.yaml epochs=1 imgsz=640 batch=8 device=0
```

训练过程中可以另开一个 PowerShell 窗口执行：

```
nvidia-smi
```

如果 GPU 被使用，应能看到 Python 进程占用显存。

------

## 1.4 Windows 环境注意事项

| 问题         | 建议                                             |
| ------------ | ------------------------------------------------ |
| 路径包含空格 | 使用双引号                                       |
| 中文路径     | 尽量避免                                         |
| 权限问题     | 不要把项目放在 `C:\Program Files`                |
| 反斜杠转义   | CLI 中推荐使用 `/` 或给路径加引号                |
| workers 卡住 | Windows 下可把 `workers=4` 改为 `workers=0`      |
| CUDA 不可用  | 检查 `torch.cuda.is_available()` 和 `nvidia-smi` |

推荐项目路径：

```
D:/your_project/
```

不推荐：

```
D:/项目/检测训练/
C:/Program Files/yolo_project/
```

------

# 2. LabelImg 标注文件检查

## 2.1 LabelImg YOLO txt 标注格式

LabelImg 导出的 YOLO 检测格式通常为：

```
class_id x_center y_center width height
```

示例：

```
1 0.5125 0.4382 0.1234 0.0876
```

字段含义：

| 字段       | 含义                              |
| ---------- | --------------------------------- |
| `class_id` | 类别编号，从 0 开始               |
| `x_center` | 检测框中心点 x 坐标，归一化到 0~1 |
| `y_center` | 检测框中心点 y 坐标，归一化到 0~1 |
| `width`    | 检测框宽度，归一化到 0~1          |
| `height`   | 检测框高度，归一化到 0~1          |

YOLO 检测数据集通常使用图片文件与同名 `.txt` 标签文件配对，标签文件中的框坐标是归一化坐标。Ultralytics 检测数据集文档也要求检测标签以类别和归一化框坐标组织。

------

## 2.2 class_id 的含义

本项目 2 类：

```
names:
  0: normal
  1: abnormal
```

则标注文件中只能出现：

```
0
1
```

示例：

```
0 0.4812 0.5021 0.2000 0.1800
1 0.7123 0.4210 0.0800 0.0600
```

表示同一张图片中有两个目标：

```
第 1 行：normal 目标
第 2 行：abnormal 目标
```

------

## 2.3 坐标必须是归一化坐标

正确：

```
1 0.5125 0.4382 0.1234 0.0876
```

错误：

```
1 328 280 79 56
```

YOLO 格式不能直接写像素坐标。

所有坐标应满足：

```
0 <= x_center <= 1
0 <= y_center <= 1
0 < width <= 1
0 < height <= 1
```

同时应满足框不越界：

```
x_center - width / 2 >= 0
x_center + width / 2 <= 1
y_center - height / 2 >= 0
y_center + height / 2 <= 1
```

------

## 2.4 每张图片对应一个同名 txt 文件

正确示例：

```
images/train/000001.jpg
labels/train/000001.txt
images/train/abc_002.png
labels/train/abc_002.txt
```

错误示例：

```
images/train/000001.jpg
labels/train/0001.txt
```

文件名不一致会导致 YOLO 找不到标签，训练时可能被当作无标签图片处理。

------

## 2.5 多目标图片如何写多行

如果一张图中有多个目标，每个目标一行：

```
0 0.3000 0.4000 0.1200 0.1000
1 0.6500 0.5200 0.0800 0.0700
1 0.7200 0.6100 0.0500 0.0400
```

表示：

```
1 个 normal 目标
2 个 abnormal 目标
```

------

## 2.6 无目标图片是否需要空 txt 文件

工程建议：**需要保留空 txt 文件。**

例如某张图片中没有任何可检测目标：

```
images/train/000088.jpg
labels/train/000088.txt
```

其中：

```
000088.txt 是空文件
```

这样做的好处：

1. 明确告诉训练程序：这张图确实没有目标。
2. 避免被误判为漏标。
3. 有利于负样本训练，降低误检。

如果你的“正常”图片中没有任何需要框出的目标，`normal` 类是否应该作为检测类别需要重新确认。检测任务学习的是“框出目标位置”，如果正常样本没有目标框，而异常样本有异常框，那么这更像“异常目标检测”；如果正常和异常都需要框出某个对象，则两类都可以作为检测类别。

------

## 2.7 检查类别编号是否只包含 0 和 1

在数据集根目录新建脚本：

```
D:/your_project/scripts/check_yolo_labels.py
```

内容如下：

```
from pathlib import Path

LABEL_DIRS = [
    Path("D:/your_project/dataset/labels/train"),
    Path("D:/your_project/dataset/labels/val"),
    Path("D:/your_project/dataset/labels/test"),
]

VALID_CLASSES = {0, 1}

bad_files = []

for label_dir in LABEL_DIRS:
    for txt_path in label_dir.glob("*.txt"):
        lines = txt_path.read_text(encoding="utf-8").strip().splitlines()

        for line_no, line in enumerate(lines, start=1):
            if not line.strip():
                continue

            parts = line.split()
            if len(parts) != 5:
                bad_files.append((txt_path, line_no, "字段数量不是 5", line))
                continue

            try:
                cls = int(parts[0])
                nums = list(map(float, parts[1:]))
            except ValueError:
                bad_files.append((txt_path, line_no, "存在非数字字段", line))
                continue

            if cls not in VALID_CLASSES:
                bad_files.append((txt_path, line_no, f"class_id 非法: {cls}", line))

            x, y, w, h = nums

            if not (0 <= x <= 1 and 0 <= y <= 1 and 0 < w <= 1 and 0 < h <= 1):
                bad_files.append((txt_path, line_no, "归一化坐标范围错误", line))

            if x - w / 2 < 0 or x + w / 2 > 1 or y - h / 2 < 0 or y + h / 2 > 1:
                bad_files.append((txt_path, line_no, "检测框越界", line))

if bad_files:
    print("发现异常标签：")
    for item in bad_files[:100]:
        print(item)
    print(f"共发现 {len(bad_files)} 个问题")
else:
    print("标签格式检查通过")
```

执行：

```
python D:/your_project/scripts/check_yolo_labels.py
```

------

## 2.8 检查图片与 txt 是否一一对应

脚本：

```
from pathlib import Path

IMAGE_EXTS = {".jpg", ".jpeg", ".png", ".bmp", ".webp"}

pairs = [
    (
        Path("D:/your_project/dataset/images/train"),
        Path("D:/your_project/dataset/labels/train"),
    ),
    (
        Path("D:/your_project/dataset/images/val"),
        Path("D:/your_project/dataset/labels/val"),
    ),
    (
        Path("D:/your_project/dataset/images/test"),
        Path("D:/your_project/dataset/labels/test"),
    ),
]

for image_dir, label_dir in pairs:
    images = {p.stem for p in image_dir.iterdir() if p.suffix.lower() in IMAGE_EXTS}
    labels = {p.stem for p in label_dir.glob("*.txt")}

    missing_labels = images - labels
    extra_labels = labels - images

    print(f"\n检查：{image_dir}")
    print(f"图片数量: {len(images)}")
    print(f"标签数量: {len(labels)}")
    print(f"缺失标签: {len(missing_labels)}")
    print(f"多余标签: {len(extra_labels)}")

    if missing_labels:
        print("缺失标签示例:", list(sorted(missing_labels))[:20])
    if extra_labels:
        print("多余标签示例:", list(sorted(extra_labels))[:20])
```

执行：

```
python D:/your_project/scripts/check_image_label_pairs.py
```

------

## 2.9 漏标、错标、框越界、空标签检查建议

| 问题   | 检查方法                 | 处理建议                         |
| ------ | ------------------------ | -------------------------------- |
| 漏标   | 随机抽样可视化检查       | 回到 LabelImg 补标               |
| 错标   | 检查类别编号与实际目标   | 修正 class_id                    |
| 框越界 | 使用脚本检查坐标         | 重新标注                         |
| 空标签 | 检查是否真的是无目标图片 | 真无目标则保留空 txt，漏标则补标 |
| 框过大 | 可视化检查               | 框紧贴目标边界                   |
| 框过小 | 可视化检查               | 框应覆盖完整目标                 |

------

# 3. YOLOv8 检测任务标准数据目录结构

推荐结构：

```
dataset/
├── images/
│   ├── train/
│   ├── val/
│   └── test/
├── labels/
│   ├── train/
│   ├── val/
│   └── test/
└── data.yaml
```

示例：

```
D:/your_project/dataset/
├── images/
│   ├── train/
│   │   ├── 000001.jpg
│   │   ├── 000002.jpg
│   │   └── ...
│   ├── val/
│   └── test/
├── labels/
│   ├── train/
│   │   ├── 000001.txt
│   │   ├── 000002.txt
│   │   └── ...
│   ├── val/
│   └── test/
└── data.yaml
```

Ultralytics 检测数据集通常通过 YAML 文件描述训练、验证、测试图片路径和类别名称。

------

## 3.1 图片与标签对应关系

| 图片目录       | 标签目录       |
| -------------- | -------------- |
| `images/train` | `labels/train` |
| `images/val`   | `labels/val`   |
| `images/test`  | `labels/test`  |

对应关系：

```
images/train/abc.jpg
labels/train/abc.txt
images/val/def.png
labels/val/def.txt
images/test/xyz.jpg
labels/test/xyz.txt
```

------

## 3.2 Windows 路径写法

推荐在 `data.yaml` 中使用 `/`：

```
path: D:/your_project/dataset
```

不推荐：

```
path: D:\your_project\dataset
```

原因是反斜杠在某些解析场景中可能引发转义问题。

------

# 4. 数据集划分方法

## 4.1 当前数据量划分

总数据量约 1500 张：

```
train : val : test = 7 : 2 : 1
```

建议：

| 数据集 | 数量       |
| ------ | ---------- |
| train  | 约 1050 张 |
| val    | 约 300 张  |
| test   | 约 150 张  |

类别比例应尽量保持一致：

| 类别 | 总数 | train  | val    | test   |
| ---- | ---- | ------ | ------ | ------ |
| 正常 | 1000 | 约 700 | 约 200 | 约 100 |
| 异常 | 500  | 约 350 | 约 100 | 约 50  |

------

## 4.2 避免数据泄漏

以下情况会导致数据泄漏：

```
同一张图片同时出现在 train 和 val
同一视频连续帧被随机分到 train 和 test
同一产品同一批次高度相似图片跨集合出现
增强后的图片和原图分到不同集合
```

工程原则：

1. 先划分，再增强。
2. 同一来源、同一批次、同一视频片段尽量放在同一 split。
3. test 集只做最终验收，不参与调参。
4. 文件名重复必须检查。

------

## 4.3 是否建议使用脚本自动划分

建议使用脚本自动划分，原因：

1. 可复现。
2. 避免人工复制出错。
3. 可保持类别比例。
4. 可同步移动图片和标签。

前提：原始数据已经按类别放好，例如：

```
raw_dataset/
├── normal/
│   ├── images/
│   └── labels/
└── abnormal/
    ├── images/
    └── labels/
```

如果你的数据不是这种结构，可以根据实际路径稍作修改。

------

## 4.4 Python 自动划分脚本示例

保存为：

```
D:/your_project/scripts/split_dataset.py
```

脚本：

```
import random
import shutil
from pathlib import Path

random.seed(42)

RAW_ROOT = Path("D:/your_project/raw_dataset")
OUT_ROOT = Path("D:/your_project/dataset")

CLASSES = ["normal", "abnormal"]
IMAGE_EXTS = {".jpg", ".jpeg", ".png", ".bmp", ".webp"}

SPLIT_RATIO = {
    "train": 0.7,
    "val": 0.2,
    "test": 0.1,
}

for split in SPLIT_RATIO:
    (OUT_ROOT / "images" / split).mkdir(parents=True, exist_ok=True)
    (OUT_ROOT / "labels" / split).mkdir(parents=True, exist_ok=True)

def copy_pair(image_path: Path, label_path: Path, split: str):
    dst_image = OUT_ROOT / "images" / split / image_path.name
    dst_label = OUT_ROOT / "labels" / split / f"{image_path.stem}.txt"

    shutil.copy2(image_path, dst_image)

    if label_path.exists():
        shutil.copy2(label_path, dst_label)
    else:
        # 无目标图片或确认为负样本时，创建空标签
        dst_label.write_text("", encoding="utf-8")

for cls_name in CLASSES:
    image_dir = RAW_ROOT / cls_name / "images"
    label_dir = RAW_ROOT / cls_name / "labels"

    images = [p for p in image_dir.iterdir() if p.suffix.lower() in IMAGE_EXTS]
    random.shuffle(images)

    n = len(images)
    n_train = int(n * SPLIT_RATIO["train"])
    n_val = int(n * SPLIT_RATIO["val"])

    split_map = {
        "train": images[:n_train],
        "val": images[n_train:n_train + n_val],
        "test": images[n_train + n_val:],
    }

    for split, split_images in split_map.items():
        for image_path in split_images:
            label_path = label_dir / f"{image_path.stem}.txt"
            copy_pair(image_path, label_path, split)

        print(f"{cls_name} -> {split}: {len(split_images)}")

print("数据集划分完成")
```

执行：

```
python D:/your_project/scripts/split_dataset.py
```

------

# 5. data.yaml 配置文件

在：

```
D:/your_project/dataset/data.yaml
```

写入：

```
path: D:/your_project/dataset
train: images/train
val: images/val
test: images/test

nc: 2
names:
  0: normal
  1: abnormal
```

------

## 5.1 字段解释

| 字段    | 作用                        |
| ------- | --------------------------- |
| `path`  | 数据集根目录                |
| `train` | 训练图片目录，相对于 `path` |
| `val`   | 验证图片目录，相对于 `path` |
| `test`  | 测试图片目录，相对于 `path` |
| `nc`    | 类别数量                    |
| `names` | 类别编号与类别名称映射      |

注意：

```
names:
  0: normal
  1: abnormal
```

必须与 LabelImg 标注中的 `class_id` 一致。

如果 `.txt` 中 `0` 实际表示异常，但 `data.yaml` 中写成：

```
0: normal
```

则训练结果会整体错位。

------

# 6. 模型选择与训练策略

## 6.1 推荐模型

当前场景：

```
任务：2 类目标检测
数据量：约 1500 张
GPU：RTX 4060 Laptop
标注：LabelImg YOLO txt
```

推荐起步模型：

```
yolov8n.pt
```

如果基线模型正常收敛，但异常类召回率不够，可以尝试：

```
yolov8s.pt
```

YOLOv8 Detect 模型是用于目标检测的模型，`yolov8n.pt`、`yolov8s.pt` 等是不同规模的检测模型；Ultralytics 官方也给出了 YOLOv8 Detect 的训练、验证、预测和导出示例。

------

## 6.2 yolov8n.pt 与 yolov8s.pt 选择

| 模型                | 优点                         | 缺点               | 建议             |
| ------------------- | ---------------------------- | ------------------ | ---------------- |
| `yolov8n.pt`        | 快、显存占用低、适合快速验证 | 表达能力较弱       | 首轮基线         |
| `yolov8s.pt`        | 精度通常更好                 | 训练更慢、显存更高 | 第二阶段优化     |
| `yolov8m.pt` 及以上 | 表达能力更强                 | 小数据集易过拟合   | 暂不建议起步使用 |

当前 1500 张数据，建议：

```
第一轮：yolov8n.pt
第二轮：yolov8s.pt 对比
```

------

## 6.3 为什么使用预训练模型

建议使用：

```
yolov8n.pt
yolov8s.pt
```

而不是：

```
yolov8n.yaml
yolov8s.yaml
```

原因：

1. 预训练模型已经学习了通用视觉特征。
2. 小数据集训练更稳定。
3. 收敛更快。
4. 对 1500 张数据更友好。
5. 从零训练容易欠拟合或不稳定。

Ultralytics 官方训练示例也支持从预训练 `.pt` 模型开始训练自定义数据集。

------

## 6.4 目标较小时是否提高 imgsz

默认建议：

```
imgsz=640
```

如果异常目标很小，例如只占图像宽高的 1%～3%，可以尝试：

```
imgsz=768
imgsz=960
```

但提高 `imgsz` 会增加：

```
显存占用
训练时间
推理时间
```

工程建议：

```
第一轮：imgsz=640
小目标漏检明显：imgsz=768
显存允许且小目标极小：imgsz=960
```

------

## 6.5 RTX 4060 Laptop batch size 建议

检测任务比分类任务更吃显存。

建议：

| 模型         | imgsz | batch 建议 |
| ------------ | ----- | ---------- |
| `yolov8n.pt` | 640   | 16         |
| `yolov8n.pt` | 768   | 8          |
| `yolov8s.pt` | 640   | 8 或 16    |
| `yolov8s.pt` | 768   | 4 或 8     |

首轮建议：

```
model=yolov8n.pt
imgsz=640
batch=16
```

如果显存溢出：

```
batch=8
```

------

# 7. 训练命令

## 7.1 基础训练命令

```
yolo detect train model=yolov8n.pt data=D:/your_project/dataset/data.yaml epochs=100 imgsz=640 batch=16 device=0 workers=4
```

------

## 7.2 参数解释

| 参数                 | 说明                            |
| -------------------- | ------------------------------- |
| `detect train`       | 执行目标检测训练                |
| `model=yolov8n.pt`   | 使用 YOLOv8 nano 检测预训练模型 |
| `data=.../data.yaml` | 数据集配置文件                  |
| `epochs=100`         | 训练 100 轮                     |
| `imgsz=640`          | 输入图像尺寸                    |
| `batch=16`           | 每批次图片数量                  |
| `device=0`           | 使用第 0 张 GPU                 |
| `workers=4`          | 数据加载线程数                  |
| `project=...`        | 训练结果保存根目录              |
| `name=...`           | 本次实验名称                    |
| `patience=...`       | 早停轮数                        |

Ultralytics CLI 训练命令遵循 `yolo TASK MODE ARGS` 的格式，例如 `yolo detect train data=... epochs=... imgsz=...`。

------

## 7.3 工程化推荐训练命令

推荐首轮正式训练：

```
yolo detect train model=yolov8n.pt data=D:/your_project/dataset/data.yaml epochs=100 imgsz=640 batch=16 device=0 workers=4 project=D:/your_project/runs name=yolov8n_det_v1 patience=20 pretrained=True
```

如果 Windows 下 workers 异常或卡住：

```
yolo detect train model=yolov8n.pt data=D:/your_project/dataset/data.yaml epochs=100 imgsz=640 batch=16 device=0 workers=0 project=D:/your_project/runs name=yolov8n_det_v1 patience=20 pretrained=True
```

------

## 7.4 小目标优化训练命令

如果目标较小、漏检明显：

```
yolo detect train model=yolov8n.pt data=D:/your_project/dataset/data.yaml epochs=120 imgsz=768 batch=8 device=0 workers=4 project=D:/your_project/runs name=yolov8n_det_img768 patience=25 pretrained=True
```

------

## 7.5 使用 yolov8s.pt 对比训练

```
yolo detect train model=yolov8s.pt data=D:/your_project/dataset/data.yaml epochs=100 imgsz=640 batch=8 device=0 workers=4 project=D:/your_project/runs name=yolov8s_det_v1 patience=20 pretrained=True
```

------

# 8. 训练过程结果解读

## 8.1 训练结果保存位置

如果使用：

```
project=D:/your_project/runs name=yolov8n_det_v1
```

结果通常保存到：

```
D:/your_project/runs/yolov8n_det_v1/
```

默认情况下，Ultralytics 会在 `runs/detect/train` 或类似目录下保存训练输出。

常见文件：

```
weights/best.pt
weights/last.pt
results.csv
results.png
confusion_matrix.png
PR_curve.png
F1_curve.png
P_curve.png
R_curve.png
```

------

## 8.2 best.pt 与 last.pt

| 文件      | 含义                 | 使用建议                 |
| --------- | -------------------- | ------------------------ |
| `best.pt` | 验证集指标最好的模型 | 推理、测试、导出优先使用 |
| `last.pt` | 最后一轮训练后的模型 | 用于断点续训或分析       |

工程部署一般使用：

```
weights/best.pt
```

------

## 8.3 box_loss

`box_loss` 表示预测框位置与真实框位置的误差。

关注点：

```
box_loss 持续下降：定位能力在提升
box_loss 不下降：框标注质量、学习率、数据难度可能有问题
box_loss 很低但 mAP 低：可能类别错误或漏标严重
```

------

## 8.4 cls_loss

`cls_loss` 表示类别分类损失。

当前 2 类：

```
normal
abnormal
```

如果 `cls_loss` 长期较高，可能原因：

1. 类别标错。
2. `data.yaml` 中 `names` 顺序与 LabelImg class_id 不一致。
3. 正常 / 异常视觉差异不明显。
4. 样本类别不均衡。

------

## 8.5 dfl_loss

`dfl_loss` 是 YOLOv8 检测框分布回归相关损失，用于提升边界框定位质量。

工程上主要观察趋势：

```
持续下降：正常
长期震荡或不下降：标注框质量、学习率、数据分布需要检查
```

------

## 8.6 precision

Precision 表示预测为目标的框中，有多少是真的。

```
precision = TP / (TP + FP)
```

工程含义：

```
precision 低：误检多
```

例如把正常区域误检为 abnormal，precision 会下降。

------

## 8.7 recall

Recall 表示真实目标中，有多少被模型检测出来。

```
recall = TP / (TP + FN)
```

工程含义：

```
recall 低：漏检多
```

当前项目需要重点关注：

```
abnormal 类 recall
```

异常漏检通常比正常误检更严重。

------

## 8.8 mAP50

`mAP50` 表示 IoU 阈值为 0.5 时的平均精度。

工程理解：

```
检测框和真实框重叠达到一定程度即可算对
```

`mAP50` 较高但 `mAP50-95` 较低，说明模型大概能找到目标，但框不够准。

------

## 8.9 mAP50-95

`mAP50-95` 是更严格的指标，综合多个 IoU 阈值。

工程理解：

```
更能反映检测框定位质量
```

最终汇报建议同时记录：

```
precision
recall
mAP50
mAP50-95
abnormal recall
```

------

## 8.10 如何判断模型是否收敛

基本收敛特征：

1. `box_loss`、`cls_loss`、`dfl_loss` 下降趋缓。
2. `mAP50`、`mAP50-95` 不再明显提升。
3. `best.pt` 长时间不更新。
4. 验证集指标稳定。

如果 20 个 epoch 没有明显提升，可依赖：

```
patience=20
```

自动早停。

------

## 8.11 如何判断过拟合

过拟合表现：

| 训练集           | 验证集              |
| ---------------- | ------------------- |
| loss 持续下降    | loss 不降反升       |
| 训练效果很好     | 验证 mAP 停滞或下降 |
| 训练图像预测准确 | 新图像误检漏检严重  |

处理：

1. 增加数据。
2. 清洗标签。
3. 增加适度增强。
4. 减少 epochs。
5. 使用更小模型。
6. 降低 `imgsz` 或模型规模。

------

## 8.12 如何判断欠拟合

欠拟合表现：

```
训练集 mAP 低
验证集 mAP 低
loss 长期较高
预测框少或类别混乱
```

处理：

1. 增加 epochs。
2. 使用 `yolov8s.pt`。
3. 提高 `imgsz`。
4. 检查标注是否错误。
5. 检查学习率。
6. 检查是否使用了正确的预训练模型。

------

## 8.13 如何判断异常类识别较差

重点查看：

```
confusion_matrix.png
PR_curve.png
results.csv
验证集预测结果
```

如果出现：

```
abnormal recall 低
abnormal 被误判为 normal
异常目标漏框
异常框置信度低
```

说明异常类识别较差。

优先处理：

```
补充异常样本
修正异常标注
提高 imgsz
尝试 yolov8s.pt
降低推理 conf
```

------

# 9. 验证与测试

## 9.1 验证集验证命令

```
yolo detect val model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt data=D:/your_project/dataset/data.yaml imgsz=640 batch=16 device=0
```

Ultralytics 支持通过 `val` 模式对训练好的模型进行验证，并输出检测指标。

------

## 9.2 指定 test 集测试

```
yolo detect val model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt data=D:/your_project/dataset/data.yaml split=test imgsz=640 batch=16 device=0
```

注意：

```
val 集用于训练过程选择模型
test 集用于最终验收
```

不要反复用 test 集调参。

------

## 9.3 查看混淆矩阵

验证完成后，在输出目录中查看：

```
confusion_matrix.png
confusion_matrix_normalized.png
```

重点看：

```
abnormal 是否被识别成 normal
normal 是否被误检成 abnormal
背景是否被误检成 abnormal
```

------

## 9.4 查看 PR 曲线

查看：

```
PR_curve.png
```

PR 曲线用于观察不同置信度阈值下 precision 和 recall 的变化。

工程关注：

```
abnormal 类 PR 曲线是否明显低于 normal 类
```

如果 abnormal 曲线很差，说明异常类样本质量、数量或标注一致性存在问题。

------

## 9.5 重点分析异常类召回率

异常检测项目中，重点不是只看总 mAP，而是看：

```
abnormal recall
abnormal 漏检数量
abnormal 低置信度样本
```

如果业务更关注“不漏检异常”，可以在推理阶段适当降低：

```
conf=0.25
```

甚至：

```
conf=0.15
```

但降低 `conf` 会增加误检，需要结合现场业务成本权衡。

------

# 10. 推理预测

## 10.1 单张图片预测

```
yolo detect predict model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt source=D:/your_project/dataset/test_images/test_001.jpg imgsz=640 conf=0.25 device=0 save=True
```

输出内容包括：

```
检测框
类别
置信度
保存后的可视化图片
```

------

## 10.2 文件夹批量预测

```
yolo detect predict model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt source=D:/your_project/test_images imgsz=640 conf=0.25 device=0 save=True
```

------

## 10.3 指定输出目录

```
yolo detect predict model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt source=D:/your_project/test_images imgsz=640 conf=0.25 device=0 save=True project=D:/your_project/runs name=predict_test
```

输出目录：

```
D:/your_project/runs/predict_test/
```

Ultralytics CLI 支持 `predict` 模式进行图像、文件夹、视频等输入的推理。

------

## 10.4 保存预测 txt 结果

如果需要保存 YOLO 格式预测结果：

```
yolo detect predict model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt source=D:/your_project/test_images imgsz=640 conf=0.25 device=0 save=True save_txt=True save_conf=True project=D:/your_project/runs name=predict_txt
```

结果中会包含预测框 `.txt` 文件，通常保存在：

```
D:/your_project/runs/predict_txt/labels/
```

------

## 10.5 置信度阈值 conf 设置建议

| 场景         | conf 建议  |
| ------------ | ---------- |
| 默认测试     | 0.25       |
| 希望减少误检 | 0.4～0.6   |
| 希望减少漏检 | 0.1～0.25  |
| 异常召回优先 | 0.15～0.25 |

示例：

```
yolo detect predict model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt source=D:/your_project/test_images conf=0.15 imgsz=640 device=0 save=True
```

------

# 11. 模型导出

Ultralytics 支持通过 `yolo export` 将训练好的模型导出到 ONNX、TorchScript 等格式。

------

## 11.1 导出 ONNX

```
yolo export model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt format=onnx imgsz=640 opset=12 simplify=True
```

导出结果通常为：

```
best.onnx
```

### ONNX 适用场景

适合：

```
ONNX Runtime
C++ 推理服务
跨平台部署
TensorRT 转换前中间格式
OpenVINO 转换
```

ONNX 格式适合跨框架、跨平台部署，Ultralytics 官方也提供 ONNX 导出说明。

------

## 11.2 导出 TorchScript

```
yolo export model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt format=torchscript imgsz=640
```

导出结果通常为：

```
best.torchscript
```

### TorchScript 适用场景

适合：

```
PyTorch 生态部署
LibTorch C++ 推理
Python 服务端推理
边缘端 PyTorch 运行环境
```

TorchScript 是 PyTorch 生态中的序列化部署方式，可用于没有完整 Python 训练代码的运行环境。

------

# 12. 常见问题与排查

## 12.1 LabelImg 标注格式错误

### 现象

```
训练报错
标签被忽略
mAP 极低
框位置明显错误
```

### 排查

检查 `.txt` 是否为 5 列：

```
class_id x_center y_center width height
```

检查是否出现像素坐标：

```
1 320 240 100 80
```

这是错误格式。

### 解决

重新导出 YOLO 格式，确保坐标归一化到 0～1。

------

## 12.2 图片和 txt 文件名不一致

### 现象

```
No labels found
大量图片被认为没有标签
训练结果很差
```

### 正确对应

```
images/train/abc.jpg
labels/train/abc.txt
```

### 解决

使用脚本检查文件名一一对应，缺失标签则补齐。无目标图片应使用空 txt。

------

## 12.3 data.yaml 路径错误

### 现象

```
Dataset not found
Images not found
FileNotFoundError
```

### 排查

确认：

```
path: D:/your_project/dataset
train: images/train
val: images/val
test: images/test
```

对应真实路径：

```
D:/your_project/dataset/images/train
D:/your_project/dataset/images/val
D:/your_project/dataset/images/test
```

### 解决

使用绝对路径，Windows 下推荐 `/`。

------

## 12.4 类别编号不匹配

### 现象

```
normal 和 abnormal 预测结果反了
混淆矩阵异常
类别名称显示错误
```

### 排查

确认 LabelImg 中：

```
0 是否真的代表 normal
1 是否真的代表 abnormal
```

确认 `data.yaml`：

```
names:
  0: normal
  1: abnormal
```

### 解决

保持 `.txt class_id` 与 `data.yaml names` 完全一致。

------

## 12.5 训练时 GPU 未使用

### 排查命令

```
nvidia-smi
python -c "import torch; print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')"
```

### 训练命令指定 GPU

```
yolo detect train model=yolov8n.pt data=D:/your_project/dataset/data.yaml epochs=100 imgsz=640 batch=16 device=0
```

### 解决

如果 `torch.cuda.is_available()` 为 False，重新安装 CUDA 版本 PyTorch，或检查显卡驱动。

------

## 12.6 CUDA / PyTorch 版本不匹配

### 现象

```
CUDA error
DLL load failed
torch.cuda.is_available() = False
```

### 排查

```
nvidia-smi
python -c "import torch; print(torch.__version__); print(torch.version.cuda); print(torch.cuda.is_available())"
```

### 解决建议

1. 确认 NVIDIA 驱动正常。
2. 确认 PyTorch 不是 CPU-only 版本。
3. 重新安装与当前环境匹配的 PyTorch CUDA 版本。

------

## 12.7 batch size 过大导致显存溢出

### 现象

```
CUDA out of memory
```

### 解决

从：

```
batch=16
```

改为：

```
batch=8
```

仍然溢出：

```
batch=4
```

如果使用 `imgsz=768` 溢出，回退：

```
imgsz=640
```

------

## 12.8 mAP 很低

### 常见原因

| 原因         | 排查方法               | 解决         |
| ------------ | ---------------------- | ------------ |
| 标签格式错误 | 检查 txt 5 列与归一化  | 修正标签     |
| 类别编号错   | 检查 names 与 class_id | 统一类别映射 |
| 漏标严重     | 可视化抽查             | 补标         |
| 框不准       | 抽查标注框             | 重标         |
| 数据太少     | 看异常样本数量         | 增加数据     |
| 目标太小     | 查看预测图             | 提高 imgsz   |
| 模型太小     | 对比 yolov8s.pt        | 使用更大模型 |

------

## 12.9 正常类和异常类样本不均衡

当前：

```
正常：约 1000
异常：约 500
```

问题：

```
模型可能偏向 normal
abnormal recall 低
异常漏检
```

解决优先级：

1. 增加异常样本。
2. 提高异常样本多样性。
3. 清洗异常类标注。
4. 对异常样本做合理增强。
5. 使用验证集重点监控 abnormal recall。

------

## 12.10 检测框偏移或漏检

### 框偏移原因

```
标注框不准
图片尺寸与标签不匹配
LabelImg 导出异常
数据增强后标注未同步
```

### 漏检原因

```
目标太小
目标样本少
conf 设置过高
标注漏标
模型能力不足
```

### 解决

```
yolo detect predict model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt source=D:/your_project/dataset/images/val imgsz=640 conf=0.15 save=True project=D:/your_project/runs name=error_analysis
```

逐张查看漏检和误检图片。

------

# 13. 性能优化建议

## 13.1 提升异常类召回率

优先级：

```
补充异常样本 > 清洗异常标签 > 调低 conf > 提高 imgsz > 尝试 yolov8s.pt
```

如果异常漏检严重，先不要盲目换大模型，应先检查：

1. 异常是否标全。
2. 异常框是否准确。
3. 异常类别编号是否正确。
4. 异常样本是否太单一。
5. 验证集是否覆盖真实场景。

------

## 13.2 是否需要增加异常样本

建议增加。

当前异常样本约 500 张，正常约 1000 张。对于异常检测项目，异常样本通常更关键。

建议目标：

```
异常样本增加到 800～1000 张
```

并覆盖：

```
不同异常类型
不同光照
不同角度
不同背景
不同尺寸
不同设备来源
轻微异常
严重异常
边界异常
```

------

## 13.3 是否需要数据增强

需要，但要合理。

YOLO 训练中通常会使用多种数据增强策略，Ultralytics 训练文档也说明训练模式支持硬件加速、增强和训练参数配置。

建议：

```
轻微旋转
轻微平移
亮度变化
尺度变化
轻微噪声
```

慎用：

```
过度裁剪
过强颜色扰动
不符合业务方向的翻转
破坏异常特征的模糊
```

如果异常特征依赖方向，例如裂纹方向、安装方向、左右结构，则不要随意翻转。

------

## 13.4 是否提高 imgsz

建议先用：

```
imgsz=640
```

如果小目标漏检明显，尝试：

```
yolo detect train model=yolov8n.pt data=D:/your_project/dataset/data.yaml epochs=120 imgsz=768 batch=8 device=0 workers=4 project=D:/your_project/runs name=yolov8n_det_img768 patience=25
```

如果显存不足：

```
降低 batch
保持 imgsz=640
```

------

## 13.5 是否尝试 yolov8s.pt

建议在 `yolov8n.pt` 基线完成后尝试。

对比实验：

```
yolo detect train model=yolov8s.pt data=D:/your_project/dataset/data.yaml epochs=100 imgsz=640 batch=8 device=0 workers=4 project=D:/your_project/runs name=yolov8s_det_v1 patience=20
```

对比指标：

```
abnormal recall
mAP50
mAP50-95
推理速度
显存占用
误检数量
漏检数量
```

如果 `yolov8s.pt` 提升不明显，优先回到数据质量优化。

------

## 13.6 如何调整 conf

推理默认可从：

```
conf=0.25
```

开始。

异常召回优先：

```
conf=0.15
```

减少误检：

```
conf=0.5
```

命令示例：

```
yolo detect predict model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt source=D:/your_project/test_images imgsz=640 conf=0.15 device=0 save=True
```

注意：

```
降低 conf：召回提升，误检增加
提高 conf：误检减少，漏检增加
```

------

## 13.7 如何分析误检和漏检样本

建议建立错误分析目录：

```
D:/your_project/error_analysis/
├── false_positive/
├── false_negative/
├── wrong_class/
└── low_confidence/
```

批量预测验证集：

```
yolo detect predict model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt source=D:/your_project/dataset/images/val imgsz=640 conf=0.15 device=0 save=True save_txt=True save_conf=True project=D:/your_project/runs name=val_error_analysis
```

分析维度：

| 错误类型       | 含义                   | 处理                                |
| -------------- | ---------------------- | ----------------------------------- |
| false positive | 正常区域被误检         | 增加负样本、提高 conf               |
| false negative | 异常漏检               | 增加异常样本、降低 conf、提高 imgsz |
| wrong class    | normal / abnormal 混淆 | 检查 class_id 和标注一致性          |
| low confidence | 置信度低               | 增加相似样本、优化标注              |

------

## 13.8 如何记录实验结果

建议建立实验记录表：

```
D:/your_project/experiment_log.xlsx
```

字段建议：

| 实验名 | 模型 | imgsz | batch | epochs | 数据版本 | mAP50 | mAP50-95 | abnormal recall | 结论 |
| ------ | ---- | ----- | ----- | ------ | -------- | ----- | -------- | --------------- | ---- |
|        |      |       |       |        |          |       |          |                 |      |

示例：

| 实验名               | 模型       | imgsz | batch | 结论         |
| -------------------- | ---------- | ----- | ----- | ------------ |
| `yolov8n_det_v1`     | yolov8n.pt | 640   | 16    | 基线         |
| `yolov8n_det_img768` | yolov8n.pt | 768   | 8     | 小目标对比   |
| `yolov8s_det_v1`     | yolov8s.pt | 640   | 8     | 模型规模对比 |

工程原则：

```
一次只改一个关键变量
每次保留 data.yaml、训练命令、结果目录、best.pt
```

------

# 14. 推荐完整执行流程

## Step 1：安装与检查环境

```
pip install -U ultralytics
yolo version
yolo checks
python -c "import torch; print(torch.__version__); print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')"
```

------

## Step 2：整理目录

```
D:/your_project/dataset/
├── images/
│   ├── train/
│   ├── val/
│   └── test/
├── labels/
│   ├── train/
│   ├── val/
│   └── test/
└── data.yaml
```

------

## Step 3：检查标签

```
python D:/your_project/scripts/check_yolo_labels.py
python D:/your_project/scripts/check_image_label_pairs.py
```

------

## Step 4：配置 data.yaml

```
path: D:/your_project/dataset
train: images/train
val: images/val
test: images/test

nc: 2
names:
  0: normal
  1: abnormal
```

------

## Step 5：冒烟训练

```
yolo detect train model=yolov8n.pt data=D:/your_project/dataset/data.yaml epochs=1 imgsz=640 batch=8 device=0 workers=4 project=D:/your_project/runs name=smoke_test
```

如果 Windows workers 异常：

```
yolo detect train model=yolov8n.pt data=D:/your_project/dataset/data.yaml epochs=1 imgsz=640 batch=8 device=0 workers=0 project=D:/your_project/runs name=smoke_test
```

------

## Step 6：正式训练

```
yolo detect train model=yolov8n.pt data=D:/your_project/dataset/data.yaml epochs=100 imgsz=640 batch=16 device=0 workers=4 project=D:/your_project/runs name=yolov8n_det_v1 patience=20 pretrained=True
```

------

## Step 7：验证集评估

```
yolo detect val model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt data=D:/your_project/dataset/data.yaml imgsz=640 batch=16 device=0
```

------

## Step 8：测试集评估

```
yolo detect val model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt data=D:/your_project/dataset/data.yaml split=test imgsz=640 batch=16 device=0
```

------

## Step 9：单张预测

```
yolo detect predict model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt source=D:/your_project/test_images/test_001.jpg imgsz=640 conf=0.25 device=0 save=True
```

------

## Step 10：批量预测

```
yolo detect predict model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt source=D:/your_project/test_images imgsz=640 conf=0.25 device=0 save=True project=D:/your_project/runs name=predict_test
```

------

## Step 11：导出 ONNX

```
yolo export model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt format=onnx imgsz=640 opset=12 simplify=True
```

------

## Step 12：导出 TorchScript

```
yolo export model=D:/your_project/runs/yolov8n_det_v1/weights/best.pt format=torchscript imgsz=640
```

------

# 15. 当前项目推荐配置

建议首轮配置：

```
模型：yolov8n.pt
任务：detect
imgsz：640
batch：16
epochs：100
device：0
workers：4
patience：20
数据比例：train/val/test = 7/2/1
```

推荐首轮命令：

```
yolo detect train model=yolov8n.pt data=D:/your_project/dataset/data.yaml epochs=100 imgsz=640 batch=16 device=0 workers=4 project=D:/your_project/runs name=yolov8n_det_v1 patience=20 pretrained=True
```

若异常目标小、漏检明显：

```
yolo detect train model=yolov8n.pt data=D:/your_project/dataset/data.yaml epochs=120 imgsz=768 batch=8 device=0 workers=4 project=D:/your_project/runs name=yolov8n_det_img768 patience=25 pretrained=True
```

若 `yolov8n.pt` 已收敛但异常类召回仍不足：

```
yolo detect train model=yolov8s.pt data=D:/your_project/dataset/data.yaml epochs=100 imgsz=640 batch=8 device=0 workers=4 project=D:/your_project/runs name=yolov8s_det_v1 patience=20 pretrained=True
```

最终验收重点：

```
不要只看总 mAP
必须看 abnormal recall
必须看 confusion matrix
必须看漏检样本
必须看现场真实图片预测结果
```