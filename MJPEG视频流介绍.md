<img src="C:\Users\wangk\AppData\Roaming\Typora\typora-user-images\image-20260418135338310.png" alt="image-20260418135338310" style="zoom:33%;" />

**MJPEG（Motion JPEG）** 是一种非常直观的视频流编码方式，本质上就是：

> 👉 **把一帧一帧的静态 JPEG 图片连续播放，形成视频**

每一帧都是独立压缩的 JPEG 图像，没有帧之间的依赖关系。

------

## 二、MJPEG 的核心特点

### 1）编码方式

- 每一帧：单独 JPEG 压缩
- 没有：
  - I帧 / P帧 / B帧
  - 运动补偿
  - 帧间预测

👉 可以理解为：
 **视频 = JPEG图片序列 + 时间顺序**

------

### 2）优点

- ✅ 实现简单（编码/解码容易）
- ✅ 延迟低（无需等待参考帧）
- ✅ 任意帧可直接访问（适合抓拍）
- ✅ 对丢包不敏感（丢一帧不影响其他帧）

------

### 3）缺点

- ❌ 带宽占用大（没有利用时间冗余）
- ❌ 压缩效率低
- ❌ 不适合长时间、高分辨率视频传输

------

## 三、MJPEG 的传输形式

### 常见协议/封装方式：

#### 1）HTTP MJPEG（最常见）

浏览器可直接播放：

```
Content-Type: multipart/x-mixed-replace; boundary=frame
```

数据格式类似：

```sql
--frame
Content-Type: image/jpeg

(binary JPEG data)
--frame
Content-Type: image/jpeg

(binary JPEG data)
```

👉 应用：

- 摄像头 Web 预览
- 嵌入式设备视频输出

------

#### 2）RTSP + MJPEG

- MJPEG 也可以封装在 RTSP 流中
- 多用于监控设备

------

#### 3）文件格式

- AVI（MJPEG编码）
- MOV（部分支持）

------

## 四、MJPEG 的处理方式（开发角度）

### 1）解码流程

1. 接收流（HTTP / RTSP）
2. 按 boundary 分割帧
3. 提取 JPEG 数据
4. 解码为图像（RGB/YUV）

------

### 2）处理流程示意

```mathematica
MJPEG Stream
   ↓
Frame Split（按boundary分帧）
   ↓
JPEG Decode
   ↓
Image Processing（OpenCV等）
   ↓
Display / Save / AI分析
```

------

### 3）常用工具/库

- OpenCV
- FFmpeg
- 浏览器 `<img src="mjpeg_url">`

------

## 五、MJPEG vs 主流视频编码

<img src="C:\Users\wangk\AppData\Roaming\Typora\typora-user-images\image-20260418135540202.png" alt="image-20260418135540202" style="zoom:33%;" /><img src="C:\Users\wangk\AppData\Roaming\Typora\typora-user-images\image-20260418135616025.png" alt="image-20260418135616025" style="zoom:33%;" />

| 特性       | MJPEG    | H.264 / H.265 |
| ---------- | -------- | ------------- |
| 压缩方式   | 帧内压缩 | 帧间压缩      |
| 带宽       | 高       | 低            |
| 延迟       | 低       | 中            |
| 实现复杂度 | 简单     | 高            |
| 抗丢包     | 强       | 较弱          |
| 随机访问   | 极好     | 一般          |

------

## 六、扩展：其他常见视频流类型

### 1）H.264（AVC）

- H.264
- 当前最主流编码

**特点：**

- 高压缩比
- GOP结构（I/P/B帧）
- 广泛支持（浏览器/硬件）

👉 应用：

- 直播
- 视频会议
- 监控

------

### 2）H.265（HEVC）

- H.265

**特点：**

- 比 H.264 节省约 30–50% 带宽
- 计算复杂度更高

👉 应用：

- 4K/8K视频
- 高清监控

------

### 3）RTSP 视频流

- RTSP

**特点：**

- 实时性强
- 常配合 H.264/H.265

👉 用于：

- IPC 摄像头
- NVR 系统

------

### 4）RTMP

- RTMP

**特点：**

- 低延迟
- 常用于推流

👉 用于：

- 直播推流（OBS → 服务器）

------

### 5）HLS（HTTP Live Streaming）

- HLS

**特点：**

- 分片（.ts + m3u8）
- 延迟较高（几秒）

👉 用于：

- 网页视频播放
- Apple生态

------

### 6）WebRTC

- WebRTC

**特点：**

- 超低延迟（<1秒）
- 点对点传输

👉 用于：

- 视频通话
- 实时互动直播

------

## 七、总结（关键理解）

- **MJPEG = 一串 JPEG 图片**
- 优点：简单、低延迟、易处理
- 缺点：带宽大、不高效

👉 如果你在做：

- 📷 摄像头快速预览 → MJPEG 很合适
- 📡 大规模视频传输 → 用 H.264 / H.265
- ⚡ 实时互动 → WebRTC