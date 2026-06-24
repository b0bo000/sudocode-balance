# SudoCode Balance

![SudoCode Logo](plugins/sudocode-balance/assets/logo.svg)

按你的明确请求，在 Codex、Claude Code 或 Hermes 中查询你的 SudoCode 账户余额。这个仓库包含
Codex 插件和通用 MCP 客户端配置示例，统一连接同一个只读 MCP 服务：
`https://api.sudocode.chat/mcp`，返回账户级钱包余额、套餐余额和总余额。

插件不会创建 API Key，不会扣费，不会充值，也不会管理账户。它不会后台定时查询，也不应该因为普通 SudoCode 或编程请求自动查询余额。

## 客户端支持

- Codex：安装这个 Codex 插件，插件内置 MCP 配置。
- Claude Code：不安装 Codex 插件，直接把 `https://api.sudocode.chat/mcp` 添加为 HTTP MCP server。
- Hermes：不安装 Codex 插件，直接把同一个 MCP server 添加到 `~/.hermes/config.yaml` 或用 `hermes mcp add`。
- 其他 MCP 客户端：使用 Streamable HTTP MCP endpoint `https://api.sudocode.chat/mcp`，并按客户端的 OAuth 流程授权。

如果客户端要求显式填写 OAuth resource，使用：

```text
https://api.sudocode.chat/mcp
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

## Codex 登录

安装后先登录一次 MCP：

```bash
codex mcp login sudocode-balance
```

Codex 会打开浏览器，进入 `https://api.sudocode.chat`。用你的 SudoCode 账号密码登录并确认授权。

登录只是授权，不会触发余额查询。只有当你明确询问 SudoCode 余额、剩余额度、钱包余额、套餐余额或总余额时，Codex 才会调用只读余额工具。

登录成功后，新开一个 Codex 会话，然后问：

```text
查询我的 SudoCode 余额
```

## Claude Code 使用

Claude Code 不读取 `.codex-plugin/plugin.json`，直接添加远程 HTTP MCP server：

```bash
claude mcp add --scope user --transport http sudocode-balance https://api.sudocode.chat/mcp
```

然后打开 Claude Code，问：

```text
查询我的 SudoCode 余额
```

首次使用时 Claude Code 会按 MCP OAuth 流程要求你授权 SudoCode。授权完成后，新开一个 Claude Code 会话再试。

如果你想用项目级 `.mcp.json`，可以参考：

```text
clients/claude-code.mcp.json
```

## Hermes 使用

Hermes 可以用命令添加：

```bash
hermes mcp add sudocode-balance --url https://api.sudocode.chat/mcp --auth oauth
hermes mcp login sudocode-balance
```

也可以把示例配置合并到 `~/.hermes/config.yaml`：

```text
clients/hermes.config.yaml
```

示例里只允许加载 `sudocode_balance` 这个只读工具，并关闭 MCP resources/prompts。

## Windows 找不到 codex 命令

如果 PowerShell 提示 `codex` 不是命令，可以用下面的方式找到 Codex CLI：

```powershell
$codex = Get-ChildItem "$env:LOCALAPPDATA\OpenAI\Codex" -Recurse -Filter codex.exe |
  Sort-Object LastWriteTime -Descending |
  Select-Object -First 1 -ExpandProperty FullName

& $codex mcp login sudocode-balance
```

登录后重新打开 Codex。

## 常见提示

如果看到：

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

Claude Code 或 Hermes 如果连接后没有出现余额工具，先确认 MCP server 已添加，再按客户端提示完成 OAuth 授权，最后重开会话或重载 MCP。

## 登录一次后多久有效

正常情况下只需要登录一次。

SudoCode 会签发一个永久但可撤销的 bearer token。之后，只有当你明确要求查询 SudoCode 余额时，已授权的 MCP 客户端才会在该请求中使用这个 token 调用只读余额工具；不会后台定时查询，也不会因普通 SudoCode 或编程请求自动查余额。

以下情况需要重新登录：

- 你主动撤销了插件授权
- 本地 Codex 配置或认证缓存被清理
- 换了一台电脑或换了一个系统用户
- 服务端管理员撤销了授权

## 隐私和权限

MCP 授权只请求一个权限：

```text
sudocode.balance
```

这个权限只允许读取账户级余额信息。

服务端只保存 token hash，不保存明文 token。

## 更新插件

```bash
codex plugin marketplace upgrade sudocode
codex plugin add sudocode-balance@sudocode
```

更新后建议新开一个 Codex 会话。

## 卸载插件

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
clients/claude-code.mcp.json
clients/hermes.config.yaml
plugins/sudocode-balance/.codex-plugin/plugin.json
plugins/sudocode-balance/.mcp.json
plugins/sudocode-balance/assets/logo.svg
```
