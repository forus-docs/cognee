# Cognee Search Types Guide

## Overview

Cognee provides multiple search strategies optimized for different use cases. Each search type uses different retrieval mechanisms and returns different result formats.

## Search Types

### 1. GRAPH_COMPLETION (Recommended for Q&A)

**Purpose**: Natural language question answering using full graph context and LLM reasoning.

**How it works**:
1. Converts query to embeddings
2. Finds relevant graph nodes and relationships
3. Traverses graph for additional context
4. Sends context + query to LLM
5. Returns conversational response

**Best for**:
- Complex questions requiring reasoning
- Multi-hop queries across relationships
- Generating comprehensive answers
- Understanding entity connections

**Example Query**:
```python
result = await cognee.search(
    "How do the main components of the system interact?",
    search_type="GRAPH_COMPLETION"
)
```

**Response Format**:
```
"The main components interact through a pipeline architecture where 
the API server receives requests, passes them to the pipeline engine, 
which orchestrates tasks that process data through various database 
adapters. The MCP server provides an alternative interface..."
```

### 2. RAG_COMPLETION

**Purpose**: Traditional RAG (Retrieval Augmented Generation) without graph structure.

**How it works**:
1. Performs vector similarity search
2. Retrieves relevant document chunks
3. Sends chunks + query to LLM
4. Returns LLM-generated response

**Best for**:
- Direct document retrieval
- Fact-finding from specific sources
- When graph relationships aren't needed
- Faster than GRAPH_COMPLETION

**Example Query**:
```python
result = await cognee.search(
    "What does the documentation say about authentication?",
    search_type="RAG_COMPLETION"
)
```

**Response Format**:
```
"According to the documentation, authentication uses JWT tokens 
with support for multi-tenant isolation. Users must include 
the Bearer token in the Authorization header..."
```

### 3. INSIGHTS

**Purpose**: Retrieve structured entity relationships and semantic connections.

**How it works**:
1. Identifies entities in query
2. Finds related entities in graph
3. Returns relationship paths
4. No LLM processing (fast)

**Best for**:
- Understanding concept relationships
- Knowledge mapping
- Entity exploration
- Graph structure analysis

**Example Query**:
```python
result = await cognee.search(
    "pipeline tasks",
    search_type="INSIGHTS"
)
```

**Response Format**:
```json
{
  "entities": [
    {
      "name": "Pipeline",
      "type": "Component",
      "relationships": [
        {"type": "contains", "target": "Task"},
        {"type": "orchestrates", "target": "TaskRun"}
      ]
    }
  ],
  "paths": [
    ["Pipeline", "executes", "Task"],
    ["Task", "produces", "DataChunk"]
  ]
}
```

### 4. CHUNKS

**Purpose**: Retrieve raw text segments that semantically match the query.

**How it works**:
1. Converts query to embeddings
2. Performs vector similarity search
3. Returns matching text chunks
4. No LLM or graph processing

**Best for**:
- Finding specific passages
- Getting exact citations
- Retrieving source content
- Fastest search type

**Example Query**:
```python
result = await cognee.search(
    "error handling",
    search_type="CHUNKS"
)
```

**Response Format**:
```json
[
  {
    "chunk": "Error handling in Cognee uses structured exceptions...",
    "score": 0.92,
    "metadata": {
      "source": "architecture.md",
      "position": 1234
    }
  }
]
```

### 5. SUMMARIES

**Purpose**: Return pre-generated hierarchical summaries of content.

**How it works**:
1. Matches query to summary topics
2. Retrieves relevant summaries
3. Returns multi-level summaries
4. Pre-computed during cognify

**Best for**:
- Quick overviews
- Document abstracts
- Topic exploration
- High-level understanding

**Example Query**:
```python
result = await cognee.search(
    "database architecture",
    search_type="SUMMARIES"
)
```

**Response Format**:
```json
{
  "summaries": [
    {
      "level": "high",
      "content": "Cognee uses three databases simultaneously..."
    },
    {
      "level": "detailed",
      "content": "The vector database stores embeddings using..."
    }
  ]
}
```

### 6. CODE

**Purpose**: Search specifically for code elements with syntax awareness.

**How it works**:
1. Parses query for code patterns
2. Searches code graph structure
3. Returns functions, classes, implementations
4. Includes dependency context

**Best for**:
- Finding function implementations
- Understanding code structure
- Locating specific patterns
- API usage examples

**Example Query**:
```python
result = await cognee.search(
    "async task execution",
    search_type="CODE"
)
```

**Response Format**:
```json
{
  "results": [
    {
      "type": "function",
      "name": "run_tasks_async",
      "file": "tasks/runner.py",
      "line": 45,
      "code": "async def run_tasks_async(tasks: List[Task])...",
      "dependencies": ["Task", "Pipeline"]
    }
  ]
}
```

### 7. CYPHER

**Purpose**: Direct graph database queries using Cypher query language.

**How it works**:
1. Executes raw Cypher query
2. Returns graph query results
3. No processing or transformation
4. Full graph database access

**Best for**:
- Advanced graph queries
- Custom traversals
- Debugging graph structure
- Power users

**Example Query**:
```python
result = await cognee.search(
    "MATCH (n:Entity)-[r]->(m:Entity) WHERE n.name = 'Pipeline' RETURN n, r, m LIMIT 10",
    search_type="CYPHER"
)
```

**Response Format**:
```json
{
  "nodes": [...],
  "relationships": [...],
  "properties": {...}
}
```

### 8. FEELING_LUCKY

**Purpose**: Automatically select the best search type for the query.

**How it works**:
1. Analyzes query intent
2. Uses LLM to classify query type
3. Selects optimal search strategy
4. Executes selected search
5. Returns results in appropriate format

**Best for**:
- General-purpose queries
- When unsure which type to use
- User-facing applications
- Adaptive search experience

**Example Query**:
```python
result = await cognee.search(
    "any query here",
    search_type="FEELING_LUCKY"
)
# Automatically chooses best search type
```

## Performance Comparison

| Search Type | Speed | Context | LLM Usage | Best For |
|------------|-------|---------|-----------|----------|
| CHUNKS | Fastest | None | No | Raw retrieval |
| INSIGHTS | Fast | Graph | No | Relationships |
| SUMMARIES | Fast | Pre-computed | No | Overviews |
| CODE | Medium | Code graph | No | Code search |
| RAG_COMPLETION | Medium | Documents | Yes | Document Q&A |
| GRAPH_COMPLETION | Slower | Full graph | Yes | Complex Q&A |
| CYPHER | Variable | Custom | No | Advanced queries |
| FEELING_LUCKY | Variable | Adaptive | Maybe | General use |

## Choosing the Right Search Type

### Decision Tree

1. **Need a conversational answer?**
   - Yes → GRAPH_COMPLETION or RAG_COMPLETION
   - No → Continue

2. **Looking for specific text/quotes?**
   - Yes → CHUNKS
   - No → Continue

3. **Need entity relationships?**
   - Yes → INSIGHTS
   - No → Continue

4. **Want an overview/summary?**
   - Yes → SUMMARIES
   - No → Continue

5. **Searching for code?**
   - Yes → CODE
   - No → Continue

6. **Need custom graph query?**
   - Yes → CYPHER
   - No → FEELING_LUCKY

## Advanced Usage

### Combining Search Types

```python
# Get both raw chunks and intelligent response
chunks = await cognee.search(query, search_type="CHUNKS")
answer = await cognee.search(query, search_type="GRAPH_COMPLETION")

# Use insights to explore, then get details
relationships = await cognee.search("Pipeline", search_type="INSIGHTS")
for entity in relationships["entities"]:
    details = await cognee.search(entity["name"], search_type="SUMMARIES")
```

### Search Parameters

```python
# Limit results
results = await cognee.search(
    query="test",
    search_type="CHUNKS",
    limit=5
)

# Target specific dataset
results = await cognee.search(
    query="architecture",
    search_type="GRAPH_COMPLETION",
    dataset_name="documentation"
)

# Include metadata
results = await cognee.search(
    query="error",
    search_type="CHUNKS",
    include_metadata=True
)
```

## Error Handling

Each search type may fail differently:

- **GRAPH_COMPLETION/RAG_COMPLETION**: LLM API errors, rate limits
- **CHUNKS/INSIGHTS**: Vector database connection issues
- **CYPHER**: Invalid query syntax
- **CODE**: Code graph not built
- **SUMMARIES**: No summaries generated

Always handle errors appropriately:

```python
try:
    result = await cognee.search(query, search_type="GRAPH_COMPLETION")
except LLMRateLimitError:
    # Fallback to non-LLM search
    result = await cognee.search(query, search_type="CHUNKS")
except Exception as e:
    logger.error(f"Search failed: {e}")
```