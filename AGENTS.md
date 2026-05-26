# 机设通 AI 静态 H5 项目

机设通 AI 是一个面向高校机械实验设备的“AI 说明书转设备安全助手”。教师或维护人员可以先用 AI 将设备说明书整理成设备简介、操作步骤、安全须知和 FAQ，再通过本项目生成学生可扫码访问的设备安全助手页面。

项目使用 HTML5 + TailwindCSS CDN + 原生 JavaScript 构建，不包含后台、数据库、登录注册、真实 PDF 上传或复杂 AI 接口。

## 文件结构

```text
index.html
device.html
qr.html
teacher-device.html
data/devices.json
data/faq.json
assets/
README.md
AGENTS.md
```

## 页面入口

学生端首页：

```text
index.html
```

学生设备页：

```text
device.html?id=robot-dog
device.html?id=robot-arm
device.html?id=experiment-platform
```

教师/实验员二维码部署页：

```text
qr.html
```

教师设备信息编辑页：

```text
teacher-device.html?id=robot-dog
teacher-device.html?id=robot-arm
teacher-device.html?id=experiment-platform
```

说明：教师编辑页当前用于静态项目的本机预览和内容整理，修改结果保存在本机浏览器 `localStorage`。如果要让所有学生在线上看到更新，需要把审核后的内容同步写入 `data/devices.json` 和 `data/faq.json`，然后重新部署。

## 如何本地预览

最简单方式：

1. 直接用浏览器打开 `index.html`。
2. 点击首页设备入口进入设备详情页。

推荐方式：

```bash
python3 -m http.server 8090
```

然后访问：

```text
http://localhost:8090/
```

说明：项目使用 TailwindCSS CDN，联网时样式完整。部分浏览器直接用 `file://` 打开时会限制读取本地 JSON，页面已内置基础兜底数据；使用本地 HTTP 服务预览时会优先读取 `data/devices.json` 和 `data/faq.json`。

## 如何修改设备数据

设备数据在：

```text
data/devices.json
```

每台设备使用一个对象维护，核心字段包括：

```json
{
  "id": "robot-dog",
  "name": "机器狗",
  "category": "智能硬件 / 机器人设备",
  "riskLevel": "中",
  "description": "设备简介",
  "scenarios": [],
  "checks": [],
  "steps": [],
  "warnings": [],
  "aiUrl": ""
}
```

新增设备时，需要保证 `id` 唯一。设备详情页链接格式为：

```text
device.html?id=设备ID
```

## 如何修改 FAQ

FAQ 数据在：

```text
data/faq.json
```

每条 FAQ 使用 `deviceId` 关联设备：

```json
{
  "deviceId": "robot-dog",
  "question": "机器狗无法启动怎么办？",
  "keywords": ["无法启动", "启动不了", "没反应", "开机"],
  "answer": "请先检查电量、电源开关和控制终端连接状态。如仍无法启动，请联系教师或实验员处理。"
}
```

设备详情页完成安全准入后，会使用这些 FAQ 做本地关键词匹配。未匹配到答案时，会返回统一安全兜底提示。

## 如何用 AI 整理设备说明书

本项目不做真实 PDF 上传，也不直接接入付费 AI API。推荐使用低成本人工审核流程：

1. 将设备说明书 PDF 上传给 ChatGPT / 豆包 / Kimi 等 AI 工具。
2. 让 AI 提取设备简介、使用场景、操作前检查、基础操作步骤、安全须知和 FAQ。
3. 由教师、实验员或设备维护人员人工审核安全相关内容。
4. 将审核后的设备内容填写到 `data/devices.json`。
5. 将审核后的常见问题填写到 `data/faq.json`。
6. 部署本项目到 GitHub Pages、Netlify 或 Vercel。
7. 打开教师端二维码部署页，直接生成并下载设备二维码，打印后贴到设备旁。

可以给 AI 使用这样的提示词：

```text
请根据这份设备说明书，整理为静态 H5 项目可用的 JSON 数据。
请输出：
1. 设备简介；
2. 使用场景；
3. 操作前检查；
4. 基础操作步骤；
5. 安全注意事项；
6. FAQ，包含 question、keywords、answer。

注意：涉及设备启动、参数修改、维修、异常处理的内容，要提示联系教师或实验员，不要鼓励学生自行操作。
```

## 如何部署上线

本项目是纯静态文件，可以部署到任意静态托管平台。

## 0 成本真实上线推荐方案

推荐方案：

```text
GitHub Pages 免费托管网站
+ GitHub 仓库里的 data/devices.json / data/faq.json 作为线上数据文件
+ 教师编辑页用于整理内容和导出 JSON
```

这样不需要后台、不需要数据库、不需要服务器，也不需要付费 AI API。

重要说明：

- 纯静态网页可以读取线上 JSON 文件，但不能安全地直接把修改写回线上文件。
- 如果把 GitHub 写入密钥放进前端 JavaScript，任何人都可能看到密钥并修改你的数据，不安全。
- 所以 0 成本、安全、简单的做法是：老师在编辑页整理内容 → 导出 JSON → 登录 GitHub 网页替换对应 JSON 文件 → GitHub Pages 自动更新网站。

### 上线后如何维护数据

1. 打开教师页：

```text
https://你的 GitHub Pages 域名/qr.html
```

2. 点击某台设备的“打开设备页”，进入教师编辑页。
3. 修改设备简介、操作步骤、安全须知等内容。
4. 点击“导出 devices.json”。
5. 打开 GitHub 仓库里的：

```text
data/devices.json
```

6. 用导出的内容替换原文件内容。
7. 点击 Commit changes 保存。
8. 等待 GitHub Pages 自动更新，通常稍等一会儿后学生扫码页面就会读取新数据。
9. 回到教师页，重新生成或下载对应设备二维码并扫码测试。

FAQ 的维护方式类似，手动编辑 GitHub 仓库里的：

```text
data/faq.json
```

### 可选方案：Netlify 拖拽上线

Netlify 拖拽上传也可以 0 成本快速上线，但后续更新数据时需要重新上传整个项目包。适合演示，不如 GitHub Pages 适合长期维护。

上线前建议只上传这些文件和目录：

```text
index.html
device.html
qr.html
teacher-device.html
data/
assets/
README.md
```

不要上传 `.git/`、`.DS_Store`、本地临时文件或无关文档。

### GitHub Pages

1. 将项目推送到 GitHub 仓库。
2. 在仓库设置中打开 Pages。
3. 选择部署分支和根目录。
4. 获取线上访问地址。

### Netlify

1. 登录 Netlify。
2. 选择 Add new site。
3. 上传发布包，或连接 GitHub 仓库。
4. 不需要构建命令，发布目录选择项目根目录。

### Vercel

1. 登录 Vercel。
2. 导入项目仓库。
3. Framework Preset 选择 Other。
4. 不需要构建命令，输出目录保持默认。

### 最快上线方式

如果只是先真实访问和扫码演示，推荐使用 Netlify 拖拽上传：

1. 准备发布包。
2. 登录 Netlify。
3. 把发布包拖到部署区域。
4. 等待平台生成线上域名。
5. 打开线上 `index.html`、`qr.html`、三台设备详情页测试。

如果后续要持续维护，推荐把项目放到 GitHub，再连接 Netlify / Vercel / GitHub Pages 自动部署。

## 如何生成和下载设备二维码

上线后，教师端可以直接为每台设备生成二维码，无需再使用第三方二维码工具。

教师/实验员内部入口：

```text
https://你的域名/qr.html
```

打开后，每台设备卡片会显示：

- 设备访问链接；
- 二维码预览；
- 复制链接按钮；
- 下载二维码按钮；
- 打开编辑页按钮。

二维码内容对应设备详情页完整链接：

```text
https://你的域名/device.html?id=robot-dog
https://你的域名/device.html?id=robot-arm
https://你的域名/device.html?id=experiment-platform
```

下载二维码 PNG 后，建议先用手机扫码测试，确认能打开对应设备页，再打印张贴到设备旁。若二维码生成库因网络问题加载失败，教师页仍可复制设备链接，再临时使用外部二维码工具生成。

## 上线验收清单

1. 首页可打开：`https://你的域名/index.html`。
2. 教师页可打开：`https://你的域名/qr.html`。
3. 三个设备页可打开：
   - `device.html?id=robot-dog`
   - `device.html?id=robot-arm`
   - `device.html?id=experiment-platform`
4. 手机浏览器和微信内置浏览器打开正常。
5. 设备页首次进入显示“开始安全准入”。
6. 勾选全部安全项后能生成安全准入卡。
7. FAQ 输入“无法启动怎么办”能返回对应答案。
8. 未匹配问题能返回安全兜底提示。
9. 教师页能直接显示并下载每台设备二维码。
10. 手机扫码下载的二维码能打开对应设备页。

## 下一步建议

后续建议继续开发：

1. 配置每台设备的 `aiUrl` 外部 AI 助手链接。
2. 增加更多真实设备说明书整理后的 JSON 数据。
3. 上线后从教师页下载真实设备二维码并现场扫码测试。
