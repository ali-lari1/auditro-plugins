# Auditro plugins

Install Auditro into Claude Code or Codex in two commands. The plugin connects your AI client to the Auditro MCP server (`https://app.auditro.co/mcp`) and adds the `auditro-icv` skill, which teaches the agent how to use Auditro's tools safely.

Auditro computes every score, figure and estimate. The AI client selects governed records and asks for approval, and Auditro keeps tenant isolation, permissions, evidence and audit history.

## Claude Code

```sh
claude plugin marketplace add ali-lari1/auditro-plugins
claude plugin install auditro@auditro
```

Inside a session you can use `/plugin marketplace add ali-lari1/auditro-plugins`, then `/plugin install auditro@auditro`. Run `/mcp` and choose **auditro** to sign in.

## Codex

```sh
codex plugin marketplace add ali-lari1/auditro-plugins
codex plugin add auditro@auditro
```

Codex opens Auditro sign-in when the plugin is installed.

## Sign-in and access

Sign-in uses OAuth. You authorize one Auditro organization and choose read-only or full access. No API key is needed. Revoke a connection at any time at `https://app.auditro.co/settings/mcp`.

## Other clients

ChatGPT, Cursor and any MCP client that supports remote Streamable HTTP with OAuth can connect directly to `https://app.auditro.co/mcp`. Setup guides: https://app.auditro.co/docs/mcp

Support: https://auditro.co/support · Privacy: https://auditro.co/privacy · Terms: https://auditro.co/terms
