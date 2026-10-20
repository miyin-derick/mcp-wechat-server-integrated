# 微信 MCP · 图片处理与监听集成

基于 [Howardzhangdqs/mcp-wechat-server](https://github.com/Howardzhangdqs/mcp-wechat-server) 的个人扩展版本，在原有微信消息工具上增加图片下载与解密能力，并提供连接 OpenClaw 的常驻监听示例。

## 与上游的关系

扫码登录、文本收发、长轮询、打字状态和登录状态持久化来自上游。本仓库新增的内容集中在以下两部分：

| 扩展 | 内容 | 代码入口 |
| --- | --- | --- |
| 图片处理 | 暴露消息中的媒体信息；新增 `download_image` 工具，下载、解密图片并返回类型、大小与内容，可选择保存到本地 | [media.ts](src/tools/media.ts)、[工具注册](src/index.ts) |
| 监听示例 | 长轮询微信消息，支持唤醒 / 休眠词，将文字与图片问题转发到 OpenClaw，并提供 macOS launchd 配置示例 | [示例说明](examples/wechat-claw-daemon/README.md) |

技术栈：TypeScript · Bun · MCP SDK · Zod。

## 功能

- **7 个 MCP 工具**：登录、消息轮询、文本发送、打字状态、图片下载与解密
- **扫码登录**：生成终端文本二维码、PNG 图片、URL 链接三种方式
- **长轮询监听**：阻塞等待新消息，最长可等待 7 天
- **打字状态**：Agent 处理消息时自动显示"对方正在输入..."
- **状态持久化**：重启不丢失登录凭证和消息游标
- **独立运行**：无需 OpenClaw 框架，开箱即用

## 快速开始

使用本仓库源码运行扩展版本：

```bash
git clone https://github.com/miyin-derick/mcp-wechat-server-integrated.git
cd mcp-wechat-server-integrated
bun install
bun run start
```

### Claude Desktop / 支持 `mcpServers` 的客户端

将 `/absolute/path/to/mcp-wechat-server-integrated` 替换为本地仓库的绝对路径；确保客户端能找到 `bun`，必要时也为它填写绝对路径。

```json
{
  "mcpServers": {
    "wechat": {
      "command": "bun",
      "args": ["run", "/absolute/path/to/mcp-wechat-server-integrated/src/index.ts"]
    }
  }
}
```

### OpenCode

```json
{
  "mcp": {
    "wechat": {
      "type": "local",
      "command": ["bun", "run", "/absolute/path/to/mcp-wechat-server-integrated/src/index.ts"],
      "enabled": true
    }
  }
}
```

原 README 中的 `bunx mcp-wechat-server` 安装的是上游 npm 包。要使用本仓库的图片处理扩展，请采用上面的本地源码配置。

### 常驻监听示例

连接 OpenClaw、配置唤醒词和 macOS launchd 的步骤见 [WeChat Claw Daemon](examples/wechat-claw-daemon/README.md)。这是需要自行配置的集成示例，依赖本地 OpenClaw 与有效的微信登录状态。

## 环境要求

- [Bun](https://bun.sh/) >= 1.0.0

## 使用方法

以下基础登录和消息操作沿用上游流程，截图来自上游项目。

### 1. 扫码登录

AI Agent 调用 `login_qrcode` 生成二维码，你可以通过以下方式扫码：

- 在终端运行 `cat ~/.mcp-wechat-server/qrcode.txt` 查看二维码
- 打开图片文件 `~/.mcp-wechat-server/qrcode.png`
- 将链接复制到微信中打开

> **提示**：如果手机扫码后页面一直加载，请切换到**移动数据网络**（关闭 WiFi）。

<img src="assets/opencode_screenshot1.png" alt="OpenCode 登录二维码" width="600">

### 2. 确认登录

Agent 调用 `check_qrcode_status` 确认登录状态。

### 3. 开始聊天

登录成功后，Agent 会自动执行以下流程：

1. 调用 `get_messages` 轮询新消息
2. 收到消息后调用 `send_typing` 显示"正在输入..."
3. 处理完成后调用 `send_text_message` 回复
4. 回复完毕后调用 `send_typing` 取消打字状态

<img src="assets/wechat_screenshot1.jpg" alt="微信收发消息" width="400">

<img src="assets/opencode_screenshot2.png" alt="OpenCode Agent 交互" width="600">

<img src="assets/wechat_screenshot2.jpg" alt="微信收到回复" width="400">

## 工具列表

| 工具 | 说明 |
|------|------|
| `login_qrcode` | 生成微信登录二维码 |
| `check_qrcode_status` | 检查二维码是否已扫码确认 |
| `logout` | 退出登录并清除凭证 |
| `get_messages` | 拉取新消息（`wait=true` 阻塞等待直到收到消息） |
| `send_text_message` | 发送文本消息 |
| `send_typing` | 发送或取消"正在输入"状态 |
| `download_image` | 下载并解密图片，可选择保存到本地 |

## 数据存储

所有数据保存在 `~/.mcp-wechat-server/` 目录下：

| 文件 | 说明 |
|------|------|
| `account.json` | Bot Token 和用户 ID（权限 600） |
| `state.json` | 消息游标和上下文 Token |
| `qrcode.png` | 生成的二维码图片 |
| `qrcode.txt` | 生成的终端二维码文本 |

## 本地开发

```bash
bun run dev
```

## 来源与许可

上游项目：[Howardzhangdqs/mcp-wechat-server](https://github.com/Howardzhangdqs/mcp-wechat-server)。本仓库保留上游提交历史；图片处理和监听示例的扩展记录可在提交历史中查看。

许可证沿用上游声明：MIT。
