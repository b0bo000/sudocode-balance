# SudoCode Balance for Claude Code

这是 Claude Code 版本的插件包。

它不包含独立业务逻辑，只是把同一个只读 MCP 接到 Claude Code 上：

```text
https://api.sudocode.chat/mcp
```

## 从 GitHub marketplace 安装

这个仓库根目录有 `.claude-plugin/marketplace.json`。Claude Code 可以把整个仓库当 marketplace，然后安装这个目录里的插件。

先启动 Claude Code：

```bash
claude
```

在 Claude Code 里执行：

```text
/plugin marketplace add b0bo000/sudocode-balance
/plugin install sudocode-balance@sudocode
/reload-plugins
/mcp
```

然后在 `/mcp` 面板里给 `sudocode-balance` 完成 OAuth 授权。

## 本地测试

如果你在这个仓库根目录，可以临时加载插件：

```bash
claude --plugin-dir ./claude/sudocode-balance
```

这个方式只对当前会话生效，不会安装到用户级 Claude 配置。

## 直接 MCP 配置

不想装插件时，用 Claude Code 自带的 MCP 命令：

```bash
claude mcp add --scope user --transport http sudocode-balance https://api.sudocode.chat/mcp
claude mcp list
claude mcp get sudocode-balance
```

然后启动 Claude Code：

```bash
claude
```

在 Claude Code 里执行：

```text
/mcp
```

再完成 OAuth 授权。

## 服务器或无浏览器环境

服务器上建议固定 OAuth callback 端口：

```bash
claude mcp add --scope user --transport http --callback-port 33497 sudocode-balance https://api.sudocode.chat/mcp
```

本地电脑开端口转发：

```bash
ssh -N -L 33497:127.0.0.1:33497 user@server
```

然后服务器上运行 `claude`，在 `/mcp` 面板里授权。

也可以直接用服务器 Token：

```bash
export SUDOCODE_MCP_TOKEN='复制个人设置里生成的服务器 Token'

claude mcp add --scope user --transport http sudocode-balance https://api.sudocode.chat/mcp \
  --header "Authorization: Bearer ${SUDOCODE_MCP_TOKEN}"
```

查询时明确说：

```text
查询我的 SudoCode 余额
```
