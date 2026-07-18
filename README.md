# AI-Chat

一个基于 Flask、Ollama 和 Live2D 的本地 AI 聊天应用。项目提供一个带有二次元傲娇人设的聊天界面，后端通过 Ollama 调用本地大语言模型，前端使用单页 HTML 实现聊天窗口、流式回复、角色互动、情绪气泡和基础 token 统计。

项目默认使用 `qwen3:32b` 模型，适合在本地机器或局域网环境中运行和调试。

## 功能特点

- 本地 AI 聊天：通过 Ollama 调用本地模型，不依赖外部在线 API。
- 傲娇角色人设：后端内置角色 prompt，使回复保持二次元傲娇少女风格。
- 流式输出：使用 Server-Sent Events 实现逐段返回，前端实时显示回复内容。
- Live2D 展示：前端加载 Shizuku Live2D 模型，提供角色展示和点击互动。
- 情绪互动：支持随机情绪文本、点击气泡和长时间无操作后的孤独消息。
- 聊天历史：前端维护对话上下文，并在请求时发送给后端。
- token 估算：前端按字符长度粗略估算输入和输出 token 数量。
- 局域网访问：Flask 默认监听 `0.0.0.0:5000`，同一局域网设备可以访问。

## 项目结构

```text
AI-Chat/
├── app.py              # Flask 后端入口，提供页面、聊天接口和静态资源
├── chat.html           # 前端单页应用，包含 HTML/CSS/JavaScript
├── emotions.txt        # 角色情绪和互动文本
├── flask.log           # Flask 运行日志
├── ollama.png          # AI 头像
├── ollama_user.png     # 用户头像
├── srs.md              # 软件需求和设计说明
├── LICENSE             # 许可证
└── README.md           # 项目说明
```

## 环境要求

- Python 3.8 或更高版本
- Ollama
- 本地可运行的大语言模型
- 建议内存 16GB 以上；如果使用 `qwen3:32b`，对内存和显存要求较高

Python 依赖：

```bash
pip install flask flask-cors requests
```

## 准备 Ollama

安装 Ollama 后，拉取默认模型：

```bash
ollama pull qwen3:32b
```

启动 Ollama 服务：

```bash
ollama serve
```

后端默认请求地址是：

```text
http://localhost:11434/api/generate
```

如果需要换模型，可以修改 `app.py` 中的配置：

```python
MODEL_NAME = "qwen3:32b"
```

## 运行项目

进入项目目录：

```bash
cd AI-Chat
```

安装依赖：

```bash
pip install flask flask-cors requests
```

启动服务：

```bash
python app.py
```

浏览器打开：

```text
http://localhost:5000
```

如果想在局域网内访问，查看运行机器的局域网 IP，然后在其他设备访问：

```text
http://<运行机器IP>:5000
```

## 后端接口

### `GET /`

返回主聊天页面 `chat.html`。

### `POST /api/chat`

发送聊天消息，后端调用 Ollama 并以 SSE 形式返回流式内容。

请求示例：

```json
{
  "message": "你好",
  "history": [
    {
      "role": "user",
      "content": "你是谁？"
    },
    {
      "role": "assistant",
      "content": "哼，我才不是特意来陪你聊天的呢。"
    }
  ]
}
```

返回内容使用 `text/event-stream`，每段数据格式类似：

```text
data: {"content": "回复片段"}
```

完成时返回：

```text
data: {"done": true}
```

### `GET /api/emotions`

读取并返回 `emotions.txt` 中的情绪文本。

### `GET /ollama.png`

返回 AI 头像。

### `GET /ollama_user.png`

返回用户头像。

## 前端说明

`chat.html` 是一个独立单页应用，主要包含：

- 聊天窗口和消息气泡
- 用户输入框
- AI 与用户头像
- 动态背景
- Live2D 角色加载
- 点击互动层
- 情绪气泡
- 关键词触发的全屏特效
- token 估算展示

Live2D 模型通过 jsDelivr CDN 加载：

```text
live2d-widget@3.1.4
live2d-widget-model-shizuku@1.0.5
```

如果网络无法访问 CDN，Live2D 角色可能不会显示，但基础聊天功能仍可使用。

## 常见问题

### 页面能打开，但聊天没有回复

先确认 Ollama 服务已经启动：

```bash
ollama serve
```

再确认模型已经下载：

```bash
ollama list
```

如果没有 `qwen3:32b`，需要先拉取：

```bash
ollama pull qwen3:32b
```

### 模型太大，机器跑不动

可以在 `app.py` 中把 `MODEL_NAME` 改成较小的本地模型，例如已经安装好的其他 Ollama 模型。

修改后重启 Flask 服务。

### Live2D 不显示

Live2D 资源来自 CDN。检查浏览器控制台和网络连接，确认可以访问 jsDelivr。

### 局域网设备打不开页面

确认：

- Flask 服务仍在运行。
- 防火墙允许访问 5000 端口。
- 访问地址使用运行机器的局域网 IP。
- 运行机器和访问设备在同一局域网内。

## 注意事项

- 当前项目适合本地实验和学习，不建议直接暴露到公网。
- 默认开启 Flask `debug=True`，生产环境使用前需要关闭 debug 并增加认证和安全配置。
- 聊天历史由前端维护，刷新页面后不会自动恢复。
- token 统计只是按字符数粗略估算，不等同于模型真实 tokenizer 结果。
- `qwen3:32b` 对硬件要求较高，首次加载和回复速度取决于本机性能。

## 相关文档

更完整的软件需求和设计说明见：

- [srs.md](./srs.md)
