# YOLOv8 学习路线图

先说明一点：Ultralytics 官方文档现在把 YOLO 系列统一放在 “Ultralytics YOLO” 文档体系下，命令行仍然是 `yolo TASK MODE ARGS` 这种格式，例如 `yolo detect train ...`、`yolo predict ...`。YOLOv8 模型页面仍提供 YOLOv8 的训练和推理示例；官方支持的视觉任务包括 detect、segment、classify、pose、OBB、track 等。不同版本的仓库/API 可能略有变化，遇到命令不一致时，以你本地仓库的 README 和官方文档为准。

------

## 一、整体学习路线图

### 阶段 1：先会跑

**一句话目标：** 先把 YOLOv8 当成一个现成工具用起来。

你应该学：

- 用命令行检测图片、视频、摄像头。
- 用 Python API 加载模型、推理图片。
- 看懂输出目录、检测框、类别、置信度。

为什么要学：

因为新手最容易一开始就陷入源码和论文细节。YOLOv8 最好的入门方式不是先推公式，而是先跑通完整流程，知道“输入什么、输出什么”。

学完能达到：

你能拿一张图片或一段视频，让 YOLOv8 自动画出检测框。

推荐实践任务：

用 `yolov8n.pt` 检测 5 张图片、一段视频和摄像头画面。

自测问题：

1. `source` 参数表示什么？
2. `conf` 参数调高后，检测结果会变多还是变少？
3. 检测结果一般保存在哪里？

------

### 阶段 2：理解输入输出

**一句话目标：** 知道目标检测不是“识别整张图”，而是“找出图中每个物体的位置和类别”。

你应该学：

- 图片输入格式。
- 检测结果由类别、置信度、边界框组成。
- 边界框坐标含义。
- NMS 后处理的直觉。

为什么要学：

因为训练自己的数据集时，标签文件其实就是在告诉模型：“这张图里有什么物体，框在哪里”。

学完能达到：

你能解释一条检测结果：

```
person 0.87 [x1, y1, x2, y2]
```

表示什么意思。

推荐实践任务：

用 Python 打印每个检测框的类别名、置信度、坐标。

自测问题：

1. 边界框为什么需要 4 个数？
2. 置信度是不是等于“模型一定正确”？
3. 为什么同一个物体可能会被预测出多个框？

------

### 阶段 3：理解模型结构

**一句话目标：** 知道 YOLOv8 大致由 backbone、neck、head 三部分组成。

你应该学：

- backbone：提取图像特征。
- neck：融合不同尺度特征。
- head：输出检测结果。
- 小目标、大目标为什么需要不同尺度特征。

为什么要学：

这是看源码和理解训练的基础。

学完能达到：

你可以用通俗语言解释：“模型不是直接看像素猜答案，而是先把图片变成多层特征，再在特征上预测框和类别。”

推荐实践任务：

打印模型结构：

```
yolo detect predict model=yolov8n.pt source=bus.jpg
```

或 Python 中：

```
from ultralytics import YOLO

model = YOLO("yolov8n.pt")
print(model.model)
```

自测问题：

1. backbone 像人的哪个能力？
2. neck 为什么要融合多层特征？
3. head 的最终任务是什么？

------

### 阶段 4：理解训练过程

**一句话目标：** 明白训练时模型在不断修正自己的预测框、类别和置信度。

你应该学：

- 前向传播。
- 损失函数。
- 反向传播。
- optimizer 更新参数。
- epoch、batch size、learning rate、imgsz。

为什么要学：

训练自己的数据集时，很多问题不是代码错，而是训练参数、数据质量、标签质量、类别不均衡造成的。

学完能达到：

你能解释训练日志里的 loss 大致代表什么，知道指标上升或下降说明什么。

推荐实践任务：

用 COCO128 或自己准备的小数据集训练 3～5 个 epoch，观察 loss 和 mAP 变化。

自测问题：

1. loss 下降一定代表模型真实效果好吗？
2. epoch 越多一定越好吗？
3. batch size 太大会出现什么问题？

------

### 阶段 5：训练自己的数据

**一句话目标：** 能把自己的图片和标注变成 YOLOv8 能训练的数据集。

你应该学：

- images/labels 目录结构。
- YOLO 标签格式。
- `data.yaml` 写法。
- 训练、验证、推理、导出。
- precision、recall、mAP。

为什么要学：

这是从“会用别人模型”到“解决自己问题”的关键一步。

学完能达到：

你能训练一个识别自己目标的小模型，比如识别安全帽、零件、车辆、宠物、缺陷等。

推荐实践任务：

准备 100～300 张图片，标注 1～3 个类别，训练一个 `best.pt`。

自测问题：

1. YOLO 标签里的坐标为什么是归一化的？
2. `train`、`val`、`test` 数据集分别有什么作用？
3. `best.pt` 和 `last.pt` 有什么区别？

------

### 阶段 6：阅读源码

**一句话目标：** 不追求一次读懂全部源码，而是沿着“推理主线”和“训练主线”看。

你应该学：

- 推理入口。
- 数据预处理。
- 模型前向传播。
- 后处理。
- 训练器。
- 数据加载。
- loss 计算。
- 保存权重。

为什么要学：

读源码不是为了背文件名，而是为了知道模型运行时每一步在做什么。

学完能达到：

你能大致定位：预测在哪里、loss 在哪里、数据加载在哪里、结果后处理在哪里。

推荐实践任务：

在关键函数里打断点或 `print()`，看图片从路径变成 tensor，再变成检测框。

自测问题：

1. 推理时是否需要标签？
2. 训练时为什么既需要图片又需要标签？
3. 后处理是在模型内部完成，还是在模型输出之后完成？

------

# 二、YOLOv8 能做什么？

Ultralytics YOLO 支持多种视觉任务，官方文档中列出的常见任务包括 detect、segment、classify、pose、OBB、track 等。目标检测任务的输出是边界框、类别标签和置信度；实例分割会进一步输出物体轮廓或 mask；姿态估计输出关键点；OBB 则用于旋转框检测。

## 1. 目标检测 detect

**一句话解释：** 找出图片里有哪些物体，并用矩形框框出来。

生活类比：

你看一张街景照片，会说：

“这里有一个人，这里有一辆车，这里有一个红绿灯。”

目标检测模型做的事情很像你眼睛加大脑的组合：

- 看整张图。
- 找出可能有物体的位置。
- 判断每个位置是什么类别。
- 给每个判断一个可信程度。

技术解释：

输入是一张图片，输出通常是多组结果：

```
类别 class
置信度 confidence
边界框 bbox
```

例如：

```
person 0.92 [120, 80, 260, 430]
car    0.85 [300, 200, 520, 360]
```

表示模型认为：

- 第一个框里是人，可信度 0.92。
- 第二个框里是车，可信度 0.85。

简单伪代码：

```
image = read_image("street.jpg")
results = model(image)

for box in results:
    print(box.class_name, box.confidence, box.coordinates)
```

YOLOv8 中的实际作用：

目标检测是 YOLOv8 最核心、最常用的任务。比如：

- 安防摄像头检测人。
- 工厂检测缺陷。
- 自动驾驶检测车辆、行人、交通标志。
- 零售场景检测商品。
- 农业检测果实、病虫害。

常见误区：

- 目标检测不是只判断“图片里有没有猫”，而是要知道“猫在哪里”。
- 检测框不是人工画出来的，而是模型预测出来的。
- 置信度高不代表绝对正确，只是模型更有把握。

------

## 2. 实例分割 segment

**一句话解释：** 不只框出物体，还要描出物体的具体轮廓。

通俗类比：

目标检测像用矩形框圈住一只狗；实例分割像用剪刀沿着狗的边缘剪出来。

适合场景：

- 医学图像分割。
- 工业缺陷轮廓。
- 人像抠图。
- 道路区域识别。

------

## 3. 图像分类 classify

**一句话解释：** 判断整张图片属于哪个类别。

通俗类比：

给你一张图片，只问你：“这是猫还是狗？”

它不关心猫在哪里，只关心整张图是什么。

适合场景：

- 垃圾分类。
- 产品种类分类。
- 病害类别分类。

------

## 4. 姿态估计 pose

**一句话解释：** 找出人体或物体的关键点。

通俗类比：

不是只框出一个人，而是标出鼻子、肩膀、手腕、膝盖等关键点。

适合场景：

- 健身动作分析。
- 人体姿态识别。
- 体育动作捕捉。
- 行为分析。

------

## 5. 旋转框检测 OBB

**一句话解释：** 用可以旋转的矩形框检测倾斜物体。

通俗类比：

普通检测框像横平竖直的快递盒；旋转框可以斜着贴合物体。

适合场景：

- 遥感图像中的飞机、船只、建筑。
- 倾斜文本检测。
- 工业零件检测。

------

# 三、YOLOv8 基本使用方法

## 0. Windows 环境建议

先确认环境：

```
nvidia-smi
```

看显卡驱动和 CUDA 支持情况。

再确认 PyTorch 是否能用 GPU：

```
python -c "import torch; print(torch.__version__); print(torch.cuda.is_available()); print(torch.cuda.get_device_name(0) if torch.cuda.is_available() else 'CPU')"
```

如果输出：

```
True
NVIDIA ...
```

说明 PyTorch 能调用 GPU。

安装或更新 Ultralytics：

```
pip install -U ultralytics
```

进入你的 YOLOv8 项目或工作目录：

```
cd D:\projects\yolov8
```

------

## 1. 命令行方式使用 YOLOv8

官方 CLI 的基本格式是：

```
yolo TASK MODE ARGS
```

例如目标检测推理：

```
yolo detect predict model=yolov8n.pt source="D:\data\images\test.jpg" conf=0.25 save=True
```

参数解释：

```
detect   表示任务是目标检测
predict  表示模式是推理/预测
model    使用哪个模型权重
source   输入来源，可以是图片、文件夹、视频、摄像头
conf     置信度阈值，低于这个值的框会被过滤
save     是否保存可视化结果
```

------

## 2. Python API 使用 YOLOv8

官方 Python API 支持加载模型、训练、验证、预测等操作。

```
from ultralytics import YOLO

# 1. 加载预训练模型
model = YOLO("yolov8n.pt")

# 2. 对图片进行预测
results = model.predict(
    source=r"D:\data\images\test.jpg",
    conf=0.25,
    save=True
)

# 3. 查看结果
for result in results:
    print(result.boxes)
```

背后的含义：

- `YOLO("yolov8n.pt")`：加载一个已经训练好的小模型。
- `source`：输入图片路径。
- `conf=0.25`：只保留置信度大于 0.25 的结果。
- `save=True`：保存画好框的图片。
- `result.boxes`：检测框结果。

------

## 3. 图片检测示例

命令行：

```
yolo detect predict model=yolov8n.pt source="D:\data\images\cat.jpg" conf=0.25 save=True
```

Python：

```
from ultralytics import YOLO

model = YOLO("yolov8n.pt")

results = model.predict(
    source=r"D:\data\images\cat.jpg",
    conf=0.25,
    save=True
)

for result in results:
    boxes = result.boxes

    for box in boxes:
        cls_id = int(box.cls[0])
        conf = float(box.conf[0])
        xyxy = box.xyxy[0].tolist()

        print("类别ID:", cls_id)
        print("置信度:", conf)
        print("边界框:", xyxy)
```

`xyxy` 表示：

```
[x1, y1, x2, y2]
```

也就是：

```
左上角 x
左上角 y
右下角 x
右下角 y
```

------

## 4. 视频检测示例

命令行：

```
yolo detect predict model=yolov8n.pt source="D:\data\videos\test.mp4" conf=0.25 save=True
```

Python：

```
from ultralytics import YOLO

model = YOLO("yolov8n.pt")

results = model.predict(
    source=r"D:\data\videos\test.mp4",
    conf=0.25,
    save=True
)
```

视频推理的本质：

```
视频 = 很多帧图片
YOLOv8 会逐帧检测
再把检测结果保存成新视频
```

------

## 5. 摄像头检测示例

Windows 通常默认摄像头编号是 `0`。

命令行：

```
yolo detect predict model=yolov8n.pt source=0 conf=0.25 show=True
```

Python：

```
from ultralytics import YOLO

model = YOLO("yolov8n.pt")

model.predict(
    source=0,
    conf=0.25,
    show=True
)
```

参数解释：

```
source=0      使用默认摄像头
show=True     弹窗显示检测结果
conf=0.25     置信度阈值
```

如果有多个摄像头，可以尝试：

```
source=1
source=2
```

------

## 6. 如何查看检测结果

默认情况下，结果通常保存在：

```
runs\detect\predict
runs\detect\predict2
runs\detect\predict3
```

你可以在 Windows 中打开：

```
explorer runs\detect\predict
```

如果你想指定项目名：

```
yolo detect predict model=yolov8n.pt source="D:\data\images" project="D:\yolo_runs" name="exp1" save=True
```

输出位置：

```
D:\yolo_runs\exp1
```

------

## 7. 如何理解类别、置信度和边界框

假设检测结果是：

```
dog 0.91 [100, 80, 300, 420]
```

含义是：

```
类别：dog
置信度：0.91
边界框：左上角坐标是 (100, 80)，右下角坐标是 (300, 420)
```

分层讲解：

### 类别 class

1. 一句话解释：模型认为这个框里的物体是什么。
2. 通俗类比：你看见一个动物，说“这是狗”。
3. 技术解释：模型对每个候选位置输出类别概率。
4. 简单代码：

```
cls_id = int(box.cls[0])
class_name = model.names[cls_id]
```

1. YOLOv8 中作用：决定框上显示 `person`、`car`、`dog` 等名称。
2. 常见误区：类别 ID 不是类别名称，需要通过 `model.names` 映射。

### 置信度 confidence

1. 一句话解释：模型对这个检测结果有多确定。
2. 通俗类比：你说“我 90% 确定这是狗”。
3. 技术解释：通常综合了目标存在性和类别判断的可信程度。
4. 简单代码：

```
conf = float(box.conf[0])
```

1. YOLOv8 中作用：过滤低质量检测框。
2. 常见误区：置信度不是准确率。

### 边界框 bbox

1. 一句话解释：物体在图片中的位置。
2. 通俗类比：用矩形便利贴把物体圈起来。
3. 技术解释：常见格式包括 `xyxy`、`xywh`。
4. 简单代码：

```
xyxy = box.xyxy[0].tolist()
xywh = box.xywh[0].tolist()
```

1. YOLOv8 中作用：用于画框、裁剪目标、统计位置。
2. 常见误区：训练标签里的 `xywh` 通常是归一化坐标，推理输出里的像素坐标不是一回事。

------

# 四、YOLOv8 底层原理通俗解释

## 1. YOLO 这个名字是什么意思？

YOLO = **You Only Look Once**。

一句话解释：

模型只看一遍图片，就同时预测出物体位置和类别。

通俗类比：

传统方法像“先扫一遍哪里可能有东西，再逐个确认是什么”；YOLO 像高手看一眼照片，就说出“左边有人，右边有车，远处有红绿灯”。

技术解释：

YOLO 把目标检测做成一个端到端的神经网络预测问题，一次前向传播直接输出多个检测框、类别和置信度。

伪代码：

```
image = preprocess(image)
predictions = model(image)
detections = postprocess(predictions)
```

YOLOv8 中作用：

YOLOv8 的优势之一是推理流程简洁，适合实时检测。

常见误区：

“只看一次”不是说模型只训练一次，而是说推理时一次前向传播完成主要预测。

------

## 2. 为什么 YOLO 是 one-stage detector？

一句话解释：

YOLO 不先单独生成候选区域，而是一步直接预测检测结果。

通俗类比：

两阶段方法像：

```
第一步：先圈出所有可疑区域
第二步：再判断每个区域是什么
```

YOLO 像：

```
直接在整张图上同时找位置和类别
```

技术解释：

one-stage detector 直接在特征图上预测边界框、类别和置信度。two-stage detector 通常先生成 region proposals，再分类和回归。

YOLOv8 中作用：

速度快，适合实时应用。

常见误区：

one-stage 不代表一定比 two-stage 准，也不代表结构一定简单，只是检测流程不同。

------

## 3. YOLO 和两阶段检测方法有什么区别？

| 对比项   | YOLO / one-stage   | Faster R-CNN / two-stage |
| -------- | ------------------ | ------------------------ |
| 流程     | 一步预测           | 先找候选框，再分类       |
| 速度     | 通常更快           | 通常更慢                 |
| 实时性   | 更适合实时         | 较难实时                 |
| 结构理解 | 对新手更直观       | 流程更复杂               |
| 使用场景 | 工业部署、视频检测 | 高精度研究场景较多       |

通俗说：

YOLO 像“扫一眼全图直接答题”；两阶段方法像“先圈重点，再精读”。

------

## 4. 一张图片输入 YOLOv8 后经过哪些步骤？

大致流程：

```
原始图片
→ resize / letterbox
→ 转成 tensor
→ 归一化
→ backbone 提取特征
→ neck 融合特征
→ head 预测框、类别
→ 后处理 NMS
→ 输出检测结果
```

简单伪代码：

```
img = load_image(path)
img = preprocess(img)

pred = model(img)

boxes = postprocess(pred)

draw_boxes(img, boxes)
```

你可以把它理解成：

```
看图
→ 提取线条、纹理、形状
→ 组合成物体特征
→ 判断哪里有什么
→ 删除重复答案
→ 输出最终结果
```

------

## 5. backbone、neck、head 是什么？

### backbone

1. 一句话解释：负责提取图像特征。
2. 通俗类比：像人的眼睛和初级视觉系统，先看边缘、颜色、纹理、形状。
3. 技术解释：由卷积模块组成，把图片变成多层特征图。
4. 伪代码：

```
features = backbone(image)
```

1. YOLOv8 中作用：从图片中提取有用信息。
2. 常见误区：backbone 不是直接输出检测框。

------

### neck

1. 一句话解释：负责融合不同尺度的特征。
2. 通俗类比：像把“近处细节”和“远处全局信息”结合起来。
3. 技术解释：将浅层高分辨率特征和深层语义特征融合。
4. 伪代码：

```
fused_features = neck(features)
```

1. YOLOv8 中作用：帮助同时检测大目标和小目标。
2. 常见误区：neck 不是可有可无，小目标检测很依赖它。

------

### head

1. 一句话解释：负责输出最终预测。
2. 通俗类比：像答题卡，最终写上“这里是人，那里是车”。
3. 技术解释：在特征图上预测边界框、类别分数等。
4. 伪代码：

```
predictions = head(fused_features)
```

1. YOLOv8 中作用：产生检测框、类别、置信度。
2. 常见误区：head 输出的原始结果还需要后处理。

------

## 6. 模型是怎么预测边界框的？

一句话解释：

模型不是“画框”，而是预测一组数字，再把数字转换成框。

通俗类比：

你告诉别人一个物体的位置，可以说：

```
它的中心在这里，宽这么多，高这么多
```

技术解释：

检测框通常可以用：

```
x_center, y_center, width, height
```

或：

```
x1, y1, x2, y2
```

表示。

伪代码：

```
box = model_output_to_box(pred)
```

YOLOv8 中作用：

head 会在不同尺度的特征图上预测框的位置。

常见误区：

模型预测的不是图片上一个“真实矩形对象”，而是数字坐标。

------

## 7. 模型是怎么预测类别的？

一句话解释：

对每个候选框，模型会判断它像哪个类别。

通俗类比：

看到一个动物，你脑中会比较：

```
像猫？像狗？像鸟？
```

技术解释：

模型会输出每个类别的分数，分数最高的类别通常作为预测类别。

伪代码：

```
scores = [0.01, 0.92, 0.03]
class_id = argmax(scores)
```

YOLOv8 中作用：

决定每个框的类别名称。

常见误区：

分类结果依赖训练数据。没训练过的类别，模型一般检测不出来。

------

## 8. 模型怎么判断“这里有一个物体”？

一句话解释：

模型会给候选位置一个“像不像目标”的分数。

通俗类比：

你扫一眼图片，有些区域明显像物体，有些只是背景。

技术解释：

YOLO 系列会根据框位置、类别分数等信息生成检测置信度。最终通过 `conf` 阈值过滤结果。

伪代码：

```
if confidence > 0.25:
    keep_box()
```

YOLOv8 中作用：

过滤背景区域和低质量预测。

常见误区：

置信度低不一定完全错，可能只是模型不确定。

------

## 9. NMS 或后处理在做什么？

NMS = Non-Maximum Suppression，非极大值抑制。

一句话解释：

从多个重复框里，只保留最好的那个。

通俗类比：

五个人同时指着同一只狗说“这里有狗”，NMS 会说：

“你们说的是同一个目标，保留最靠谱的那一个答案。”

技术解释：

如果多个框重叠很高，并且类别相同，保留置信度最高的框，删除其他框。

伪代码：

```
boxes = sort_by_confidence(boxes)

for box in boxes:
    if box overlaps too much with selected_box:
        remove(box)
```

YOLOv8 中作用：

减少重复检测框，让最终结果更干净。

常见误区：

NMS 不是模型训练出来的，它通常是后处理规则。

------

## 10. anchor-free 是什么意思？YOLOv8 为什么说是 anchor-free？

一句话解释：

anchor-free 表示模型不再依赖预先设计的一堆 anchor 框来预测目标。

通俗类比：

anchor-based 像提前准备很多不同大小的模板框，让模型在模板基础上调整。

anchor-free 像模型直接学习“目标中心在哪里、框应该多大”。

技术解释：

早期 YOLO 常使用 anchor boxes，即预设不同宽高比的参考框。YOLOv8 检测头采用 anchor-free 思路，减少对 anchor 设计的依赖。

伪代码理解：

```
# anchor-based 思路
box = anchor + predicted_offset

# anchor-free 思路
box = predicted_position_and_size
```

YOLOv8 中作用：

训练和迁移到新数据集时更方便，不需要特别关心 anchor 聚类。

常见误区：

anchor-free 不代表没有任何网格或特征点，只是不依赖预设 anchor 框模板。

------

## 11. 训练时模型到底在学习什么？

一句话解释：

模型在学习如何从图片特征推断出正确的框、类别和置信度。

通俗类比：

像小孩看很多带答案的图片：

```
这是一只猫，框在这里。
这是一个人，框在这里。
这是汽车，框在这里。
```

看多了以后，小孩学会自己找物体。

技术解释：

训练时，模型先预测，然后和人工标注对比，计算损失，再通过反向传播调整参数。

伪代码：

```
for images, labels in dataloader:
    predictions = model(images)
    loss = compute_loss(predictions, labels)
    loss.backward()
    optimizer.step()
```

YOLOv8 中作用：

让模型从随机或预训练参数，逐渐变成适合你任务的检测器。

常见误区：

模型不是记住每张图片，而是学习可泛化的视觉规律。

------

## 12. 损失函数大概由哪些部分组成？

不用复杂公式，直觉上主要包括：

### 框的位置损失

预测框和真实框差多远。

```
框偏了 → 惩罚
框越准 → 惩罚越小
```

### 分类损失

类别有没有判断错。

```
真实是狗，预测成猫 → 惩罚
真实是狗，预测成狗 → 惩罚小
```

### 置信度/目标相关损失

模型是否正确判断这里有没有目标。

```
背景被当成物体 → 惩罚
有物体却没检测到 → 惩罚
```

YOLOv8 中，训练的核心就是让这些损失整体下降。

常见误区：

loss 不是单一含义，它通常由多个部分组成。

------

## 13. 为什么需要很多标注好的图片？

一句话解释：

模型需要从大量例子中学习规律。

通俗类比：

你只看过一张猫的照片，可能只认识那一只猫；看过各种颜色、姿态、光照、角度的猫，才真正知道“猫”的概念。

技术解释：

数据越丰富，模型越不容易过拟合，泛化能力越好。

好数据应该包含：

- 不同角度。
- 不同光照。
- 不同背景。
- 不同大小。
- 遮挡情况。
- 正常和困难样本。

常见误区：

图片数量不是唯一因素，标签质量非常关键。

------

## 14. 常见训练参数影响什么？

| 参数      | 作用         | 通俗理解           | 常见问题                   |
| --------- | ------------ | ------------------ | -------------------------- |
| `epochs`  | 训练轮数     | 看几遍教材         | 太少学不够，太多可能过拟合 |
| `batch`   | 每次喂几张图 | 每次做几道题再改错 | 太大爆显存，太小训练不稳定 |
| `lr`      | 学习率       | 每次改错步子多大   | 太大震荡，太小学得慢       |
| `imgsz`   | 输入图片尺寸 | 看图时缩放到多大   | 大图更准但更慢、更耗显存   |
| `device`  | 训练设备     | 用 CPU 还是 GPU    | Windows 下常用 `device=0`  |
| `workers` | 数据加载线程 | 几个人帮你搬数据   | Windows 出问题可设小一些   |

------

# 五、训练自己的 YOLOv8 数据集

## 1. 数据集怎么准备？

假设你要检测两类目标：

```
helmet
person
```

你需要准备：

```
图片：jpg/png
标签：txt
配置：data.yaml
```

建议起步数量：

```
每类至少 100～300 个目标实例
```

如果只是学习流程，几十张也能跑通，但效果不一定好。

------

## 2. 图片和标签怎么组织？

推荐结构：

```
D:\datasets\helmet\
│
├── images\
│   ├── train\
│   │   ├── 0001.jpg
│   │   ├── 0002.jpg
│   │   └── ...
│   │
│   ├── val\
│   │   ├── 0101.jpg
│   │   └── ...
│   │
│   └── test\
│       ├── 0201.jpg
│       └── ...
│
├── labels\
│   ├── train\
│   │   ├── 0001.txt
│   │   ├── 0002.txt
│   │   └── ...
│   │
│   ├── val\
│   │   ├── 0101.txt
│   │   └── ...
│   │
│   └── test\
│       ├── 0201.txt
│       └── ...
│
└── data.yaml
```

要求：

```
images/train/0001.jpg
对应
labels/train/0001.txt
```

文件名必须对应，只是后缀不同。

------

## 3. YOLO 格式标签是什么？

每张图片对应一个 `.txt` 文件。

如果图片里有两个物体，标签文件可能是：

```
0 0.512 0.438 0.210 0.350
1 0.300 0.600 0.180 0.420
```

每一行表示一个目标。

格式是：

```
class_id x_center y_center width height
```

------

## 4. class、x_center、y_center、width、height 是什么意思？

假设一行标签：

```
0 0.512 0.438 0.210 0.350
```

含义：

```
class_id = 0
目标中心点 x = 图片宽度的 51.2%
目标中心点 y = 图片高度的 43.8%
框宽度 = 图片宽度的 21.0%
框高度 = 图片高度的 35.0%
```

注意：

这些不是像素坐标，而是归一化坐标。

------

## 5. 为什么坐标要归一化？

一句话解释：

为了让不同尺寸的图片用统一格式表示。

通俗类比：

你说“物体在图片中间偏右一点”，不管图片是 640×480 还是 1920×1080，这个描述都能成立。

技术解释：

归一化坐标范围通常是 0 到 1：

```
x_center = 目标中心点像素 x / 图片宽度
y_center = 目标中心点像素 y / 图片高度
width    = 框像素宽度 / 图片宽度
height   = 框像素高度 / 图片高度
```

伪代码：

```
x_center_norm = x_center_pixel / image_width
y_center_norm = y_center_pixel / image_height
w_norm = box_width_pixel / image_width
h_norm = box_height_pixel / image_height
```

常见误区：

YOLO 标签中不是 `x1 y1 x2 y2`，而是中心点加宽高。

------

## 6. data.yaml 怎么写？

示例：

```
path: D:/datasets/helmet

train: images/train
val: images/val
test: images/test

names:
  0: helmet
  1: person
```

也可以写成：

```
train: D:/datasets/helmet/images/train
val: D:/datasets/helmet/images/val
test: D:/datasets/helmet/images/test

names:
  0: helmet
  1: person
```

解释：

```
path   数据集根目录
train  训练图片路径
val    验证图片路径
test   测试图片路径，可选
names  类别 ID 到类别名称的映射
```

重要：

标签路径不用在 `data.yaml` 里单独写，YOLOv8 会根据 `images` 自动去找对应的 `labels`。

------

## 7. 如何启动训练？

Windows + NVIDIA GPU 命令示例：

```
yolo detect train model=yolov8n.pt data="D:\datasets\helmet\data.yaml" epochs=100 imgsz=640 batch=16 device=0 workers=4 project="D:\yolo_runs" name="helmet_yolov8n"
```

参数解释：

```
detect train        目标检测训练
model=yolov8n.pt    从 YOLOv8 nano 预训练模型开始训练
data=...            数据集配置文件
epochs=100          训练 100 轮
imgsz=640           输入图片尺寸
batch=16            每批 16 张图片
device=0            使用第 0 块 NVIDIA GPU
workers=4           数据加载线程数
project=...         输出根目录
name=...            本次实验名称
```

如果显存不够，降低 batch：

```
batch=8
```

甚至：

```
batch=4
```

如果 Windows 下 dataloader 报错，可以试：

```
workers=0
```

------

## 8. 如何验证模型效果？

训练完成后，通常会生成：

```
D:\yolo_runs\helmet_yolov8n\weights\best.pt
D:\yolo_runs\helmet_yolov8n\weights\last.pt
```

验证命令：

```
yolo detect val model="D:\yolo_runs\helmet_yolov8n\weights\best.pt" data="D:\datasets\helmet\data.yaml" imgsz=640 device=0
```

含义：

```
val       在验证集上评估
model     使用训练好的 best.pt
data      使用同一份数据配置
imgsz     验证输入尺寸
device    使用 GPU
```

------

## 9. 如何用 best.pt 推理？

图片：

```
yolo detect predict model="D:\yolo_runs\helmet_yolov8n\weights\best.pt" source="D:\test_images" conf=0.25 save=True
```

视频：

```
yolo detect predict model="D:\yolo_runs\helmet_yolov8n\weights\best.pt" source="D:\test_videos\demo.mp4" conf=0.25 save=True
```

摄像头：

```
yolo detect predict model="D:\yolo_runs\helmet_yolov8n\weights\best.pt" source=0 conf=0.25 show=True
```

------

## 10. 如何判断训练是否成功？

主要看：

```
1. 训练过程没有报错
2. loss 整体下降
3. precision / recall / mAP 有提升
4. 用 best.pt 检测新图片效果还不错
5. 不只是训练集效果好，验证集也要好
```

你可以查看输出目录中的图表，例如：

```
results.png
confusion_matrix.png
PR_curve.png
```

------

## 11. precision、recall、mAP50、mAP50-95 是什么意思？

### precision 精确率

一句话解释：

模型检测出来的目标里，有多少是真的。

通俗类比：

你报警 10 次，其中 8 次真的有问题，precision 就高。

适合关注：

```
误报多不多
```

常见场景：

工业缺陷检测里，如果误报太多，工人会很烦。

------

### recall 召回率

一句话解释：

真实存在的目标里，模型找到了多少。

通俗类比：

现场有 10 个缺陷，模型找到了 8 个，recall 就是比较高。

适合关注：

```
漏检多不多
```

常见场景：

安全帽检测里，漏检没戴安全帽的人可能更严重。

------

### mAP50

一句话解释：

在 IoU 阈值 0.5 下的平均检测性能。

通俗类比：

框和真实框重合程度超过 50%，就算比较合格。

特点：

相对宽松。

------

### mAP50-95

一句话解释：

从 IoU 0.5 到 0.95 多个严格程度下综合评估。

通俗类比：

不只看“框差不多对”，还看“框得准不准”。

特点：

更严格，更能反映框的定位质量。

------

## 12. 训练效果不好怎么排查？

按优先级排查：

### 1. 标签是否正确

最常见问题。

检查：

- 类别 ID 是否从 0 开始。
- 框是否框准。
- 有没有漏标。
- 有没有错标。
- 图片和标签文件名是否对应。

### 2. 数据量是否太少

表现：

- 训练集效果好，验证集差。
- 换场景就检测不到。

解决：

- 增加图片。
- 增加不同光照、角度、背景。
- 加入困难样本。

### 3. 类别是否不均衡

例如：

```
person 有 5000 个
helmet 只有 100 个
```

模型可能偏向多数类。

解决：

- 补充少数类数据。
- 清理重复数据。

### 4. 图片尺寸是否合适

小目标多，可以尝试：

```
imgsz=960
```

但会更耗显存。

### 5. 训练轮数是否不够

可以从：

```
epochs=50
```

增加到：

```
epochs=100
epochs=200
```

### 6. 模型太小或太大

`yolov8n.pt` 很快，但能力有限。

可以尝试：

```
model=yolov8s.pt
model=yolov8m.pt
```

显存不够就回到 `n` 或 `s`。

### 7. 置信度阈值设置不合适

推理时：

```
conf=0.25
```

如果漏检多，可以试：

```
conf=0.15
```

如果误检多，可以试：

```
conf=0.4
```

------

# 六、逐步理解 YOLOv8 源码

不要一上来读所有源码。你要抓住两条主线：

```
推理主线
训练主线
```

不同版本源码路径可能会变，以你本地仓库为准。Ultralytics 文档当前强调模型有 Train、Val、Predict、Export、Track、Benchmark 等模式；CLI 和 Python API 都是围绕这些模式组织的。

------

## 1. 推理主线

目标：

```
输入图片
→ 加载模型
→ 预处理
→ 前向传播
→ 后处理
→ 输出结果
```

建议先看这些位置：

```
ultralytics/
├── engine/
│   ├── model.py          # YOLO 类通常会调用这里的统一模型接口
│   ├── predictor.py      # 推理流程主线
│   ├── results.py        # Results、Boxes 等结果对象
│
├── models/
│   └── yolo/
│       └── detect/
│           ├── predict.py    # 检测任务的 predictor
│           ├── train.py      # 检测任务的 trainer
│           └── val.py        # 检测任务的 validator
│
├── nn/
│   ├── tasks.py          # 模型构建、DetectionModel 等
│   └── modules/          # 网络模块
│
├── utils/
│   ├── ops.py            # NMS、坐标变换等工具函数
│   └── plotting.py       # 画框、可视化
```

你可以从 Python API 开始追：

```
from ultralytics import YOLO

model = YOLO("yolov8n.pt")
results = model.predict(source="test.jpg")
```

阅读顺序：

### 第一步：看 `YOLO("yolov8n.pt")`

关注问题：

```
模型权重在哪里加载？
模型结构在哪里创建？
```

关键变量：

```
model
ckpt
self.model
```

### 第二步：看 `model.predict(...)`

关注问题：

```
source 是怎么传进去的？
predictor 是什么时候创建的？
```

关键变量：

```
source
predictor
args
```

### 第三步：看预处理

关注问题：

```
图片怎么 resize？
图片怎么变成 tensor？
BGR/RGB 是否转换？
是否除以 255？
```

关键变量：

```
im
img
orig_img
```

### 第四步：看前向传播

关注问题：

```
输入 tensor 形状是什么？
输出 predictions 形状是什么？
```

关键变量：

```
preds
features
```

### 第五步：看后处理

关注问题：

```
NMS 在哪里做？
坐标如何从缩放图映射回原图？
```

关键变量：

```
pred
boxes
conf
cls
```

### 第六步：看 Results

关注问题：

```
result.boxes.xyxy 是怎么来的？
result.names 是什么？
```

关键变量：

```
Results
Boxes
xyxy
xywh
conf
cls
```

新手可以先跳过：

```
分布式训练
混合精度细节
各种导出格式
TensorRT/OpenVINO 部署细节
复杂 callback 系统
```

------

## 2. 训练主线

目标：

```
加载数据
→ 数据增强
→ 构建模型
→ 前向传播
→ 计算损失
→ 反向传播
→ 更新参数
→ 验证与保存模型
```

从命令开始：

```
yolo detect train model=yolov8n.pt data="D:\datasets\helmet\data.yaml" epochs=10 imgsz=640
```

或 Python：

```
from ultralytics import YOLO

model = YOLO("yolov8n.pt")

model.train(
    data=r"D:\datasets\helmet\data.yaml",
    epochs=10,
    imgsz=640,
    batch=8,
    device=0
)
```

建议阅读顺序：

### 第一步：训练入口

看：

```
ultralytics/engine/trainer.py
```

重点关注：

```
train()
_train()
```

理解：

```
整个训练循环在哪里？
每个 epoch 做了什么？
每个 batch 做了什么？
```

关键变量：

```
epoch
batch
loss
optimizer
scheduler
metrics
```

------

### 第二步：检测任务训练器

看：

```
ultralytics/models/yolo/detect/train.py
```

重点关注：

```
检测任务和通用训练器有什么不同？
模型怎么构建？
数据集怎么构建？
validator 怎么指定？
```

关键变量：

```
DetectionTrainer
train_loader
model
validator
```

------

### 第三步：数据加载和增强

常见位置：

```
ultralytics/data/
```

重点关注：

```
图片路径怎么读？
标签怎么读？
增强在哪里做？
batch 怎么拼起来？
```

关键变量：

```
img
labels
bboxes
cls
batch
```

新手重点看：

```
一张图片如何对应一个 txt 标签
标签如何变成 tensor
```

------

### 第四步：模型结构

看：

```
ultralytics/nn/tasks.py
ultralytics/nn/modules/
```

重点关注：

```
模型 YAML 怎么解析？
backbone、neck、head 怎么搭起来？
```

关键变量：

```
cfg
model
layers
Detect
```

新手可以先不深究每个模块公式，只要知道：

```
卷积模块负责提特征
融合模块负责多尺度信息
Detect head 负责输出预测
```

------

### 第五步：loss 计算

常见位置：

```
ultralytics/utils/loss.py
```

重点关注：

```
预测结果和真实标签怎么匹配？
box loss 怎么算？
class loss 怎么算？
DFL 等细节可以后看
```

关键变量：

```
preds
batch
targets
loss
loss_items
```

新手最容易卡在：

```
模型输出的 shape 很复杂
标签匹配逻辑比较绕
DFL/TaskAlignedAssigner 等术语陌生
```

建议策略：

第一次看 loss，不要逐行死磕。先抓住：

```
预测框 vs 真实框
预测类别 vs 真实类别
总损失 = 多个损失加权相加
```

------

### 第六步：验证和保存模型

看：

```
ultralytics/models/yolo/detect/val.py
ultralytics/engine/validator.py
```

重点关注：

```
mAP 在哪里统计？
best.pt 根据什么保存？
验证集何时跑？
```

关键变量：

```
metrics
mAP
best_fitness
```

------

# 七、重要概念分层讲解

下面把几个最关键概念再用统一格式讲一遍。

------

## 概念 1：目标检测

### 1. 一句话解释

找出图片中每个目标的位置和类别。

### 2. 通俗类比

像你在照片上用笔圈出“人”“车”“狗”。

### 3. 技术解释

模型输入图片，输出多个边界框，每个框带类别和置信度。

### 4. 简单代码

```
from ultralytics import YOLO

model = YOLO("yolov8n.pt")
results = model("test.jpg")

for result in results:
    for box in result.boxes:
        print(box.cls, box.conf, box.xyxy)
```

### 5. YOLOv8 中实际作用

这是 YOLOv8 detect 任务的核心。

### 6. 常见误区

目标检测不是图像分类；它不仅要知道“是什么”，还要知道“在哪里”。

------

## 概念 2：边界框

### 1. 一句话解释

边界框是圈住目标的矩形。

### 2. 通俗类比

像给图片里的物体贴一个矩形标签。

### 3. 技术解释

常见表示方式：

```
xyxy: x1, y1, x2, y2
xywh: x_center, y_center, width, height
```

### 4. 简单代码

```
xyxy = box.xyxy[0].tolist()
xywh = box.xywh[0].tolist()
```

### 5. YOLOv8 中实际作用

推理时用于画框；训练时用于和真实框比较。

### 6. 常见误区

训练标签中的 YOLO 格式是归一化 `xywh`，不是像素级 `xyxy`。

------

## 概念 3：置信度

### 1. 一句话解释

模型对检测结果的相信程度。

### 2. 通俗类比

像你说“我 90% 确定这是猫”。

### 3. 技术解释

低于 `conf` 阈值的预测会被过滤。

### 4. 简单代码

```
model.predict("test.jpg", conf=0.25)
```

### 5. YOLOv8 中实际作用

控制输出结果数量。

### 6. 常见误区

置信度高不等于一定正确，置信度低也不等于一定错误。

------

## 概念 4：NMS

### 1. 一句话解释

去掉重复框，只保留最好的框。

### 2. 通俗类比

几个人都指着同一个物体，最后只采纳最靠谱的那个人。

### 3. 技术解释

根据框的重叠程度和置信度进行筛选。

### 4. 简单伪代码

```
boxes = sort_by_confidence(boxes)
selected = []

for box in boxes:
    if not too_overlap(box, selected):
        selected.append(box)
```

### 5. YOLOv8 中实际作用

让最终检测结果更干净。

### 6. 常见误区

NMS 不是训练出来的能力，而是后处理规则。

------

## 概念 5：loss

### 1. 一句话解释

loss 衡量模型预测和真实答案差多远。

### 2. 通俗类比

像考试扣分：框错了扣分，类别错了扣分，漏检也扣分。

### 3. 技术解释

检测 loss 通常包含框损失、分类损失等部分。

### 4. 简单伪代码

```
pred = model(images)
loss = criterion(pred, labels)
loss.backward()
optimizer.step()
```

### 5. YOLOv8 中实际作用

指导模型更新参数。

### 6. 常见误区

loss 低不一定代表实际检测效果一定好，要结合验证集 mAP 看。

------

## 概念 6：mAP

### 1. 一句话解释

mAP 是目标检测中综合衡量模型效果的指标。

### 2. 通俗类比

不是只看你答对多少，还看你框得准不准、漏没漏、误报多不多。

### 3. 技术解释

mAP 综合 precision-recall 曲线以及 IoU 阈值。

### 4. 简单理解

```
mAP50     较宽松
mAP50-95  更严格
```

### 5. YOLOv8 中实际作用

常用来判断 `best.pt`。

### 6. 常见误区

只看 mAP 不够，还要实际看预测图片。

------

# 八、7～14 天实践学习计划

下面给你一个 10 天计划，比较适合你的背景。

------

## Day 1：跑通推理

学习目标：

熟悉 YOLOv8 的基本命令。

理论内容：

- YOLO 是什么。
- detect/predict 是什么。
- 图片、视频、摄像头输入。

实践任务：

```
yolo detect predict model=yolov8n.pt source="D:\data\images\test.jpg" save=True
yolo detect predict model=yolov8n.pt source="D:\data\videos\test.mp4" save=True
yolo detect predict model=yolov8n.pt source=0 show=True
```

检查标准：

- 能找到输出目录。
- 能看到画好框的图片或视频。
- 能解释 `model`、`source`、`conf`。

常见问题：

- `yolo` 命令找不到：检查 ultralytics 是否安装在当前 Python 环境。
- 摄像头打不开：尝试 `source=1`。
- GPU 没用上：检查 `torch.cuda.is_available()`。

------

## Day 2：用 Python API 推理

学习目标：

能用 Python 控制 YOLOv8。

理论内容：

- `YOLO()` 加载模型。
- `model.predict()` 推理。
- `Results` 和 `Boxes`。

实践任务：

写一个 `predict_image.py`：

```
from ultralytics import YOLO

model = YOLO("yolov8n.pt")

results = model.predict(
    source=r"D:\data\images\test.jpg",
    conf=0.25,
    save=True
)

for result in results:
    for box in result.boxes:
        cls_id = int(box.cls[0])
        conf = float(box.conf[0])
        xyxy = box.xyxy[0].tolist()
        name = model.names[cls_id]

        print(name, conf, xyxy)
```

检查标准：

- 能打印类别名。
- 能打印置信度。
- 能打印坐标。

常见问题：

- 路径报错：Windows 路径建议用 `r"D:\xxx\xxx.jpg"`。
- 类别只显示 ID：使用 `model.names[cls_id]`。

------

## Day 3：理解目标检测输入输出

学习目标：

彻底理解检测结果。

理论内容：

- class。
- confidence。
- bbox。
- `xyxy` 和 `xywh`。

实践任务：

把检测结果保存成 txt 或 CSV。

示例：

```
from ultralytics import YOLO
import csv

model = YOLO("yolov8n.pt")
results = model.predict(r"D:\data\images\test.jpg", conf=0.25)

with open("detections.csv", "w", newline="", encoding="utf-8") as f:
    writer = csv.writer(f)
    writer.writerow(["class_id", "class_name", "confidence", "x1", "y1", "x2", "y2"])

    for result in results:
        for box in result.boxes:
            cls_id = int(box.cls[0])
            class_name = model.names[cls_id]
            conf = float(box.conf[0])
            x1, y1, x2, y2 = box.xyxy[0].tolist()

            writer.writerow([cls_id, class_name, conf, x1, y1, x2, y2])
```

检查标准：

- 能解释 CSV 每一列。
- 能知道坐标是相对原图还是归一化。

常见问题：

- 坐标有小数：这是正常的，模型输出是浮点数。

------

## Day 4：理解 YOLOv8 原理

学习目标：

能用自己的话解释 YOLOv8 为什么能检测物体。

理论内容：

- one-stage detector。
- backbone、neck、head。
- NMS。
- anchor-free。

实践任务：

打印模型结构：

```
from ultralytics import YOLO

model = YOLO("yolov8n.pt")
print(model.model)
```

检查标准：

- 能说出 backbone/neck/head 的作用。
- 能解释 NMS 为什么需要。

常见问题：

- 模型结构太长看不懂：先不要逐层研究，只标出大概模块。

------

## Day 5：准备自己的数据集

学习目标：

能整理 YOLO 格式数据集。

理论内容：

- images/labels。
- train/val/test。
- YOLO 标签格式。
- `data.yaml`。

实践任务：

创建目录：

```
D:\datasets\helmet\
├── images\train
├── images\val
├── labels\train
├── labels\val
└── data.yaml
```

写 `data.yaml`：

```
path: D:/datasets/helmet
train: images/train
val: images/val

names:
  0: helmet
  1: person
```

检查标准：

- 图片和标签文件名能一一对应。
- 标签每行都是 5 个字段。
- 类别 ID 从 0 开始。

常见问题：

- 标签坐标不是 0～1：说明没有归一化。
- 类别 ID 写成类别名：YOLO 标签里要写数字 ID。

------

## Day 6：训练第一个自定义模型

学习目标：

跑通训练流程。

理论内容：

- epoch。
- batch。
- imgsz。
- device。
- best.pt / last.pt。

实践任务：

```
yolo detect train model=yolov8n.pt data="D:\datasets\helmet\data.yaml" epochs=50 imgsz=640 batch=8 device=0 workers=4 project="D:\yolo_runs" name="helmet_first"
```

检查标准：

- 训练能开始。
- 能看到 loss。
- 能生成 `best.pt`。

常见问题：

- CUDA out of memory：减小 `batch` 或 `imgsz`。
- Windows workers 报错：设 `workers=0`。
- 找不到标签：检查 images 和 labels 目录结构。

------

## Day 7：验证和推理自己的模型

学习目标：

知道模型训练得好不好。

理论内容：

- precision。
- recall。
- mAP50。
- mAP50-95。
- 混淆矩阵。

实践任务：

验证：

```
yolo detect val model="D:\yolo_runs\helmet_first\weights\best.pt" data="D:\datasets\helmet\data.yaml" imgsz=640 device=0
```

推理：

```
yolo detect predict model="D:\yolo_runs\helmet_first\weights\best.pt" source="D:\datasets\helmet\images\val" conf=0.25 save=True
```

检查标准：

- 能看懂基本指标。
- 能打开预测图片检查效果。

常见问题：

- 指标不错但实际很差：可能验证集太简单或数据泄漏。
- 检测框偏移：检查标签是否准确。

------

## Day 8：排查训练效果

学习目标：

知道效果不好从哪里改。

理论内容：

- 数据质量。
- 类别不均衡。
- 过拟合。
- 欠拟合。
- 置信度阈值。

实践任务：

分别尝试：

```
conf=0.15
conf=0.25
conf=0.5
```

比较输出结果。

检查标准：

- 能判断误检多还是漏检多。
- 知道该调数据、参数还是阈值。

常见问题：

- 只调参数不改数据：很多检测问题本质是数据问题。

------

## Day 9：阅读推理源码主线

学习目标：

能追踪一次 predict 的流程。

理论内容：

- model.predict。
- predictor。
- preprocess。
- inference。
- postprocess。
- results。

实践任务：

在源码中搜索：

```
def predict
preprocess
postprocess
non_max_suppression
Results
Boxes
```

检查标准：

- 能说出图片从路径到检测框经历了什么。
- 能找到 NMS 大概在哪里。

常见问题：

- 看到很多抽象类：先看 detect 任务相关文件，不要追所有父类细节。

------

## Day 10：阅读训练源码主线

学习目标：

能追踪一次 train 的流程。

理论内容：

- trainer。
- dataloader。
- model forward。
- loss。
- backward。
- optimizer。
- validator。

实践任务：

在源码中搜索：

```
DetectionTrainer
build_dataset
get_dataloader
loss
optimizer.step
validator
```

检查标准：

- 能解释一个 batch 如何参与训练。
- 能找到 loss 计算的大概位置。
- 能知道 best.pt 何时保存。

常见问题：

- loss 细节看不懂：先不硬啃 DFL、assigner，先抓整体流程。

------

# 九、建议你的学习方式

你接下来最稳的学习顺序是：

```
第 1 步：用 yolov8n.pt 熟练推理
第 2 步：打印并理解 Results / Boxes
第 3 步：理解 class、conf、bbox、NMS
第 4 步：准备一个 1～3 类的小数据集
第 5 步：训练 best.pt
第 6 步：验证、推理、排查问题
第 7 步：读推理源码
第 8 步：读训练源码
第 9 步：再深入 loss、assigner、模型结构细节
```

你现在不要急着啃源码全部细节。YOLOv8 的正确入门节奏应该是：

```
先知道怎么用
再知道输出是什么
再知道模型大概怎么得到这些输出
再知道训练时怎么让输出变准
最后再看源码是怎么实现的
```

最重要的三个抓手：

```
1. 一张图片输入后，如何变成检测框？
2. 一个标签 txt，如何指导模型学习？
3. 一个 best.pt，如何从训练中被选出来？
```

只要这三件事打通，YOLOv8 的主线你就已经掌握了。