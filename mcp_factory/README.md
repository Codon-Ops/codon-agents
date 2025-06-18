# MCP Factory

This repository contains a framework for creating Model Context Protocol (MCP) servers that expose various tools via JSON-RPC endpoints.

## Directory Structure

```
mcp_factory/
├── mcp_factory.py          # Core factory function for creating MCP servers
├── templates/              # Template implementations for different MCP servers
│   ├── collaboration_mcp.py
│   ├── complicance_mcp.py
│   ├── enrichment_mcp.py
│   ├── graph_mcp.py
│   └── ontology_mcp.py
└── tools/                  # Tool implementations (currently empty)
```

## Core Component

### mcp_factory.py

The core module that provides the `create_mcp_server` function to generate FastAPI applications that expose MCP-compatible JSON-RPC endpoints. This factory function handles all the boilerplate code for setting up an MCP server.

## Templates

The `templates/` directory contains pre-configured MCP server implementations for different purposes:

### ontology_mcp.py

An MCP server for ontology-related operations, providing tools for:
- `fetch_ontology`: Fetch ontology data from various sources
- `rdf_to_jsonld`: Convert RDF data to JSON-LD format

### graph_mcp.py

An MCP server for interacting with Neo4j graph databases, providing tools for:
- `execute_cypher_read`: Execute read-only Cypher queries
- `execute_cypher_write`: Execute write operations with Cypher

### Other Templates

- `collaboration_mcp.py`: Collaboration-related tools
- `complicance_mcp.py`: Compliance-related tools
- `enrichment_mcp.py`: Data enrichment tools

## Usage

To create and run an MCP server, import the relevant template and start it with a ASGI server like Uvicorn:

```python
import uvicorn
from templates.ontology_mcp import app

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

## Creating New MCP Servers

To create a new MCP server:

1. Create a new file in the `templates/` directory
2. Import the necessary components:
   ```python
   from mcp_factory import create_mcp_server, ToolSpec
   from your_tools import YourToolInput, YourToolOutput, your_tool_handler
   ```
3. Define your tool specifications:
   ```python
   tool_specs = [
       ToolSpec(
           name="your_tool",
           input_schema=YourToolInput,
           output_schema=YourToolOutput,
           handler=your_tool_handler
       ),
   ]
   ```
4. Create the FastAPI app:
   ```python
   app = create_mcp_server(
       title="Your MCP Service",
       version="1.0.0",
       tool_specs=tool_specs,
       prefix="/mcp/your_prefix"
   )
   ```