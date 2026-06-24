# SudoCode Balance

![SudoCode Logo](plugins/sudocode-balance/assets/logo.svg)

按你的明确请求查询 SudoCode 账户余额。这个仓库提供 Codex 插件，并同时给
Claude Code、Hermes 和其他支持远程 HTTP MCP + OAuth 的客户端提供接入说明。

余额 MCP 连接到：

```text
https://api.sudocode.chat/mcp
```

它只返回账户级钱包余额、套餐余额和总余额。它不会创建 API Key，不会扣费，不会充值，也不会管理账户。

## 触发规则

只在你明确询问 SudoCode 余额、剩余额度、钱包余额、套餐余额或总余额时使用。

不要在这些场景里调用余额工具：

- 你说“不要查余额”“不用查余额”“先别查余额”
- 你只是配置 Codex、Claude Code、Hermes 或 MCP
- 你只是询问登录、充值、价格、套餐、API Key 或账户管理
- 你只是进行普通编程、代码分析或文档写作

如果意图不明确，客户端应该先问你是否需要查询余额。

## 支持方式

| 客户端 | 支持方式 |
| --- | --- |
| Codex | 安装这个 Codex plugin，然后登录 MCP |
| Claude Code | 直接添加远程 HTTP MCP server |
| Hermes | 添加远程 OAuth MCP server 或写入 mcpServers JSON |
| 其他 MCP 客户端 | 使用 mcpServers JSON，前提是客户端支持远程 HTTP MCP 和 OAuth |

完整网页文档：

```text
https://sudocode.chat/docs/mcp-balance
```

## Codex 安装

先添加 SudoCode 插件源：

```bash
codex plugin marketplace add b0bo000/sudocode-balance --ref main
```

再安装余额插件：

```bash
codex plugin add sudocode-balance@sudocode
```

安装后登录 MCP：

```bash
codex mcp login sudocode-balance
```

Codex 会打开浏览器，进入 `https://api.sudocode.chat`。用你的 SudoCode 账号登录并确认授权。

登录只是授权，不会触发余额查询。登录成功后，新开一个 Codex 会话，然后问：

```text
查询我的 SudoCode 余额
```

### Windows 找不到 codex 命令

如果 PowerShell 提示 `codex` 不是命令，可以用下面的方式找到 Codex CLI：

```powershell
$codex = Get-ChildItem "$env:LOCALAPPDATA\OpenAI\Codex" -Recurse -Filter codex.exe |
  Sort-Object LastWriteTime -Descending |
  Select-Object -First 1 -ExpandProperty FullName

& $codex mcp login sudocode-balance
```

登录后重新打开 Codex。

## 远程服务器和 VS Code Remote

如果 Codex、Claude Code 或 Hermes 运行在 SSH 服务器上，OAuth 回调通常是：

```text
http://127.0.0.1:<端口>/callback/...
```

这表示“服务器上的客户端”正在监听服务器自己的 `127.0.0.1`。如果你把授权 URL 复制到另一台电脑直接打开，浏览器会回调到那台电脑自己的 `127.0.0.1`，服务器上的客户端收不到 token。

可用方式有两种。

### 方式一：端口转发后继续 OAuth

先运行 MCP 登录命令。Codex 会打印授权 URL，并提示浏览器打开失败时手动复制 URL：

```bash
codex mcp login sudocode-balance
```

从 URL 的 `redirect_uri` 里找到端口，例如：

```text
redirect_uri=http://127.0.0.1:33497/callback/...
```

在本地电脑另开一个终端，转发同一个端口到服务器：

```bash
ssh -N -L 33497:127.0.0.1:33497 user@server
```

保持 MCP 登录命令和 SSH 转发都运行中，再用本地浏览器打开授权 URL。

VS Code Remote 用户也可以用 Ports 面板转发同一个端口。

Claude Code 可以固定 callback 端口，服务器场景更方便：

```bash
# 本地电脑
ssh -N -L 33497:127.0.0.1:33497 user@server

# 服务器
claude mcp add --scope user --transport http --callback-port 33497 sudocode-balance https://api.sudocode.chat/mcp
```

### 方式二：生成服务器 Token

进入 SudoCode 个人设置，在 MCP 授权设备卡片里生成“服务器 Token”。Token 只显示一次，服务端只保存 token hash。

Codex 可以用环境变量读取 bearer token：

```bash
export SUDOCODE_MCP_TOKEN='复制个人设置里生成的 token'
codex mcp add sudocode-balance \
  --url https://api.sudocode.chat/mcp \
  --oauth-resource https://api.sudocode.chat/mcp \
  --bearer-token-env-var SUDOCODE_MCP_TOKEN
```

Claude Code 可以把 token 写成请求头：

```bash
claude mcp add --scope user --transport http sudocode-balance https://api.sudocode.chat/mcp \
  --header "Authorization: Bearer 复制个人设置里生成的 token"
```

不用时在 SudoCode 个人设置里撤销对应 MCP 授权。

## Claude Code 接入

Claude Code 不安装 Codex 插件，直接把 SudoCode 余额 MCP 添加为远程 HTTP server：

```bash
claude mcp add --scope user --transport http sudocode-balance https://api.sudocode.chat/mcp
```

首次使用时按 Claude Code 的提示完成浏览器 OAuth 授权。授权完成后，在 Claude Code 里明确询问：

```text
查询我的 SudoCode 余额
```

## Hermes 接入

Hermes 如果提供 MCP server UI，选择远程 HTTP/OAuth MCP server，并填写：

```text
Name: sudocode-balance
URL: https://api.sudocode.chat/mcp
OAuth Resource: https://api.sudocode.chat/mcp
```

如果 Hermes 使用 JSON 配置，可以写入：

```json
{
  "mcpServers": {
    "sudocode-balance": {
      "type": "http",
      "url": "https://api.sudocode.chat/mcp",
      "oauth_resource": "https://api.sudocode.chat/mcp"
    }
  }
}
```

登录后 Hermes 会持久化 OAuth token。后续只有在你明确要求查询余额时才应该调用只读余额工具。

## 通用 MCP JSON

其他支持远程 HTTP MCP 和 OAuth 的客户端也可以使用同一段配置：

```json
{
  "mcpServers": {
    "sudocode-balance": {
      "type": "http",
      "url": "https://api.sudocode.chat/mcp",
      "oauth_resource": "https://api.sudocode.chat/mcp"
    }
  }
}
```

如果某个客户端不支持远程 HTTP MCP 或 OAuth，它不能直接使用这个余额 MCP。

## 常见提示

如果 Codex 看到：

```text
The sudocode-balance MCP server is not logged in.
Run `codex mcp login sudocode-balance`.
```

说明插件已经安装，但还没有完成授权。执行：

```bash
codex mcp login sudocode-balance
```

然后新开一个 Codex 会话即可。

如果看到：

```text
MCP startup incomplete (failed: sudocode-balance)
```

通常也是没有登录，或者登录后当前会话还没重新加载插件。先登录，再新开会话。

## 登录一次后多久有效

正常情况下只需要登录一次。

SudoCode 会签发一个永久但可撤销的 bearer token。之后，只有当你明确要求查询 SudoCode 余额时，客户端才会在该请求中使用这个 token 调用只读余额工具；不会后台定时查询，也不会因普通 SudoCode 或编程请求自动查余额。

以下情况需要重新登录：

- 你主动撤销了插件授权
- 本地客户端配置或认证缓存被清理
- 换了一台电脑或换了一个系统用户
- 服务端管理员撤销了授权

## 隐私和权限

余额 MCP 只请求一个权限：

```text
sudocode.balance
```

这个权限只允许读取账户级余额信息。

服务端只保存 token hash，不保存明文 token。

## 更新 Codex 插件

```bash
codex plugin marketplace upgrade sudocode
codex plugin add sudocode-balance@sudocode
```

更新后建议新开一个 Codex 会话。

## 卸载和撤销授权

卸载 Codex 插件：

```bash
codex plugin remove sudocode-balance
```

卸载本地插件不会自动撤销服务端授权。如果需要撤销授权，请进入 SudoCode MCP 授权管理页面：

```text
https://api.sudocode.chat/mcp/oauth/authorizations
```

## 仓库结构

```text
.agents/plugins/marketplace.json
plugins/sudocode-balance/.codex-plugin/plugin.json
plugins/sudocode-balance/.mcp.json
plugins/sudocode-balance/assets/logo.svg
```
