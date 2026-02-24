# Quick Start Guide: Juniper MCP Server Setup (WSL + Docker + VSCode)

This guide walks you through setting up the Juniper MCP Server on Windows using WSL2, Docker, and VSCode.

## Prerequisites

- Windows with WSL2 (Ubuntu)
- Docker Desktop installed, configured and running for WSL2
- VSCode installed on Windows
- SSH key for accessing Junos devices

## 1. Clone and Build

```bash
cd ~/repositories
git clone https://github.com/Juniper/junos-mcp-server.git
cd junos-mcp-server
docker build -t junos-mcp-server:latest .
```

## 2. Create `devices.json`

Create a `devices.json` file in the repository with your device configurations:

```json
{
  "DEVICE-NAME": {
    "ip": "192.168.1.1",
    "port": 22,
    "username": "netops",
    "auth": {
      "type": "ssh_key",
      "private_key_path": "/app/config/ssh_key_name"
    }
  }
}
```

**⚠️ Critical:** Use `/app/config/` path (container path), not your WSL host path!

### Example with Real Paths

If your SSH key is at `/home/username/.ssh/netops`, you'll:
- Mount it: `-v /home/username/.ssh/netops:/app/config/netops`
- Reference it in `devices.json`: `"private_key_path": "/app/config/netops"`

## 3. Configure VSCode MCP

Edit `C:\Users\<your-username>\AppData\Roaming\Code\User\mcp.json`:

```json
{
  "servers": {
    "Juniper-mcp-server": {
      "command": "wsl",
      "args": [
        "-d",
        "Ubuntu",
        "docker",
        "run",
        "--rm",
        "-i",
        "--user",
        "1000:1000",
        "-v",
        "/home/username/repositories/junos-mcp-server/devices.json:/app/config/devices.json",
        "-v",
        "/home/username/.ssh/netops:/app/config/netops",
        "junos-mcp-server:latest"
      ],
      "type": "stdio"
    }
  }
}
```

**Important configuration flags:**
- `-i` only (no `-t` - TTY breaks JSON-RPC protocol)
- `--user 1000:1000` (fixes SSH key permission issues in container)
- Use absolute WSL paths for volume mounts
- `-d Ubuntu` should match your WSL distribution name (check with `wsl -l`)

## 4. Start & Use

1. **Reload VSCode**: Press `Ctrl+Shift+P` → "Developer: Reload Window"
2. **Start MCP Server**: Press `Ctrl+Shift+P` → "MCP: List Servers" → Start "Juniper-mcp-server"
3. **Use in Copilot Chat**:

### Example Commands

```
Get device facts for DEVICE-NAME
```

```
Show interfaces on DEVICE-NAME
```

```
Show the routing table for DEVICE-NAME
```

```
Show config diff against rollback 3 on DEVICE-NAME
```

```
What's the VLAN configuration on DEVICE-NAME?
```

```
Show BGP neighbors on DEVICE-NAME
```

```
Run 'show version' on all routers
```

## Common Pitfalls & Solutions

| Issue | Problem | Solution |
|-------|---------|----------|
| **JSON parsing errors** | Using `-it` flag | Use `-i` only (remove `-t`) |
| **Authentication failures** | Wrong path in `devices.json` | Use container path `/app/config/` not host path |
| **Permission denied on SSH key** | Container can't read key | Add `--user 1000:1000` flag |
| **Server not appearing** | VSCode not reloaded | Reload window after config changes |
| **WSL distribution error** | Wrong distro name | Check with `wsl -l` and update `-d` flag |

## Troubleshooting

### Test Docker Container Manually

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}' | \
docker run --rm -i \
  -v /path/to/devices.json:/app/config/devices.json \
  -v /path/to/ssh_key:/app/config/ssh_key \
  --user 1000:1000 \
  junos-mcp-server:latest
```

Should return a JSON response with server capabilities if working correctly.

### Check SSH Key Permissions

```bash
docker run --rm \
  -v /path/to/ssh_key:/app/config/ssh_key \
  --user 1000:1000 \
  junos-mcp-server:latest \
  ls -la /app/config/ssh_key
```

Should show readable permissions (e.g., `-rw-------`).

### View MCP Server Logs

In VSCode:
1. Open Output panel: `Ctrl+Shift+U`
2. Select "MCP Servers" from dropdown
3. Look for connection errors or authentication issues

## Additional Resources

- [Main README](README.md) - Full documentation
- [GitHub Repository](https://github.com/Juniper/junos-mcp-server)
- [Juniper Community Blog](https://community.juniper.net/blogs/victor-ganjian/2025/11/01/network-automation-with-ai-and-junos-mcp-server)
- [MCP Documentation](https://modelcontextprotocol.io/)

## Next Steps

Once you have the server running:
- Explore all available MCP tools with "What tools are available?"
- Try executing show commands on your devices
- Use Jinja2 templates for configuration management
- Add multiple devices to your `devices.json`

---

**Need Help?** Check the [main README](README.md) for detailed documentation or open an issue on [GitHub](https://github.com/Juniper/junos-mcp-server/issues).
