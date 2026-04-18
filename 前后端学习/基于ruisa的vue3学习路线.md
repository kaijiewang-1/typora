# vite.config.js

Vue 3 + Vite 项目的配置文件，用于配置开发服务器、模块解析等。

```JS
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { fileURLToPath, URL } from 'node:url'

export default defineConfig({//导出配置
  plugins: [vue()],//启用vue3插件，让vite能编译和处理vue单文件组件.vue文件
  resolve: {
    alias: {
      '@': fileURLToPath(new URL('./src', import.meta.url)),//
    },
  },
  server: {
    port: 5173,
    proxy: {
      '/api': { target: 'http://127.0.0.1:8080', changeOrigin: true, ws: true },
      '/docs': { target: 'http://127.0.0.1:8080', changeOrigin: true },
      '/openapi.json': { target: 'http://127.0.0.1:8080', changeOrigin: true },
    },
  },
})

```

1. **`/api` 路径代理**
   - `target: 'http://127.0.0.1:8080'`：转发到本地 8080 端口的后端服务
   - `changeOrigin: true`：修改请求头中的 origin 字段为目标 URL
   - `ws: true`：支持 WebSocket 协议代理
2. **`/docs` 路径代理**
   - 转发 API 文档页面到后端服务（通常是 Swagger/OpenAPI 文档）
3. **`/openapi.json` 路径代理**
   - 转发 OpenAPI 规范文件到后端服务

## 实际工作流程示例

当你在前端代码中发起请求时：

```
javascript// 前端代码
fetch('/api/users')
```

**实际流程**：

1. 请求发送到 `http://localhost:5173/api/users`

2. Vite 开发服务器检测到 `/api` 前缀

3. 根据代理配置，将请求转发到 `http://127.0.0.1:8080/api/users`

4. 后端服务处理请求并返回响应

   # index.html

```HTML
<body>
  <div id="app"></div>
  <script type="module" src="/src/main.js"></script>
</body>
```

- **Vue 应用的挂载点**
- Vue 会将整个应用渲染到这个 `<div>` 元素内
- 在 `main.js` 中会通过 `app.mount('#app')` 挂载
- 初始为空，由 Vue 动态填充内容



### 应用启动流程

```
浏览器加载 index.html
    ↓
解析到 <script src="/src/main.js">
    ↓
加载 main.js（Vue 应用入口）
    ↓
创建 Vue 应用实例
    ↓
挂载到 <div id="app">
    ↓
页面渲染完成
```

## main.js

```js
import { createApp } from 'vue'
import App from './App.vue'
import './assets/app.css'

createApp(App).mount('#app')

```

#### **步骤 1：`createApp(App)`**

- 创建一个 Vue 应用实例
- 将 `App` 组件设置为根组件
- 返回一个应用实例对象

#### **步骤 2：`.mount('#app')`**

- 将应用**挂载**到 DOM 元素上
- `'#app'` 是 CSS 选择器，指向 `index.html` 中的 `<div id="app"></div>`
- 执行后，Vue 会接管该元素，渲染整个应用

这个文件虽然只有 5 行代码，但完成了：

- ✅ 初始化 Vue 3 应用
- ✅ 设置根组件
- ✅ 加载全局样式
- ✅ 挂载到 DOM

### 执行流程

```
1. 导入 createApp 函数
       ↓
2. 导入根组件 App.vue
       ↓
3. 加载全局样式 app.css
       ↓
4. 创建 Vue 应用实例（以 App 为根组件）
       ↓
5. 挂载到 #app 元素
       ↓
6. Vue 开始渲染，页面显示
```

# app.css

### **全局重置与基础样式**（第 1-10 行）

```css
* { margin: 0; padding: 0; box-sizing: border-box; }
body {
    font-family: 'Segoe UI', system-ui, sans-serif;
    background: #080c18;
    color: #d0dff5;
    height: 100vh;
    overflow: hidden;
    display: flex;
    flex-direction: column;
}
```

#### **含义：**

- **`*` 通配符选择器**：清除所有元素的默认边距和内边距
- **`box-sizing: border-box`**：盒模型计算包含 padding 和 border
- `body` 样式
  - 使用系统字体栈（优先 Segoe UI）
  - 深蓝黑色背景 `#080c18`
  - 浅蓝灰色文字 `#d0dff5`
  - 占满整个视口高度
  - 禁止滚动（内部元素独立滚动）
  - Flexbox 垂直布局

------

### 2. **头部导航栏样式**（第 11-45 行）

```css
.header {
    height: 56px;
    background: #0b1022;
    border-bottom: 1px solid #1b2f55;
    padding: 0 20px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    flex-shrink: 0;
}
```

#### **关键子元素：**

| 类名           | 作用       | 特点                           |
| :------------- | :--------- | :----------------------------- |
| `.header-left` | 左侧区域   | Flex 布局，间距 12px           |
| `.logo`        | Logo 图标  | 36x36px，渐变蓝色背景，圆角    |
| `.title`       | 主标题     | **渐变文字效果**（青蓝到紫色） |
| `.subtitle`    | 副标题     | 小号灰色文字                   |
| `.status-pill` | 状态胶囊   | 圆角边框容器，显示连接状态     |
| `.status-dot`  | 状态指示灯 | 红色（离线）/绿色（在线）      |
| `.link-docs`   | 文档链接   | 悬停变亮效果                   |

#### **渐变文字技术：**

```css
.title {
    background: linear-gradient(135deg, #00c8ff, #a78bfa);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
}
```

- 将渐变背景裁剪到文字形状
- 文字本身透明，显示背景渐变

------

### 3. **主体布局**（第 47-66 行）

```css
.main {
    flex: 1;              /* 占据剩余空间 */
    display: flex;        /* 水平排列两列 */
    min-height: 0;        /* 允许收缩 */
    border-top: 1px solid #142240;
}
.col-chat {               /* 左侧聊天列 */
    width: 44%;
    border-right: 1px solid #182a4a;
    min-width: 320px;
    background: #0a0e18;
}
.col-side {               /* 右侧面板列 */
    flex: 1;              /* 自适应宽度 */
}
```

**布局结构：**

```
┌─────────────────────────────────────┐
│           Header (56px)             │
├──────────────┬──────────────────────┤
│              │                      │
│  col-chat    │     col-side         │
│   (44%)      │      (flex:1)        │
│              │                      │
└──────────────┴──────────────────────┘
```

------

### 4. **面板标题样式**（第 67-83 行）

```css
.panel-title {
    padding: 10px 16px;
    font-size: 11px;
    font-weight: 600;
    color: #5a8ac0;
    border-bottom: 1px solid #182a4a;
    background: #0c1220;
}
.panel-title::before {
    content: '';
    display: inline-block;
    width: 3px; height: 11px;
    background: linear-gradient(to bottom, #00c8ff, #0055ff);
    border-radius: 2px;
    margin-right: 8px;
    vertical-align: middle;
}
```

- **伪元素 `::before`**：在标题前添加蓝色渐变装饰条
- 增强视觉层次感

------

### 5. **聊天消息区域**（第 85-116 行）

```css
.agent-messages {
    flex: 1;
    overflow-y: auto;     /* 垂直滚动 */
    padding: 14px;
    display: flex;
    flex-direction: column;
    gap: 10px;            /* 消息间距 */
}
```

#### **消息气泡样式：**

| 类名             | 对齐方式              | 背景色          | 圆角特点   |
| :--------------- | :-------------------- | :-------------- | :--------- |
| `.msg.user`      | 右对齐 (`flex-end`)   | 蓝色渐变        | 右下角直角 |
| `.msg.assistant` | 左对齐 (`flex-start`) | 深蓝背景 + 边框 | 左下角直角 |

```css
.msg.user {
    align-self: flex-end;
    background: linear-gradient(135deg, #0055ff, #00c8ff);
    border-bottom-right-radius: 4px;  /* 小圆角指向用户 */
}
```

#### **自定义滚动条：**

```css
.agent-messages::-webkit-scrollbar { width: 4px; }
.agent-messages::-webkit-scrollbar-thumb { 
    background: #1a3060; 
    border-radius: 2px; 
}
```

------

### 6. **快捷操作按钮**（第 118-136 行）

```css
.quick-btn {
    padding: 5px 10px;
    background: #0f2248;
    border: 1px solid #1e4a9a;
    border-radius: 12px;
    color: #6a9acc;
    font-size: 11px;
    cursor: pointer;
}
.quick-btn:hover { 
    background: #1a3a7a; 
    color: #00c8ff; 
    border-color: #00c8ff; 
}
```

- 悬停时高亮为青色
- 禁用状态降低透明度

------

### 7. **调试工具栏**（第 138-198 行）

```css
.agent-debug-bar {
    display: none;        /* 默认隐藏 */
    background: #1a1520;  /* 紫黑色背景 */
    border-top: 1px solid #4a3060;
}
.agent-debug-bar.show { display: flex; }  /* 激活时显示 */
```

#### **任务指示器动画：**

```css
.agent-task-spinner {
    border: 2px solid #1a3060;
    border-top-color: #00c8ff;
    animation: spin 1s linear infinite;
}
@keyframes spin { to { transform: rotate(360deg); } }
```

- 旋转加载动画

------

### 8. **输入区域**（第 200-266 行）

```css
.agent-input {
    padding: 12px 14px;
    background: #0c1528;
    border-top: 1px solid #1b3060;
    display: flex;
    gap: 10px;
}
```

#### **麦克风按钮动画：**

```css
.agent-mic-btn.listening {
    border-color: #00e87a;
    animation: mic-pulse 1.2s ease-in-out infinite;
}
@keyframes mic-pulse {
    0%, 100% { box-shadow: 0 0 0 2px rgba(0, 232, 122, 0.2); }
    50%      { box-shadow: 0 0 0 6px rgba(0, 232, 122, 0.35); }
}
```

- 录音时绿色脉冲扩散效果

#### **发送按钮：**

```css
.agent-send-btn {
    background: linear-gradient(135deg, #00c8ff, #0055ff);
    border-radius: 50%;  /* 圆形 */
}
```

------

### 9. **终端日志区域**（第 268-307 行）

```css
.term-wrap {
    flex: 1;
    background: #060910;  /* 最深色背景 */
}
.term-bar {
    /* macOS 风格窗口控制按钮 */
}
.tdot.r { background: #ff5f57; }  /* 红 */
.tdot.y { background: #febc2e; }  /* 黄 */
.tdot.g { background: #28c840; }  /* 绿 */
```

#### **日志颜色分类：**

```css
.log-line .t { color: #1e3050; }  /* 时间戳 - 暗灰 */
.log-line .i { color: #00c8ff; }  /* 信息 - 青色 */
.log-line .e { color: #ff6b6b; }  /* 错误 - 红色 */
.log-line .s { color: #00e87a; }  /* 成功 - 绿色 */
```

------

### 10. **任务流程面板**（第 309-347 行）

```css
.flow-panel {
    max-height: 38%;
    min-height: 160px;
    border-top: 1px solid #182a4a;
}
.task-grid {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 8px;
}
```

- **响应式网格布局**：自动填充，最小 200px 宽
- 任务卡片自适应排列

------

### 11. **提示框（Toast）**（第 349-359 行）

```css
.toast-container {
    position: fixed; 
    top: 64px; 
    right: 16px; 
    z-index: 300;
}
.toast.error { 
    background: #2a0a0a; 
    border: 1px solid #ff444444; 
    color: #ff6b6b; 
}
@keyframes slideIn { 
    from { transform: translateX(12px); opacity: 0; } 
    to { transform: none; opacity: 1; } 
}
```

- 右上角固定定位
- 滑入动画效果

------

### 12. **摄像头预览弹窗**（第 361-432 行）

```css
.camera-modal {
    display: none;
    position: fixed;
    inset: 0;           /* top/right/bottom/left: 0 */
    z-index: 500;
    align-items: center;
    justify-content: center;
}
.camera-modal.open { display: flex; }
.camera-modal-backdrop {
    background: rgba(4, 8, 18, 0.88);
    backdrop-filter: blur(4px);  /* 毛玻璃效果 */
}
```

- 全屏遮罩层
- 背景模糊效果
- 居中显示图片

------

### 13. **辅助样式**（第 434-450 行）

```css
#app {
    height: 100vh;
    display: flex;
    flex-direction: column;
    overflow: hidden;
}
.msg {
    white-space: pre-wrap;   /* 保留换行和空格 */
    word-break: break-word;  /* 长单词换行 */
}
```

------

## 样式应用机制

### 1. **CSS 优先级规则**

浏览器按以下顺序确定样式优先级（从高到低）：

```
1. !important 声明
2. 内联样式 (style="...")
3. ID 选择器 (#app)
4. 类选择器 (.header)、属性选择器、伪类
5. 元素选择器 (body)、伪元素 (::before)
6. 通配符 (*)
```

**示例：**

```
css/* 优先级低 */
.msg { color: #fff; }

/* 优先级高 */
.msg.user { color: #00c8ff; }

/* 优先级更高 */
#app .msg.user { color: #00e87a; }
```

------

### 2. **样式继承机制**

某些属性会自动从父元素继承：

```
cssbody {
    font-family: 'Segoe UI', system-ui, sans-serif;
    color: #d0dff5;
}
/* 所有子元素默认继承这些属性，除非被覆盖 */
```

**可继承属性：** `color`, `font-family`, `font-size`, `line-height` 等
**不可继承属性：** `margin`, `padding`, `border`, `background` 等

------

### 3. **Flexbox 布局原理**

文件中大量使用 Flexbox：

```
css.header {
    display: flex;           /* 启用 Flex 布局 */
    align-items: center;     /* 垂直居中 */
    justify-content: space-between;  /* 两端对齐 */
}
```

**关键属性：**

- `flex: 1` → `flex-grow: 1`，占据剩余空间
- `flex-shrink: 0` → 不允许收缩
- `gap: 10px` → 子元素间距

------

### 4. **级联（Cascading）规则**

当多个规则冲突时：

```
css/* 后定义的覆盖先定义的 */
.msg { color: #fff; }
.msg { color: #000; }  /* 这个生效 */

/* 更具体的选择器优先 */
.msg { color: #fff; }
.msg.user { color: #00c8ff; }  /* 这个对 .msg.user 生效 */
```

------

### 5. **Vue 组件中的样式应用**

在 Vue 组件中使用这些类：

```
vue<template>
  <div class="header">
    <div class="logo">🤖</div>
    <div class="title">智能体</div>
  </div>
  <div class="agent-messages">
    <div class="msg user">你好</div>
    <div class="msg assistant">Hello!</div>
  </div>
</template>
```

**样式匹配过程：**

1. Vue 渲染 DOM 元素
2. 浏览器解析 `class` 属性
3. 在 CSS 文件中查找匹配的类选择器
4. 应用对应的样式规则
5. 处理冲突和继承

------

## 配色方案总结

| 用途     | 颜色值    | 视觉效果           |
| :------- | :-------- | :----------------- |
| 主背景   | `#080c18` | 深蓝黑             |
| 次背景   | `#0b1022` | 稍亮的深蓝         |
| 强调色   | `#00c8ff` | 青色（链接、按钮） |
| 成功色   | `#00e87a` | 绿色（在线状态）   |
| 警告色   | `#ff6b6b` | 红色（错误）       |
| 文字主色 | `#d0dff5` | 浅蓝灰             |
| 文字辅色 | `#5a8ac0` | 中蓝色             |
| 边框色   | `#1b2f55` | 深蓝灰             |

## 