# SudoCode Balance

![SudoCode Logo](plugins/sudocode-balance/assets/logo.svg)

这个仓库把同一个只读余额能力拆成三个入口，但共用同一个后端：

- Codex plugin
- Claude Code plugin
- Hermes MCP 配置样本

共享的服务地址是：

```text
https://api.sudocode.chat/mcp
```

它只读账户余额、套餐余额和总余额，不会创建 Key、扣费、充值或做账户管理。

在线文档：

```text
https://sudocode.chat/docs/mcp-balance
```

## 仓库结构

```text
README.md
plugins/sudocode-balance/    Codex 插件包
claude/sudocode-balance/     Claude Code 插件包
hermes/                      Hermes 配置样本
```

## 什么时候用哪一个

- 你在 Codex 里用，就装 Codex plugin。
- 你在 Claude Code 里用，就装 Claude plugin。
- 你在 Hermes 里用，就直接加 MCP。
- 你只想接通一个 MCP 客户端，就看 `hermes/` 里的样本配置。

## Codex

Codex 这边建议直接装插件，因为它把触发规则、说明和 MCP 配置一起打包好了。你也可以只接 MCP，但默认发行方式是插件。

```bash
codex plugin marketplace add b0bo000/sudocode-balance --ref main
codex plugin add sudocode-balance@sudocode
codex mcp login sudocode-balance
```

登录后，新开一个 Codex 会话，再明确说：

```text
查询我的 SudoCode 余额
```

如果你在远程服务器、VS Code Remote 或没有浏览器跳转的环境里，优先用下面两个办法之一：

1. SSH 端口转发后继续 OAuth。
2. 用 SudoCode 个人设置里的服务器 Token。

## Claude Code

Claude Code 不读取 Codex plugin。这个仓库给 Claude Code 单独提供了：

- 根目录 `.claude-plugin/marketplace.json`：让 Claude Code 把这个仓库当 marketplace。
- `claude/sudocode-balance/.claude-plugin/plugin.json`：Claude plugin 清单。
- `claude/sudocode-balance/.mcp.json`：Claude plugin 内置的远程 MCP 配置。

### 推荐方式：从这个仓库安装 Claude plugin

在终端进入 Claude Code：

```bash
claude
```

然后在 Claude Code 里执行 slash commands：

```text
/plugin marketplace add b0bo000/sudocode-balance
/plugin install sudocode-balance@sudocode
/reload-plugins
/mcp
```

在 `/mcp` 面板里找到 `sudocode-balance`，按提示完成 OAuth 授权。授权完成后，新开或 reload 一个 Claude Code 会话，再明确询问：

```text
查询我的 SudoCode 余额
```

### 本地开发测试

如果你已经 clone 了这个仓库，可以临时加载插件目录：

```bash
git clone https://github.com/b0bo000/sudocode-balance.git
cd sudocode-balance
claude --plugin-dir ./claude/sudocode-balance
```

这只对当前 Claude Code 会话生效，适合测试插件包本身。

### 不装插件，直接配置 MCP

如果你只想让 Claude Code 接入余额 MCP，不需要 Claude plugin，可以直接加远程 HTTP MCP：

```bash
claude mcp add --scope user --transport http sudocode-balance https://api.sudocode.chat/mcp
```

确认配置已写入：

```bash
claude mcp list
claude mcp get sudocode-balance
```

然后进入 Claude Code：

```bash
claude
```

在 Claude Code 里执行：

```text
/mcp
```

再在 MCP 面板里完成 OAuth 授权。

### 服务器或无浏览器环境

如果 Claude Code 跑在 SSH 服务器、VS Code Remote、容器或 CI-like shell 里，浏览器 OAuth 要保证回调能回到服务器上的 Claude Code。

先在服务器上用固定 callback 端口添加 MCP：

```bash
claude mcp add --scope user --transport http --callback-port 33497 sudocode-balance https://api.sudocode.chat/mcp
```

然后在本地电脑开端口转发：

```bash
ssh -N -L 33497:127.0.0.1:33497 user@server
```

保持转发窗口打开，在服务器上启动 Claude Code：

```bash
claude
```

进入 `/mcp` 面板授权。这样浏览器里的 `127.0.0.1:33497` 会被转发回服务器上的 Claude Code。

如果你不想走 OAuth，也可以在 SudoCode 个人设置里生成服务器 Token，然后把 token 作为请求头配置给 Claude Code：

```bash
export SUDOCODE_MCP_TOKEN='复制个人设置里生成的服务器 Token'

claude mcp add --scope user --transport http sudocode-balance https://api.sudocode.chat/mcp \
  --header "Authorization: Bearer ${SUDOCODE_MCP_TOKEN}"
```

这种方式不需要浏览器跳转。不用时去 SudoCode 个人设置里撤销这个服务器 Token。

## Hermes

Hermes 不读取 Codex plugin。它直接接 MCP。

推荐命令：

```bash
hermes mcp add sudocode-balance --url https://api.sudocode.chat/mcp --auth oauth
```

如果你想手动看样本配置，打开 `hermes/config.sample.yaml`。

## 远程服务器场景

如果客户端运行在 SSH 服务器上，OAuth 回调通常会落到服务器自己的 `127.0.0.1`。这时把授权 URL 复制到另一台电脑直接打开，往往只会让那台电脑自己完成回调，服务器上的客户端收不到 token。

可用两条路。具体命令看上面各客户端章节：

### 方式一：端口转发

先让客户端使用固定 callback 端口，再把这个端口从本地转到服务器。

```bash
ssh -N -L 33497:127.0.0.1:33497 user@server
```

然后在客户端的 MCP 授权入口里继续 OAuth。

### 方式二：服务器 Token

在 SudoCode 个人设置里生成服务器 Token。Token 只显示一次，服务端只保存 hash。把它作为 bearer token 传给客户端。

这条路适合：

- 服务器上的 Codex CLI
- 服务器上的 Claude Code
- 服务器上的 Hermes
- 不能打开浏览器的环境

## 触发规则

这个能力只应在你明确想查余额时触发。

不要在这些场景里调用余额工具：

- 你说“不要查余额”“不用查余额”“先别查余额”
- 你只是在配置 Codex、Claude Code、Hermes 或 MCP
- 你只是在问登录、充值、价格、套餐、API Key 或账户管理
- 你只是在做普通编程、代码分析或文档写作

如果意图不清楚，先问一句，不要直接查。

## 详细入口

- [Codex 插件包](plugins/sudocode-balance/README.md)
- [Claude Code 插件包](claude/sudocode-balance/README.md)
- [Hermes 配置样本](hermes/README.md)
