# Aperture Lab · AI 智能修图网站

基于原生 HTML + CSS + JavaScript 的 AI 图片修图网站，支持首页功能选择、上传图片、AI 编辑器、历史记录等完整功能。

## 目录结构

```
网站设计/
├── index.html          # 主页面（首页 + 编辑器）
├── css/
│   └── style.css       # 样式文件
├── js/
│   └── script.js       # 交互逻辑 + AI 服务层
└── README.md           # 部署与 API 对接说明（本文件）
```

## 快速预览

直接用浏览器打开 `index.html` 即可预览。默认使用本地 Canvas 模拟 AI 处理（演示模式）。

## 部署上线

### 方式一：静态网站托管（推荐）

将整个文件夹上传到任意静态托管服务：

- **Vercel / Netlify**：拖拽文件夹即可部署，免费额度足够
- **GitHub Pages**：推送到 GitHub 仓库，开启 Pages 功能
- **阿里云 OSS / 腾讯云 COS**：上传到对象存储，开启静态网站托管
- **自有服务器**：放到 Nginx / Apache 的网站根目录

部署后访问对应网址即可，发给别人也能打开。

### 方式二：Nginx 部署示例

```nginx
server {
    listen 80;
    server_name yourdomain.com;
    root /var/www/aperture-lab;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }

    # 静态资源缓存
    location ~* \.(css|js|png|jpg|svg)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
}
```

## 对接真实 AI API

### 第一步：修改配置

打开 `js/script.js`，找到顶部的 `CONFIG` 对象：

```javascript
var CONFIG = {
    API: {
        useApi: true,                                    // 改为 true 启用真实 API
        endpoint: 'https://api.yourdomain.com/v1/enhance', // 你的后端接口地址
        timeout: 30000,
        token: 'your-api-token'                          // 如需要鉴权，填入 token
    },
    // ...
};
```

也可以在浏览器控制台动态修改：
```javascript
AIEnhance.config.API.useApi = true;
AIEnhance.config.API.endpoint = 'https://your-api.com/enhance';
```

### 第二步：后端接口规范

**请求**：
- 方法：`POST`
- Content-Type：`multipart/form-data`
- 字段：
  - `image`：图片文件（File）
  - `prompt`：修改需求描述（String）
- 可选请求头：`Authorization: Bearer {token}`

**响应**（JSON）：
```json
{
    "imageUrl": "data:image/png;base64,iVBORw0KGgo..."
}
```

支持的返回格式：
- Base64 Data URL：`data:image/png;base64,...`
- 图片 URL：`https://.../result.png`（需支持跨域访问）

### 第三步：后端示例（Node.js Express）

```javascript
const express = require('express');
const multer = require('multer');
const cors = require('cors');

const app = express();
app.use(cors());

const storage = multer.memoryStorage();
const upload = multer({ storage });

app.post('/api/v1/enhance', upload.single('image'), async (req, res) => {
    try {
        const imageFile = req.file;
        const prompt = req.body.prompt;

        // 调用你的 AI 模型服务
        const resultImage = await callYourAIService(imageFile, prompt);

        res.json({
            imageUrl: `data:image/png;base64,${resultImage.toString('base64')}`
        });
    } catch (err) {
        res.status(500).json({ error: err.message });
    }
});

app.listen(3000);
```

## 功能说明

| 模块 | 说明 |
|------|------|
| 首页功能卡片 | 4 大分类、14+ 修图功能，点击选择后滚动到上传区 |
| 上传区 | 支持拖拽上传 / 点击上传，格式校验、大小限制 |
| AI 编辑器 | 三栏布局：输入素材 / 结果预览 / 控制面板 |
| 自然语言描述 | 在控制面板输入需求，AI 理解并执行 |
| 常用约束标签 | 点击快速填充常用描述词 |
| 历史记录 | 每次生成自动保存，支持查看和下载 |
| AI 对话助手 | 右下角常驻，智能问答修图建议 |
| 响应式适配 | PC / 平板 / 手机完美适配 |

## 浏览器兼容性

- Chrome / Edge 90+
- Firefox 88+
- Safari 14+
- 移动端浏览器均支持

## 技术栈

- 原生 HTML5 + CSS3 + JavaScript (ES5+)
- 无框架依赖，纯静态，部署即用
- AI 服务层可插拔，轻松对接任意后端
