# SudoCode Balance

Read your SudoCode account balance from Codex through a read-only remote MCP server.

## Install

```bash
codex plugin marketplace add b0bo000/sudocode-balance --ref main
codex plugin add sudocode-balance@sudocode
```

Start a new Codex thread after installing, then ask:

```text
Check my SudoCode balance.
```

Codex will open the SudoCode OAuth authorization flow the first time the plugin is used.

## What It Does

This plugin connects Codex to:

```text
https://api.sudocode.chat/mcp
```

It exposes one MCP tool:

```text
sudocode_balance
```

The tool returns account-level wallet and subscription balance in CNY. It does not create API keys, spend balance, recharge, or manage the account.

## Authentication

The plugin uses standard MCP OAuth against:

```text
https://api.sudocode.chat
```

Users sign in on the API domain and authorize a permanent, revocable bearer token. The server stores only a hash of that token.

## Files

```text
.agents/plugins/marketplace.json
plugins/sudocode-balance/.codex-plugin/plugin.json
plugins/sudocode-balance/.mcp.json
```
