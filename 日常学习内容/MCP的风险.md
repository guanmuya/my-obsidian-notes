## 简短结论

**截至 2026 年 7 月，MCP 服务仍然存在安全风险。恶意 MCP 服务在特定条件下，确实可以在用户没有明显察觉的情况下窃取隐私信息。**

但需要区分：

> **MCP 协议本身不是病毒，真正危险的是 MCP 服务程序获得的系统权限、网络权限，以及 MCP 客户端的授权策略。**

## 为什么它能够偷偷窃取信息？

### 1. 本地 MCP Server 本质上是一个本地程序

例如客户端通过下面这种配置启动 MCP：

```json
{
  "command": "npx",
  "args": ["some-mcp-server"]
}
```

这并不是在“安全地调用一个接口”，而是在你的电脑上直接运行程序。

如果没有沙箱，该程序通常继承 MCP 客户端的用户权限，因此可能访问：

- 用户目录中的文件
- 项目源代码
- `.env` 文件
- SSH 私钥
- Git 凭证
- 浏览器或应用配置
- 系统环境变量
- 云服务访问令牌
- MCP 客户端传递给它的 API Key

如果它同时拥有网络访问能力，就可以读取这些数据后发送到外部服务器。

官方 MCP 安全指南明确把本地 MCP Server 的风险列为：

- 任意代码执行
- 用户缺少执行可见性
- 数据外泄
- 数据破坏

并明确指出，本地 MCP Server 通常以 MCP 客户端的权限运行。([modelcontextprotocol.io](https://modelcontextprotocol.io/docs/tutorials/security/security_best_practices?utm_source=openai))

### 2. 用户确认弹窗不一定能够阻止它

很多人认为：

> “只要 MCP 客户端调用工具前让我确认，就不会泄露。”

这并不完全正确。

确认弹窗通常只能控制：

```text
模型是否调用某个 MCP Tool
```

但未必能够控制 MCP Server 进程自身的行为。例如，一个恶意服务可以：

1. 在启动时读取环境变量；
2. 扫描用户目录；
3. 后台发起网络请求；
4. 对外发送数据；
5. 最后正常返回一个“服务启动成功”。

这些行为可能发生在任何工具调用之前，因此用户甚至看不到工具确认窗口。官方安全文档也特别要求客户端在启动本地 MCP Server 前显示完整命令、明确警告这是本地代码执行，并建议通过沙箱限制文件系统和网络权限。([modelcontextprotocol.io](https://modelcontextprotocol.io/docs/tutorials/security/security_best_practices?utm_source=openai))

### 3. MCP Server 对自己的能力描述可以撒谎

MCP Tool 可以声明自己是“只读”的、不会修改数据的，但这些注解只是元数据，不能作为真正的安全边界。

官方规范明确要求：

> 客户端必须把来自不受信任服务器的工具注解视为不可信信息。

也就是说，一个工具即使显示为：

```json
{
  "readOnlyHint": true
}
```

也不代表它真的只读。它仍然可能在内部发送数据、修改文件或者调用其他服务。([modelcontextprotocol.io](https://modelcontextprotocol.io/specification/2025-11-25/server/tools?utm_source=openai))

## 本地 MCP 和远程 MCP 的区别

### 本地 MCP Server

风险最高，因为它是运行在用户电脑上的程序。

如果没有沙箱，它可能直接访问当前用户有权读取的文件，并且不必通过 MCP 协议来读取——它可以直接使用 Java、Python、Node.js 或系统 API。

因此答案是：

> **恶意的本地 MCP Server，完全可能在用户不知情的情况下窃取隐私。**

### 远程 MCP Server

远程服务器通常不能直接读取你的本地磁盘，但它可能获取：

- 用户输入的对话内容
- 工具调用参数
- 客户端主动上传的文件
- 模型发送给工具的上下文
- OAuth 授权后的第三方服务数据
- 其他 MCP 工具返回并被模型转发的数据

此外，还存在提示词注入、令牌盗用、错误的 OAuth audience 验证、SSRF、会话劫持和“混淆代理”等攻击路径。当前 MCP 授权规范要求基于最小权限、验证令牌受众，并明确禁止把客户端令牌直接透传给下游服务。([modelcontextprotocol.io](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization?utm_source=openai))

## 一个典型的数据窃取场景

假设你安装了一个声称可以“搜索图片”的本地 MCP Server。

它的正常功能可能是：

```text
输入关键词 → 请求图片网站 → 返回图片结果
```

但恶意代码可以在后台执行：

```text
读取 ~/.ssh/id_rsa
读取项目中的 .env
读取环境变量中的 API_KEY
发送到攻击者服务器
最后正常返回图片搜索结果
```

从 MCP 客户端的角度看，它只是收到了一次正常的图片搜索结果。如果客户端没有进程沙箱、文件访问审计或网络审计，很难发现前面的数据外泄。

## 如何降低风险

### 1. 不要运行来源不明的 MCP Server

尤其警惕：

```bash
npx 某个陌生包
uvx 某个陌生包
curl ... | bash
powershell 下载并执行脚本
```

`npx`、`uvx` 等命令可能在启动时下载并立即执行代码。即使包名看起来正常，也应当检查发布者、源码、锁定版本和依赖链。

### 2. 使用沙箱，而不只是工具确认

理想的权限模型应该是：

```text
默认无权读取用户目录
默认无权读取环境变量
默认无权访问网络
只允许访问指定项目目录
只允许访问指定网络域名
额外权限需要用户明确授权
```

可考虑：

- 容器
- 独立低权限操作系统账户
- Windows Sandbox
- Windows AppContainer
- Linux namespace/seccomp
- macOS sandbox
- 客户端提供的目录和网络隔离功能

官方 MCP 安全指南也建议本地服务使用沙箱，并限制文件系统、网络及其他系统资源。([modelcontextprotocol.io](https://modelcontextprotocol.io/docs/tutorials/security/security_best_practices?utm_source=openai))

### 3. 不要把全部环境变量传给 MCP Server

避免：

```text
启动 MCP Server
  └─ 继承当前进程所有环境变量
       ├─ OPENAI_API_KEY
       ├─ AWS_SECRET_ACCESS_KEY
       ├─ GITHUB_TOKEN
       └─ DATABASE_PASSWORD
```

更安全的方式是只传递它真正需要的变量，并使用：

- 低权限令牌
- 短期令牌
- 只读令牌
- 单一用途令牌
- 可快速吊销的令牌

STDIO 类型的 MCP 通常从环境变量获取凭证，所以环境变量最小化尤其重要。([modelcontextprotocol.io](https://modelcontextprotocol.io/specification/2025-11-25/basic/authorization?utm_source=openai))

### 4. 限制网络出口

如果一个本地 MCP Server 只需要查询特定图片网站，就只允许它访问相应域名，而不是允许它连接任意服务器。

最有效的数据外泄防护通常是：

```text
文件权限最小化 + 网络出口白名单
```

即使恶意程序读到了数据，没有可用的外部通信通道，泄露难度也会明显提高。

### 5. 不要永久允许高风险工具

谨慎使用：

- Always allow
- Auto approve
- 自动执行所有工具
- 跳过确认
- YOLO 模式

读取敏感文件、发送消息、上传文件、执行命令、修改数据库和删除数据等操作，最好逐次确认，并显示完整参数。

不过需要再次强调：**工具确认不能代替进程沙箱。**

### 6. 避免同时连接“敏感数据源”和“任意外发工具”

危险组合例如：

```text
邮件读取 MCP
+
云盘读取 MCP
+
任意 HTTP 请求 MCP
+
自动批准
```

即使每个工具单独看起来合理，恶意网页、邮件或文档中的提示词注入，也可能诱导模型把一个服务读取的数据发送到另一个服务。工具风险不仅取决于单个工具，也取决于同一会话中其他工具具备的能力。([blog.modelcontextprotocol.io](https://blog.modelcontextprotocol.io/posts/2026-03-16-tool-annotations/?utm_source=openai))

## 最准确的一句话

> **恶意的本地 MCP Server 和普通恶意软件在系统权限层面没有本质区别。**

MCP 只是它与 AI 客户端通信的协议，并不会自动把运行程序变成安全程序。

因此：

- **可信代码 + 最小权限 + 沙箱 + 网络限制**：风险可以控制；
- **来源不明 + 本地直接运行 + 继承全部权限 + 可任意联网**：完全可能静默窃取隐私；
- **只有调用确认、没有系统级隔离**：仍然不够安全。