# Control PyMOL via AI CLI (Claude Code or Kiro)

PyMOL MCP connects AI coding assistants — [Claude Code](https://claude.ai/code) or [Kiro](https://kiro.dev) — to [PyMOL](https://pymol.org) via the Model Context Protocol, letting you control molecular visualizations with natural language. Describe what you want — fetch a structure, highlight active-site residues, color by secondary structure, measure distances — and the AI translates it into PyMOL commands in real time. It works locally on Mac/Windows or remotely on an HPC cluster through a simple SSH reverse tunnel, with no plugins or GUI interaction required.

<p align="center">
  <img src="pymol_claude_code.gif" alt="PyMOL MCP Demo"/>
</p>

## How It Works

The AI picks from three MCP tools (`run_command`, `run_python`, `pymol_get`) to talk to PyMOL over XML-RPC.

![Information Flow](flow.svg)

## Prerequisites

- [PyMOL](https://pymol.org)
- [Claude Code](https://claude.ai/code) **or** [Kiro](https://kiro.dev)
- [pixi](https://pixi.sh) (Python package manager — used for all platforms)

---

## Setup with Kiro

### Mac (Local)

**1. Clone the repo**
```bash
git clone https://github.com/nagarh/pymol-claude-code
cd pymol-claude-code
```

**2. Install dependencies with pixi**
```bash
pixi init
pixi add python mcp
```

**3. Configure the MCP server**

Create `.kiro/settings/mcp.json` in the repo:
```json
{
  "mcpServers": {
    "pymol": {
      "command": "/path/to/pymol-claude-code/.pixi/envs/default/bin/python",
      "args": ["/path/to/pymol-claude-code/pymol_mcp_server.py"],
      "disabled": false,
      "autoApprove": ["run_command", "run_python", "pymol_get"]
    }
  }
}
```

Replace `/path/to/pymol-claude-code` with the actual path (use `pwd` to find it). The pixi Python path is always `<project-dir>/.pixi/envs/default/bin/python`.

**4. Start PyMOL with XML-RPC**
```bash
/Applications/PyMOL.app/Contents/bin/pymol -R
```

**5. Open Kiro in the repo folder**

The MCP server connects automatically. Verify it shows as connected in the MCP Server view in the Kiro feature panel.

---

### Linux (Local)

**1. Clone the repo**
```bash
git clone https://github.com/nagarh/pymol-claude-code
cd pymol-claude-code
```

**2. Install dependencies with pixi**
```bash
pixi init
pixi add python mcp
```

**3. Configure the MCP server**

Create `.kiro/settings/mcp.json`:
```json
{
  "mcpServers": {
    "pymol": {
      "command": "/path/to/pymol-claude-code/.pixi/envs/default/bin/python",
      "args": ["/path/to/pymol-claude-code/pymol_mcp_server.py"],
      "disabled": false,
      "autoApprove": ["run_command", "run_python", "pymol_get"]
    }
  }
}
```

**4. Start PyMOL with XML-RPC**
```bash
pymol -R
```

**5. Open Kiro in the repo folder**

---

### HPC (Remote via SSH Tunnel)

PyMOL runs on your local Mac/Linux machine; Kiro runs on the HPC. An SSH reverse tunnel bridges them.

**1. Clone the repo on HPC**
```bash
git clone https://github.com/nagarh/pymol-claude-code
cd pymol-claude-code
```

**2. Install dependencies with pixi on HPC**
```bash
pixi init
pixi add python mcp
```

**3. Choose a free port on HPC**
```bash
PORT=49123
ss -tlnp | grep $PORT || echo "port is FREE"
```

**4. Update the port in `pymol_mcp_server.py`**
```python
pymol = xmlrpc.client.ServerProxy('http://localhost:<PORT>')
```

**5. Configure the MCP server on HPC**

Create `.kiro/settings/mcp.json`:
```json
{
  "mcpServers": {
    "pymol": {
      "command": "/path/to/pymol-claude-code/.pixi/envs/default/bin/python",
      "args": ["/path/to/pymol-claude-code/pymol_mcp_server.py"],
      "disabled": false,
      "autoApprove": ["run_command", "run_python", "pymol_get"]
    }
  }
}
```

**6. On your local machine — start PyMOL and the SSH reverse tunnel**
```bash
pymol -R
ssh -R <PORT>:localhost:9123 <user>@<hpc-address>
```

**7. Open Kiro on the HPC**

---

### Windows (Local)

**1. Clone the repo**
```cmd
git clone https://github.com/nagarh/pymol-claude-code
cd pymol-claude-code
```

**2. Install dependencies with pixi**
```cmd
pixi init
pixi add python mcp
```

**3. Fix the error log path in `pymol_mcp_server.py`**

The default path `/tmp/pymol_error.txt` is Unix-only. Update the `run_python` tool to use a cross-platform temp directory:
```python
import tempfile, os
error_file = os.path.join(tempfile.gettempdir(), 'pymol_error.txt')
```

**4. Configure the MCP server**

Create `.kiro\settings\mcp.json`:
```json
{
  "mcpServers": {
    "pymol": {
      "command": "C:\\path\\to\\pymol-claude-code\\.pixi\\envs\\default\\Scripts\\python.exe",
      "args": ["C:\\path\\to\\pymol-claude-code\\pymol_mcp_server.py"],
      "disabled": false,
      "autoApprove": ["run_command", "run_python", "pymol_get"]
    }
  }
}
```

Note: on Windows the pixi Python is at `.pixi\envs\default\Scripts\python.exe` (not `bin/python`).

**5. Start PyMOL with XML-RPC**
```cmd
"C:\Program Files\PyMOL\PyMOL\PyMOLWin.exe" -R
```

Adjust the path to match your PyMOL installation.

**6. Open Kiro in the repo folder**

---

## Setup with Claude Code

### Mac (Local)

**1. Clone the repo**
```bash
git clone https://github.com/nagarh/pymol-claude-code
cd pymol-claude-code
```

**2. Install dependencies**
```bash
python3 -m venv venv
source venv/bin/activate
pip install mcp
```

**3. Register the MCP server**
```bash
claude mcp add pymol $(pwd)/venv/bin/python3 $(pwd)/pymol_mcp_server.py
claude mcp list
# verify: pymol_mcp_server.py - ✓ Connected
```

**4. Start PyMOL and open Claude Code**
```bash
/Applications/PyMOL.app/Contents/bin/pymol -R
claude
```

### Linux / HPC (Remote via SSH Tunnel)

**1. Clone the repo on HPC and install dependencies**
```bash
git clone https://github.com/nagarh/pymol-claude-code
cd pymol-claude-code
pip install mcp
```

**2. Choose a free port and update `pymol_mcp_server.py`**
```bash
ss -tlnp | grep 49123 || echo "port is FREE"
```
```python
pymol = xmlrpc.client.ServerProxy('http://localhost:<PORT>')
```

**3. Register the MCP server with Claude Code on HPC**
```bash
claude mcp add pymol python3 ./pymol_mcp_server.py
```

**4. On your local machine — start PyMOL and the SSH reverse tunnel**
```bash
pymol -R
ssh -R <PORT>:localhost:9123 <user>@<hpc-address>
```

**5. Open Claude Code on HPC**
```bash
claude
```

### Auto-approve PyMOL MCP Permissions (Claude Code)

By default Claude Code asks for approval on every tool call. To disable this:
```bash
echo '{"permissions":{"allow":["Bash(*)","mcp__pymol__*"]}}' > ~/.claude/settings.json
```

---

## Usage

Once connected, the AI can control PyMOL directly. Example prompts:

- `fetch 1hho and show the protein as cartoon`
- `remove water molecules and show surface`
- `select residues within 4 angstroms of the ligand`
- `color the helices salmon and the ligand yellow`

---

## Troubleshooting

**MCP server shows `failed` or `ENOENT`**
- Make sure pixi dependencies are installed (`pixi add python mcp`)
- Verify the Python path in `mcp.json` exists: `.pixi/envs/default/bin/python` (Mac/Linux) or `.pixi\envs\default\Scripts\python.exe` (Windows)
- Check that PyMOL is running with `-R` before connecting

**Connection refused**
- PyMOL must be started with `pymol -R` before the MCP server can connect
- Verify the port in `pymol_mcp_server.py` matches your tunnel port (HPC only)

**Port is stuck on HPC**
- Always press `Ctrl+C` on the SSH tunnel before closing the terminal
- Use a different port if stuck: `ss -tlnp | grep <PORT> || echo "FREE"`
- Stuck ports are released by HPC's sshd automatically after some time

---

## Author

**Name:** Hemant Nagar  
**Email:** hn533621@ohio.edu
