# Cognee Architecture Overview

## System Architecture

Cognee is a memory layer for AI agents that transforms unstructured data into intelligent knowledge graphs. The system follows an ECL (Extract-Cognify-Load) pipeline architecture with multiple deployment modes and database backends.

## Core Components

### 1. Main Application (`cognee/`)
The core Python package implementing the knowledge graph engine:
- **Pipeline Engine**: Orchestrates data processing workflows
- **Task System**: Modular async tasks for data transformation
- **Database Adapters**: Interfaces for vector, graph, and relational databases
- **LLM Gateway**: Unified interface for multiple LLM providers

### 2. API Server (`cognee/api/`)
FastAPI-based REST API providing:
- RESTful endpoints for all operations
- WebSocket support for real-time updates
- JWT authentication and multi-tenant support
- Rate limiting and middleware

### 3. Frontend Application (`cognee-frontend/`)
React/Next.js web interface featuring:
- Knowledge graph visualization
- Data management dashboard
- Search interface
- Pipeline monitoring

### 4. MCP Server (`cognee-mcp/`)
Model Context Protocol server enabling:
- IDE integration (Cursor, VSCode)
- Multiple transport modes (stdio, SSE, HTTP)
- Tool-based interface for AI assistants
- Background task management

## Data Flow Architecture

### ECL Pipeline Pattern

```
Extract → Cognify → Load → Search
```

1. **Extract Phase**
   - Load data from various sources (files, text, databases)
   - Classify document types
   - Convert to unified format
   - Store raw data with metadata

2. **Cognify Phase**
   - Chunk text into semantic units
   - Extract entities and relationships
   - Generate embeddings
   - Build knowledge graph
   - Create hierarchical summaries

3. **Load Phase**
   - Store embeddings in vector database
   - Save graph structure in graph database
   - Persist metadata in relational database
   - Index for efficient retrieval

4. **Search Phase**
   - Multiple search strategies
   - Graph traversal for context
   - Semantic similarity matching
   - LLM-enhanced responses

## Database Architecture

### Triple Database Strategy
Cognee uses three specialized databases simultaneously:

1. **Vector Database** (embeddings & similarity search)
   - LanceDB (default, embedded)
   - ChromaDB
   - PGVector
   - Neptune Analytics

2. **Graph Database** (relationships & traversal)
   - Kuzu (default, embedded)
   - Neo4j
   - FalkorDB
   - Neptune Analytics

3. **Relational Database** (metadata & configuration)
   - SQLite (default, embedded)
   - PostgreSQL

## Module Organization

### Core Modules (`cognee/modules/`)
- **chunking/**: Text segmentation strategies
- **data/**: Dataset and data management
- **graph/**: Graph operations and models
- **ingestion/**: Data loading and classification
- **pipelines/**: Workflow orchestration
- **retrieval/**: Search and context retrieval
- **users/**: Authentication and permissions

### Infrastructure Layer (`cognee/infrastructure/`)
- **databases/**: Database adapters and interfaces
- **llm/**: LLM provider integration
- **loaders/**: File format handlers
- **files/**: Storage abstraction (local, S3)

### Task Library (`cognee/tasks/`)
- **chunks/**: Text chunking tasks
- **graph/**: Entity and relationship extraction
- **summarization/**: Content summarization
- **repo_processor/**: Code analysis
- **temporal_awareness/**: Time-based graph operations

## Deployment Modes

### 1. Python Package
```python
import cognee
await cognee.add("data")
await cognee.cognify()
results = await cognee.search("query")
```

### 2. API Server
```bash
uvicorn cognee.api.server:app --reload
# Endpoints at http://localhost:8000
```

### 3. Docker Deployment
```bash
docker-compose up
# Includes all services and databases
```

### 4. MCP Integration
```bash
python cognee-mcp/src/server.py
# Integrates with AI IDEs
```

## Key Design Patterns

### 1. Async-First Architecture
All I/O operations use async/await for scalability

### 2. Provider Pattern
Pluggable backends for databases and LLMs

### 3. Pipeline Tasks
Composable units of work that can be chained

### 4. Repository Pattern
Abstract data access behind clean interfaces

### 5. Multi-Tenant Design
Isolated datasets and permissions per user/organization

## Configuration Management

Environment variables control behavior:
- `LLM_PROVIDER`: Choose LLM backend
- `GRAPH_DB_PROVIDER`: Select graph database
- `VECTOR_DB_PROVIDER`: Choose vector store
- `FILE_STORAGE_PROVIDER`: Local or S3 storage

## Performance Optimizations

1. **Batching**: Process data in configurable batch sizes
2. **Caching**: LRU cache for embeddings and LLM responses
3. **Connection Pooling**: Reuse database connections
4. **Rate Limiting**: Configurable limits per user/endpoint
5. **Async Processing**: Non-blocking I/O throughout

## Extensibility Points

1. **Custom Loaders**: Add support for new file formats
2. **Task Plugins**: Create custom processing tasks
3. **Database Adapters**: Integrate new database backends
4. **LLM Providers**: Add new language models
5. **Search Strategies**: Implement custom retrievers

## Security Features

1. **JWT Authentication**: Secure API access
2. **Multi-Tenant Isolation**: Data separation
3. **Permission System**: Fine-grained access control
4. **Rate Limiting**: Prevent abuse
5. **Input Validation**: Sanitize all inputs

## Monitoring & Observability

1. **Structured Logging**: JSON logs with context
2. **Pipeline Telemetry**: Track task execution
3. **Metrics Collection**: Performance monitoring
4. **Health Endpoints**: Service status checks
5. **Error Tracking**: Detailed error reporting