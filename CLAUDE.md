# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Cognee is a memory layer for AI agents that enables dynamic knowledge graph construction from various data sources. It provides an ECL (Extract, Cognify, Load) pipeline architecture for building intelligent memory systems with support for multiple database backends, LLM providers, and deployment modes.

## Common Development Commands

### Python Package Development
```bash
# Install with uv (recommended)
uv sync --all-extras  # Install all optional dependencies
uv sync              # Install core dependencies only

# Run tests
pytest                                  # Run all tests
pytest cognee/tests/unit/             # Run unit tests only
pytest cognee/tests/integration/      # Run integration tests
pytest -k "test_name"                  # Run specific test by name
pytest -x                              # Stop on first failure
pytest --cov=cognee                    # With coverage report

# Code quality
ruff check cognee/                     # Lint code
ruff format cognee/                    # Format code
mypy cognee/                          # Type checking

# Development server
uvicorn cognee.api.server:app --reload --port 8000  # Run API server locally
```

### Frontend Development
```bash
cd cognee-frontend
npm install          # Install dependencies
npm run dev          # Start development server (port 3000)
npm run build        # Build for production
npm run lint         # Run linting
```

### MCP Server Development
```bash
cd cognee-mcp
uv sync --dev --all-extras
python src/server.py                           # Run with stdio (default)
python src/server.py --transport sse          # Run with SSE transport
python src/server.py --transport http         # Run with HTTP transport
```

### Docker Development
```bash
# Build and run all services
docker-compose up --build

# Run specific profiles
docker-compose --profile ui up       # Include frontend
docker-compose --profile mcp up      # Include MCP server
docker-compose --profile neo4j up    # Include Neo4j database
docker-compose --profile postgres up # Include PostgreSQL

# Individual services
docker-compose up cognee             # Main API only
docker-compose up frontend           # Frontend only
docker-compose up cognee-mcp        # MCP server only
```

## Architecture & Code Structure

### Core Pipeline Architecture (ECL)
The system follows an Extract-Cognify-Load pattern:

1. **Extract Phase** (`cognee/modules/ingestion/`)
   - Data loaders in `cognee/infrastructure/loaders/` handle various formats (PDF, audio, images, text)
   - Classification in `cognee/modules/ingestion/classify/` determines data types
   - Data items stored with metadata tracking

2. **Cognify Phase** (`cognee/tasks/`)
   - `chunks/` - Text chunking with configurable strategies
   - `graph/` - Entity extraction and relationship inference
   - `ontology/` - Ontology generation from data
   - `summarization/` - Hierarchical summarization
   - `code/` - Source code analysis and graph building
   - Pipeline orchestration in `cognee/modules/pipelines/`

3. **Load Phase** (`cognee/infrastructure/databases/`)
   - Vector storage for embeddings (LanceDB, ChromaDB, PGVector)
   - Graph storage for relationships (Kuzu, Neo4j, FalkorDB)
   - Relational storage for metadata (SQLite, PostgreSQL)

### Multi-Database Strategy
The system uses three database types simultaneously:

```python
# Vector DB for semantic search
from cognee.infrastructure.databases.vector import get_vector_database
vector_db = await get_vector_database()

# Graph DB for relationships
from cognee.infrastructure.databases.graph import get_graph_database
graph_db = await get_graph_database()

# Relational DB for metadata
from cognee.infrastructure.databases.relational import get_relational_database
relational_db = await get_relational_database()
```

### API Structure (`cognee/api/v1/`)
- RESTful endpoints with FastAPI
- Authentication via JWT tokens (`cognee/modules/users/`)
- Multi-tenant support with permissions (`cognee/modules/users/permissions/`)
- Rate limiting in `cognee/api/v1/middleware/`
- WebSocket support for real-time updates

### Search Types (`cognee/modules/retrieval/`)
- `GRAPH_COMPLETION` - Natural language Q&A using graph context + LLM
- `CHUNKS` - Raw text segments matching query
- `INSIGHTS` - Entity relationships and connections
- `SUMMARIES` - Pre-generated hierarchical summaries
- `CODE` - Code-specific search

### Key Design Patterns

1. **Provider Pattern** - Pluggable database/LLM providers
   ```python
   # Example: cognee/infrastructure/databases/vector/factory.py
   def get_vector_database(provider: str = None) -> VectorDBInterface
   ```

2. **Pipeline Tasks** - Composable async tasks
   ```python
   # Example: cognee/tasks/chunks/chunk_by_paragraph.py
   async def chunk_by_paragraph(data_chunks: list[DataChunk]) -> list[DataChunk]
   ```

3. **Repository Pattern** - Data access abstraction
   ```python
   # Example: cognee/modules/data/repository/
   class DataRepository(Protocol)
   ```

## Environment Configuration

Create a `.env` file based on `.env.template`:

```bash
# LLM Configuration
LLM_API_KEY=your_openai_key
LLM_PROVIDER=openai  # or litellm, ollama, etc.

# Database Configuration
GRAPH_DB_PROVIDER=kuzu  # or neo4j, falkordb
VECTOR_DB_PROVIDER=lancedb  # or chromadb, pgvector
RELATIONAL_DB_PROVIDER=sqlite  # or postgres

# For PostgreSQL
DB_HOST=localhost
DB_PORT=5432
DB_NAME=cognee_db
DB_USERNAME=cognee
DB_PASSWORD=cognee

# For Neo4j
NEO4J_URI=bolt://localhost:7687
NEO4J_USER=neo4j
NEO4J_PASSWORD=password

# Storage
FILE_STORAGE_PROVIDER=local  # or s3
S3_BUCKET_NAME=cognee-bucket
S3_REGION=us-east-1
```

## Testing Strategy

### Unit Tests
- Test individual functions/classes in isolation
- Mock external dependencies (databases, LLMs)
- Located in `cognee/tests/unit/`

### Integration Tests
- Test complete workflows with real databases
- Located in `cognee/tests/integration/`
- Database-specific tests in subdirectories

### Running Specific Test Suites
```bash
# Database-specific tests
pytest cognee/tests/integration/databases/neo4j/
pytest cognee/tests/integration/databases/chromadb/

# Feature-specific tests
pytest cognee/tests/integration/ingestion/
pytest cognee/tests/integration/graph/
pytest cognee/tests/integration/permissions/
```

## MCP Server Integration

The MCP (Model Context Protocol) server enables IDE integration:

### Available Tools
- `add(text/file)` - Add data to cognee
- `cognify()` - Process data into knowledge graph
- `codify(repo_path)` - Analyze code repositories
- `search(query, search_type)` - Query knowledge
- `list_data()` - List datasets and items
- `delete(data_id)` - Remove specific data
- `prune()` - Reset entire memory
- `cognify_status()` - Check processing status

### Transport Modes
- `stdio` - Standard I/O (default, for Cursor/VSCode)
- `sse` - Server-Sent Events (for web clients)
- `http` - HTTP streaming (for production deployments)

## Key Files to Understand

1. **`cognee/cognee.py`** - Main entry point with high-level API
2. **`cognee/api/server.py`** - FastAPI application setup
3. **`cognee/modules/pipelines/pipeline.py`** - Pipeline orchestration logic
4. **`cognee/tasks/graph/extract_graph_from_chunks.py`** - Core graph extraction
5. **`cognee/infrastructure/databases/*/factory.py`** - Database provider factories
6. **`cognee/modules/retrieval/*/search.py`** - Search implementations
7. **`cognee-mcp/src/server.py`** - MCP server implementation

## Performance Considerations

1. **Async Processing** - All database operations and LLM calls are async
2. **Batching** - Process data in batches for efficiency
3. **Caching** - LRU caching for embeddings and LLM responses
4. **Connection Pooling** - Database connection reuse
5. **Rate Limiting** - Configurable limits per user/endpoint

## Debugging Tips

1. **Enable debug logging**:
   ```python
   import logging
   logging.basicConfig(level=logging.DEBUG)
   ```

2. **API debugging**:
   ```bash
   uvicorn cognee.api.server:app --reload --log-level debug
   ```

3. **MCP debugging**:
   ```bash
   export MCP_LOG_LEVEL=DEBUG
   python cognee-mcp/src/server.py
   ```

4. **Database queries**:
   - Enable query logging in database providers
   - Use graph visualization endpoint for Neo4j/Kuzu

## Common Pitfalls

1. **Always use async/await** for database and LLM operations
2. **Check provider compatibility** - Not all features work with all database providers
3. **Memory usage** - Large documents can consume significant memory during processing
4. **Rate limits** - Respect LLM provider rate limits
5. **Database migrations** - Run Alembic migrations when schema changes

## Critical Resource Management Lessons (MUST READ)

### The Resource Overload Problem
Cognee can crash with `vector validation error (empty vectors)` when overloaded. This happens because:
- The embedding engine (`OllamaEmbeddingEngine.py:89`) sends ALL requests simultaneously: `asyncio.gather(*all_requests)`
- Local Ollama on a 16GB machine can handle ~5-10 concurrent requests, not 100+
- When overloaded, Ollama returns empty vectors → pipeline crashes

### How to Avoid Resource Overload

#### When Processing Documents
```python
# ❌ WRONG - This will crash
for doc in ten_documents:
    await cognee.add(doc)  # Rapid fire = overload

# ✅ CORRECT - Process one at a time
await cognee.add(doc1)
await cognee.cognify()
status = await cognee.cognify_status()
# Wait for COMPLETED before adding next document
```

#### Document Size Guidelines
- **Safe**: < 500 lines per document
- **Risky**: 500-1000 lines (may work, monitor closely)
- **Dangerous**: > 1000 lines (likely to fail on 16GB machines)
- **Solution**: Break large documents into smaller chunks

#### Best Practices for Local Development (Ollama)
1. **One document at a time** - Never batch process
2. **Wait for completion** - Always check `cognify_status()` 
3. **Start fresh if failed** - Use `prune()` to reset after errors
4. **Monitor logs** - Check for "empty vector" errors
5. **Small test first** - Start with a tiny document to verify setup

### Codify Known Issues
The `codify()` function may fail with `IndexError` in dependency parsing:
- Error location: `/cognee/tasks/repo_processor/get_local_dependencies.py:268`
- Cause: Parser encounters unexpected code syntax patterns
- Workaround: Process smaller directories or individual files

### Search Type Performance

When knowledge base is properly built:
- **CHUNKS**: Fast, reliable for exact text retrieval
- **GRAPH_COMPLETION**: Slower but provides context (requires LLM)
- **INSIGHTS**: Fast for relationship queries
- **File-based search**: Always fastest for code/implementation details

### Environment Setup for Stability

```bash
# For local Ollama (16GB machines)
LLM_PROVIDER=ollama
LLM_MODEL=llama3.2:3b  # Smaller model = more stable
EMBEDDING_MODEL=nomic-embed-text
EMBEDDING_DIMENSIONS=768

# Theoretical settings (not yet implemented in Cognee)
# EMBEDDING_BATCH_SIZE=5  # Process 5 at a time
# EMBEDDING_CONCURRENT_LIMIT=3  # Max parallel requests
# PIPELINE_MAX_WORKERS=2  # Limit pipeline parallelism
```

### When to Use What

**Use File-Based Search When:**
- Looking for exact implementation details
- Need instant results
- Working with code files
- System resources are limited
- Cognee processing has failed

**Use Cognee Search When:**
- Need semantic understanding
- Want context and explanations
- Building relationships between concepts
- Have successfully processed documents
- Can afford setup time