# SudoCode Balance for Claude Code

这是 Claude Code 版本的插件包。

它不包含独立业务逻辑，只是把同一个只读 MCP 接到 Claude Code 上。

本地测试：

```bash
claude --plugin-dir ./claude/sudocode-balance
```

如果你把 `claude/sudocode-balance/` 这个目录单独打包成 zip，也可以用 URL 加载：

```bash
claude --plugin-url <plugin-package-zip-url>
```

如果你在远程服务器上，优先看仓库根目录的 [README](../../README.md) 里的端口转发和服务器 Token 说明。
