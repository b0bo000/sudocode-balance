# SudoCode Balance

Read your SudoCode account balance from Codex through a read-only remote MCP server.

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
.codex-plugin/plugin.json
.mcp.json
```
