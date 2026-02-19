<p align="center">
  <a href="README.md">English</a> | <a href="README.ja.md">日本語</a> | <strong>中文</strong> | <a href="README.es.md">Español</a> | <a href="README.fr.md">Français</a> | <a href="README.hi.md">हिन्दी</a> | <a href="README.it.md">Italiano</a> | <a href="README.pt-BR.md">Português</a>
</p>

<p align="center">
  <img src="logo.png" alt="mcp-aside 标志" width="280" />
</p>

<h1 align="center">mcp-aside</h1>

<p align="center">
  一个让AI在对话中随时记录想法的MCP服务器 — 不会打断当前任务。
</p>

<p align="center">
  <a href="#快速开始">快速开始</a> &middot;
  <a href="#工作原理">工作原理</a> &middot;
  <a href="#工具">工具</a> &middot;
  <a href="#配置">配置</a> &middot;
  <a href="#许可证">许可证</a>
</p>

---

## 为什么

LLM容易遗忘。一个灵光一闪、一个半成形的顾虑、一句"我们应该回来处理这个"却再也没有回来。**mcp-aside**为模型提供了一个专用收件箱 — 带有速率限制、去重和自动过期功能，这样收件箱本身不会成为问题。

把它想象成对话旁边的便利贴。模型写笔记，你（或模型）在合适的时候阅读。

## 工作原理

1. 模型调用`aside.push`，附带优先级标记的想法。
2. 护栏检查重复、速率限制和TTL上限。
3. 如果通过，插入内容进入内存收件箱。
4. 客户端通过`notifications/resources/updated`收到通知。
5. 任何人都可以通过`interject://inbox`资源读取收件箱。

无数据库。无持久化。服务器停止，收件箱消失 — 这是设计使然。

## 快速开始

```bash
npm install
npm run build
node build/index.js
```

服务器通过**stdio**使用MCP。将任何MCP兼容客户端指向它：

```json
{
  "mcpServers": {
    "aside": {
      "command": "node",
      "args": ["build/index.js"]
    }
  }
}
```

## 工具

| 工具 | 功能 |
|---|---|
| `aside.push` | 将插入内容推送到收件箱。接受`text`、`priority`（low/med/high）、`reason`、`tags`、`expiresAt`、`source`和`meta`。 |
| `aside.configure` | 运行时调整护栏 — TTL上限、速率限制、去重窗口、通知阈值。 |
| `aside.clear` | 清空收件箱。 |
| `aside.status` | 收件箱大小和当前护栏配置的只读快照。 |

## 资源

| URI | 描述 |
|---|---|
| `interject://inbox` | 待处理插入内容的JSON数组，按最新排序。过期项目在读取时过滤。 |

## 护栏

所有设置均可通过`aside.configure`配置。默认值：

| 设置 | 默认值 | 控制内容 |
|---|---|---|
| `defaultTtlSeconds` | 600（10分钟） | 未设置明确过期时间时插入内容的存活时间 |
| `maxTtlSeconds` | 3600（1小时） | TTL硬上限，即使调用者请求更长时间 |
| `dedupeWindowSeconds` | 300（5分钟） | 相同优先级+文本+原因 = 在此窗口内被抑制 |
| `rateLimitWindowSeconds` | 60 | 速率限制的滑动窗口 |
| `rateLimitMax` | low: 6, med: 3, high: 1 | 每个优先级每个窗口的最大推送次数 |
| `notifyAtOrAbove` | high | 仅对此优先级及以上的项目发送日志通知 |

## 配置

### 定时触发器

内置定时器每5分钟触发一次，推送低优先级的"有阻碍吗？"签到。它遵循与手动推送相同的护栏（因此会像其他内容一样被去重或限速）。在`index.ts`中注释掉`startTimerTrigger`调用即可禁用。

### MCP检查器

本地测试用：

```
Transport: STDIO
Command:   node
Args:      build/index.js
```

## 备注

- 日志输出到**stderr** — stdout保留给MCP JSON-RPC。
- 收件箱是临时的。重启 = 全新开始。
- 插入内容按最新排序存储。过期项目在每次读取和推送时清理。

## 许可证

[MIT](LICENSE)
