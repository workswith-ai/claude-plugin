# workswith.ai for Claude

A Claude Code plugin that connects Claude to your organization on workswith.ai: its Types, Records and
Processes, and the Actions you may invoke. It adds one MCP server, `https://auth.workswith.ai/mcp`.

## Install

```
/plugin marketplace add workswith-ai/claude-plugin
/plugin install workswith-ai@workswith-ai
```

Then run `/mcp`, choose **plugin:workswith-ai:workswith-ai** and sign in. Your browser opens workswith.ai, where you sign in as
usual, check the organization it names (**Use another org** picks a different one), and choose what Claude may do: *Read*, *Act*, *Build* and *Agents*. Tick **Act** and **Agents**
together: that lets Claude hand work to workswith.ai's system agent (apps, Ontology changes, connections,
automations), which the plugin's skill relies on. You need the *Connect apps* permission
in the org. Revoke the connection at any time under Desk › Connected apps.

To connect another organization later, revoke the connection and sign in again from `/mcp`.

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
