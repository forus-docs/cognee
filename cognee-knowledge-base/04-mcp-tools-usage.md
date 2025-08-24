# Cognee MCP Tools Usage Guide

## Overview

The Cognee MCP (Model Context Protocol) server enables AI assistants and IDEs to interact with Cognee's knowledge graph system through a standardized tool interface.

## Installation & Setup

### 1. Install MCP Server
```bash
cd cognee-mcp
uv sync --dev --all-extras
```

### 2. Configure Transport Mode
```bash
# Standard I/O (for Cursor/VSCode)
python src/server.py

# Server-Sent Events (for web clients)
python src/server.py --transport sse

# HTTP streaming (for production)
python src/server.py --transport http --port 8080
```

### 3. IDE Configuration

#### Cursor/VSCode
Add to `.mcp.json`:
```json
{
  "servers": {
    "cognee": {
      "command": "python",
      "args": ["/path/to/cognee-mcp/src/server.py"],
      "env": {
        "LLM_API_KEY": "your-key"
      }
    }
  }
}
```

## Available MCP Tools

### 1. cognee_add_developer_rules

**Purpose**: Ingest developer rule files into Cognee's memory layer.

**Parameters**:
- `base_path` (string): Root path for resolving files (default: ".")
- `graph_model_file` (string, optional): Custom schema file path
- `graph_model_name` (string, optional): Schema class name

**Usage Example**:
```python
await mcp.cognee_add_developer_rules(
    base_path="/project/root"
)
```

**What it does**:
1. Loads predefined developer files (.cursorrules, CLAUDE.md, etc.)
2. Assigns them to 'developer_rules' node set
3. Processes them into knowledge graph
4. Makes patterns available for retrieval

### 2. cognify

**Purpose**: Transform data into structured knowledge graphs.

**Parameters**:
- `data` (string, required): Text or file path to process
- `graph_model_file` (string, optional): Custom graph schema
- `graph_model_name` (string, optional): Schema class name

**Usage Example**:
```python
# Process text directly
await mcp.cognify(
    data="Cognee is a memory layer for AI agents..."
)

# Process a file
await mcp.cognify(
    data="/path/to/document.md"
)
```

**Process Steps**:
1. Document classification
2. Text chunking
3. Entity extraction
4. Relationship inference
5. Graph construction
6. Embedding generation
7. Summary creation

### 3. codify

**Purpose**: Analyze code repositories and build code knowledge graphs.

**Parameters**:
- `repo_path` (string, required): Path to code repository

**Usage Example**:
```python
await mcp.codify(
    repo_path="/Users/project/my-app"
)
```

**What it analyzes**:
- Function definitions and calls
- Class hierarchies
- Module dependencies
- Import relationships
- Code patterns
- Documentation strings

### 4. search

**Purpose**: Query the knowledge graph with various search strategies.

**Parameters**:
- `search_query` (string, required): Your question or search terms
- `search_type` (string, required): Search strategy to use

**Search Types**:
- `GRAPH_COMPLETION`: Natural language Q&A with graph context
- `RAG_COMPLETION`: Traditional RAG without graph
- `CHUNKS`: Raw text segments
- `INSIGHTS`: Entity relationships
- `SUMMARIES`: Pre-generated summaries
- `CODE`: Code-specific search
- `CYPHER`: Direct graph queries
- `FEELING_LUCKY`: Auto-select best type

**Usage Examples**:
```python
# Natural language question
result = await mcp.search(
    search_query="How does the authentication system work?",
    search_type="GRAPH_COMPLETION"
)

# Find code implementations
result = await mcp.search(
    search_query="async task execution",
    search_type="CODE"
)

# Get relationships
result = await mcp.search(
    search_query="Pipeline Task",
    search_type="INSIGHTS"
)
```

### 5. save_interaction

**Purpose**: Log user-agent interactions for learning.

**Parameters**:
- `data` (string, required): Interaction details to save

**Usage Example**:
```python
await mcp.save_interaction(
    data="Q: How to implement auth? A: Use JWT tokens with..."
)
```

### 6. get_developer_rules

**Purpose**: Retrieve learned developer patterns and rules.

**Parameters**: None

**Usage Example**:
```python
rules = await mcp.get_developer_rules()
# Returns bullet-pointed list of coding patterns
```

### 7. list_data

**Purpose**: List all datasets and data items with IDs.

**Parameters**:
- `dataset_id` (string, optional): Filter by specific dataset

**Usage Example**:
```python
# List all data
all_data = await mcp.list_data()

# List specific dataset
dataset_items = await mcp.list_data(
    dataset_id="62fad84d-3aa5-506d-9011-ae6778128887"
)
```

**Response Format**:
```
Datasets:
- main_dataset (id: uuid)
  - document1.md (id: uuid)
  - code.py (id: uuid)
```

### 8. delete

**Purpose**: Remove specific data from datasets.

**Parameters**:
- `data_id` (string, required): UUID of data item
- `dataset_id` (string, required): UUID of dataset
- `mode` (string, optional): "soft" or "hard" (default: "soft")

**Usage Example**:
```python
await mcp.delete(
    data_id="123e4567-e89b-12d3-a456-426614174000",
    dataset_id="62fad84d-3aa5-506d-9011-ae6778128887",
    mode="soft"
)
```

**Deletion Modes**:
- `soft`: Removes data but keeps shared entities
- `hard`: Also removes orphaned entity nodes

### 9. prune

**Purpose**: Reset entire Cognee knowledge graph.

**Parameters**: None

**Usage Example**:
```python
await mcp.prune()
# WARNING: Deletes all data permanently!
```

### 10. cognify_status

**Purpose**: Check cognify pipeline status.

**Parameters**: None

**Usage Example**:
```python
status = await mcp.cognify_status()
```

**Response Format**:
```
Pipeline Status:
- ID: uuid
- Status: PROCESSING | COMPLETED | ERRORED
- Progress: 75%
- Current Task: Extracting entities
```

### 11. codify_status

**Purpose**: Check code analysis pipeline status.

**Parameters**: None

**Usage Example**:
```python
status = await mcp.codify_status()
```

## Common Workflows

### Initial Setup
```python
# 1. Add developer rules
await mcp.cognee_add_developer_rules()

# 2. Process initial documentation
await mcp.cognify(data="/path/to/docs")

# 3. Analyze codebase
await mcp.codify(repo_path="/path/to/code")

# 4. Check status
await mcp.cognify_status()
await mcp.codify_status()
```

### Knowledge Building
```python
# Add new content
await mcp.cognify(data="New information to remember")

# Save interactions for learning
await mcp.save_interaction(
    data="User asked about X, solution was Y"
)

# Query the knowledge
result = await mcp.search(
    search_query="previous solutions",
    search_type="GRAPH_COMPLETION"
)
```

### Data Management
```python
# List current data
data_list = await mcp.list_data()

# Delete specific item
await mcp.delete(
    data_id="item-uuid",
    dataset_id="dataset-uuid"
)

# Complete reset (careful!)
await mcp.prune()
```

## Error Handling

### Common Errors

1. **Pipeline Errors**
```python
status = await mcp.cognify_status()
if "ERRORED" in status:
    # Check logs for details
    # Retry with smaller data
```

2. **LLM Rate Limits**
```python
try:
    result = await mcp.search(query, "GRAPH_COMPLETION")
except Exception as e:
    if "rate limit" in str(e):
        # Use non-LLM search
        result = await mcp.search(query, "CHUNKS")
```

3. **Missing Configuration**
```python
# Ensure environment variables are set
export LLM_API_KEY=your-key
export LLM_PROVIDER=openai
```

## Best Practices

### 1. Memory Initialization
Always initialize memory at conversation start:
```python
await mcp.cognify_status()  # Check existing knowledge
rules = await mcp.get_developer_rules()  # Load patterns
```

### 2. Incremental Learning
Save successful interactions:
```python
# After solving a problem
await mcp.save_interaction(
    data=f"Problem: {problem}\nSolution: {solution}"
)
await mcp.cognify(data=solution)  # Process for future use
```

### 3. Search Strategy
Start broad, then narrow:
```python
# First, try intelligent search
result = await mcp.search(query, "FEELING_LUCKY")

# If need specifics, use targeted search
chunks = await mcp.search(query, "CHUNKS")
relationships = await mcp.search(query, "INSIGHTS")
```

### 4. Dataset Organization
Use meaningful dataset names:
```python
await mcp.cognify(
    data=architecture_doc,
    dataset_name="architecture"
)
await mcp.cognify(
    data=api_doc,
    dataset_name="api_reference"
)
```

## Transport-Specific Usage

### stdio (Default)
```python
# Direct tool calls in IDE
result = mcp.search("query", "CHUNKS")
```

### SSE (Server-Sent Events)
```javascript
// Web client
const eventSource = new EventSource('/mcp/stream');
eventSource.onmessage = (event) => {
    const result = JSON.parse(event.data);
};
```

### HTTP
```bash
# REST API calls
curl -X POST http://localhost:8080/tools/search \
  -H "Content-Type: application/json" \
  -d '{"query": "test", "search_type": "CHUNKS"}'
```

## Debugging

### Enable Debug Logging
```bash
export MCP_LOG_LEVEL=DEBUG
python src/server.py
```

### Check Tool Registration
```python
# List available tools
tools = await mcp.list_tools()
print(tools)
```

### Monitor Pipeline Progress
```python
# Start long operation
await mcp.cognify(large_document)

# Check progress periodically
while True:
    status = await mcp.cognify_status()
    print(status)
    if "COMPLETED" in status or "ERRORED" in status:
        break
    await asyncio.sleep(5)
```