# SudoCode Balance

![SudoCode Logo](plugins/sudocode-balance/assets/logo.svg)

在 Codex 中查询你的 SudoCode 账户余额。插件通过只读 MCP 工具连接
`https://api.sudocode.chat/mcp`，返回账户级钱包余额、套餐余额和总余额。

插件不会创建 API Key，不会扣费，不会充值，也不会管理账户。

## 安装

先添加 SudoCode 插件源：

```bash
codex plugin marketplace add b0bo000/sudocode-balance --ref main
```

再安装余额插件：

```bash
codex plugin add sudocode-balance@sudocode
```

## 登录

安装后先登录一次 MCP：

```bash
codex mcp login sudocode-balance
```

Codex 会打开浏览器，进入 `https://api.sudocode.chat`。用你的 SudoCode 账号密码登录并确认授权。

登录成功后，新开一个 Codex 会话，然后问：

```text
查询我的 SudoCode 余额
```

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

## 登录一次后多久有效

正常情况下只需要登录一次。

SudoCode 会签发一个永久但可撤销的 bearer token。之后 Codex 会自动使用这个 token 调用余额工具，不需要每次输入账号密码。

以下情况需要重新登录：

- 你主动撤销了插件授权
- 本地 Codex 配置或认证缓存被清理
- 换了一台电脑或换了一个系统用户
- 服务端管理员撤销了授权

## 隐私和权限

插件只请求一个权限：

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
plugins/sudocode-balance/.codex-plugin/plugin.json
plugins/sudocode-balance/.mcp.json
plugins/sudocode-balance/assets/logo.svg
```
