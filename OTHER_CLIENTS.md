# Using MCP for Crestron with other AI apps

MCP for Crestron is a standard MCP server that runs over stdio, so any app that can
launch an MCP server can use it, with any model that handles tool calls well. Claude
Desktop has a one-click installer (the `.mcpb`). Everything else runs it from npm.

Every setup needs the same three things:

- the command `npx -y mcp-for-crestron@latest`
- `CRESTRON_HOST`, the processor's IP address or hostname
- `CRESTRON_KEY`, the processor's secure key, shown on the MCP Server Config module's
  `Key` output (setting it turns on encryption and secure authentication)

Optional extras are `CRESTRON_PORT` (default `50794`) and `CRESTRON_ROOM`, a room to
start in so that requests naming no room apply to it.

The `@latest` tag matters. `npx` caches packages, so without it an app can keep running
the copy it downloaded the first time after a new version is out.

## LibreChat

Add the server to `librechat.yaml`, and keep the key in LibreChat's `.env` file rather
than in the YAML:

```yaml
mcpServers:
  crestron:
    type: stdio
    command: npx
    args:
      - -y
      - mcp-for-crestron@latest
    env:
      CRESTRON_HOST: "10.0.1.38"
      CRESTRON_KEY: "${CRESTRON_KEY}"
```

```bash
# .env
CRESTRON_KEY=<the processor's secure key>
```

Restart LibreChat after changing either file, then pick the Crestron tools in a chat.

## Claude Code

```bash
claude mcp add crestron \
  --env CRESTRON_HOST=10.0.1.38 \
  --env CRESTRON_KEY=<the processor's secure key> \
  -- npx -y mcp-for-crestron@latest
```

## Any other MCP host

Most hosts take a JSON block like this one:

```json
{
  "mcpServers": {
    "crestron": {
      "command": "npx",
      "args": ["-y", "mcp-for-crestron@latest"],
      "env": {
        "CRESTRON_HOST": "10.0.1.38",
        "CRESTRON_KEY": "<the processor's secure key>"
      }
    }
  }
}
```

The machine running the app needs Node.js 18 or later and must be able to reach the
processor on the network.

## Choosing a model

The product is model-agnostic, but the model has to be good at tool calling. In field
use, Claude Sonnet and Qwen3.5-35B-A3B have both driven a real system flawlessly, Qwen
at a fraction of the cost. DeepSeek V4.1 Flash struggled: it repeated moves, pressed
buttons while a shade was still travelling, and at times wrote its tool calls out as
text instead of making them. If the assistant behaves erratically, try a stronger
tool-calling model before anything else.
