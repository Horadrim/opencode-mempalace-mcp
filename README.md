# MemPalace MCP Setup for OpenCode on Windows 11

## Complete Working Guide

This guide walks you through installing and configuring MemPalace as an MCP server for OpenCode on Windows 11.

---

## Prerequisites

- Python 3.9+ installed and accessible in PATH
- Node.js (for mcp-proxy) - Download from https://nodejs.org/
- OpenCode installed on your system
- Administrator rights (may be needed for some installations)

---

## Step 1: Install MemPalace

Open a PowerShell or Command Prompt as Administrator:

"pip install mempalace"

Verify installation:

"mempalace --version"

Expected output: "mempalace 3.0.0" or similar.

---

## Step 2: Initialize Your Memory Palace

Choose a directory for your project, navigate inside it and initialize MemPalace:

"mempalace init ."

This creates the palace structure in "~/.mempalace/".

### Mine Your Existing Data

"mempalace mine ."
"mempalace mine .\chats --mode convos"
"mempalace status"

---

## Step 3: Install mcp-proxy (HTTP to stdio bridge)

MemPalace's MCP server uses stdio transport, but OpenCode requires remote (HTTP). mcp-proxy bridges this gap.

### Install via npm:

"npm install -g @modelcontextprotocol/mcp-proxy"

Or using the Python version (alternative):

"pip install mcp-proxy"

Note: The npm version is more reliable on Windows. We'll use it in this guide.

---

## Step 4: Start the MemPalace MCP Server

Create a batch file to easily start the server.

### Create "start_mempalace.bat" in your project directory:

```
@echo off
title MemPalace MCP Server
echo ========================================
echo   MemPalace MCP Server Starting...
echo ========================================
echo.
echo Server will run on: http://localhost:6060/sse
echo.
echo Keep this window open while using OpenCode
echo.
echo Press Ctrl+C to stop the server
echo ========================================
echo.

npx mcp-proxy --port 6060 -- python -m mempalace.mcp_server

pause
```
### Run the server:

Double-click "start_mempalace.bat" or run in terminal:

`.\start_mempalace.bat`

### Expected output:
```
[I ...] Serving MCP Servers via SSE:
[I ...]   - http://127.0.0.1:6060/sse
INFO:     Uvicorn running on http://127.0.0.1:6060
```
### Verify the server is working:

Open another PowerShell window and run:

`curl http://localhost:6060/sse`

### Expected response:

"event: endpoint"
"data: /messages/?session_id=6356cb572a87447d8e76b69964d57003"

✓ If you see something similar to this, the server is running correctly!

---

## Step 5: Configure OpenCode

Add MemPalace to your OpenCode configuration.

### Locate your OpenCode config file:

Typically found at:
- "%USERPROFILE%\.config\opencode\opencode.jsonc"
- "%APPDATA%\OpenCode\config.json"
- "~\.opencode\config.json"
- Or within your project's ".opencode/" directory

### Add the MemPalace server configuration:

{
  "mcpServers": {
    "mempalace": {
      "type": "remote",
      "url": "http://localhost:6060/sse",
      "enabled": true
    }
  }
}

If your OpenCode version uses a different structure, try:

{
  "mcp_servers": {
    "mempalace": {
      "transport": "sse",
      "url": "http://localhost:6060/sse",
      "enabled": true
    }
  }
}

Or for older configurations:

{
  "servers": {
    "mempalace": {
      "type": "remote",
      "url": "http://localhost:6060/sse"
    }
  }
}

### Restart OpenCode

After saving the configuration, fully restart OpenCode to load the MCP server.

---

## Step 6: Test the Integration

Once OpenCode restarts, test if MemPalace is working:

### Ask OpenCode:

"Can you list all the tools available in my MemPalace MCP server?"

### Expected response: OpenCode should list 19 MemPalace tools including:
- mempalace_search
- mempalace_status
- mempalace_list_wings
- mempalace_kg_query
- mempalace_add_drawer
- etc.

### Test a simple search:

"What's the current status of my MemPalace palace?"

Or:

"Search my memory palace for anything related to 'filter'"

---

## Step 7: Using MemPalace Day-to-Day

With the server running and OpenCode configured, MemPalace works automatically.

### Examples of what you can ask OpenCode:

| Question | What MemPalace Does |
|----------|---------------------|
| "What did we decide about the database schema?" | Searches past conversations |
| "Show me the discussion about API rate limiting" | Retrieves verbatim chat history |
| "What issues did we encounter with Python 3.14?" | Finds relevant debugging sessions |
| "List all decisions made last week" | Queries knowledge graph with temporal filters |

### Storing New Conversations:

MemPalace automatically mines conversations as you work. You can also manually mine:

"mempalace mine . --mode convos"
"mempalace mine . --mode convos --extract general"

---

## Running the Server Automatically

### Option A: Start Minimized (Recommended)

1. Right-click "start_mempalace.bat"
2. Select Create Shortcut
3. Right-click the shortcut → Properties
4. Set Run: to Minimized
5. Place shortcut in your Startup folder ("shell:startup")

### Option B: Windows Service (Advanced)

Use NSSM (Non-Sucking Service Manager) to run as a background service:

"nssm install MemPalaceMCP"
"nssm set MemPalaceMCP application "C:\Program Files\nodejs\npx.cmd""
"nssm set MemPalaceMCP arguments "mcp-proxy --port 6060 -- python -m mempalace.mcp_server""
"nssm start MemPalaceMCP"

---

## Troubleshooting

### Server won't start (port already in use)

"netstat -ano | findstr :6060"
"taskkill /PID 12345 /F"

### OpenCode shows "Connection refused"

- Verify the server is running ("curl http://localhost:6060/sse")
- Check firewall isn't blocking port 6060
- Ensure OpenCode config has the correct URL

### MemPalace commands not found

"pip install --upgrade mempalace"
"python -m mempalace --help"

### SSE error in OpenCode

Try alternative endpoint in config:

"url": "http://localhost:6060"

Or:

"url": "http://127.0.0.1:6060/sse"

---

## Quick Reference Card

| Component | Command/Location |
|-----------|------------------|
| Start Server | ".\start_mempalace.bat" |
| Server URL | "http://localhost:6060/sse" |
| Test Server | "curl http://localhost:6060/sse" |
| Initialize Palace | "mempalace init ." |
| Mine Data | "mempalace mine ." |
| Search | "mempalace search "query"" |
| Status | "mempalace status" |
| Config File | OpenCode config.json |
| Palace Location | "~/.mempalace/" |

---

## Summary

You now have:

✓ MemPalace installed locally
✓ Memory palace initialized for your project
✓ mcp-proxy bridging stdio to HTTP
✓ Server running on port 6060
✓ OpenCode configured to connect
✓ Working MCP integration

Your AI coding assistant now has persistent, searchable memory across all your conversations and code sessions—completely local and free.

---

## Getting Help

- MemPalace GitHub: https://github.com/milla-jovovich/mempalace
- OpenCode Documentation: Check your OpenCode installation docs
- mcp-proxy Issues: https://github.com/modelcontextprotocol/mcp-proxy

---

Last updated: April 8, 2026
Tested on Windows 11 with Python 3.14, Node.js 22+, and OpenCode
