# workswith.ai for Claude

A Claude Code plugin that connects Claude to your organization on workswith.ai: its Types, Records and
Processes, and the Actions you may invoke. It adds one MCP server, `https://<org>.workswith.ai/mcp`.

## Install

```
/plugin marketplace add workswith-ai/claude-plugin
/plugin install workswith-ai@workswith-ai
```

Claude Code asks for your **organization key** when the plugin is enabled: the first part of your org's
address, `acme` for `https://acme.workswith.ai`. From a shell, give it at install:

```
claude plugin install workswith-ai@workswith-ai --config org=acme
```

Then run `/mcp`, choose **plugin:workswith-ai:ww** and sign in. Your browser opens workswith.ai, where you sign in as
usual and choose what Claude may do: *read*, *act*, and *build*. You need the *Connect apps* permission
in the org. Revoke the connection at any time under Desk › Connected apps.

To change the org later: `/plugin` › Installed › workswith.ai › Configure options, then restart Claude Code.

## How sign-in works

The plugin names only the MCP address. Claude Code reads that host's protected-resource metadata, which
names workswith.ai's authorization server, registers itself there and signs in with PKCE. No client id or
secret is configured, and the token is for your org alone.

## Privacy

[Privacy policy](https://workswith.ai/privacy). MIT licensed; see `LICENSE`.

## Developing

```
claude plugin validate --strict .
claude --plugin-dir .
```
