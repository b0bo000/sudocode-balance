---
name: sudocode-balance
description: Use only when the user explicitly asks to check SudoCode wallet balance, package balance, remaining quota, or total balance.
---

# SudoCode Balance

Use the `sudocode-balance` MCP server only for explicit SudoCode balance requests.

Do not call the balance tool when the user is only configuring Claude Code, MCP, Codex, Hermes, login, pricing, recharge, API keys, account settings, or general programming work.

If the user says not to check balance, do not call the tool.

If the request is ambiguous, ask whether they want to query their SudoCode balance before using the tool.
