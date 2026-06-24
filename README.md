# SudoCode Balance

![SudoCode Logo](plugins/sudocode-balance/assets/logo.svg)

在 Codex 中通过只读远程 MCP 服务查询你的 SudoCode 账户余额。

## 安装

```bash
codex plugin marketplace add b0bo000/sudocode-balance --ref main
codex plugin add sudocode-balance@sudocode
```

安装后新开一个 Codex 会话，然后直接问：

```text
查询我的 SudoCode 余额。
```

第一次使用时，Codex 会打开 SudoCode OAuth 授权流程。授权页面在 `https://api.sudocode.chat`。

## 功能

插件会把 Codex 连接到：

```text
https://api.sudocode.chat/mcp
```

插件只提供一个 MCP 工具：

```text
sudocode_balance
```

这个工具返回账户级余额，单位为人民币 CNY，包括钱包余额、套餐余额和总余额。

它不会创建 API Key，不会扣费，不会充值，也不会管理账户。

## 授权

插件使用标准 MCP OAuth。用户在 `https://api.sudocode.chat` 登录并授权后，SudoCode 会签发一个永久但可撤销的 bearer token。

服务端只保存 token hash，不保存明文 token。

## 仓库结构

```text
.agents/plugins/marketplace.json
plugins/sudocode-balance/.codex-plugin/plugin.json
plugins/sudocode-balance/.mcp.json
plugins/sudocode-balance/assets/logo.svg
```
