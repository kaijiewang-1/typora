

# PicGo + Typora 图片管理终极方案（小白友好版）

## 一、为什么需要 PicGo + Typora？

Typora 是一款「所见即所得」的 Markdown 编辑器，写文档时经常需要插入图片。但直接拖拽图片到 Typora 会生成本地路径，换设备或分享文档时图片会丢失。

**PicGo** 是一款开源的「图片上传工具」，支持一键将图片上传到图床（如 GitHub、Gitee、阿里云 OSS 等），生成稳定可访问的网络链接。搭配 Typora 后，拖拽图片即可自动上传并插入图床链接，彻底解决图片管理痛点！

## 二、准备工作

### 1. 安装 PicGo

- **方式 1（推荐）：下载桌面版**
  访问 PicGo 官网，下载对应系统的安装包（Windows/macOS/Linux），直接安装。
  https://molunerfinn.com/assets/img/PicGo.2e8c4b30.png

- **方式 2（极客向）：npm 安装**
  若已安装 Node.js，命令行执行：

  bash

  复制

  ```bash
  npm install picgo -g
  ```

### 2. 注册一个图床（以 GitHub 为例）

PicGo 支持多种图床，推荐 **GitHub + jsdelivr**（免费、稳定、全球可访问）。
​**步骤：​**​

1. 注册 GitHub 账号，创建一个公开仓库（如命名 `my-pic-bed`）。
2. 生成 GitHub Token：
   - 进入 GitHub Settings → Developer settings → Personal access tokens。
   - 点击 `Generate new token`，勾选 `repo` 权限（必须），生成 Token（复制保存，仅展示一次）。

## 三、配置 PicGo 上传到 GitHub

### 1. 安装 GitHub 图床插件（若需要）

新版 PicGo 核心已内置 GitHub 上传支持，无需额外插件。若提示缺失，可手动安装：
打开 PicGo → 插件设置 → 搜索 `picgo-plugin-github-uploader` → 安装。

### 2. 配置 GitHub 图床参数

1. 打开 PicGo → 图床设置 → 找到 `GitHub` 配置项。

2. 填写以下信息：

   - `Repo`：你的仓库名（格式：`用户名/仓库名`，如 `wangkaijie/my-pic-bed`）。
   - `Token`：刚才复制的 GitHub Token。
   - `Branch`：分支（默认 `main` 或 `master`）。
   - `Custom URL`：图片访问路径（必填！推荐 `https://cdn.jsdelivr.net/gh/你的用户名/仓库名@分支名`）。
   - `Path`：图片在仓库中的存储路径（如 `/img/`，可选）。

   示例配置：
   https://picgo.github.io/PicGo-Doc/zh/guide/github.png

### 3. 测试上传

点击 PicGo 界面的「上传区」，拖拽一张本地图片进去，或点击「选择图片」上传。
上传成功后，会显示图片的 GitHub 原始链接和 jsdelivr 加速链接（推荐用后者，更快）。

## 四、Typora 集成 PicGo（自动上传图片）

### 1. 配置 Typora 图片上传设置

1. 打开 Typora → `文件 → 偏好设置 → 图像`。

2. 找到「上传服务」部分，选择 `PicGo`。

3. 点击「选择 PicGo 路径」：

   - 若用桌面版 PicGo，选择安装目录下的 `PicGo.exe`（Windows）或 `PicGo.app`（macOS）。
   - 若用 npm 安装，路径通常是 `node_modules/picgo/bin/picgo.js`（或直接填 `picgo` 命令，需确保环境变量已配置）。

   https://typora.io/assets/images/image_setting.png

### 2. 开启「自动上传图片」

在 Typora 图像设置中，勾选：

- `插入图片时自动上传`（拖拽/粘贴图片自动上传）。
- `对本地位置应用上述规则`（可选，控制本地图片是否上传）。

### 3. 测试 Typora + PicGo 联动

在 Typora 中拖拽一张图片，或复制粘贴图片：

- PicGo 会自动上传图片到 GitHub 图床。
- Typora 中图片会自动替换为图床链接（如 `https://cdn.jsdelivr.net/gh/wangkaijie/my-pic-bed@main/img/test.png`）。

## 五、常见问题解决

### 1. 上传失败：Token 权限不足

检查 GitHub Token 是否勾选了 `repo` 权限（必须），或重新生成 Token。

### 2. 图片链接无法访问

- 确认 `Custom URL` 格式正确（`https://cdn.jsdelivr.net/gh/用户名/仓库名@分支名`）。
- 检查 GitHub 仓库是否为公开（私有仓库需 token 有权限，但 jsdelivr 可能无法访问私有仓库）。

### 3. Typora 不自动上传

- 确认 PicGo 路径配置正确（可在命令行手动运行 `picgo upload 图片路径` 测试）。
- 检查 Typora 图像设置中「自动上传」是否勾选。

## 六、扩展：其他图床推荐

- **Gitee**（国内访问快）：类似 GitHub，需在 Gitee 创建仓库并生成 Token，配置 PicGo 的 Gitee 图床。
- **阿里云 OSS**（付费但稳定）：适合需要大容量、高带宽的用户，配置需填写 Bucket、AccessKey 等。
- **腾讯云 COS**：类似阿里云 OSS，配置方式类似。

## 总结

通过 PicGo + Typora，你可以实现：
✅ 拖拽图片自动上传到图床
✅ 自动生成稳定网络链接
✅ 告别本地图片丢失问题

赶紧试试吧！有具体问题可以留言～ 😊
