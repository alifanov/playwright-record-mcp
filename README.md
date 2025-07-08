[![MseeP.ai Security Assessment Badge](https://mseep.net/pr/korwabs-playwright-record-mcp-badge.png)](https://mseep.ai/app/korwabs-playwright-record-mcp)

# Playwright Record MCP

Playwright Record MCP is a Model Context Protocol (MCP) server that provides browser automation capabilities using [Playwright](https://playwright.dev/). This server adds video recording functionality to record browser interactions. It enables LLMs (Large Language Models) to interact with web pages through structured accessibility snapshots, without requiring screenshots or visual models.

## Key Features

- **Fast and lightweight**: Uses Playwright's accessibility tree, not pixel-based input.
- **LLM-friendly**: No vision models needed, operates purely on structured data.
- **Deterministic tool application**: Avoids ambiguity common with screenshot-based approaches.
- **Video recording**: Ability to record browser interactions as video.

## Use Cases

- Web navigation and form-filling
- Data extraction from structured content
- LLM-driven automated testing
- General-purpose browser interaction for agents
- Recording and analyzing browser interactions

## Installation

### Installation via NPM

```bash
npm install @playwright/record-mcp
```

Or

```bash
npx @playwright/record-mcp
```

### Configuration Example

#### NPX

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/record-mcp@latest"
      ]
    }
  }
}
```

### Installation in VS Code

You can install the Playwright Record MCP server using VS Code CLI:

```bash
# For VS Code
code --add-mcp '{"name":"playwright","command":"npx","args":["@playwright/record-mcp@latest"]}'
```

```bash
# For VS Code Insiders
code-insiders --add-mcp '{"name":"playwright","command":"npx","args":["@playwright/record-mcp@latest"]}'
```

After installation, the Playwright Record MCP server will be available for use with your GitHub Copilot agent in VS Code.

## CLI Options

The Playwright Record MCP server supports the following command-line options:

- `--browser <browser>`: Browser or Chrome channel to use. Possible values:
  - `chrome`, `firefox`, `webkit`, `msedge`
  - Chrome channels: `chrome-beta`, `chrome-canary`, `chrome-dev`
  - Edge channels: `msedge-beta`, `msedge-canary`, `msedge-dev`
  - Default: `chrome`
- `--caps <caps>`: Comma-separated list of capabilities to enable, possible values: tabs, pdf, history, wait, files, install. Default is all.
- `--cdp-endpoint <endpoint>`: CDP endpoint to connect to
- `--executable-path <path>`: Path to the browser executable
- `--headless`: Run browser in headless mode (headed by default)
- `--port <port>`: Port to listen on for SSE transport
- `--user-data-dir <path>`: Path to the user data directory
- `--vision`: Run server that uses screenshots (Aria snapshots are used by default)
- `--record`: Record browser interactions as video (new feature)
- `--record-path <path>`: Path to save recording files (default: ./recordings)
- `--record-format <format>`: Recording format, possible values: mp4, webm (default: mp4)

## User Data Directory

Playwright Record MCP will launch the browser with a new profile, located at:

- Windows: `%USERPROFILE%\AppData\Local\ms-playwright\mcp-chrome-profile`
- macOS: `~/Library/Caches/ms-playwright/mcp-chrome-profile`
- Linux: `~/.cache/ms-playwright/mcp-chrome-profile`

All login information will be stored in that profile; you can delete it between sessions if you'd like to clear the offline state.

## Running Headless Browser (Browser without GUI)

This mode is useful for background or batch operations.

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/record-mcp@latest",
        "--headless"
      ]
    }
  }
}
```

## Using Video Recording

To use the video recording feature, use the `--record` flag:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/record-mcp@latest",
        "--record"
      ]
    }
  }
}
```

To specify the recording file save path:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/record-mcp@latest",
        "--record",
        "--record-path", "./my-recordings"
      ]
    }
  }
}
```

To specify the recording format:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/record-mcp@latest",
        "--record",
        "--record-format", "webm"
      ]
    }
  }
}
```

## Running Headed Browser on Linux without DISPLAY

When running a headed browser on a system without a display or from worker processes of IDEs,
run the MCP server from an environment with DISPLAY and pass the `--port` flag to enable SSE transport.

```bash
npx @playwright/record-mcp@latest --port 8931
```

Then, in the MCP client config, set the `url` to the SSE endpoint:

```json
{
  "mcpServers": {
    "playwright": {
      "url": "http://localhost:8931/sse"
    }
  }
}
```

## Docker

**NOTE:** The Docker implementation currently only supports headless Chromium.

```json
{
  "mcpServers": {
    "playwright": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "--init", "mcp/playwright-record"]
    }
  }
}
```

To build with Docker:

```bash
docker build -t mcp/playwright-record .
```

## Tool Modes

The tools are available in two modes:

1. **Snapshot Mode** (default): Uses accessibility snapshots for better performance and reliability
2. **Vision Mode**: Uses screenshots for visual-based interactions

To use Vision Mode, add the `--vision` flag when starting the server:

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": [
        "@playwright/record-mcp@latest",
        "--vision"
      ]
    }
  }
}
```

Vision Mode works best with computer use models that are able to interact with elements using X-Y coordinate space, based on the provided screenshot.

## Programmatic Usage with Custom Transports

```javascript
import http from 'http';

import { createServer } from '@playwright/record-mcp';
import { SSEServerTransport } from '@modelcontextprotocol/sdk/server/sse.js';

http.createServer(async (req, res) => {
  // ...

  // Creates a headless Playwright Record MCP server with SSE transport
  const mcpServer = await createServer({ headless: true, record: true });
  const transport = new SSEServerTransport('/messages', res);
  await mcpServer.connect(transport);

  // ...
});
```

## Snapshot-based Interactions

- **browser_snapshot**
  - Description: Capture accessibility snapshot of the current page, this is better than screenshot
  - Parameters: None

- **browser_click**
  - Description: Perform click on a web page
  - Parameters:
    - `element` (string): Human-readable element description used to obtain permission to interact with the element
    - `ref` (string): Exact target element reference from the page snapshot

- **browser_drag**
  - Description: Perform drag and drop between two elements
  - Parameters:
    - `startElement` (string): Human-readable source element description used to obtain the permission to interact with the element
    - `startRef` (string): Exact source element reference from the page snapshot
    - `endElement` (string): Human-readable target element description used to obtain the permission to interact with the element
    - `endRef` (string): Exact target element reference from the page snapshot

- **browser_hover**
  - Description: Hover over element on page
  - Parameters:
    - `element` (string): Human-readable element description used to obtain permission to interact with the element
    - `ref` (string): Exact target element reference from the page snapshot

- **browser_type**
  - Description: Type text into editable element
  - Parameters:
    - `element` (string): Human-readable element description used to obtain permission to interact with the element
    - `ref` (string): Exact target element reference from the page snapshot
    - `text` (string): Text to type into the element
    - `submit` (boolean, optional): Whether to submit entered text (press Enter after)
    - `slowly` (boolean, optional): Whether to type one character at a time. Useful for triggering key handlers in the page. By default entire text is filled in at once.

## Video Recording Tools (New Feature)

- **browser_record_start**
  - Description: Start recording browser interactions
  - Parameters:
    - `path` (string, optional): Path to save the recording file
    - `format` (string, optional): Recording format (mp4 or webm)

- **browser_record_stop**
  - Description: Stop and save browser interaction recording
  - Parameters: None

- **browser_record_pause**
  - Description: Pause the current recording
  - Parameters: None

- **browser_record_resume**
  - Description: Resume a paused recording
  - Parameters: None

- **browser_record_list**
  - Description: Return a list of current recording files
  - Parameters: None

## Examples

### Starting and Stopping Video Recording

```javascript
// Start video recording
await mcpServer.invoke('browser_record_start', {
  path: './my-recordings/test-recording.mp4',
  format: 'mp4'
});

// Perform browser navigation
await mcpServer.invoke('browser_navigate', {
  url: 'https://example.com'
});

// Interact with the page
const snapshot = await mcpServer.invoke('browser_snapshot');
// Find elements in the snapshot...

// Stop video recording
await mcpServer.invoke('browser_record_stop');
```

## Supported Browsers

- Chrome
- Firefox
- WebKit
- Microsoft Edge

## Requirements

- Node.js 18 or higher
- The required browser must be installed (or use the `browser_install` tool to install it)

## License

Apache-2.0 license
