# Claude Code Installation Guide

This guide explains how to install and use the Infrastructure Diagram MCP Server with Claude Code.

## Prerequisites

### 1. Install GraphViz

GraphViz is required for diagram generation.

**Windows:**
```bash
winget install graphviz
```

**macOS:**
```bash
brew install graphviz
```

**Linux (Ubuntu/Debian):**
```bash
sudo apt-get install graphviz
```

After installation, verify GraphViz is in your PATH:
```bash
dot -V
```

### 2. Install Python 3.12+

Ensure Python 3.12 or later is installed:
```bash
python --version
```

## Installation Methods

### Method 1: Using uvx (Recommended)

Create or edit `.mcp.json` in your project root:

```json
{
  "mcpServers": {
    "infrastructure-diagrams": {
      "type": "stdio",
      "command": "uvx",
      "args": [
        "infrastructure-diagram-mcp-server"
      ]
    }
  }
}
```

### Method 2: Using pip + Local Package

1. Install the package:
```bash
pip install infrastructure-diagram-mcp-server
```

2. Configure `.mcp.json`:
```json
{
  "mcpServers": {
    "infrastructure-diagrams": {
      "type": "stdio",
      "command": "python",
      "args": [
        "-m",
        "infrastructure_diagram_mcp_server"
      ]
    }
  }
}
```

### Method 3: Development (Local Source)

1. Clone and install in editable mode:
```bash
git clone https://github.com/andrewmoshu/diagram-mcp-server.git
cd diagram-mcp-server
pip install -e .
```

2. Configure `.mcp.json`:
```json
{
  "mcpServers": {
    "infrastructure-diagrams": {
      "type": "stdio",
      "command": "python",
      "args": [
        "-m",
        "infrastructure_diagram_mcp_server"
      ]
    }
  }
}
```

## Windows-Specific Configuration

On Windows, GraphViz headers may need to be explicitly added to the environment.

Add these environment variables to your `.mcp.json`:

```json
{
  "mcpServers": {
    "infrastructure-diagrams": {
      "type": "stdio",
      "command": "uvx",
      "args": [
        "infrastructure-diagram-mcp-server"
      ],
      "env": {
        "INCLUDE": "C:\\Program Files\\Graphviz\\include",
        "LIB": "C:\\Program Files\\Graphviz\\lib",
        "PATH": "...existing PATH...;C:\\Program Files\\Graphviz\\bin"
      }
    }
  }
}
```

## Usage

After configuration, restart Claude Code to load the MCP server.

### Available Tools

1. **list_icons** - Discover available diagram icons by provider (aws, gcp, azure, k8s, onprem, etc.)
2. **get_diagram_examples** - Get example code for different diagram types
3. **generate_diagram** - Generate diagrams from Python code

### Example Usage in Claude Code

```
User: Create an AWS diagram showing a load balancer, EC2 instances, and RDS database

Claude: [Uses list_icons to find AWS icons]
Claude: [Uses get_diagram_examples to get AWS examples]
Claude: [Uses generate_diagram to create the diagram]
```

## Troubleshooting

### Error: `module 'signal' has no attribute 'SIGALRM'`

This occurs on Windows with older versions. Update to the latest version which uses ThreadPoolExecutor for cross-platform timeout.

### Error: `ExecutableNotFound: dot`

GraphViz is not installed or not in PATH. Install GraphViz and ensure `dot -V` works.

### Error: `'unicodeescape' codec can't decode bytes`

This is fixed in the latest version. Update your installation.

### MCP Server Not Loading

1. Verify `.mcp.json` is in the project root
2. Restart Claude Code after configuration changes
3. Check the MCP server logs for errors

## Diagram Output

Generated diagrams are saved to:
- `{workspace}/generated-diagrams/` (when workspace_dir is provided)
- System temp directory as fallback

Output formats:
- `.png` - Image file for viewing
- `.drawio` - Editable diagram for draw.io/diagrams.net
